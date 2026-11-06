---
type: "harness"
title: "oh-my-claudecode(OMC): Claude Code 위에 얹는 무거운 멀티 에이전트 랄프톤 하네스"
aliases:
  - "OMC"
  - "오마이클로드코드"
  - "oh-my-claude-sisyphus"
sources:
  - "raw/oh-my-claudecode/README.ko.md"
  - "raw/oh-my-claudecode/README.md"
  - "raw/oh-my-claudecode/skills/ralph/SKILL.md"
  - "raw/oh-my-claudecode/skills/autopilot/SKILL.md"
  - "raw/oh-my-claudecode/skills/team/SKILL.md"
  - "raw/oh-my-claudecode/skills/deep-interview/SKILL.md"
  - "raw/oh-my-claudecode/hooks/hooks.json"
  - "raw/oh-my-claudecode/scripts/persistent-mode.mjs"
related:
  - "[[harness/gajae-code]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/verification-commands]]"
  - "[[concepts/ralph-loop]]"
  - "[[concepts/harness-subsystems]]"
tags:
  - "type/harness"
  - "topic/loop-engineering"
  - "topic/role-separation"
  - "topic/verification"
  - "topic/tools-extensibility"
updated: "2026-11-06"
---

# oh-my-claudecode(OMC): Claude Code 위에 얹는 무거운 멀티 에이전트 랄프톤 하네스

## 왕초보 설명

- **이게 뭐야?** Claude Code(Anthropic이 만든 터미널용 AI 코딩 도구)에 끼워 쓰는 **확장 팩(플러그인)**이다. 혼자 일하던 Claude 한 명을 **전문가 여러 명이 있는 팀**처럼 움직이게 바꾼다. README의 표현으로는 "당신의 Claude가 스테로이드를 맞았습니다."
- **랄프톤과 무슨 관계야?** OMC의 `ralph`는 [[concepts/ralph-loop|Ralph 루프]](목표를 주면 끝날 때까지 스스로 반복)를 그대로 구현한 모드다. AI가 "이제 그만할게요"라고 말하려 하면 **훅(hook, 특정 순간에 자동으로 끼어드는 스크립트)**이 그 종료를 막고 "아직 안 끝났다, 계속해"라고 다시 밀어 넣는다.
- **왜 "무겁다"고 해?** 스킬 48개, 전문 에이전트 19개, 훅 명령 30개(이벤트 11종)가 함께 돈다. 단계마다 다른 에이전트와 다른 크기의 모델을 쓰고, 여러 작업자를 동시에 띄운다. 큰 일을 오래 맡기기에는 강력하지만, 처음 쓰는 사람에게는 무엇이 돌고 있는지 보기 어렵다.
- **나는 뭘 하면 돼?** Claude Code 안에서 세 단계면 시작할 수 있다.

```text
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
/omc-setup
```

그다음 원하는 걸 말하면 된다. 예: `autopilot: build a REST API for managing tasks`. 뭘 만들지 막연하면 `/deep-interview "만들고 싶은 것"`부터 한다.

## 구조 요약

### 한 장 그림: 실행 모드 고르기

```text
 막연한 아이디어 ──► /deep-interview ──► /ralplan ──► /autopilot (Phase 2부터 시작)
                     (모호함 ≤ 20%)       (Planner·Architect·Critic 합의)

 "끝까지 해"      ──► ralph     prd.json의 이야기(story)를 하나씩: 구현 → 검증 → 다음
 "알아서 다 해"   ──► autopilot Phase 0 확장 → 1 계획 → 2 실행 → 3 QA(최대 5회) → 4 다각도 검증 → 5 정리
 "여럿이 나눠서"  ──► /team N:executor   team-plan → team-prd → team-exec → team-verify → team-fix(반복)
 다른 AI CLI 동원 ──► omc team 2:codex / 2:gemini / 1:claude   (tmux 창마다 실제 CLI 프로세스)
```

### 계속 돌게 만드는 장치: Stop 훅

