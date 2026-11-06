---
type: "concept"
title: "기능 목록(하네스 프리미티브)"
aliases:
  - "feature_list.json"
  - "기능 목록"
  - "feature list"
  - "패스 상태 게이팅"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/feature_list.json"
related:
  - "[[concepts/definition-of-done]]"
  - "[[concepts/one-feature-at-a-time]]"
  - "[[concepts/continuity-artifacts]]"
tags:
  - "type/concept"
  - "topic/feature-list"
updated: "2026-11-06"
---

# 기능 목록(하네스 프리미티브)

## 왕초보 설명

- **이게 뭐야?** 만들 기능을 한 줄씩 적은 파일이다. 줄마다 "무엇이 되면 되나 / 어떤 명령으로 확인하나 / 지금 상태(시작 안 함·하는 중·막힘·통과)"를 적는다.
- **왜 필요해?** AI가 "다 했어요"라고 할 때 무엇이 끝났는지 이 파일로 확인한다. 다음 대화의 AI도 이 파일만 보고 어디서 이어갈지 안다.
- **나는 뭘 하면 돼?** 프로젝트 루트에 `feature_list.json`을 두고, `AGENTS.md`에 "검사가 통과해야만 상태를 pass로 바꾼다"를 적는다.

## 정의

모든 기능을 `(동작 설명, 검증 명령, 현재 상태)` 삼중 구조로 담은 기계 판독 파일. 하네스의 스케줄러·검증기·핸드오프 보고기·진행 추적기가 이것을 읽으며, `passing`으로 가는 유일한 경로는 검증 명령 성공이다(패스 상태 게이팅).

## 왜 중요한가

- "완료"의 뜻을 에이전트 밖에 고정한다. 없으면 에이전트는 "꽤 완성돼 보인다"를 기준으로 쓴다.
- 비구조 메모("장바구니 거의 완료")는 다음 세션이 해석할 수 없다. 구조화 목록은 상태를 즉시 복원하게 한다.
- 아직 통과 못 한 항목 수가 하네스의 역압력이 되어 완료를 끌어낸다.

## 등장하는 곳

- [[lectures/lecture-08]]: 프리미티브, 삼중 구조, 4상태 기계, 패스 상태 게이팅, 세분화 기준을 제시한다.
- [[projects/project-03]]: `feature_list.json` 11개 항목을 "한 번에 하나, 증거 없이 pass 금지"로 운영한다. 앞 실습 [[projects/project-01]]·[[projects/project-02]]도 같은 파일을 쓴다.
- [[harness/oh-my-claudecode]]의 `prd.json`, [[harness/gajae-code]]의 `goals.json`이 하네스 수준의 같은 구조다.

## 관련 개념

- [[concepts/definition-of-done]]: 각 항목의 "검증 명령"이 완료 기준이다.
- [[concepts/one-feature-at-a-time]]: 목록에서 한 번에 하나만 `active`.
- [[concepts/continuity-artifacts]]: 목록 자체가 가장 중요한 인계 파일이다.

## 상충·열린 질문

- 강의와 코스 실습의 형식이 다르다. [[lectures/lecture-08]]은 항목마다 `verification`(검증 명령) 필드와 `not_started`/`active`/`blocked`/`passing` 네 상태를 요구한다. [[projects/project-03]]의 실제 `feature_list.json`은 `id, name, description, status, evidence, testedAt` 필드에 상태 `not-started`/`pass`만 쓰고, 검증 명령 필드가 없다. 증거(`evidence`)는 명령이 아니라 서술문이다.
- 강의는 "상태를 직접 수정하지 말고 검증 스크립트가 바꾼다"고 하지만, 코스 실습의 정책은 에이전트가 `feature_list.json`을 직접 `"pass"`로 고친다.
