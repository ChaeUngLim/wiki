---
type: "harness"
title: "Codex의 하네스 설계: 저장소가 사실의 원천, AGENTS.md는 디렉터리 페이지"
aliases:
  - "Codex"
  - "codex"
  - "Codex CLI"
  - "OpenAI Codex"
sources:
  - "raw/learn-harness-engineering/docs/ko/harness-designs/codex/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/codex/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md"
  - "raw/gajae-code/AGENTS.md"
related:
  - "[[concepts/repo-system-of-record]]"
  - "[[concepts/map-not-manual]]"
  - "[[concepts/harness-subsystems]]"
  - "[[harness/claude-code]]"
  - "[[syntheses/claude-code-vs-codex]]"
tags:
  - "type/harness"
  - "topic/system-of-record"
  - "topic/instructions"
  - "topic/scope-control"
updated: "2026-11-06"
---

# Codex의 하네스 설계: 저장소가 사실의 원천, AGENTS.md는 디렉터리 페이지

## 왕초보 설명

- **이게 뭐야?** OpenAI의 터미널 AI 코딩 도구 Codex를 하네스 관점에서 뜯어본 코스 글이다. Codex는 기능을 잔뜩 넣기보다 **규칙(규약)**으로 승부한다: 프로젝트 지식은 저장소 문서에 두고, `AGENTS.md`는 그 문서들의 **목차**로만 쓴다.
- **비유:** 두꺼운 백과사전 한 권 대신, 도서관 입구의 안내도 한 장. 안내도가 각 서가(`docs/`)를 가리킨다.
- **나는 뭘 하면 돼?** `AGENTS.md`를 100줄 안팎으로 짧게 쓰고 "절대 어기면 안 되는 규칙 + 확인 명령(테스트·린트)"만 적는다. 자세한 설명은 `docs/`에 나눠 두고 링크한다. 작업마다 따로 작업 폴더(git worktree)를 쓰면 서로 엉키지 않는다.

## 구조 요약

**한 문장**(원문): 저장소가 사실의 원천(repository as the system of record)이고, `AGENTS.md`는 디렉터리 페이지일 뿐이며, 엔지니어링의 가치는 환경을 설계하고 의도를 표현하며 피드백 루프를 구축하는 데 있다. 이 분야의 이름을 붙인 OpenAI 《Harness Engineering》 글이 Codex로 제품을 만든 경험을 정리한 것이라, Codex 분석은 곧 그 글의 배경 실천 분석이다. Codex CLI 자체는 Rust로 만든 오픈소스 단일 바이너리다.

**다섯 하위 시스템으로 보기**(원문 매핑 표)

| 하위 시스템 | Codex의 구현 | 원문 평가 |
| --- | --- | --- |
| 지시 | `AGENTS.md` 디렉터리 페이지 + `docs/` 분할 + 불변 조건 | "지도를 줘라"를 정의한 교과서적 구현 |
| 도구 | worktree 격리 + `spawn_agent` subagent | 환경으로 경계를 강제 |
| 환경 | 독립 worktree + 관찰 가능성 스택 | worktree 격리가 대표 특징 |
| 상태 | Write 전략(상태를 파일·문서에) | 내장 메모리가 아니라 규약에 의존 |
| 피드백 | 검증 명령을 규약에 + 승인 정책 + plan mode | 피드백 경로를 기본값으로 |

**지시: 디렉터리 페이지**. 거대한 지시 파일은 커버리지·최신성·소유권·교차 링크를 기계적으로 검사하기 어렵고 현실과 어긋난다. 그래서 `AGENTS.md`를 약 100줄로 제한하고, 넘치면 `docs/`로 나눠 필요할 때 읽게 한다([[concepts/map-not-manual]], [[lectures/lecture-04]]). 함께 쓰는 원칙: **구현을 세세하게 관리하지 말고 불변 조건을 강제하라**(강제 제약 + 검증 명령만, 방법은 모델에게, [[lectures/lecture-02]]).

**컨텍스트: Write-Select-Compress-Isolate**(커뮤니티가 Codex에 매핑한 틀)

| 전략 | 뜻 | Codex에서 |
| --- | --- | --- |
| Write | 컨텍스트를 창 밖에 영속화 | 결론은 문서, 상태는 파일([[concepts/repo-system-of-record]]) |
| Select | 필요한 token만 창으로 | `AGENTS.md`가 길 안내, 파일은 필요할 때 |
| Compress | 중요한 것만 남김 | 자동 compaction + 수동 `/compact`, `compact_prompt` 설정 |
| Isolate | 경계별로 나눔 | subagent마다 다른 컨텍스트 |

