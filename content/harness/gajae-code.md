---
type: "harness"
title: "Gajae-Code(gjc): 계획 먼저, 증거로 끝내는 가벼운 랄프톤 하네스"
aliases:
  - "가재코드"
  - "gjc"
  - "Gajae-Code"
  - "gajae 하네스"
sources:
  - "raw/gajae-code/README.ko.md"
  - "raw/gajae-code/README.md"
  - "raw/gajae-code/NOTICE.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/deep-interview/SKILL.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/ralplan/SKILL.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/ultragoal/SKILL.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/autoresearch/SKILL.md"
  - "raw/gajae-code/packages/coding-agent/src/prompts/agents/critic.md"
  - "raw/gajae-code/packages/coding-agent/src/prompts/agents/executor.md"
  - "raw/gjc-beginner-guide.html"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/verification-commands]]"
  - "[[concepts/ralph-loop]]"
  - "[[harness/oh-my-claudecode]]"
tags:
  - "type/harness"
  - "topic/loop-engineering"
  - "topic/role-separation"
  - "topic/verification"
  - "topic/context-continuity"
updated: "2026-11-06"
---

# Gajae-Code(gjc): 계획 먼저, 증거로 끝내는 가벼운 랄프톤 하네스

> 가볍다 = 기본 기능이 적고(스킬 4개 + 역할 4개) 흐름이 하나라는 뜻이다. 작업 시간을 뜻하지 않는다.

## 왕초보 설명

- **이게 뭐야?** `gjc`(가재코드)는 터미널에서 쓰는 AI 코딩 도우미다. "이거 고쳐줘"라고 하면 코드를 읽고, 계획하고, 고치고, 제대로 됐는지 확인까지 한다. Claude, ChatGPT, Gemini 같은 AI를 **이미 내고 있는 월 구독**으로 연결해 쓴다.
- **하네스로서 특징은?** "바로 고치지 말고 먼저 묻고, 계획하고, 검토받은 다음 고친다." 그리고 "다 했어요"를 말이 아니라 **증거 기록**으로 끝낸다.
- **랄프톤이 뭐야?** 랄프톤(Ralph-thon)은 "Ralph 루프"를 쓰는 해커톤을 부르는 말이다. [[concepts/ralph-loop|Ralph 루프]]는 AI에게 목표를 주고 **끝날 때까지 스스로 반복**(만들기 → 확인 → 고치기)하게 하는 방식이다. gjc에는 `ralph`라는 명령이 없다. 대신 `deep-interview → ralplan → ultragoal` 세 단계와 자동 넘김 설정으로 같은 일을 한다.
- **나는 뭘 하면 돼?** 처음엔 아래 세 줄만 기억하면 된다.

```text
/skill:deep-interview 하고 싶은 일을 대충 적기     ← AI가 질문하며 요구사항을 정리 (코드 안 건드림)
/skill:ralplan                                      ← 계획 → 구조 검토 → 비판 (코드 안 건드림)
"Approve execution via ultragoal" 선택              ← 승인하면 목표별로 끝까지 실행
```

- **주의:** 실행 단계(ultragoal)는 git 커밋 기록으로 "무엇이 바뀌었는지"를 계산한다. git을 쓰지 않거나 커밋이 하나도 없는 폴더에서는 목표를 "완료"로 기록하지 못하고 막힌다. 오래 돌릴 때는 `gjc --tmux --worktree 작업이름`으로 원본과 분리된 폴더에서 돌린다.

## 구조 요약

### 한 장 그림

