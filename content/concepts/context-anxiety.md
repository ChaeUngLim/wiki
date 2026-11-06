---
type: "concept"
title: "컨텍스트 불안(context anxiety)"
aliases:
  - "컨텍스트 불안"
  - "조기 수렴"
  - "context anxiety"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md"
related:
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/verification-commands]]"
tags:
  - "type/concept"
  - "topic/context-continuity"
  - "topic/verification"
updated: "2026-11-06"
---

# 컨텍스트 불안(context anxiety)

## 왕초보 설명

- **이게 뭐야?** AI가 "대화창이 거의 찼다"고 느끼면 **서둘러 대충 끝내 버리는** 버릇이다. 검사를 건너뛰고 쉬운 방법을 고른다.
- **비유:** 시험 종료 5분 전, 남은 객관식을 아무거나 찍는 학생.
- **나는 뭘 하면 돼?** 대화가 길어져 AI가 갑자기 "완료했습니다"를 서두르면 의심한다. 긴 작업은 중간에 인계 파일을 쓰게 하고 새 대화로 넘긴다. 완료는 검증 명령 결과로만 인정한다.

## 정의

Anthropic이 장기 실행 에이전트에서 관찰한 현상. 에이전트가 인식된 컨텍스트 한계에 다가가면 **조기 수렴**한다. 작업을 서둘러 마무리하고, 검증 단계를 건너뛰고, 최선 대신 단순한 해결책을 택한다.

## 왜 중요한가

- "완료"라는 말과 실제 정확성의 차이(검증 격차)를 키운다.
- 대처법이 모델마다 다르다. Sonnet 4.5에선 불안이 심해 컴팩션(앞 대화 요약)만으론 부족했고 컨텍스트 리셋(새 세션 + 인계 파일)이 필요했다. Opus 4.5에선 크게 줄어 컴팩션으로 관리할 수 있었다. 하네스는 대상 모델을 알고 설계해야 한다.
- 컴팩션은 불안을 없애지 못한다. 에이전트는 컨텍스트가 한때 컸다는 걸 알고 여전히 서두른다.

## 등장하는 곳

- [[lectures/lecture-01]]: 다섯 방어 레이어 중 검증 피드백 문제의 한 예로 처음 소개한다.
- [[lectures/lecture-05]]: Anthropic 2026년 3월 연구를 인용해 모델별 차이와 컴팩션 vs 리셋을 다룬다.

## 관련 개념

- [[concepts/continuity-artifacts]]: 리셋으로 불안을 피하는 데 필요한 인계 파일.
- [[concepts/verification-commands]]: 서둘러 끝내려 해도 통과해야 하는 기계 판정.
- [[concepts/ralph-loop]]: 조기 "완료" 선언을 하네스가 막고 계속 돌리는 구조.

## 상충·열린 질문

없음.