세부: 커뮤니티 소스 분석에 따르면 `build_environment_update_item`은 매번 전체 시스템 컨텍스트를 다시 붙이지 않고 CWD·git 브랜치·파일 시스템 중 **바뀐 필드만** 낸다.

**도구와 경계**

1. **git worktree 환경 격리**: 각 작업을 독립 worktree에서 돌리고 로컬 관찰 스택(로그·메트릭·트레이스)으로 변경마다 검증. 경계를 프롬프트로 부탁하지 않고 환경으로 강제한다([[lectures/lecture-07]]).
2. **커널 수준 subagent**: `spawn_agent`/`wait_agent`. subagent는 부모 `AGENTS.md`를 상속하되 자기 컨텍스트에서 돈다. 설정은 `.codex/agents/*.toml`(모델·지시를 따로 지정).

**피드백**: `AGENTS.md`에 검증 명령을 적어 "제대로 했는지 확인하는 방법"을 저장소의 일부로 만든다([[concepts/verification-commands]]). 테스트·CI·문서·관찰 설정까지 Codex가 생성한다. 승인 정책과 plan mode는 고위험 작업 전에 계획과 승인을 요구한다.

**원문이 꼽는 참고할 설계**: ① `AGENTS.md`를 약 100줄 디렉터리 페이지로 ② 불변 조건만, 구현은 맡기기 ③ worktree로 환경 격리 ④ 환경 컨텍스트는 변경분만 ⑤ subagent로 컨텍스트 격리.

## 다른 하네스와의 비교

| 항목 | [[harness/claude-code\|claude-code]] | codex (이 페이지) | [[harness/deepseek\|deepseek]] | [[harness/pi\|pi]] | [[harness/oh-my-claudecode\|oh-my-claudecode]] | [[harness/gajae-code\|gajae-code]] |
| --- | --- | --- | --- | --- | --- | --- |
| 정체 | Anthropic의 공식 CLI. 단순한 while 루프 + 루프를 둘러싼 시스템 | OpenAI의 오픈소스 Rust 단일 바이너리 CLI. 《Harness Engineering》 글의 배경 | `dsh`, 2026-08 Developer Preview. Agent = Model + Environment + Tools + State, 모든 것이 plugin(Cordis 커널) | "minimal agent harness". npm `@earendil-works/pi-coding-agent`. 작은 커널 + 프로그래밍 가능한 확장 | Claude Code 위에 얹는 플러그인 | 독립 실행 CLI 하네스(oh-my-pi 계열) |
| 지시 | `CLAUDE.md` 4범위(조직/사용자/프로젝트/로컬) + 하위 디렉터리 필요 시 로드 + auto memory(첫 200줄 또는 25KB) | `AGENTS.md` = 약 100줄 디렉터리 페이지 + `docs/` 분할, 불변 조건과 검증 명령만 | 규칙·스킬을 plugin으로 주입. `CLAUDE.md` 같은 내장 규약 없음 | `AGENTS.md` 계층 로드(전역 → 상위 디렉터리 → 현재, `CLAUDE.md` 호환) + `SYSTEM.md` replace/append | 스킬 48개, 에이전트 19개 + 키워드 자동 감지 | 번들 스킬 4개 + 역할 4개만, 명시적으로 부를 때만 |
| 계획 단계 | 원문(분석 글)에 서술 없음 | plan mode(고위험 작업 전 계획 작성·승인 요청) | 원문(분석 글)에 서술 없음 | 원문(분석 글)에 서술 없음 | `deep-interview`(모호함 ≤20%) → `ralplan` | `deep-interview`(기본 기준 5%) → `ralplan`(최대 5회, 코드로 수정 차단) |
| 실행 루프 | 모델 호출 → 도구 실행 → 관찰 반복. [[lectures/lecture-13]]: `/goal`, `/loop`, Routines | 원문(분석 글)에 서술 없음. [[lectures/lecture-13]]: `/goal`(수동 활성화), Automations, `codex exec` + cron | 이벤트 파이프라인(`turn/start` → … → `step/end`). agent 루프 자체도 교체 가능한 plugin | while 루프. TUI 외에 print/JSON 모드·RPC·SDK로 외부 프로그램이 구동 | `ralph`(prd.json 이야기별 반복), `autopilot`, `team` 5단계 | `ultragoal`(goals.json 목표별 반복 + goal 모드) |
| 계속 돌게 하는 장치 | `Stop` hook이 완료 선언 시점에 개입 가능(OMC가 이 지점을 사용) | 원문(분석 글)에 서술 없음. [[lectures/lecture-13]]: 스레드 자동화·독립 실행형 자동화 | `agent/turn-stopping` 이벤트를 받아 추론 계속 여부 결정 | hooks로 외부(파일 감시·Webhook·CI)에서 메시지를 넣어 agent를 깨움 | Stop 훅(`persistent-mode`)이 턴 종료를 막음, 안전 상한 기본 500회 | goal 모드 연속 실행 + `nudgeBudget`(목표당 기본 10회) |
| 상태 | append-only `history.jsonl` + `/resume`·fork + 5단계 compaction | Write 전략(상태를 파일·문서에) + 자동·수동 `/compact`(`compact_prompt` 설정) | append-only Session Event Log | session 트리(`/tree`, 한 파일에 모든 브랜치) + 교체 가능한 compaction(최근 약 2만 token 유지) | `.omc/state/sessions/<id>/prd.json`, `progress.txt` | `.gjc/_session-<id>/ultragoal/goals.json`, `ledger.jsonl` |
| 피드백 | permissions 7모드 + ML 분류기, `PostToolUse` hook 강제 검사 | 검증 명령을 `AGENTS.md` 규약에, 작업마다 git worktree + 로그·메트릭·트레이스로 검증 | `tools/pre-execute`의 permission·guard·policy·hook + "Model-visible means logged" 런타임 불변 조건 | 강제 테스트 gate 없음. 검증 명령은 사용자가 `AGENTS.md`에. 커뮤니티 확장 session-summary·extract-patterns·telemetry | `omc ralph verify`가 기준선 대비 새 실패만 판정, 리뷰어 검증 | 목표별 quality-gate + 마지막 경계 검토 + 종료 critic `OKAY` |
| 역할 분리 | subagent 대화를 별도 sidechain 파일로 격리 | `spawn_agent`/`wait_agent` subagent, `.codex/agents/*.toml`로 모델·지시 지정 | SubAgent도 교체 가능한 능력 접합부(capability seam) | 원문(분석 글)에 서술 없음 | 단계별로 다른 에이전트·모델 등급(haiku/sonnet/opus) | 읽기 전용 planner·architect·critic vs 쓰기 가능한 executor |
| 사람 개입 | permissions 정책에 따라 고위험 작업은 질문 또는 거부 | 승인 정책(approval policies) + plan mode | 원문(분석 글)에 서술 없음 | hooks로 위험 명령 가로채기(permissions gate)·경로 보호 | 매직 키워드만 말하면 자동 진행 | 기본은 계획 승인 1회. `autoHandoff=ultragoal`이면 0회 |
| 왕초보에게 | 기본 내장이 많아 바로 쓴다. 확장 4종(스킬·MCP·hooks·subagent)의 역할 구분은 배워야 한다 | 기능보다 규약. 저장소를 잘 정리해 둘수록 효과가 커진다 | 자유도가 가장 높지만 설정 비용도 가장 크다. 아직 Developer Preview | 가장 가볍지만 필요한 장치는 직접 만들거나 패키지로 설치해야 한다 | 기능이 많아 강력하지만 무겁다 | 흐름이 하나라 배우기 쉽다. 대신 git 필수 |

