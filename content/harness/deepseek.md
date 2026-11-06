---
type: "harness"
title: "DeepSeek Harness의 설계: 모든 것이 plugin, 모델이 본 것은 모두 기록"
aliases:
  - "DeepSeek Harness"
  - "deepseek"
  - "dsh"
sources:
  - "raw/learn-harness-engineering/docs/ko/harness-designs/deepseek/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/deepseek/index.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/ultragoal/SKILL.md"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/continuity-artifacts]]"
  - "[[lectures/lecture-11]]"
tags:
  - "type/harness"
  - "topic/tools-extensibility"
  - "topic/observability"
  - "topic/harness-model"
updated: "2026-11-06"
---

# DeepSeek Harness의 설계: 모든 것이 plugin, 모델이 본 것은 모두 기록

## 왕초보 설명

- **이게 뭐야?** DeepSeek이 낸 `dsh`(DeepSeek Harness)를 뜯어본 코스 글이다. 특징은 **하네스를 특정 AI 모델에서 떼어 낸 독립 운영체제**처럼 만든 것. 모델 연결, 도구, 대화 기록, 심지어 AI가 도는 루프까지 전부 갈아 끼울 수 있는 부품(plugin)이다.
- **비유:** 레고 기본판. 판(커널)은 아무 기능도 없고, 바퀴·엔진·조종석을 전부 블록으로 끼운다. 대신 처음부터 조립할 게 많다.
- **나는 뭘 하면 돼?** 직접 쓰기보다는 설계 아이디어 두 가지를 가져가면 된다. ① "AI가 본 것은 전부 기록한다". 나중에 왜 그렇게 했는지 되짚을 수 있다. ② 검사·권한·기억 기능을 루프 안에 하드코딩하지 말고 "도구 실행 전" 같은 지점에 끼우는 방식으로 붙인다. 아직 Developer Preview(맛보기 단계)다.

## 구조 요약

**한 문장**: 공식 정의 **Agent = Model + Environment + Tools + State**. 전통적 coding agent가 "LLM + 고정 루프 + 고정 도구"라면, `dsh`는 "모델 + plugin 커널(Cordis)"이다. 커널은 plugin 로드·언로드·의존성·이벤트만 맡고 **agent 능력을 하나도 소유하지 않는다**. 아키텍처 문서 인용: "Every part of the product is a plugin, including the model adapter, the tool registry, the session log, and the agent loop itself", "There is no privileged core to patch". 2026년 8월 Developer Preview, 저장소 `deepseek-ai/deepseek-harness`(MIT).

**다섯 하위 시스템으로 보기**(원문 매핑 표)

| 하위 시스템 | DeepSeek Harness의 구현 | 원문 평가 |
| --- | --- | --- |
| 지시 | 규칙·스킬을 모두 plugin으로 주입 | 매우 자유롭지만 `CLAUDE.md` 같은 내장 규약 없음 |
| 도구 | Service Definition → Provider → Consumer 능력 접합부 | 도구 하위 시스템 표준화의 극치 |
| 환경 | sandbox·FS·Shell Provider를 모두 교체(원격 E2B 포함) | 환경이 완전히 교체 가능 |
| 상태 | append-only Session Event Log + "Model-visible means logged" | 관찰 가능성이 제1원칙 |
| 피드백 | `tools/pre-execute`의 permission·guard·policy·hook | 피드백을 이벤트화 |

**핵심 1: 능력 접합부(capability seam)** `후보:capability-seam`. 능력마다 세 역할로 나눈다.

```
Service Definition   (인터페이스 선언)
        ↓
Service Provider     (구현: 예 Local FS / E2B FS / Remote FS)
        ↓
Consumer             (사용자: 보통 모델에 노출되는 도구)
```

Shell, Subprocess, Sandbox, Web, LLM, SubAgent도 같은 구조다. agent가 "구체적인 도구"가 아니라 "능력 인터페이스"에 의존하므로, Provider를 바꿔도 모델이 보는 도구 모양은 그대로이고 환경만 바뀐다.

**핵심 2: 이벤트 파이프라인**

```
turn/start → claim input → assemble(system prompt / context / tools)
  → agent/pre-step → step/start → LLM request(agent/request) → llm/stream
  → assistant/message → tool/call
  → tools/pre-execute(permission / guard / policy / hook)
  → tools/execute → tools/post-execute → tool/result → step/end → next turn
```

`turn/*`, `step/*`, `user/message`, `assistant/*`, `tool/*`는 영속화되는 session 이벤트이고, `agent/pre-step`, `agent/request`, `llm/stream`, `tools/*`는 plugin이 받는 확장 지점이다. 안전 검사는 `tools/pre-execute`, 메모리 주입은 `agent/pre-step`, 모델 요청 수정은 `agent/request`, 계속 추론할지는 `agent/turn-stopping`에서 한다. 루프 자체를 고치지 않고 listener로 붙인다. [[lectures/lecture-11]]의 "관측 가능성은 하네스 안에"보다 더 나아가 **루프의 모든 단계를 이벤트 지점으로** 만든다.

**핵심 3: "Model-visible means logged"**. append-only Session Event Log에 대한 런타임 불변 조건이다. "모델 요청에 들어가는 모든 것은 로그에서 재구성할 수 있어야 하며, 런타임 불변 조건이 이를 강제한다." 로그는 덮어쓰지 않고 추가만 하므로 session 상태를 재생할 수 있다. 원문은 이를 [[lectures/lecture-12]]의 "클린 상태 인계"를 엔지니어링으로 보장하는 장치로 본다([[concepts/continuity-artifacts]]).

**원문이 꼽는 참고할 설계**: ① 루프의 모든 단계를 이벤트 지점으로 ② 능력 접합부 표준화 ③ Model-visible means logged ④ append-only session 로그.

## 다른 하네스와의 비교

| 항목 | [[harness/claude-code\|claude-code]] | [[harness/codex\|codex]] | deepseek (이 페이지) | [[harness/pi\|pi]] | [[harness/oh-my-claudecode\|oh-my-claudecode]] | [[harness/gajae-code\|gajae-code]] |
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

- 원문의 대비: Pi·Claude Code·Codex는 "하나의 구체적인 agent" 안에서 하네스를 최적화하고, DeepSeek Harness는 하네스를 **모델과 독립된 운영체제**로 정의해 agent를 그 위의 교체 가능한 애플리케이션으로 만든다. 대가는 높은 설정 비용.
- [[harness/gajae-code]]의 `ultragoal`도 `ledger.jsonl`에 체크포인트를 추가만 하는 기록을 남기지만, 그 대상은 "목표별 검증 증거"이고 `dsh`처럼 "모델이 본 모든 입력"은 아니다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/harness-designs/deepseek/index.md`
- en: `raw/learn-harness-engineering/docs/en/harness-designs/deepseek/index.md`
- 원문이 드는 1차 출처: DeepSeek Harness 공식 사이트, `deepseek-ai/deepseek-harness` 저장소, `docs/architecture.md`·`cordis-primer.md`·`capability-seams.md`·`subsystems/session.md`. 이 위키는 외부 문서를 직접 확인하지 않았다(출시 시점 2026-08도 원문 주장).
- 원문 끝 관련 강의는 강의 2·11·12다.
