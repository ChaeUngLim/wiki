---
type: "harness"
title: "Claude Code의 하네스 설계: 계층형 메모리·5단계 compaction·hooks"
aliases:
  - "Claude Code"
  - "claude-code"
  - "클로드 코드"
sources:
  - "raw/learn-harness-engineering/docs/ko/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/index.md"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/map-not-manual]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[harness/oh-my-claudecode]]"
  - "[[syntheses/claude-code-vs-codex]]"
tags:
  - "type/harness"
  - "topic/harness-model"
  - "topic/context-continuity"
  - "topic/tools-extensibility"
  - "topic/verification"
updated: "2026-11-06"
---

# Claude Code의 하네스 설계: 계층형 메모리·5단계 compaction·hooks

## 왕초보 설명

- **이게 뭐야?** Anthropic이 만든 터미널 AI 코딩 도구 Claude Code를 "하네스(AI를 감싸는 작업 환경)" 관점에서 뜯어본 코스 글이다. 핵심은 AI가 도는 루프 자체는 단순하고, **그 루프를 둘러싼 장치들**(기억 파일, 대화 압축, 권한, 자동 검사)이 신뢰성을 만든다는 것.
- **비유:** 자동차 엔진(모델)은 그대로인데 안전벨트·ABS·블랙박스(하네스)가 사고를 줄인다.
- **나는 뭘 하면 돼?** 프로젝트 폴더에 `CLAUDE.md`를 두고 빌드·테스트 명령을 적는다. 처음이면 `/init`이 자동으로 초안을 만든다. 대화가 길어지면 `/compact`, 새 주제면 `/clear`. 반드시 지킬 검사는 말로 부탁하지 말고 hook(작업 전후에 자동으로 도는 스크립트)으로 건다.

## 구조 요약

**한 문장**: 모델 호출 → 도구 실행 → 결과 관찰 → 재호출을 반복하는 단순한 while 루프. 코드 대부분은 루프 안이 아니라 **루프를 둘러싼 시스템**(permissions, compaction 파이프라인, 확장, subagent 오케스트레이션, session 저장소)에 있다.

**다섯 하위 시스템으로 보기**([[concepts/harness-subsystems]], 원문 매핑 표)

| 하위 시스템 | Claude Code의 구현 | 원문 평가 |
| --- | --- | --- |
| 지시 | 범위별 계층화(조직/사용자/프로젝트/로컬) + 자동 메모리 | 계층형 메모리의 모범 구현 |
| 도구 | 스킬 + MCP + hooks + subagent 네 가지 확장 | 책임 구분이 명확한 핵심 강점 |
| 환경 | 프로젝트 내 설정 + `settings.json` | `CLAUDE.md`에서 사용자가 직접 설명하는 데 의존 |
| 상태 | 추가 지향 session 저장소 + 5단계 compaction + resume/fork | 장기 작업 연속성의 참고 구현 |
| 피드백 | permissions 분류기 + `PostToolUse` hooks 강제 검사 | 조기 완료 선언 방지를 결정론적 메커니즘으로 |

**지시: `CLAUDE.md` 네 범위**(넓은 범위 → 좁은 범위 순으로 로드, 더 구체적인 지시가 컨텍스트에 더 늦게 들어감)

| 범위 | 위치 | 내용 |
| --- | --- | --- |
| 조직 정책 | 예: `/etc/claude-code/CLAUDE.md` | 회사 규범(IT/DevOps 관리) |
| 사용자 | `~/.claude/CLAUDE.md` | 여러 프로젝트에 쓰는 개인 선호 |
| 프로젝트 | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 구조·기술 스택·검증 명령, 저장소에서 공유 |
| 로컬 | `./CLAUDE.local.md` | 프로젝트 안 개인 선호, 보통 `.gitignore` |

추가로 하위 디렉터리 `CLAUDE.md`는 그 디렉터리 파일을 읽을 때만 로드되고, **auto memory**는 Claude가 직접 쓰는 메모로 session마다 최대 첫 200줄 또는 25KB를 로드한다. 원문은 이것을 [[lectures/lecture-04]] "거대한 지시 파일" 문제의 제품화된 답으로 본다([[concepts/map-not-manual]]).

**컨텍스트: 5단계 compaction 파이프라인**(VILA Lab 소스 분석 인용): 무손실 가지치기(중복 도구 결과 제거) → 구조화된 정제 → 마지막에야 손실 있는 LLM 요약, 과도한 압축을 막는 회로 차단기. 여기에 모든 기록을 `history.jsonl`에 추가만 하는 session 저장소, `/resume` 복구와 fork가 결합된다. "인계는 기억이 아니라 저장 계층으로 보장한다."