- `hooks/hooks.json`은 Claude Code의 거의 모든 순간(프롬프트 제출, 도구 사용 전후, 하위 에이전트 시작·종료, 압축 직전, 멈춤, 세션 종료)에 스크립트를 건다.
- 핵심은 **Stop**(AI가 턴을 끝내려는 순간)에 걸린 `persistent-mode.mjs`다. ralph·ultragoal·autopilot·team 같은 모드가 켜져 있으면 `decision: "block"`을 돌려줘 종료를 막고, 남은 일을 다시 지시한다.
- 무한 루프 방지 상한: 기본 500회. `OMC_SECURITY=strict`이면 200회로 줄어든다.
- 안전 예외: 컨텍스트 한도 초과나 인증 실패(401/403)로 멈출 때는 막지 않는다. 다른 Stop 훅이 이미 돌고 있을 때도 다시 막지 않는다(Claude Code의 반복 차단 보호와 충돌하지 않도록).

비유하면, 퇴근하려는 직원을 문 앞에서 "체크리스트 아직 남았어요"라며 돌려보내는 경비원이다.

### ralph 한 바퀴

1. **PRD 준비**: `.omc/state/sessions/<id>/prd.json`에 일을 이야기(user story) 단위로 쪼개고, 이야기마다 확인 가능한 완료 기준을 쓴다. 자동 생성된 "구현 완료" 같은 뭉뚱그린 기준은 반드시 구체적으로 바꾼다.
2. **기준선 기록**: 첫 바퀴에 `omc ralph verify --write-baseline`. 빌드·테스트·린트(`feedbackCommands`)를 돌려 "원래부터 실패하던 것"을 적어 둔다.
3. **이야기 하나 구현 → 검증**: `omc ralph verify`가 기준선 대비 **새로 생긴 실패만** 문제로 본다. AI가 테스트 결과를 눈으로 대충 보고 판단하는 것은 금지다.
4. **모든 이야기 통과 → 리뷰어 검증**: 기본 리뷰어는 `architect`. `--critic=critic` 또는 `--critic=codex`로 바꿀 수 있다. 거절되면 고치고 같은 리뷰어에게 다시 받는다.
5. **정리(deslop)와 종료**. 진행과 배운 점은 `progress.txt`에 쌓인다.

추가 장치: 측정해 보니 완료 기준이 틀렸다면 몰래 지우지 않고, 원문과 근거를 남기는 "기준 수정 기록(`criterionAmendments`)"으로만 바꾼다. 저장소 성격(`prototype`/`production`/`library`)에 따라 검증 강도가 달라진다. 사람이 없을 때는 `omc ralph afk`로 돌린다.

### 다섯 하위 시스템으로 보기

[[concepts/harness-subsystems|하네스 5개 하위 시스템]]으로 나누면 이렇다.

| 하위 시스템 | OMC에서 맡는 것 | 왕초보용 한 줄 |
| --- | --- | --- |
| 지시 | 스킬 48개(`ralph`, `autopilot`, `team`, `ralplan`, `deep-interview`, `ultragoal` …), 에이전트 정의 19개, 매직 키워드(`ralph`, `autopilot`, `team` …) 자동 감지 훅 | 말만 하면 맞는 매뉴얼이 자동으로 펼쳐진다 |
| 도구 | Claude Code 도구 + 하위 에이전트 + MCP 도구 + tmux 위 Codex·Gemini·Claude CLI 작업자 | 다른 회사 AI까지 팀원으로 부른다 |
| 환경 | Claude Code 플러그인 실행 환경, tmux, 실험 기능 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | Claude Code가 먼저 깔려 있어야 한다 |
| 상태 | `.omc/state/`(모드별 상태 파일, 세션별 `prd.json`), `progress.txt`, `.omc/plans/`, `.omc/specs/`, 프로젝트 메모리 훅, 압축 직전 저장 훅 | 대화가 길어져 요약돼도 진행 기록은 파일에 남는다 |
| 피드백 | `omc ralph verify` 기준선 비교, 단계별 검증 에이전트(`verifier`, `code-reviewer`, `security-reviewer`), autopilot QA 반복(최대 5회, 같은 오류 3번이면 멈추고 보고), 하위 에이전트 산출물 검사 훅 | 검사 담당이 여러 명이고, 검사 방법도 기계로 고정했다 |

### 모델 크기 나눠 쓰기

