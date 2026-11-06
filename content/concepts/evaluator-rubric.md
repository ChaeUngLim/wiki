---
type: "concept"
title: "평가자 루브릭"
aliases:
  - "Evaluator rubric"
  - "채점 루브릭"
  - "evaluator-rubric.md"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/single-role/evaluator-rubric.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/plan-gen-eval/evaluator-rubric.md"
related:
  - "[[concepts/sprint-contract]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/concept"
  - "topic/verification"
  - "topic/observability"
updated: "2026-11-06"
---

# 평가자 루브릭

## 왕초보 설명

- **이게 뭐야?** "좋은 것 같다 / 별로다" 대신 **항목별 점수표**로 결과를 채점하는 것이다(루브릭 = 채점 기준표).
- **비유:** 수행평가 채점표. "내용 5점, 발표 3점, 시간 엄수 2점"이 있으면 누가 채점해도 비슷하게 나온다.
- **나는 뭘 하면 돼?** 검토 AI에게 "정확성·구조 준수·테스트·접근성 같은 항목마다 점수와 근거를 적어라"라고 시키고, 낮은 항목을 고친 뒤 다시 채점받는다.

## 정의

품질 평가를 주관적 판단에서 **차원별·증거 기반 채점**으로 바꾸는 표. 서로 다른 평가자가 같은 출력에 비슷한 점수를 주도록 한다.

## 왜 중요한가

- 루브릭이 없으면 평가자는 암묵적 가정에 기대고, 같은 결과가 평가자마다 다르게 평가된다.
- 모호한 거부("느낌이 안 좋다")가 구체적이고 고칠 수 있는 결함 목록으로 바뀐다.
- 수정 전후 점수를 남기면 개선이 기록된다.

## 등장하는 곳

- [[lectures/lecture-11]]: 코드 정확성 / 아키텍처 준수 / 테스트 커버리지 × A~D 예시.
- [[projects/project-05]]: 세 변형마다 `evaluator-rubric.md`(1~5점 척도, 항목별 점수와 메모, 종합 점수). gen-eval과 plan-gen-eval은 수정 회차별 점수(2.8→3.1→3.3, 4.2→4.6→4.8→4.9)를 남긴다.
- [[projects/project-06]]: solution `evaluator-rubric.md` 종합 5.0/5. 도구 목록에 "평가자 루브릭"이 있다.

## 관련 개념

- [[concepts/quality-document]]: 루브릭은 한 작업을, 품질 문서는 코드베이스 전체를 채점한다.

- [[concepts/sprint-contract]]: 계약의 수용 기준이 루브릭 채점의 기준이 된다.
- [[concepts/maker-checker-separation]]: 루브릭은 검사자가 쓰는 도구다.

## 상충·열린 질문

- lecture-11은 루브릭이 "서로 다른 평가자가 비슷한 점수"를 주게 한다고 하지만, project-05에서는 변형마다 평가자가 다르고(single-role은 자기 평가) 같은 결과를 여러 평가자가 채점한 기록은 없다. 재현성 주장은 이 저장소에서 검증되지 않는다.
