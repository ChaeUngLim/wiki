---
type: "concept"
title: "컴포넌트 제거 실험과 하네스 단순화"
aliases:
  - "Component ablation"
  - "하네스 단순화"
  - "Harness simplification"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/map-not-manual]]"
tags:
  - "type/concept"
  - "topic/harness-model"
updated: "2026-11-06"
---

# 컴포넌트 제거 실험과 하네스 단순화

## 왕초보 설명

- **이게 뭐야?** 하네스 규칙이나 장치를 **하나씩 꺼 보고** 결과가 나빠지는지 재는 방법이다. 안 나빠지면 그 장치는 지워도 된다.
- **비유:** 약을 여러 개 먹고 있을 때 의사와 상의해 하나씩 끊어 보는 것. 끊어도 괜찮은 약은 원래 필요 없었던 것이다.
- **왜 필요해?** 규칙은 AI가 못 하는 걸 메우려고 생기는데, AI가 똑똑해지면 그 규칙이 짐이 된다.
- **나는 뭘 하면 돼?** 한 달에 한 번, 같은 작업 묶음을 정해 두고 규칙 하나를 빼서 돌려 본다. 결과가 그대로면 지우고, 나빠지면 되돌리거나 더 가벼운 규칙으로 바꾼다.

## 정의

모델을 고정한 채 하네스 구성 요소(하위 시스템, 규칙, 도구)를 하나씩 비활성화하고 같은 벤치마크 작업의 결과 하락폭으로 그 구성 요소의 한계 기여도를 재는 실험. 이를 정기적으로 돌려 기여가 없는 구성 요소를 제거하는 운영이 하네스 단순화다.

## 왜 중요한가

- 모든 하네스 구성 요소는 "모델이 스스로 안정적으로 못 한다"는 가정 위에 있고, 모델이 좋아지면 가정이 낡는다(Anthropic).
- 쓸모없는 구성 요소는 컨텍스트와 시간만 쓴다([[concepts/map-not-manual]]의 "거대 지시 파일" 문제와 같은 방향).
- 측정 없이 지우면 필요한 것까지 지울 수 있다.

## 등장하는 곳

- [[lectures/lecture-02]]: 모델 고정, 하위 시스템 하나씩 제거, 하락폭 해석(중복 / 약한 설계 / 해당 작업에서 덜 쓰임). Anthropic은 모델이 강해질수록 일부 컴포넌트는 덜 중요해지지만 새 컴포넌트가 늘 생긴다고 보고했다.
- [[lectures/lecture-12]]: "매달 하나를 골라 일시 비활성화 → 벤치마크 → 저하 없으면 영구 제거, 저하되면 복원 또는 더 가벼운 대안으로 교체". 연습 문제 3이 이 실습이다.

- [[projects/project-06]]: 캡스톤의 마지막 단계가 "하네스 절제 실험"(구성 요소를 하나씩 제거해 무엇이 중요한지 확인)이다. 결과 파일은 저장소에 없다.

## 관련 개념

- [[concepts/harness-subsystems]]: 제거 실험의 단위가 다섯 하위 시스템이다.
- [[concepts/map-not-manual]]: 지시를 줄이는 다른 경로.

## 상충·열린 질문

- 두 강의 모두 "결과가 저하되지 않으면"의 판정 기준(어떤 지표, 몇 번 실행, 허용 오차)을 정하지 않는다.