`team`은 단계마다 기본 에이전트와 모델 등급을 정해 둔다. 예: `team-plan`은 `explore`(haiku)와 `planner`(opus), `team-exec`는 `executor`(sonnet), `team-verify`는 `verifier`(sonnet). README는 이 라우팅으로 토큰을 30~50% 아낀다고 주장한다. 이 수치의 측정 근거는 README에 없다.

### 특징 한눈에

| 특징 | 왕초보용 설명 | 근거 |
| --- | --- | --- |
| Claude Code 플러그인 | 혼자 도는 프로그램이 아니라 Claude Code에 끼워 쓴다. Claude Code와 그 구독(또는 API 키)이 먼저 있어야 한다 | README 빠른 시작 |
| 매직 키워드 | `autopilot`, `ralph`, `ralplan`, `team` 같은 단어를 말하면 훅이 알아채고 해당 모드를 켠다 | README 매직 키워드, `keyword-detector.mjs` |
| 실행 모드 여러 개 | Team(단계별 팀), omc team(다른 AI CLI 작업자), Autopilot(전 과정 자동), Ralph(끝까지 반복), Pipeline(순서대로 변환) | README 실행 모드 |
| 전문 에이전트 19개 | 분석가, 설계자, 디버거, 디자이너, 보안 리뷰어, 테스트 엔지니어, 문서 작성자 등으로 나뉜다 | `agents/` |
| 모델 크기 자동 배분 | 쉬운 일은 작은 모델(haiku), 어려운 추론은 큰 모델(opus)에 맡긴다 | README, team SKILL.md |
| 다른 회사 AI를 작업자로 | tmux 창마다 Codex·Gemini·Claude CLI를 실제로 띄워 일을 나눈다. 일이 끝나면 창을 닫는다 | README tmux CLI 워커 |
| Stop 훅으로 끝까지 | AI가 멈추려 하면 훅이 막고 남은 일을 다시 시킨다(상한 기본 500회) | `persistent-mode.mjs` |
| 상태바(HUD) | 화면 아래에서 지금 도는 모드와 진행 상황을 본다 | README HUD |
| 스킬 학습 | 디버깅하며 알게 된 요령을 스킬 파일로 뽑아 두고, 비슷한 상황에서 자동으로 불러온다(`/skillify`) | README 커스텀 스킬 |
| 운영 편의 | 사용량 한도가 풀리면 자동 재개(`omc wait`), 끝나면 Telegram/Discord/Slack 알림 | README 유틸리티 |

### 장점 (왕초보 관점)

1. **시작이 쉽다.** 설치 3줄 뒤 "autopilot: 만들고 싶은 것"이라고만 말하면 된다. 명령어를 외울 필요가 적다.
2. **큰 일에 강하다.** 여러 에이전트·여러 회사 AI를 동시에 굴리고, 단계마다 전문 에이전트를 붙인다. 큰 덩어리를 쪼개 병렬로 처리하는 데 맞춰져 있다.
3. **잘 안 멈춘다.** Stop 훅이 "다 했어요"를 막고 완료 기준이 채워질 때까지 다시 돌린다. 완료 기준 자체도 측정 근거 없이 지우지 못하게 기록을 남긴다.
4. **검증을 기계에 맡긴다.** `omc ralph verify`가 처음부터 실패하던 것과 새로 생긴 실패를 구분해 준다. AI가 결과를 눈으로 보고 "괜찮아 보여요"라고 하는 걸 막는다.
5. **보이는 게 많다.** HUD 상태바, 토큰 사용량 추적, 세션 요약 알림이 있다.

### 단점과 주의할 점

