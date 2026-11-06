---
type: "harness"
title: "Pi의 하네스 설계: 극도로 단순한 커널 + 프로그래밍 가능한 확장"
aliases:
  - "Pi"
  - "pi"
  - "Pi coding agent"
  - "pi-coding-agent"
sources:
  - "raw/learn-harness-engineering/docs/ko/harness-designs/pi/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/pi/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/deepseek/index.md"
  - "raw/gajae-code/NOTICE.md"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/loop-engineering]]"
  - "[[harness/gajae-code]]"
tags:
  - "type/harness"
  - "topic/tools-extensibility"
  - "topic/context-continuity"
  - "topic/instructions"
updated: "2026-11-06"
---

# Pi의 하네스 설계: 극도로 단순한 커널 + 프로그래밍 가능한 확장

## 왕초보 설명

- **이게 뭐야?** 스스로를 "minimal agent harness(최소한의 하네스)"라고 부르는 터미널 AI 코딩 도구 Pi를 뜯어본 코스 글이다. 기본 기능은 일부러 적게 두고, 필요한 건 **확장(내가 짜 넣는 작은 프로그램)이나 남이 만든 패키지**로 붙인다.
- **비유:** 빈 원룸에 이사 가기. 가구가 거의 없지만 원하는 대로 채울 수 있다. 대신 직접 사 와야 한다.
- **나는 뭘 하면 돼?** 프로젝트에 `AGENTS.md`를 두면 Pi가 읽는다(`CLAUDE.md`도 읽음). 테스트 같은 확인 명령은 Pi가 강제하지 않으니 `AGENTS.md`에 직접 적는다. 대화가 길어지면 `/compact`, 예전 지점으로 돌아가려면 `/tree`.

## 구조 요약

**한 문장**: 커널 최소화 + 확장의 프로그래밍화. 컨텍스트 엔지니어링을 시스템 프롬프트 바깥에 두고, Pi가 사용자 대신 하네스를 정하는 게 아니라 **사용자(심지어 Pi 자신)가 하네스를 고치게** 한다. 포지셔닝 원문: "Ask Pi to build what you want, or install a package that does it your way". npm 패키지 `@earendil-works/pi-coding-agent`.

**네 가지 맞춤 계층**: 확장(Extensions, 생명주기 이벤트에 붙는 TypeScript hooks) / 스킬(필요할 때 로드하는 능력 패키지, [[concepts/progressive-disclosure|점진적 공개]]) / 프롬프트 템플릿(`/name`으로 펼치는 Markdown) / 테마(TUI 외형).

**다섯 하위 시스템으로 보기**(원문 매핑 표, 원문이 "주관적 비교용"이라 밝힘)

| 하위 시스템 | Pi의 구현 | 원문 평가 |
| --- | --- | --- |
| 지시 | `AGENTS.md` 계층 로드 + `SYSTEM.md` | 계층은 명확하지만 규칙은 사용자가 작성 |
| 도구 | 스킬 필요 시 로드 + 전체 생명주기 확장 hooks | 도구 시스템을 프로그래밍 가능한 표면으로 |
| 환경 | `SYSTEM.md`로 환경 자기 설명, 런타임 환경은 사용자가 `AGENTS.md`에 | 재현성은 사용자 설명에 의존 |
| 상태 | session 트리 + 맞춤 compaction + `PROGRESS.md` | session 간 연속성·복구가 핵심 |
| 피드백 | 검증 명령은 사용자 정의, session-summary·extract-patterns로 메커니즘화 | 메커니즘은 주고 내용은 사용자가 |

**지시**: `AGENTS.md` 로드 순서는 전역 `~/.pi/agent/AGENTS.md` → 상위 디렉터리를 단계별로 → 현재 `./AGENTS.md`(`CLAUDE.md` 호환). `SYSTEM.md`는 프로젝트별로 기본 시스템 프롬프트를 교체(replace)하거나 덧붙인다(append). 시스템 프롬프트 자체는 극도로 단순하게 두고 규칙은 스킬·확장으로 필요할 때만 나타나게 해, [[lectures/lecture-04]]의 거대 지시 파일 문제를 피한다([[concepts/map-not-manual]]).

**상태와 컨텍스트**(원문이 가장 자세히 다룸)

