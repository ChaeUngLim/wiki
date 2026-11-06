---
type: "lecture"
title: "13강. 수동 프롬프팅에서 자율 루프로"
aliases:
  - "수동 프롬프팅에서 자율 루프로"
  - "From Manual Prompting to Autonomous Loops"
  - "루프 엔지니어링"
  - "Loop Engineering"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/autoresearch/SKILL.md"
related:
  - "[[concepts/loop-engineering]]"
  - "[[concepts/ralph-loop]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/continuity-artifacts]]"
tags:
  - "type/lecture"
  - "topic/loop-engineering"
updated: "2026-11-06"
---

# 13강. 수동 프롬프팅에서 자율 루프로

## 왕초보 설명

- **이게 뭐야?** 지금까지는 내가 키보드 앞에서 AI에게 "이거 해, 다음엔 저거 해"를 하나씩 시켰다. 이 강의는 그 "다음 지시"를 **시스템이 대신 하게** 만드는 법(루프 엔지니어링)이다.
- **비유:** 세탁기. 빨래 넣고 "표준 코스"만 누르면 헹굼·탈수는 기계가 알아서 하고 끝나면 삐 울린다. 나는 코스(목표)와 끝 조건만 정한다.
- **가장 쉬운 시작:** `/goal "모든 테스트 통과, 린트 경고 0"`처럼 **목표 + 확인 방법 + 멈출 조건**을 한 번에 준다. 끝났는지는 AI 자신이 아니라 테스트나 다른 AI가 판정한다.
- **나는 뭘 하면 돼?** 일주일에 두 번 이상 손으로 하는 일 하나를 골라 목표·확인·멈춤 조건을 적고 한 번 돌려 본다. 진행 상황은 마크다운 파일 하나에 적게 한다. 루프가 만든 코드는 **반드시 읽는다**(안 읽으면 내 프로젝트를 내가 모르게 된다).

## 핵심 요약

- 1~12강은 "사람이 매번 시작 버튼을 누른다"는 가정 위에 있었다. 루프 엔지니어링은 그 버튼을 시스템에 넘긴다. **하네스를 대체하지 않고 그 위에 한 층을 쌓는다**: 하네스는 한 번의 실행을 믿을 만하게, 루프는 지속 실행을 자율적으로 만든다([[concepts/loop-engineering]]).
- **`/goal`이 가장 단순한 루프**: 목표 + 검증 방법 + 중단 조건. 완료 판단을 작업한 에이전트가 아니라 독립 심판(다른 모델·스크립트·테스트)이 한다.
- **여섯 프리미티브**(Addy Osmani): 자동화(심장 박동) / 워크트리(격리) / 스킬(프로젝트 지식) / 커넥터(MCP로 실제 도구) / 서브 에이전트(메이커·체커 분리) / 외부 상태(메모리, 나머지 모두의 척추).
- **생성자/평가자 분리가 루프의 가장 중요한 신뢰성 장치**다. "모델은 자기 출력의 최고 변호사다."([[concepts/maker-checker-separation]])
- 루프는 생성을 거의 공짜로 만들고 **판단을 희소 자원**으로 남긴다.

## 핵심 개념

**루프 유형** (트리거 × 중단 방식)

| 유형 | 트리거 | 중단 조건 | Claude Code | Codex |
| --- | --- | --- | --- | --- |
| 턴 기반 | 매 프롬프트 수동 | 에이전트 판단 또는 사용자 중단 | 일반 채팅 | 일반 채팅 |
| 목표 기반 | 목표 제시 | 독립 평가자 확인 또는 최대 턴 | `/goal` | `/goal`(수동 활성화 필요) |
| 시간 기반 | N분/시간마다 | 수동 중단 또는 작업 후 종료 | `/loop` | 스레드 자동화 |
| 이벤트 기반 | PR·CI 실패·새 이슈 | 처리 후 또는 재시도 한도 | Routines(API / GitHub Webhook) | 독립 실행형 자동화 + 플러그인 |

**`/goal` vs `/loop`**: 한 문장 테스트 "이 일에 끝이 있는가?" 끝이 있으면 `/goal`(마라톤), 계속 지켜봐야 하면 `/loop`(알람 시계). `/loop 10m "계속 결제 시스템 구현해"`는 틀렸다. `/loop`는 매번 같은 지시를 독립 실행하고 지난번 진행을 기억하지 못한다.

**루프 내부 vs 외부**: 내부(1~12강)는 사람이 엔진이다. 외부에서는 시스템이 트리거 → 작업 발견 → 서브 에이전트 할당 → 검증 → 상태 저장을 돌리고, 사람은 ① 시작 전 목표·중단 조건 정의 ② 결과 검토 ③ 시스템이 길을 잃을 때 규칙 조정만 한다.

**Karpathy autoresearch** (강의가 꼽은 모범 루프)