- 원문의 대비: [[harness/claude-code]]는 메모리·permissions·subagent를 모두 커널에 넣는 "덧셈", Codex는 커널을 절제하고 책임을 저장소 규약과 컨텍스트 엔지니어링에 두는 "뺄셈".
- [[harness/gajae-code]] 저장소 자신의 `AGENTS.md`는 스스로를 "아키텍처 지도 + `docs/`에서 유도할 수 없는 규칙"으로 정의하고 깊은 내용은 `docs/`로 보낸다. 디렉터리 페이지 발상은 같지만 길이는 206줄로, 이 글이 권하는 약 100줄의 두 배다. 또한 gjc는 계획·실행 워크플로(스킬 4개)를 기본으로 내장한다는 점에서 Codex보다 "덧셈" 쪽이다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/harness-designs/codex/index.md`
- en: `raw/learn-harness-engineering/docs/en/harness-designs/codex/index.md`
- 원문이 드는 1차 출처: OpenAI 《Harness Engineering》, OpenAI 《AGENTS.md》 규약, `github.com/openai/codex`, 커뮤니티 글 "Context Engineering for Codex CLI", 커뮤니티 소스 분석 `codex-harness-internals`. 이 위키는 외부 문서를 직접 확인하지 않았다.

**원문 간 차이**

- 같은 OpenAI 사례를 이 글은 "몇 주 만에(in a matter of weeks) 100만 줄 넘게, 모든 줄을 Codex가 작성"이라고, [[lectures/lecture-13]]은 "작은 팀이 새 저장소에서 다섯 달 동안 약 100만 줄, 약 1,500 PR"이라고 쓴다. 기간이 다르다.
- `AGENTS.md` 길이: 이 글과 [[lectures/lecture-02]]는 약 100줄, [[lectures/lecture-04]]는 50~200줄을 권한다([[concepts/map-not-manual]]의 열린 질문).