**도구: 네 가지 확장과 책임 분리**

| 확장 | 맡는 것 | 한 줄 |
| --- | --- | --- |
| 스킬 `SKILL.md` | 어떻게 하는가 | 트리거 단어로 자동 로드, [[concepts/progressive-disclosure\|점진적 공개]] |
| MCP | 어디에 연결하는가 | JSON-RPC로 외부 시스템 연결 |
| Hooks | 언제 강제하는가 | `PreToolUse`·`PostToolUse`·`Stop` 등 생명주기 이벤트에 붙는 결정론적 스크립트 |
| Plugin / Subagents | 누구에게 나누는가 | 전문 agent에게 하위 작업 |

`CLAUDE.md`는 "무엇인가"를 맡는다. 이 층을 섞으면(예: MCP가 할 일을 `CLAUDE.md`에 적기) 컨텍스트 누수가 생긴다.

**피드백**

1. **Permissions**: 7가지 모드 + ML 분류기. 저위험은 허용, 고위험은 정책에 따라 질문·거부. 경계를 프롬프트로 부탁하지 않고 런타임에서 강제([[lectures/lecture-07]]).
2. **Hooks**: `PostToolUse`가 도구 실행 뒤 검사를 강제하고 결과를 컨텍스트에 기록, `Stop`은 완료 선언 시점에 개입. Anthropic이 관찰한 "agent가 자기 작업을 자신 있게 칭찬"하는 현상에 대한 결정론적 대응([[lectures/lecture-09]], [[concepts/maker-checker-separation]]).
3. **Subagent**: 대화 기록을 독립 sidechain 파일에 저장해 부모 컨텍스트를 부풀리지 않는다.

**관찰·영속화**: `history.jsonl` 전체 기록, `/compact`·`/clear`·`/init` 명령. `/init`은 코드베이스를 분석해 빌드 명령·테스트·규약을 담은 초기 `CLAUDE.md`를 만든다([[lectures/lecture-06]]의 초기화를 명령 하나로).

**원문이 꼽는 참고할 설계**: ① 지시를 범위별로 계층화 ② compaction은 무손실 먼저, 손실 나중의 깔때기 ③ hooks로 결정론적 검사 ④ subagent 컨텍스트 격리 ⑤ session 저장소는 추가 지향·재생 가능.

## 다른 하네스와의 비교

| 항목 | claude-code (이 페이지) | [[harness/codex\|codex]] | [[harness/deepseek\|deepseek]] | [[harness/pi\|pi]] | [[harness/oh-my-claudecode\|oh-my-claudecode]] | [[harness/gajae-code\|gajae-code]] |
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

- 코스 개관 글(`harness-designs/index.md`)은 네 제품을 이렇게 한 줄로 대비한다: Pi = 극도로 단순한 커널 + 프로그래밍 가능한 확장, Claude Code = 완전한 실행 환경, Codex = 저장소를 사실의 원천으로 삼고 `AGENTS.md`는 디렉터리 페이지로, DeepSeek = 하네스 자체를 모델과 독립된 런타임으로.
- [[harness/oh-my-claudecode]]는 Claude Code **위에** 얹는 플러그인이다. 이 페이지의 장치 중 OMC가 직접 기대는 것은 `Stop` hook(지속 모드 `persistent-mode`가 턴 종료를 막음)과 스킬·에이전트 정의다. 즉 "Claude Code가 주는 것 = hooks·스킬·subagent라는 부품, OMC가 더하는 것 = 그 부품으로 짠 `ralph`·`team`·`autopilot` 워크플로와 키워드 자동 감지".
- [[harness/gajae-code]]는 Claude Code를 쓰지 않는 독립 CLI지만, 프로젝트 지시 파일·스킬·역할 agent라는 같은 층 구분을 쓴다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/harness-designs/claude-code/index.md`
- en: `raw/learn-harness-engineering/docs/en/harness-designs/claude-code/index.md`
- 개관: `raw/learn-harness-engineering/docs/ko/harness-designs/index.md` (코스 하네스 4개의 aux로 등록)
- 원문이 드는 1차 출처: Claude Code 공식 문서(Memory, Skills, MCP, Hooks, Sub-agents), VILA Lab 《Dive into Claude Code》 소스 분석 보고서(5단계 compaction, 7 permissions 모드, sidechain, `history.jsonl`), Anthropic 《Effective harnesses for long-running agents》. 이 위키는 이 외부 문서들을 직접 확인하지 않았다.
- 원문 끝 관련 강의는 강의 3·9·10이다. 본문은 강의 4·5·6·7·10도 언급한다.
