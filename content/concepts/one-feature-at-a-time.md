---
type: "concept"
title: "한 번에 기능 하나(WIP=1)"
aliases:
  - "WIP=1"
  - "한 번에 하나씩"
  - "one feature at a time"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/AGENTS.md"
related:
  - "[[concepts/definition-of-done]]"
  - "[[concepts/ralph-loop]]"
tags:
  - "type/concept"
  - "topic/scope-control"
updated: "2026-11-06"
---

# 한 번에 기능 하나(WIP=1)

## 왕초보 설명

- **이게 뭐야?** AI가 동시에 진행하는 일을 **딱 하나**로 묶는 규칙이다. 하나를 끝내고(검사 통과 + 기록 + 커밋) 나서야 다음을 시작한다.
- **비유:** 뷔페에서 접시를 쌓지 말고 한 접시씩. 다 먹고 다음 접시.
- **왜 필요해?** 여러 개를 동시에 벌이면 하나도 안 끝나고, 반쯤 된 코드끼리 엉켜 다음 사람(다음 대화)이 길을 잃는다.
- **나는 뭘 하면 돼?** `AGENTS.md`에 "기능 목록에서 하나만 골라 → 그것만 고치고 → 검사 → pass와 증거 기록 → 커밋 → 그다음 기능"을 적는다.

## 정의

진행 중(active) 작업 수를 1로 제한하는 범위 통제 규칙. 칸반의 WIP 제한을 에이전트에 적용한 것으로, 현재 작업이 검증된 완료에 이르기 전에는 새 작업을 시작하지 않으며 현재 작업 범위 밖 코드를 건드리지 않는다.

## 왜 중요한가

- 주의력(컨텍스트)은 유한하다. 작업 k개에 나누면 작업당 C/k가 되어 임계값 아래로 떨어진다.
- 과욕과 미완성의 악순환을 끊는다. 강의 사례에서 같은 8개 기능 프로젝트의 완료율이 37.5% → 87.5%가 됐다(측정 출처 표기 없는 사례).
- 세션이 끊겨도 "진행 중"이 하나뿐이라 이어받기 쉽다.

## 등장하는 곳

- [[lectures/lecture-07]]: WIP=1, 완료 증거, 범위 표면(DAG), 검증 완료율로 이론을 세운다.
- [[projects/project-03]]: solution `AGENTS.md`가 "Project 03의 핵심 규율"로 6단계 정책과 기능 의존 그래프를 둔다. 범위 밖 수정을 "버그와 회귀의 가장 흔한 원인"이라 명시한다.
- [[harness/oh-my-claudecode]]의 `ralph`(이야기 하나씩), [[harness/gajae-code]]의 `ultragoal`(목표 하나씩 checkpoint)은 이 규칙을 루프 구조로 강제한다.

## 관련 개념

- [[concepts/feature-list]]: 하나를 고르는 목록이자 상태를 기록하는 파일.
- [[concepts/definition-of-done]]: "하나를 끝냈다"의 판정 기준.
- [[concepts/ralph-loop]]: 한 작업이 끝날 때까지 하네스가 계속 돌리는 구조.
- [[concepts/continuity-artifacts]]: 작업 상태를 파일로 외부화하는 방법.

## 상충·열린 질문

없음.
