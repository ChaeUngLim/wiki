---
type: "concept"
title: "품질 문서"
aliases:
  - "Quality document"
  - "quality-document.md"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/projects/project-06/starter/quality-document.md"
  - "raw/learn-harness-engineering/projects/project-06/solution/quality-document.md"
related:
  - "[[concepts/clean-state]]"
  - "[[concepts/evaluator-rubric]]"
  - "[[concepts/continuity-artifacts]]"
tags:
  - "type/concept"
  - "topic/clean-state"
  - "topic/observability"
updated: "2026-11-06"
---

# 품질 문서

## 왕초보 설명

- **이게 뭐야?** 프로젝트의 부분(모듈)마다 **성적표**를 매겨 저장소에 두는 문서다. "결제 모듈: C, 테스트 불안정" 같은 식이다.
- **비유:** 건강검진 결과표. 매년 같은 항목을 재니까 어디가 나빠지는지 보인다.
- **나는 뭘 하면 돼?** `quality-document.md`를 만들어 모듈별 등급과 이유를 적고, 주기적 정리 때 갱신하게 한다. 새 세션에는 "가장 낮은 등급부터 고쳐라"라고 시킨다. 단, 적힌 근거는 가끔 직접 실행해 확인한다.

## 정의

각 모듈(또는 차원)의 품질 등급과 근거를 지속적으로 기록하는 **살아 있는 산출물**. 일회성 평가가 아니라 코드베이스가 강해지는지 약해지는지 보여 주는 추적기다.

## 왜 중요한가

- 저하되고 있다는 걸 알아야 고칠 수 있다.
- 새 세션이 우선순위를 즉시 정할 수 있다(가장 낮은 등급 먼저).
- 주기적 정리 루프의 입력·출력이 된다.

## 등장하는 곳

- [[lectures/lecture-12]]: 모듈별 등급(A~) + 검증 통과 / 에이전트 이해 가능성 / 테스트 안정성 / 아키텍처 경계 / 코드 컨벤션 항목 예시. 주간 정리에서 갱신.
- [[projects/project-06]]: starter(종합 D+, 치명적 격차·행동 항목 목록)와 solution(종합 A, 품질 증거와 "검증 대상" 목록) 두 버전. 도구 목록에 "품질 문서 템플릿"이 있다.

## 관련 개념

- [[concepts/clean-state]]: 주기적 정리 모드에서 품질 문서를 갱신한다.
- [[concepts/evaluator-rubric]]: 루브릭은 한 작업 결과를, 품질 문서는 코드베이스 전체를 채점한다.
- [[concepts/continuity-artifacts]]: 다음 세션이 읽는 상태 파일의 하나.

## 상충·열린 질문

- 등급은 누가 매기고 근거를 어떻게 검증하나? [[projects/project-06]] solution 품질 문서는 "체크리스트 30개 통과", "`benchmark.sh` 모든 작업 완료"를 근거로 들지만, 체크리스트 항목은 41개이고 `benchmark.sh`는 직접 실행했을 때 첫 작업에서 실패했다. 품질 문서도 [[concepts/maker-checker-separation|검사자 분리]] 없이는 자기 평가가 된다.