```text
 사람: 대충 적은 요구
        │
        ▼
 ① deep-interview ── 질문 → "모호함 점수" 계산 → 기준 이하가 될 때까지 반복 → 명세(spec) 파일
        │                                        (읽기 전용)
        ▼
 (선택) autoresearch ── 웹·데이터로 조사 → 근거 붙은 결론(verdict)        (코드 변경 없음)
        │
        ▼
 ② ralplan ── PLANNER 계획 → ARCHITECT 구조 검토 → CRITIC 비판 → 합의까지 반복(기본 최대 5회)
        │                                        (읽기 전용)
        ▼
 승인 게이트 ── 사람이 "Approve execution via ultragoal" 선택
        │        또는 gjc.ralplan.autoHandoff = ultragoal 이면 질문 없이 통과 (= 완전 자동, 랄프톤 방식)
        ▼
 ③ ultragoal ── 계획을 목표 G001, G002 … 로 나눔
                 목표마다: 구현 → 검증 → 증거 기록(checkpoint)
                 마지막 목표에서만 무거운 검토(정리·구조 리뷰·QA) + 최종 CRITIC "OKAY"
```

### 다섯 하위 시스템으로 보기

[[concepts/harness-subsystems|하네스 5개 하위 시스템]] 틀로 나누면 이렇다.

| 하위 시스템 | gjc에서 맡는 것 | 왕초보용 한 줄 |
| --- | --- | --- |
| 지시 | `AGENTS.md`, 번들 스킬 4개(`deep-interview`, `ralplan`, `ultragoal`, `autoresearch`), 역할 프롬프트 4개(`executor`, `architect`, `planner`, `critic`) | "어떤 순서로 일하라"는 매뉴얼이 처음부터 들어 있다 |
| 도구 | 파일 읽기·검색·편집, 셸, LSP, 웹 검색, 하위 에이전트 위임. `architect`·`planner`·`critic`은 읽기 도구와 허용된 `git status`·`git diff` 등만 쓴다 | 검토하는 역할은 코드를 고칠 수 없게 손을 묶어 둔다 |
| 환경 | `--worktree`(분리된 작업 폴더), `--tmux`(창을 닫아도 계속 실행), 구독 로그인, 프로바이더 재시도 예산 | 원본을 지키는 별도 작업실 |
| 상태 | 세션마다 `.gjc/_session-<id>/` 아래 `specs/`(인터뷰 명세), `plans/ralplan/<run>/`(계획 단계별 기록), `ultragoal/goals.json`(목표 목록), `ultragoal/ledger.jsonl`(증거 일지) | 어디까지 했는지 파일로 남아서 창을 닫아도 이어진다 |
| 피드백 | 모호함 점수 게이트, 합의 게이트(Critic `OKAY` + Architect 승인), 목표별 `quality-gate`(실행한 검증 명령과 결과), 마지막 경계 검토와 종료 CRITIC 게이트 | "다 했어요"는 증거 파일이 있어야 인정된다 |

### 단계별로 조금 더

**① deep-interview — 질문으로 요구 정리**

- 질문을 하나씩 하고, 답할 때마다 요구가 얼마나 흐릿한지 "모호함 점수"를 매긴다. 점수가 기준(threshold) 아래로 내려가면 명세를 쓴다.
- 기준값: 기본 `0.05`(5%). `--quick` 0.6, `--standard` 0.5, `--deep` 0.35. 첫 줄에 `Deep Interview threshold: …`를 출력해 어떤 기준으로 도는지 보여 준다.
- 명세에는 목표, 제약, 하지 않을 것(Non-Goals), 완료 기준(Acceptance Criteria), 인터뷰 기록이 들어간다.

**② ralplan — 세 역할이 합의할 때까지 계획**

- PLANNER가 계획과 요약(원칙 3~5개, 결정 요인 3개, 선택지 2개 이상)을 쓰고, ARCHITECT와 CRITIC이 검토한다([[concepts/maker-checker-separation|일하는 쪽과 확인하는 쪽 분리]]). CRITIC은 "실행 가능하고 검증 가능한 계획만 승인"하는 읽기 전용 역할이다.
- 합의가 안 되면 고쳐서 다시 검토한다. `gjc.ralplan.maxIterations`(기본 5)를 넘으면 `PLANNING-STUCK`로 멈추고, 자동 넘김이 켜져 있어도 **실행하지 않는다**.
- 계획 기록은 `gjc ralplan --write`로만 저장된다. 계획 단계에서는 제품 코드 수정, 커밋, 푸시가 코드로 막혀 있다.

