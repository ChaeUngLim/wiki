---
type: "lecture"
title: "강의 11. 에이전트의 런타임을 관측 가능하게 만들어라"
aliases:
  - "에이전트의 런타임을 관측 가능하게 만들어라"
  - "Making the Agent's Runtime Observable"
  - "Why Observability Belongs Inside the Harness"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/deepseek/index.md"
related:
  - "[[concepts/sprint-contract]]"
  - "[[concepts/evaluator-rubric]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/lecture"
  - "topic/observability"
updated: "2026-11-06"
---

# 강의 11. 에이전트의 런타임을 관측 가능하게 만들어라

## 왕초보 설명

- **이게 뭐야?** AI가 "완료됐는데 테스트 두 개가 실패해요. 왜인지는 잘 모르겠어요"라고 할 때, 문제는 AI가 아니라 **보이는 정보가 없는 것**이라는 이야기다. 관측 가능성(observability, 안에서 무슨 일이 있었는지 밖에서 볼 수 있는 정도)을 하네스에 넣어야 한다.
- **비유:** 블랙박스 없는 사고 조사. 증거가 없으면 "아마 이랬겠지" 추측만 반복한다.
- **두 종류를 다 봐야 한다:** ① 프로그램이 **무엇을 했나**(로그, 실행 기록) ② AI가 **왜 그렇게 했나**(작업 전에 쓴 계약서, 채점표).
- **나는 뭘 하면 돼?** 큰 작업 전에 "바꿀 것 / 안 바꿀 것 / 통과 기준"을 한 장 적게 하고(스프린트 계약), 검토는 점수표(루브릭)로 하게 하고, 앱은 구조화된 로그를 남기게 한다.

## 핵심 요약

- 관측 가능성이 없으면 결정은 불확실성 속에서, 평가는 주관으로, 재시도는 맹목으로 이뤄진다. OpenAI와 Anthropic 모두 신뢰성을 **증거 문제**로 정의한다.
- **관측 가능성은 하네스 아키텍처의 속성**이다. 나중에 붙이는 기능이 아니다.
- **계층화된 관측 가능성**: 런타임 신호는 "무슨 일이 있었나", 프로세스 산출물은 "왜 그렇게 했나"를 설명한다. 둘 다 필요하다.
- 에이전트에게 로그를 맡기면 안 되는 이유: 모르는 신호는 기록하지 않고, 세션마다 형식이 다르고, 계약·루브릭 같은 프로세스 산출물은 로그로 해결되지 않는다.

## 핵심 개념

- **런타임 관측 가능성**: 로그, 트레이스, 프로세스 이벤트, 상태 확인. 하네스가 자동 수집할 것: 앱 수명 주기(시작·준비·실행·종료), 기능 경로 실행(진입·체크포인트·종료), 컴포넌트 간 데이터 흐름, 리소스 이상, 전체 오류 컨텍스트.
- **프로세스 관측 가능성**: 계획, 채점 루브릭, 인수 기준 등 하네스 결정 산출물.
- **태스크 트레이스** `후보:task-trace`: 시작부터 완료까지의 결정 경로 기록. OpenTelemetry로 표준화하면 세션 = 트레이스, 작업 = 스팬, 검증 단계 = 하위 스팬이 되어 Jaeger·Zipkin과 연결된다.
- **[[concepts/sprint-contract|스프린트 계약]]**: 코딩 전에 범위·검증 기준·제외 사항을 합의. 강의 예시는 "다크 모드 지원"(제외: 인쇄 스타일, 써드파티 컴포넌트).
- **[[concepts/evaluator-rubric|평가자 루브릭]]**: "좋은가"를 차원별 등급(코드 정확성, 아키텍처 준수, 테스트 커버리지 × A~D)으로 바꿔 평가를 재현 가능하게.

**관측 가능성 부재의 네 비용**: 정확함과 정확해 보임을 구별 못 함 / 평가가 신비주의가 됨 / 재시도가 맹목적 추측 / 세션 핸드오프 정보 절벽([[concepts/continuity-artifacts]]).

**사례: 다크 모드** (플래너-생성자-평가자 워크플로, 강의의 가정 시나리오)

| 버전 | 반복 | 시간 | 결과 |
| --- | --- | --- | --- |
| 관측 불가 | 3~4번 맹목적 재시도 | 약 45분 | 겨우 수용 가능. 평가자는 "느낌이 안 좋다"만 말함 |
| 완전 관측 | 1번 | 약 15분 | 고품질. 평가자가 차원별로 증거 인용 |

강의는 이 시나리오를 "상상해 보세요"로 소개한다. 측정된 실험이 아니다.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-09]]·[[lectures/lecture-10]]: 검증을 외부화했다면, 이 강의는 그 검증이 쓸 **증거**를 만든다.
- [[projects/project-04]]: solution의 `logger.ts`(JSON 구조화 로그, `forService`)가 런타임 관측 가능성의 실습 구현이다.
- [[projects/project-05]]: `sprint-contract.md`와 `evaluator-rubric.md`가 프로세스 관측 가능성의 실습 구현이다.
- 실습: [[projects/project-06]](ko 링크 제목 "완전한 하네스 (캡스톤)").
- 하네스 실례: [[harness/deepseek]]는 루프의 모든 단계를 이벤트 지점으로 만들고 "모델이 본 것은 모두 기록"을 런타임 불변 조건으로 강제한다. [[harness/gajae-code]]의 `ultragoal`은 `ledger.jsonl`에 체크포인트를 남기고, [[harness/oh-my-claudecode]]는 런타임 상태·세션 데이터·계획·로그·핸드오프를 `.omc/` 아래에 쓴다(README "`.omc/` state and git"). 둘 다 프로세스 산출물을 에이전트 머리 밖에 둔다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-11-why-observability-belongs-inside-the-harness/index.md`
- 표기 차이: 디렉터리 이름은 "Why Observability Belongs Inside the Harness", en 본문 제목은 "Making the Agent's Runtime Observable". 실습 링크 제목은 ko "Project 06. 완전한 하네스 (캡스톤)", en "Project 06. Build a Complete Agent Harness"이고 경로는 `project-06-runtime-observability-and-debugging`이다.
