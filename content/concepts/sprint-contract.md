---
type: "concept"
title: "스프린트 계약"
aliases:
  - "Sprint contract"
  - "sprint-contract.md"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/plan-gen-eval/sprint-contract.md"
related:
  - "[[concepts/evaluator-rubric]]"
  - "[[concepts/definition-of-done]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/concept"
  - "topic/scope-control"
  - "topic/observability"
updated: "2026-11-06"
---

# 스프린트 계약

## 왕초보 설명

- **이게 뭐야?** 코딩을 시작하기 **전에** "이번에 바꿀 것 / 안 바꿀 것 / 통과 기준"을 한 장에 적어 만드는 쪽과 검사하는 쪽이 합의하는 문서다.
- **비유:** 인테리어 공사 전 견적서. "욕실 타일 교체, 배관은 안 건드림, 물 새지 않으면 완료"를 미리 써 두면 끝나고 싸울 일이 줄어든다.
- **나는 뭘 하면 돼?** 큰 작업 앞에 `sprint-contract.md`를 쓰게 하고(범위 / 검증 기준 / 제외 사항), 검토할 때 그 문서 기준으로 하라고 지시한다.

## 정의

작업 시작 전에 생성자와 평가자가 협상하는 단기 합의. 작업 범위, 항목별 검증 기준, 제외 사항을 명시한다. 생성자와 평가자는 같은 에이전트의 다른 호출이어도 된다.

## 왜 중요한가

- **사전 정렬**: "생성자가 만든 것을 평가자가 예측 가능한 이유로 바로 거부하는" 상황을 막는다.
- 평가가 무엇을 기준으로 할지가 에이전트 머리 밖, 파일에 남는다(프로세스 관측 가능성).
- 제외 사항을 적으면 범위가 새지 않는다.

## 등장하는 곳

- [[lectures/lecture-11]]: 프로세스 관측 가능성의 핵심 도구로 정의. 예시 "다크 모드 지원"(범위 / 검증 기준 / 제외 사항).
- [[projects/project-05]]: `solution/plan-gen-eval/sprint-contract.md`가 플래너 산출물. 범위 안·밖, 역할 표, 번호 붙은 수용 기준을 둔다. 이 변형의 rubric 점수가 가장 높다(4.9/5).

## 관련 개념

- [[concepts/evaluator-rubric]]: 계약이 "무엇을"을 정하면 루브릭은 "얼마나 잘"을 채점한다.
- [[concepts/definition-of-done]]: 계약의 검증 기준은 작업 단위의 완료 정의다.
- [[concepts/maker-checker-separation]]: 계약은 만드는 쪽과 검사하는 쪽이 같은 기준을 보게 한다.

## 상충·열린 질문

- lecture-11은 계약을 생성자와 평가자가 **협상**한다고 하지만, project-05의 계약은 플래너가 쓰고 생성자·평가자가 따른다. 협상 단계의 기록은 없다.
