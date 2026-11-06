---
type: "lecture"
title: "강의 01. 강력한 모델도 실행 신뢰성을 보장하지 않는다"
aliases:
  - "강력한 모델도 실행 신뢰성을 보장하지 않는다"
  - "Why Capable Agents Still Fail"
  - "다섯 가지 방어 레이어"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-01-why-capable-agents-still-fail/index.md"
related:
  - "[[lectures/lecture-02]]"
  - "[[projects/project-01]]"
  - "[[concepts/verification-commands]]"
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/definition-of-done]]"
tags:
  - "type/lecture"
  - "topic/harness-model"
  - "topic/verification"
updated: "2026-11-06"
---

# 강의 01. 강력한 모델도 실행 신뢰성을 보장하지 않는다

## 왕초보 설명

- **이게 뭐야?** "AI가 시험 점수는 높은데 왜 내 프로젝트에선 엉망이지?"에 대한 답이다. 답은 대개 **AI가 아니라 AI에게 준 작업 환경**이 문제라는 것.
- **비유:** 경주마(모델)가 아무리 빨라도 안장·고삐 같은 마구(하네스) 없이는 멀리 못 가고 자주 떨어진다.
- **왜 중요해?** 실패할 때마다 "더 비싼 모델로 바꾸자"고 하면 돈만 쓰고 같은 실패를 반복한다. 같은 모델도 환경만 고치면 결과가 확 달라진다.
- **나는 뭘 하면 돼?** AI가 실패하면 모델 탓을 하기 전에 다섯 가지를 차례로 묻는다. ① 할 일을 분명히 말했나? ② 프로젝트 규칙을 알려줬나? ③ 실행 환경이 갖춰졌나? ④ 다 됐는지 확인할 방법을 줬나? ⑤ 지난번까지 한 일을 기록해 뒀나? 그리고 저장소 맨 위에 `AGENTS.md` 한 장부터 만든다.

## 핵심 요약

- **모델 역량 ≠ 실행 신뢰성.** Claude Sonnet 4.5는 SWE-bench Verified 77.2%였지만, 그건 이슈 설명·테스트·환경이 다 갖춰진 엄선된 작업 기준이다. 실제 프로젝트에선 이것들을 직접 마련해야 해서 수치가 더 낮다.
- **같은 모델, 다른 하네스.** Anthropic은 같은 프롬프트("2D 레트로 게임 메이커")와 같은 모델(Opus 4.5)을 비교했다. 맨손: 20분·9달러, 입력에 반응하지 않는 앱. 플래너+생성기+평가기 하네스: 6시간·200달러, 핵심 게임플레이는 작동하지만 미완성. 모델은 그대로였고 바뀐 건 마구였다.
- **실패하면 하네스부터 점검한다.** 모델 교체는 가장 비싼 선택지이고, 대부분 모델 문제가 아니다.
- **모든 실패를 다섯 방어 레이어 중 하나에 귀속시킨다**: 작업 명세, 컨텍스트 제공, 실행 환경, 검증 피드백, 상태 관리.
- `AGENTS.md` 파일 하나가 더 비싼 모델로 바꾸는 것보다 효과적일 수 있다.

## 핵심 개념

**에이전트가 막히는 다섯 지점(방어 레이어)**

| 레이어 | 무엇이 잘못되나 | 비유 |
| --- | --- | --- |
| 작업 명세 | "검색 기능 추가해"만 들으면 전문 검색인지, 페이지네이션은 있는지 에이전트가 추측한다 | 식당에서 "생선으로 주세요"만 말하기 |
| 컨텍스트 제공 | 팀 규칙(SQLAlchemy 2.0, OAuth 2.0 필수)이 머릿속·슬랙에만 있어 에이전트가 존재 자체를 모른다 | 신입에게 사내 규칙을 안 알려 주기 |
| 실행 환경 | 의존성 누락, 버전 불일치로 `pip install` 실패에 컨텍스트를 낭비한다 | 망치도 작업대도 없는 목수 |
| 검증 피드백 | 테스트·린트·검증 명령이 없어 "괜찮아 보이니 완료" | 답안지 없이 숙제 제출 |
| 상태 관리 | 세션이 바뀌면 전에 알아낸 게 사라져 매번 처음부터 탐색. 30분 넘는 작업에서 실패율이 급증 | 매일 기억을 잃는 직원 |

