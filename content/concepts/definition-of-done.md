---
type: "concept"
title: "완료 기준(Definition of Done)"
aliases:
  - "완료 기준"
  - "Definition of Done"
  - "DoD"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-09-why-agents-declare-victory-too-early/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-09-why-agents-declare-victory-too-early/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-02-agent-readable-workspace/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-02-agent-readable-workspace/index.md"
  - "raw/learn-harness-engineering/projects/project-02/solution/AGENTS.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
related:
  - "[[concepts/verification-commands]]"
  - "[[concepts/ralph-loop]]"
tags:
  - "type/concept"
  - "topic/verification"
  - "topic/scope-control"
updated: "2026-11-06"
---

# 완료 기준(Definition of Done)

## 왕초보 설명

- **이게 뭐야?** "어떻게 되면 다 한 거야?"를 **미리, 확인 가능한 문장으로** 적어 두는 것이다. 예: "테스트 통과", "앱 창이 뜬다", "목록에 pass 표시".
- **비유:** 숙제에 채점 기준표를 같이 주는 것. 기준표가 없으면 학생(AI)은 자기 마음대로 "다 했어요"라고 한다.
- **나는 뭘 하면 돼?** AI에게 일을 줄 때 "검색 기능 추가해" 대신 끝 상태를 3~5줄로 적는다. 그중 적어도 하나는 명령으로 확인할 수 있게(`npm run check` 통과 등) 쓴다.

## 정의

작업이 끝났다고 인정하는 **기계 검증 가능한 조건들의 집합**. 테스트 통과, 린트 클린, 타입 검사 통과 같은 명령 결과와, 기능 목록 상태·문서 갱신 같은 확인 가능한 산출물로 이루어진다. 명시하지 않으면 에이전트가 자기 기준을 만들어낸다.

## 왜 중요한가

- [[lectures/lecture-01]]의 "검증 격차"(완료 자신감 vs 실제 정확성)는 가장 흔한 실패 유형이고, 완료 기준이 그 격차를 재는 잣대다.
- 모호한 지시("검색 기능 추가")는 작업 명세 레이어의 실패를 부른다. 완료 기준을 쓰면 범위도 함께 정해진다.

## 등장하는 곳

- [[lectures/lecture-01]]: 핵심 용어로 정의하고, 검색 API 예시(엔드포인트, 페이지네이션 20개, 하이라이트 스니펫, pytest 통과, `mypy --strict` 통과)를 든다.
- [[projects/project-02]]: solution `AGENTS.md`가 기능 완료를 다섯 조건으로 정한다. ① `npm run check` 통과 ② 앱 창이 보임 ③ `feature_list.json`에 `"pass"`와 증거 ④ Electron 계층 경계 준수 ⑤ `docs/` 갱신.

- [[projects/project-03]]: "검증 증거 없이는 pass 금지" 규칙과 `clean-state-checklist.md`의 범위 통제 검증(기능별 증거, fail·not-started 없음)으로 완료 기준을 기능 목록 단위로 강제한다.

- [[lectures/lecture-07]]: "완료 증거"를 실행 가능한 명령(예: curl이 201 반환)으로 요구하고, 증거 없는 작업은 진행 중으로 남긴다.

- [[lectures/lecture-09]]: 완료 판단을 외부화하는 3단계 종료 검사(정적 분석 → 런타임 동작 → 시스템 수준)와 "앞 단계 실패 시 다음으로 안 감" 규칙.

- [[lectures/lecture-12]]: "세션 완료 = 작업 검증 통과 AND 클린 상태 검사 통과". 클린 상태를 완료 정의에 넣는다.

## 관련 개념

- [[concepts/clean-state]]: 세션 단위 완료 조건.

- [[concepts/feature-list]]: 기능마다 완료 기준(검증 명령)을 담는 파일.
- [[concepts/verification-commands]]: 완료 기준 중 "명령으로 확인" 부분을 구체 명령으로 적는 것.
- [[concepts/ralph-loop]]: 완료 기준이 채워질 때까지 하네스가 계속 돌리는 구조. OMC의 `prd.json` 이야기별 완료 기준, gjc의 목표별 증거가 이에 해당한다.

## 상충·열린 질문

없음.