| 파일 | 누가 편집 | 역할 |
| --- | --- | --- |
| `prepare.py` | 아무도(읽기 전용) | 데이터·토크나이저·평가 하네스 |
| `train.py`(~630줄) | AI 에이전트 | 모델·옵티마이저·훈련 루프 |
| `program.md` | 사람 | 자연어 연구 방법론(목표 `val_bpb`, 제약, 탐색 방향, 평가 규칙, "멈추지 마라") |

9단계 **래칫**: 방향 읽기 → 상태 조사 → 가설 → `train.py` 수정 → `git commit` → 고정 5분 훈련 → `val_bpb` 개선? → 예: 유지·`results.tsv` 기록 / 아니오: `git reset HEAD~1`·실패 기록 → 반복. 강의에 따르면 README 추정 시간당 약 12회, 8시간에 약 100회이고, 1라운드는 nanochat GPT-2 수준 훈련 시간을 2.02h → 1.80h(약 10.9%) 줄였다. 교훈: **에이전트에게 작업이 아니라 방법론을 줘라.**

**네 가지 조용한 비용** `후보:loop-silent-costs`

| 비용 | 뜻 | 대응 |
| --- | --- | --- |
| 검증 부채 | "괜찮아 보여"가 쌓임 | 중단 조건은 기계로 검증 가능해야 함 |
| 이해 부패 | 내 코드베이스 이해가 현실과 멀어짐 | 빠른 루프는 빠른 읽기를 요구 |
| 인지 항복 | 생각을 피하는 데 루프를 씀 | 루프는 생각을 증폭하는 데 |
| 토큰 폭발 | 전체 히스토리 재처리 시 누적 입력이 대략 2차로 증가 | 컨텍스트 압축을 첫 루프부터 |

**첫 루프 5단계**: 반복 작업 하나 → 목표·중단 조건 작성 → 메이커·체커 분리 → 마크다운 메모리 → 타이머(`/loop` 또는 cron, 하루 1회로 시작해 일주일 관찰).

**성숙도 사다리**: 1 목표 실행자(`/goal`) → 2 예약된 단일 작업 → 3 다중 에이전트(메이커·체커, 워크트리 포크) → 4 자기 공급 루프(외부 상태에서 다음 작업 발견) → 5 함대 오케스트레이션. 강의는 "대부분의 팀은 2와 3 사이"라고 한다.

**강의가 드는 외부 데이터 포인트**(이 위키는 검증하지 않음): Claude Code와 Codex가 2026년 초 `/goal`을 독립 출시, Addy Osmani가 2026-06-07 "루프 엔지니어링" 명명, Boris Cherny의 30일 259 PR(497 커밋, +40,000/-38,000줄, Claude Code + Opus 4.5), OpenAI 소규모 팀의 5개월 약 100만 줄·약 1,500 PR.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-05]]·[[concepts/continuity-artifacts]]: 외부 상태 = 루프의 메모리.
- [[lectures/lecture-08]]·[[concepts/feature-list]]: 기능 목록은 자기 공급 루프가 다음 작업을 찾는 데이터 소스.
- [[lectures/lecture-09]]: 조기 완료 선언 → 루프에서는 중단 조건을 독립 심판에게.
- [[lectures/lecture-11]]: 루프가 빠를수록 관측 가능성이 더 필요.
- [[lectures/lecture-12]]·[[concepts/clean-state]]: 각 라운드가 클린 상태를 남겨야 다음 라운드가 자동 시작된다.
- 실습: [[projects/project-07]]. 다음: [[lectures/lecture-14]](루프는 그래프의 한 노드).
- 하네스 실례: [[concepts/ralph-loop]]가 이 강의의 "목표 기반 루프"를 하네스 수준에서 구현한 것이다. [[harness/oh-my-claudecode]]의 `ralph`(검증자 판정까지 반복), [[harness/gajae-code]]의 `ultragoal`(목표별 체크포인트 + 최종 `critic` `OKAY`)과 goal 모드. gjc에도 `autoresearch`라는 스킬이 있지만 Karpathy의 autoresearch와는 별개로, 연구 결과와 판정(verdict)을 내는 워크플로다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-13-loop-engineering/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md`
- 코드 예제: `raw/learn-harness-engineering/docs/{ko,en}/lectures/lecture-13-loop-engineering/code/`(`goal-template.md`, `loop-state-template.md`, `maker-prompt.md`, `checker-prompt.md`). `code/index.md`가 약속한 루프 제어 스크립트 예시는 없다.
- 긴 소스(ko 약 46KB, en 약 38KB). 절 구성은 ko·en이 같다.
- 표기 차이: ko 제목 "13강. 수동 프롬프팅에서 자율 루프로", project-07의 링크 제목은 "L13. 왜 에이전트에게 프롬프트하는 것을 그만둬야 하는가"(en "Why You Need to Stop Prompting Your Agent").
- 강의 도입부의 1~12강 요약은 "깔끔한 인계(9, 12강)", "관찰 가능(10-11강)"으로 묶어, 각 강의의 실제 주제(9 조기 완료 선언, 10 E2E 테스트)와 다르다.