**승인 게이트 — 반자동 vs 완전 자동** (`raw/gjc-beginner-guide.html` 7장)

| | 반자동(기본값, 왕초보 추천) | 완전 자동(랄프톤 방식) |
| --- | --- | --- |
| 설정 | `gjc config set gjc.ralplan.autoHandoff off` | `gjc config set gjc.ralplan.autoHandoff ultragoal` |
| 계획이 끝나면 | 승인 질문이 뜨고 내가 고른다 | 질문 없이 바로 ultragoal 시작 |
| 장점 | 계획을 보고 고칠 수 있다 | 자리를 비워도 끝까지 간다 |
| 단점 | 승인할 때 자리에 있어야 한다(폰 알림으로 해결) | 계획이 틀려도 그대로 실행 → 반드시 worktree에서 |

**③ ultragoal — 목표별로 끝까지 실행하고 증거 남기기**

- 승인된 계획을 `@goal:` 줄로 나눠 목표 `G001`, `G002` …를 만든다. 검증이 서로 묶인 이야기(같은 기능 묶음, 같은 완료 기준)는 한 목표로 합치고, 그 안에서 `executor`를 여러 개 나눠 돌린다.
- 목표 하나가 끝날 때마다 `gjc ultragoal checkpoint --status complete --evidence "…" --quality-gate-json …`로 **무슨 검증을 돌렸고 통과했는지**를 기록한다. 변경 파일 목록과 해시(`changeSetHash`)는 git diff에서 자동 계산된다.
- 중간 목표는 가벼운 검증만 하고, **마지막 목표(경계)에서 한 번** 무거운 검토를 한다: 코드 정리, `architect` 구조 리뷰, `executor` QA를 같은 스냅숏에 동시에 돌리고 결과를 합친다.
- 끝낼 때(완료든 일시정지든) 읽기 전용 `critic`의 `OKAY`가 있어야 한다.
- 막혔을 때: AI가 해결할 수 있는 문제(`resolvable`: 테스트 실패, 설치 누락 등)면 **멈추지 않고** 계속 시도한다. 사람만 풀 수 있는 문제(`human_blocked`: 비밀번호, 결제, 승인)일 때만 일시정지할 수 있다. 실행 중에는 질문(ask)도 막혀 있어서, 결정할 일은 막힘 기록으로 남긴다.
- 끈기: `gjc.ultragoal.nudgeBudget`(기본 10)은 목표 하나에서 포기하기 전까지 다시 밀어붙이는 횟수다. "goal 모드"(`goal.enabled`)가 켜져 있어야 턴이 끝나도 다음 작업으로 이어진다.

**(선택) autoresearch — 조사 전용**

코드를 만들지 않고, 웹 검색과 데이터 실험으로 질문 하나에 대한 결론(`status`, `evidence[]`, `caveats[]`)을 낸다. 보통 deep-interview 다음, ralplan 전에 넣는다.

### 계보

