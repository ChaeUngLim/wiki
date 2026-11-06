---
type: "lecture"
title: "강의 04. 명령 파일을 여러 파일로 분산하라"
aliases:
  - "명령 파일을 여러 파일로 분산하라"
  - "Why One Giant Instruction File Fails"
  - "거대한 명령 파일 함정"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md"
related:
  - "[[lectures/lecture-03]]"
  - "[[projects/project-02]]"
  - "[[concepts/map-not-manual]]"
  - "[[concepts/repo-system-of-record]]"
tags:
  - "type/lecture"
  - "topic/instructions"
  - "topic/context-continuity"
updated: "2026-11-06"
---

# 강의 04. 명령 파일을 여러 파일로 분산하라

## 왕초보 설명

- **이게 뭐야?** AI가 실수할 때마다 `AGENTS.md`에 규칙을 한 줄씩 더하다 보면 파일이 600줄이 되고, 오히려 AI가 더 못하게 된다는 이야기다.
- **비유:** 여행 가방에 "혹시 몰라서" 계속 욱여넣으면, 속옷 한 장 찾으려고 가방을 다 비워야 한다. 해법은 패킹 큐브(주제별 작은 주머니)다.
- **왜 그래?** AI가 한 번에 볼 수 있는 양(컨텍스트)은 정해져 있다. 긴 규칙 파일이 그 자리를 먹고, 파일 **가운데**에 있는 규칙은 잘 잊는다.
- **나는 뭘 하면 돼?** `AGENTS.md`는 50~200줄짜리 **안내판**으로 둔다. 프로젝트 한두 줄 소개, 실행·검사 명령, 절대 규칙 몇 개, 그리고 "API 작업이면 `docs/api-patterns.md`를 읽어라" 같은 링크만. 자세한 건 주제별 문서로 옮긴다.

## 핵심 요약

- **악순환**: 실수 → 규칙 추가 → 잠깐 효과 → 다른 실수 → 또 추가. 파일은 커지기만 한다.
- 거대한 파일의 다섯 가지 피해: 컨텍스트 예산 잠식, 중간에서 길을 잃음, 우선순위 충돌, 유지보수 붕괴, 모순 누적.
- 해법: 진입 파일은 **라우터**(안내판)이고 백과사전이 아니다. 상세는 주제 문서로 나눠 필요할 때만 읽힌다(점진적 공개).
- 모든 규칙에 출처·적용 조건·만료 조건을 달고 정기적으로 감사한다. 규칙을 코드 의존성처럼 관리한다.
- 단, 짧은 파일이 개선을 **보장하지는 않는다**. 실제 작업으로 검증해야 한다(아래 근거 참고).

## 핵심 개념

**거대한 파일이 망가지는 이유**

| 문제 | 무슨 일이 생기나 |
| --- | --- |
| 컨텍스트 예산 잠식 | 600줄 `AGENTS.md`는 10,000~20,000 토큰. 128K 창의 8~15%가 시작 전에 사라진다 |
| 중간 손실(Lost in the Middle) | Liu et al.(2023): LLM은 긴 글의 가운데를 처음·끝보다 덜 활용한다. 300번째 줄의 보안 규칙은 무시되기 쉽다 |
| 우선순위 모호성 | "절대 `eval()` 금지"(하드 제약)와 "함수형 선호"(권장)와 "지난주 WebSocket 누수 주의"(교훈)가 같은 모양이라 중요도를 구별할 수 없다 |
| 유지보수 붕괴 | 삭제는 결과가 불확실해 안 하고, 추가는 공짜 같아서 계속한다. 신호 대 잡음비(SNR)가 계속 떨어진다 |
| 모순 누적 | "TypeScript strict"와 "일부 레거시는 any 허용"이 함께 있으면 에이전트가 매번 임의로 고른다 |

**용어**