이 다섯 레이어는 [[lectures/lecture-02]]의 [[concepts/harness-subsystems|5개 하위 시스템]](지시·도구·환경·상태·피드백)을 "고장 진단" 쪽에서 본 것이다.

**핵심 용어**

- **역량 격차(Capability Gap)**: 벤치마크 성능과 실제 작업 성능의 차이.
- **하네스 유발 실패(Harness-Induced Failure)**: 모델은 충분한데 환경의 구조적 결함 때문에 실패한 경우.
- **검증 격차(Verification Gap)**: 에이전트의 "완료" 자신감과 실제 정확성의 차이. 가장 흔한 실패 유형이다. 줄이는 도구가 [[concepts/verification-commands|검증 명령어]]다.
- **컨텍스트 불안(context anxiety)** ([[concepts/context-anxiety]]): Anthropic 관찰. 에이전트가 컨텍스트가 모자란다고 느끼면 서둘러 마무리하고 검증을 건너뛴다. 시험 종료 직전 남은 문제를 찍는 것과 같다.
- **진단 루프(Diagnostic Loop)** `후보:diagnostic-loop`: 실행 → 실패 관찰 → 특정 레이어에 귀속 → 그 레이어 수정 → 재실행. 강의가 꼽는 하네스 엔지니어링의 핵심 방법론이다. 작업별 성공·실패와 원인 레이어를 로그로 남기면 병목 레이어가 보인다.
- **완료 기준(Definition of Done)** ([[concepts/definition-of-done]]): 기계로 확인할 수 있는 조건 묶음(테스트 통과, 린트 클린, 타입 검사 통과). 없으면 에이전트가 자기 기준을 만든다. 예:

```
완료 기준:
- 새 엔드포인트 GET /api/search?q=xxx
- 페이지네이션 지원, 기본 20개 항목
- 결과에 하이라이트된 스니펫 포함
- 모든 새 코드가 pytest 통과
- 타입 검사 통과 (mypy --strict)
```

**사례 두 개**

- **OpenAI 백만 줄 실험**: 2025년 8월부터 5개월간 빈 저장소에서 Codex로 내부 제품을 만들었다. 약 100만 줄, PR 약 1,500개 병합(엔지니어 1인당 하루 3.5개), 엔지니어 3명 → 7명. 규칙: 사람은 코드를 직접 쓰지 않는다. 초기에 느렸던 이유는 모델이 아니라 환경 부족이었고, 실패 때마다 묻는 질문은 "더 열심히"가 아니라 "에이전트에게 어떤 역량이 부족한가"였다.
- **FastAPI 앱(약 15,000줄)**: 한 문장 지시만 줬을 때 Claude Sonnet은 탐색에 시간을 쓰고, 프로젝트 오류 처리 패턴을 무시하고, 구식 SQLAlchemy 문법을 쓰고, 런타임 오류가 있는데 완료를 선언했다. `AGENTS.md`, 검증 명령(`pytest tests/api/v2/ && python -m mypy src/`), 아키텍처 결정 기록을 추가하자 같은 모델이 컨벤션을 따르고 명령으로 확인했다.

## 다른 강의·프로젝트와의 연결

- 다음 강의: [[lectures/lecture-02]]가 "그럼 하네스는 정확히 무엇인가"를 5개 하위 시스템으로 정의한다.
- 실습: [[projects/project-01]]이 같은 작업을 프롬프트만 vs 최소 하네스로 돌려 이 강의의 연습 1(비교 실험)을 그대로 수행한다.
- 컨텍스트 레이어 심화: [[lectures/lecture-03]](저장소를 기록 원천으로), [[lectures/lecture-04]](지시 파일 분산).
- 상태 레이어 심화: [[lectures/lecture-05]], [[lectures/lecture-06]].
- 검증 레이어 심화: [[lectures/lecture-09]](이른 완료 선언), [[lectures/lecture-10]](종단 간 테스트).

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-01-why-capable-agents-still-fail/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-01-why-capable-agents-still-fail/index.md`
- 표기 차이: ko 강의 제목은 "강력한 모델도 실행 신뢰성을 보장하지 않는다"인데, ko project-01 페이지는 "강력한 모델이 곧 신뢰할 수 있는 실행을 의미하지는 않습니다"로 적는다.
