---
type: "lecture"
title: "강의 08. 기능 목록으로 에이전트의 행동을 제약하십시오"
aliases:
  - "기능 목록으로 에이전트의 행동을 제약하십시오"
  - "Why Feature Lists Are Harness Primitives"
  - "패스 상태 게이팅"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md"
related:
  - "[[lectures/lecture-07]]"
  - "[[concepts/feature-list]]"
  - "[[concepts/definition-of-done]]"
tags:
  - "type/lecture"
  - "topic/feature-list"
  - "topic/scope-control"
updated: "2026-11-06"
---

# 강의 08. 기능 목록으로 에이전트의 행동을 제약하십시오

## 왕초보 설명

- **이게 뭐야?** "할 일 목록"을 사람용 메모가 아니라 **AI 작업 시스템의 뼈대**로 쓰라는 이야기다. 목록의 각 줄에 "무엇이 되면 되는지 / 어떤 명령으로 확인하는지 / 지금 상태"를 적는다.
- **비유:** 친구에게 "과일 좀 사와"라고만 하면 레몬을 사 온다. "사과 3개, 빨간 것, 영수증 사진"이라고 적어 줘야 한다.
- **왜 중요해?** "장바구니 거의 완료"같은 메모로는 다음 AI가 무엇이 끝났는지 알 수 없다. 이미 끝난 기능을 다시 만들기도 한다.
- **나는 뭘 하면 돼?** `feature_list.json`(또는 Markdown 표)을 만들고, 기능마다 `동작 설명 + 검증 명령 + 상태`를 적는다. "검증 명령이 통과해야만 passing으로 바꾼다"를 `AGENTS.md`에 적는다. 기능 크기는 "대화 한 번에 끝낼 수 있는 만큼"으로 자른다.

## 핵심 요약

- 에이전트는 "완료"가 뭔지 모른다. 알려주지 않으면 자기 기준("문법 오류 없고 꽤 완성돼 보인다")을 쓴다.
- 기능 목록은 **하네스 프리미티브**다. 스케줄러, 검증기, 핸드오프 보고기, 진행 추적기가 모두 이것을 읽는다. 문서는 무시될 수 있지만 프리미티브는 우회될 수 없다.
- 각 항목은 **삼중 구조**: 동작 설명, 검증 명령, 현재 상태. 하나라도 빠지면 다리 하나 없는 세발 의자.
- 상태 전환은 **하네스가 통제**한다. 에이전트는 `passing`으로 직접 못 바꾸고 검증 요청만 낸다(패스 상태 게이팅).
- 세분화는 "한 세션 안에 완료 가능". "장바구니 구현"은 너무 넓고, "Cart 모델에 name 필드"는 너무 좁고, "사용자가 장바구니에 항목을 추가할 수 있다"가 적절하다.

## 핵심 개념

([[concepts/feature-list]])

- **상태 기계**: `not_started` → `active` → `passing`, 그리고 `blocked`. `passing`은 되돌릴 수 없다(합격한 시험 점수를 소급해 못 바꾸듯).
- **패스 상태 게이팅(Pass-state gating)**: `active` → `passing`의 유일한 경로는 검증 명령 성공.
- **단일 진실 원천**: "무엇을 해야 하는가"는 모두 이 목록에서 파생된다. 목록과 대화 기록이 모순되면 안 된다([[concepts/repo-system-of-record]]).
- **역압력(Back-pressure)**: 아직 통과 못 한 기능 수. 0이면 프로젝트 완료.

**기능 목록을 읽는 네 구성 요소**

| 구성 요소 | 하는 일 | 비유 |
| --- | --- | --- |
| 스케줄러 | 다음 `not_started` 기능 선택 | 생산 계획 |
| 검증기 | 검증 명령 실행, 전환 허용 여부 결정 | 품질 검사 |
| 핸드오프 보고기 | 목록에서 세션 인계 요약 자동 생성 | 교대 근무 보고서 |
| 진행 추적기 | 상태 분포 집계 | 대시보드 |

**항목 예시와 규칙**

```json
{
  "id": "F03",
  "behavior": "POST /cart/items with {product_id, quantity} returns 201",
  "verification": "curl -X POST http://localhost:3000/api/cart/items ... | jq .status == 201",
  "state": "passing",
  "evidence": "commit abc123, test output log"
}
```

```
## 기능 목록 규칙
- 기능 목록 파일: /docs/features.md
- 한 번에 하나의 기능만 활성화
- passing으로 표시하기 전에 검증 명령이 통과해야 함
- 기능 목록 상태를 직접 수정하지 마십시오 — 검증 스크립트가 자동으로 업데이트합니다
```

**사례: 기능 10개 전자상거래** (강의 본문 사례, 측정 출처 표기 없음): 비구조 메모로는 세 세션 후 새 세션이 상태 추론에 20분, 완료된 기능 재구현. 구조화 목록으로는 3분 만에 "F01–F05 passing, F06 active, F07–F10 not_started"를 알고 F06부터 재작업 없이 진행.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-07]]: WIP=1과 범위 표면. 이 강의는 그 범위 표면을 담는 파일 형식이다.
- [[projects/project-01]], [[projects/project-02]], [[projects/project-03]]: 코스 실습이 실제로 쓰는 `feature_list.json`. 형식이 강의와 다르다(아래).
- 실습: [[projects/project-04]]. 단, 이 실습에는 `feature_list.json`이 없다.
- 하네스 실례: [[harness/oh-my-claudecode]]의 `prd.json`은 이야기마다 완료 기준과 `passes` 상태를 두고, 완료 기준을 지우려면 근거를 남기는 수정 기록(`criterionAmendments`)을 거쳐야 한다. [[harness/gajae-code]]의 `goals.json`은 목표 상태를 `gjc ultragoal checkpoint` 명령으로만 바꾸며, 검증 증거(quality-gate)가 없으면 완료로 기록되지 않는다. 둘 다 "에이전트가 상태를 직접 못 바꾼다"는 패스 상태 게이팅의 구현이다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-08-why-feature-lists-are-harness-primitives/index.md`
- 표기 차이: ko 강의 제목은 "기능 목록으로 에이전트의 행동을 제약하십시오", ko project-04 페이지는 "기능 목록을 사용하여 에이전트가 하는 일을 제한하기"로 적는다.