- **명령 비대화**: 명령 파일이 컨텍스트 창의 10~15%를 넘으면 코드 읽기·추론 예산을 잠식하기 시작한다.
- **명령 SNR**: 파일 속 명령 중 현재 작업과 관련 있는 비율.
- **라우팅 파일** ([[concepts/map-not-manual]]): 모든 걸 담지 않고 상세 문서로 안내하는 짧은 진입 파일. 50~200줄.
- **[[concepts/progressive-disclosure|점진적 공개(Progressive Disclosure)]]**: 먼저 개요를, 필요할 때 상세를 준다. 좋은 UI가 옵션을 한 번에 쏟아내지 않는 것과 같다.

**분산하는 방법**

- 자주 필요한 건 손 닿는 곳(진입 파일), 가끔 필요한 건 치워 두고(주제 문서), 쓰지 않을 건 버린다.
- 진입 파일에는 프로젝트 개요(1~2문장), 첫 실행 명령(`make setup && make test`), 전역 하드 제약(최대 15개), 주제 문서 링크(한 줄 설명 + 언제 읽는지)만.
- 주제 문서는 50~150줄, `docs/`나 해당 모듈 옆에.
- 타입 정의, 인터페이스 주석, 설정 파일 설명처럼 코드에 직접 둘 수 있는 건 코드에 둔다. 명령 파일에 중복하지 않는다.
- 진입 파일에 꼭 둬야 하는 규칙은 맨 위나 맨 아래에. 절대 가운데에 두지 않는다.

```markdown
# AGENTS.md
## 프로젝트 개요
Python 3.11 FastAPI 백엔드, PostgreSQL 15 데이터베이스.
## 빠른 시작
- 설치: `make setup` / 테스트: `make test` / 전체 검증: `make check`
## 하드 제약
- 모든 API는 OAuth 2.0 인증을 사용해야 함
- 모든 PR은 pytest + mypy --strict + ruff check를 통과해야 함
## 주제 문서
- API 설계 패턴 (`docs/api-patterns.md`) — 엔드포인트 추가 시 필독
- 데이터베이스 규칙 (`docs/database-rules.md`) — DB 작업 수정 시 필독
```

**근거와 한계** (강의가 직접 단서를 단다)

| 출처 | 내용 | 한계 |
| --- | --- | --- |
| OpenAI | 거대 `AGENTS.md`의 문제를 겪고 약 100줄 진입 파일 + 구조화된 docs(lint·CI로 관리)로 바꿈 | 전후 성공률·보안 규칙 준수율 수치 없음 |
| ETH Zurich | 평가 환경에서 저장소 컨텍스트 파일이 성공률을 일반적으로 높이지 않았고 추론 비용은 20% 이상 증가. 사람이 쓰는 요구사항을 최소화하라고 권함 | 효과는 내용·작업에 따라 다름 |
| Lulla et al. 표 1 | gpt-5.2-codex, 10개 저장소 PR 기반 작업 124개, `AGENTS.md` 유무 비교: 중위 실행 시간 98.57초→70.34초(28.64% 감소), 중위 출력 토큰 2,925→2,440(16.58% 감소) | 효율만 측정. 거대 파일 분할 효과나 기능적 정확성은 평가 안 함 |

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-03]]: "무엇을 저장소에 둘까"를 다뤘고, 이 강의는 "어떻게 나눠 둘까"를 다룬다. 두 강의의 실습이 [[projects/project-02]]다.
- [[lectures/lecture-02]]: "지도를 줘라, 매뉴얼이 아니라"를 한 줄로 예고했다.
- [[lectures/lecture-01]]: 다섯 방어 레이어 중 "컨텍스트 제공" 레이어의 설계 지침이다.
- 하네스 실례: [[harness/gajae-code]]의 기본 기능은 스킬 4개뿐이고, 긴 규칙(예: ultragoal의 경계 검토, 종료 critic 게이트)은 별도 fragment 파일로 빼서 그 단계에서만 읽게 한다. 진입 지시를 짧게 두고 필요할 때 상세를 여는 점진적 공개 구조다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md`
- 표기 차이: ko 강의 제목은 "명령 파일을 여러 파일로 분산하라", ko project-02 페이지는 "지침을 여러 파일로 분리하기"로 적는다.