- 원문(`NOTICE.md`)상 gjc는 [`oh-my-pi`](https://github.com/can1357/oh-my-pi)를 "윗줄 계보와 구현 DNA"로 삼고, `oh-my-codex`와 `oh-my-claudecode`의 워크플로 실험에서 배운 것을 이어받았다. 그래서 스킬 이름(`deep-interview`, `ralplan`, `ultragoal`)이 [[harness/oh-my-claudecode]]와 겹치고, 각 SKILL.md에 "forked from upstream … rebranded for GJC"라고 적혀 있다.
- 코스의 [[harness/pi]] 분석 대상(`badlogic/pi-mono`)과 oh-my-pi는 이름이 비슷하지만, 둘의 관계는 이 위키가 가진 원문(`NOTICE.md`, 코스 Pi 분석)으로는 확인되지 않는다. 설계 면에서는 둘 다 작은 기본 구성에 스킬·확장을 얹는 쪽이고, gjc는 그 위에 계획·실행 워크플로 4개를 기본으로 묶었다.

### 특징 한눈에

| 특징 | 왕초보용 설명 | 근거 |
| --- | --- | --- |
| 독립 실행 프로그램 | Claude Code 같은 다른 도구 없이 `gjc` 하나만 설치하면 된다. 설치기는 미리 만든 실행 파일이라 Bun·Node도 필요 없다 | README 빠른 시작 |
| 여러 회사 구독을 그대로 | Claude Pro/Max, ChatGPT Plus/Pro(Codex), Cursor, GitHub Copilot, Kimi, Grok 등을 `/login`으로 연결한다. API 키, 로컬 모델(Ollama 등)도 된다 | README "쓰던 코딩 플랜 그대로" |
| 역할마다 다른 모델 | 계획·검토·구현 역할에 서로 다른 회사 모델을 배정할 수 있다(프리셋으로 한 번에) | README, 가이드 5장 |
| 기본 기능이 딱 4+4개 | 워크플로 스킬 4개, 역할 에이전트 4개만 들어 있다. 디스크의 다른 파일로 교체할 수 없다 | README "변경 전에 계획" |
| 내가 불러야 움직인다 | `/skill:…`로 명시적으로 불렀을 때만 워크플로가 돈다. 말 속 단어를 보고 알아서 켜지지 않는다 | README "사용자가 명시적으로 선택해 실행" |
| 계획 중엔 코드를 못 고친다 | 인터뷰·계획 단계의 코드 수정·커밋 차단이 프롬프트가 아니라 **프로그램 코드로** 걸려 있다 | ralplan SKILL.md |
| 증거 일지 | 목표마다 무엇을 검증했는지 `ledger.jsonl`에 남는다. 나중에 사람이 다시 확인할 수 있다 | ultragoal SKILL.md |
| 폰으로 답하기 | 질문·승인 요청을 Telegram/Discord/Slack으로 받아 폰에서 답한다(기본은 꺼짐) | README "휴대폰으로 답하기" |
| 토큰 절약 | 긴 파일은 구조 요약으로 먼저 보여 주고, 긴 출력은 잘라서 따로 저장한다 | README "토큰을 덜 쓰기" |
| 한국어 친화 | 화면 언어 `ui.language ko`, 한국어 README·치트시트, 별칭 `가재씨` | README, 가이드 3장 |

### 장점 (왕초보 관점)

1. **배울 게 적다.** 기억할 흐름이 `deep-interview → ralplan → ultragoal` 하나다. 스킬이 4개뿐이라 "지금 뭐가 돌고 있지?"를 헷갈릴 일이 적다.
2. **안전하다.** 계획이 끝나고 승인하기 전에는 코드가 바뀌지 않는다. 계획이 막히면(`PLANNING-STUCK`) 자동 실행 설정이 켜져 있어도 실행하지 않는다.
3. **돈을 두 번 내지 않는다.** 이미 내는 월 구독으로 돌린다. 한 회사 구독이 바닥나도 다른 회사 모델로 역할을 나눌 수 있다.
4. **"다 했어요"를 믿을 수 있다.** 완료는 AI의 말이 아니라 목표별 검증 기록과 마지막 critic의 `OKAY`로 정해진다.
5. **끝없이 돌지 않는다.** 목표 하나당 다시 밀어붙이는 횟수가 `nudgeBudget`(기본 10)으로 정해져 있어 한도를 다 태우는 사고가 적다.
6. **자리를 비워도 된다.** `--tmux`로 창을 닫아도 돌고, 막히면 폰으로 알림이 온다.

### 단점과 주의할 점

1. **git이 사실상 필수다.** ultragoal은 변경 범위를 git diff로 계산한다. git을 안 쓰거나 커밋이 하나도 없는 폴더에서는 목표를 "완료"로 기록하지 못한다. 시작 전에 `git init`과 첫 커밋을 해 둔다.
2. **처음 설정이 길다.** 로그인, 역할별 모델, 자동 넘김(`gjc.ralplan.autoHandoff`), goal 모드, 알림을 하나씩 맞춰야 한다. 왕초보 가이드가 13장짜리인 이유다.
3. **실행 중에는 AI가 질문하지 않는다.** ultragoal이 도는 동안 질문(ask)이 막혀 있다. 애매한 건 "막힘 기록"으로 남기고 계속 간다. 그래서 인터뷰 단계에서 충분히 답해 둬야 한다.
4. **베타다.** README가 직접 "거친 부분이 있을 수 있다"고 쓴다. 중요한 작업은 결과를 직접 확인한다.
5. **문서의 버전 함정.** README 설치 예시가 `v0.15.3`에 고정돼 있다. 그대로 설치하면 최신 모델이 없어 오류가 날 수 있다(가이드 2장). 설치 후 `gjc update`.
6. **다른 AI CLI를 팀으로 부리는 기능은 번들에 없다.** 목표 안에서 `executor`를 여러 개 나눠 돌리는 병렬화는 있지만, OMC의 `omc team 2:codex`처럼 다른 회사 CLI를 tmux 창마다 띄우는 모드는 없다.

## 다른 하네스와의 비교

| 항목 | [[harness/claude-code\|claude-code]] | [[harness/codex\|codex]] | [[harness/deepseek\|deepseek]] | [[harness/pi\|pi]] | [[harness/oh-my-claudecode\|oh-my-claudecode]] | gajae-code (이 페이지) |
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

### 무엇을 고를까 (왕초보용)

| 이런 상황이면 | 추천 | 이유 |
| --- | --- | --- |
| Claude Code를 이미 매일 쓰고, 큰 일을 여러 에이전트로 나눠 오래 돌리고 싶다 | [[harness/oh-my-claudecode\|OMC]] | Claude Code 위에 얹기만 하면 되고, `team`·`autopilot`이 병렬·장시간 실행에 맞춰져 있다 |
| ChatGPT·Gemini·GLM 등 Claude 말고 다른 구독도 쓰고 싶다 | gjc | `/login`으로 여러 회사 구독을 연결하고 역할마다 섞을 수 있다 |
| 처음이라 무엇이 돌고 있는지 하나하나 보면서 배우고 싶다 | gjc | 흐름이 하나이고, 승인 전에는 코드가 바뀌지 않는다 |
| "말만 하면 알아서"가 제일 중요하다 | OMC | `autopilot: …`처럼 키워드만 말하면 끝까지 자동 진행한다 |
| git을 아직 모른다 | git부터 익히기 | gjc는 git 없이는 완료 기록이 안 된다. OMC는 git을 필수로 적지 않지만, ralph의 완료 자동 대조(`gitGrep`)와 git 안전 훅(`git-guardrails`)이 git을 전제로 한다 |

## 원문 경로

- ko: `raw/gajae-code/README.ko.md` (영어 README가 기준이라고 명시)
- en: `raw/gajae-code/README.md`
- 스킬: `raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/{deep-interview,ralplan,ultragoal,autoresearch}/SKILL.md`
- 역할: `raw/gajae-code/packages/coding-agent/src/prompts/agents/{executor,architect,planner,critic}.md`
- 계보: `raw/gajae-code/NOTICE.md`
- 왕초보 가이드(사용자 작성, gjc 0.19.3 기준): `raw/gjc-beginner-guide.html`
- 스냅숏: 커밋 `d4ecb353a21528c0df07fc36239ea648eecab357`(2026-11-03), 패키지 버전 0.19.3

**원문 간 차이**

- README(ko·en)의 설치 예시는 `v0.15.3`에 고정돼 있다. 왕초보 가이드는 이 버전으로 설치하면 Claude 5.5 모델이 없어 서비스 종료된 `claude-3-5-sonnet-20240620`으로 시작해 404 오류가 난다고 기록한다. 스냅숏의 실제 버전은 0.19.3이다.
- 왕초보 가이드는 역할을 DEFAULT 포함 5개로 설명한다. 번들 역할 에이전트 파일은 4개(`executor`, `architect`, `planner`, `critic`)이고, DEFAULT는 대화 상대(메인 세션)의 모델 설정이다.