1. **Claude Code 전용이다.** Claude Code 밖에서는 못 쓴다. Codex 중심이면 별도 프로젝트 `oh-my-codex`를 쓴다(README 안내).
2. **무겁고 속이 잘 안 보인다.** 스킬 48개, 에이전트 19개, 훅 명령 30개가 함께 돈다. 이상하게 동작할 때 어느 훅·스킬 때문인지 찾기 어렵다. 도움말: `/omc-doctor`.
3. **모드가 말 한마디로 켜진다.** 프롬프트에 매직 키워드가 들어가면 의도보다 큰 모드가 켜질 수 있다. 멈추려면 `/oh-my-claudecode:cancel`.
4. **토큰을 많이 쓸 수 있다.** 루프 상한이 기본 500회로 넉넉하고, 여러 에이전트를 동시에 띄운다. 예산 상한(`OMC_RUN_BUDGET_TOKENS`)은 직접 켜야 하는 선택 기능이다.
5. **준비물이 늘어난다.** Team은 Claude Code 실험 기능(`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`)을 켜야 하고, CLI 작업자는 tmux와 `codex`·`gemini` CLI 설치가 필요하다.
6. **문서와 코드가 어긋난 곳이 있다.** README의 "32개 에이전트"와 실제 19개처럼, 숫자는 코드로 확인한다.

### 무겁다는 게 무슨 뜻인가

이 위키에서 "무겁다"는 기능의 양을 뜻한다. 작업 시간을 뜻하지 않는다. 작업자를 여럿 띄우고(`team`), 단계가 많고(`autopilot` 6단계), Stop 훅이 최대 500회까지 루프를 붙잡는다. 큰 작업을 사람 없이 오래 굴리는 데 맞춘 설계다.

## 다른 하네스와의 비교

| 항목 | [[harness/claude-code\|claude-code]] | [[harness/codex\|codex]] | [[harness/deepseek\|deepseek]] | [[harness/pi\|pi]] | oh-my-claudecode (이 페이지) | [[harness/gajae-code\|gajae-code]] |
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

고르는 법은 [[harness/gajae-code#무엇을 고를까 (왕초보용)|gajae-code 페이지의 선택 표]]에 정리했다.

- OMC는 [[harness/claude-code]] 하네스 **위에** 한 층을 더 얹는 구조다. Claude Code가 주는 것은 `CLAUDE.md` 계층, hooks(`Stop`·`PostToolUse` 등), 스킬, subagent, permissions, compaction 같은 부품이고, OMC가 더하는 것은 그 부품으로 짠 `ralph`·`team`·`autopilot` 워크플로, 키워드 자동 감지, `.omc/` 상태 파일이다. 특히 OMC의 지속 모드는 Claude Code의 `Stop` hook 지점을 이용해 턴 종료를 막는다.
- [[harness/gajae-code]]는 `NOTICE.md`에서 OMC의 워크플로 실험을 계보로 밝힌다. 두 하네스가 `deep-interview`·`ralplan`이라는 같은 이름을 쓰는 이유다. 다만 실행 루프는 OMC의 `ralph` 대신 `ultragoal`로 바꿨고, 계속 돌게 하는 방식도 Stop 훅 차단이 아니라 goal 모드 연속 실행이다.

## 원문 경로

- ko: `raw/oh-my-claudecode/README.ko.md`
- en: `raw/oh-my-claudecode/README.md`
- 스킬: `raw/oh-my-claudecode/skills/{ralph,autopilot,team,deep-interview}/SKILL.md`
- 훅: `raw/oh-my-claudecode/hooks/hooks.json`, `raw/oh-my-claudecode/scripts/persistent-mode.mjs`
- 스냅숏: 커밋 `0564555b63dfedd158669c6799d53743abd7d6bf`(2026-11-03), 패키지 버전 5.7.31, npm 이름 `oh-my-claude-sisyphus`

**원문과 코드의 차이**

- README(ko)는 "32개의 전문 에이전트"라고 쓰지만, 스냅숏의 `agents/`에는 정의 파일이 19개다(`analyst`, `architect`, `code-reviewer`, `code-simplifier`, `critic`, `debugger`, `designer`, `document-specialist`, `executor`, `explore`, `git-master`, `planner`, `qa-tester`, `scientist`, `security-reviewer`, `test-engineer`, `tracer`, `verifier`, `writer`). README 수치는 갱신되지 않은 것으로 보인다.
- README(ko)는 v4.4.0에서 Codex/Gemini MCP 서버가 "제거됩니다"라고 미래형으로 쓰지만, 스냅숏은 5.7.31이다. 현재는 `omc team …` CLI 작업자가 그 역할을 한다.
- README(ko) 곳곳에 `TODO(i18n): verify translation` 주석이 남아 있다. 번역이 애매하면 en을 기준으로 한다.