1. **프로그래밍 가능한 compaction**: 전략 자체를 확장으로 바꿀 수 있다(주제 기반, 코드 인식 요약, 다른 모델로 요약). 기본 동작은 컨텍스트 오버플로 복구 또는 보존 임계값 초과 시 트리거, 최근 약 2만 token을 남기고 이전 메시지는 "context handoff"로 요약해 단계별 연쇄 압축.
2. **동적 컨텍스트**: 확장이 매 추론 전에 메시지 주입, 기록 필터링, RAG, 장기 메모리를 구현한다. 창에 들어가기 **전에** 넣고 뺄 것을 정한다.
3. **session 트리**: "sessions are stored as trees". `/tree`로 과거 노드로 돌아가 이어 가고, 모든 브랜치는 한 파일에. HTML 내보내기·gist 공유. 요약으로 이어 붙이는 대신 구조화된 기록 재생([[concepts/continuity-artifacts]]).

**도구**: 스킬은 "self-contained capability packages that the agent loads on-demand". 트리거될 때만 상세가 컨텍스트에 들어가 **프롬프트 캐시를 깨지 않는다**(비용 관점의 설계). 확장(hooks)의 공식 활용 예: 위험 명령 가로채기(permissions gate), 작업 전환 시 코드 상태 checkpoint, 경로 보호(`.env` 쓰기 금지), 도구 출력 수정 후 전달, 외부(파일 감시·Webhook·CI)에서 메시지를 넣어 agent 깨우기.

**피드백**: 강제 테스트 gate는 내장돼 있지 않다. 커뮤니티 하네스 `pi-agent-harness`가 확장으로 피드백 루프를 짠다: session-summary(순환 `PROGRESS.md`), extract-patterns(`LESSONS.md`에 교훈 축적), telemetry(token·비용), 그리고 `VISION.md`·`PROGRESS.md`·`LESSONS.md`·`STANDARDS.md` 파일 체계.

**루프 엔지니어링과의 연결**: TUI 외에 print/JSON 모드, RPC, SDK 임베딩을 지원해 CI/CD나 다른 프로그램이 같은 하네스를 구동할 수 있다. 원문은 이것을 [[lectures/lecture-13]]의 "수동 구동에서 자동 루프로"의 전제로 본다([[concepts/loop-engineering]]).

**원문이 꼽는 참고할 설계**: ① compaction 전략을 교체 가능하게 ② session 트리로 단순 요약 대체 ③ 프롬프트 캐시 친화(스킬 필요 시 로드) ④ agent가 자기 하네스를 고칠 수 있게.

## 다른 하네스와의 비교

| 항목 | [[harness/claude-code\|claude-code]] | [[harness/codex\|codex]] | [[harness/deepseek\|deepseek]] | pi (이 페이지) | [[harness/oh-my-claudecode\|oh-my-claudecode]] | [[harness/gajae-code\|gajae-code]] |
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

- 원문의 대비: [[harness/claude-code]]는 메모리·permissions·subagent를 커널에 내장, [[harness/codex]]는 저장소 규약·환경 격리를 기본값으로, Pi는 **아무것도 대신 정하지 않고** 결정권을 확장 지점으로 넘긴다. 대가는 직접 확장을 짜거나 패키지를 설치해야 한다는 것.
- [[harness/gajae-code]]는 `NOTICE.md`에서 `can1357/oh-my-pi`를 계보로 밝힌다. 이름이 비슷하지만, oh-my-pi와 이 페이지의 Pi(`badlogic/pi-mono`)의 관계는 이 위키가 가진 원문으로는 확인되지 않는다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/harness-designs/pi/index.md`
- en: `raw/learn-harness-engineering/docs/en/harness-designs/pi/index.md`
- 원문이 드는 1차 출처: pi.dev 공식 사이트와 문서(Sessions, Extensions, Project Context), `badlogic/pi-mono`의 `packages/coding-agent/README.md`, 커뮤니티 `LabidySabidy/pi-agent-harness`. 이 위키는 외부 문서를 직접 확인하지 않았다.

**원문 안의 의문점**

- 패키지 이름이 두 가지다: npm `@earendil-works/pi-coding-agent`(첫 문단)와 hooks API `@mariozechner/pi-coding-agent/hooks`(도구 절). 소스 저장소는 `badlogic/pi-mono`로 적는다.
- 확장 절의 예시 이벤트 이름 `agent/pre-step`, `agent/request`는 [[harness/deepseek]] 글의 이벤트 파이프라인 이름과 똑같다. Pi 문서의 실제 이벤트 이름인지, 다른 글의 표현이 섞인 것인지 원문만으로는 알 수 없다.
- 원문 끝 관련 강의는 강의 2·5·13이다.
