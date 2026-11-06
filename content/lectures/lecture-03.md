---
type: "lecture"
title: "강의 03. 저장소를 단일 진실 원천으로 만들어라"
aliases:
  - "저장소를 단일 진실 원천으로 만들어라"
  - "Why the Repository Must Become the System of Record"
  - "콜드 스타트 테스트"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md"
related:
  - "[[lectures/lecture-02]]"
  - "[[projects/project-02]]"
  - "[[concepts/repo-system-of-record]]"
tags:
  - "type/lecture"
  - "topic/system-of-record"
  - "topic/context-continuity"
updated: "2026-11-06"
---

# 강의 03. 저장소를 단일 진실 원천으로 만들어라

## 왕초보 설명

- **이게 뭐야?** AI는 **저장소(프로젝트 폴더)에 적힌 것만** 안다. 슬랙 대화, 회의에서 정한 것, 선배 머릿속 규칙은 AI에게 존재하지 않는다.
- **비유:** AI는 "폴더 안에 갇힌 신입 엔지니어"다. 누구에게도 물어볼 수 없으니, 필요한 지도는 폴더 안에 붙여 둬야 한다.
- **왜 중요해?** 지도가 없으면 AI는 추측한다. 틀린 추측은 버그가 되고, 매 새 대화마다 같은 추측을 반복한다.
- **나는 뭘 하면 돼?** 새 대화를 열어 AI에게 다섯 가지를 물어본다(콜드 스타트 테스트). "이게 뭐야? 어떻게 생겼어? 어떻게 실행해? 어떻게 확인해? 지금 어디까지 했어?" 대답 못 하는 질문이 있으면 그 답을 파일로 써서 저장소에 넣는다.

## 핵심 요약

- 에이전트의 입력은 시스템 프롬프트·작업 설명, 저장소 파일, 도구 실행 결과뿐이다. **저장소에 없는 정보는 에이전트에게 존재하지 않는다**(OpenAI의 "저장소를 명세로" 원칙).
- 목표는 "문서를 더 많이"가 아니라 "결정 정보를 올바른 장소에". 모듈 옆 50줄 `ARCHITECTURE.md`가 아무도 관리 안 하는 Confluence 500쪽보다 낫다.
- 저장소 품질은 **콜드 스타트 테스트**로 잰다.
- 상태 관리는 데이터베이스의 **ACID** 원칙을 빌려 설계한다.
- **지식 부패**(코드와 어긋난 문서)가 가장 큰 적이다. 낡은 문서는 문서가 없는 것보다 위험하다.

## 핵심 개념

**저장소 = 기록의 원천** ([[concepts/repo-system-of-record]]). 프로젝트 결정, 아키텍처 제약, 실행 상태, 검증 기준의 최종 권위는 저장소다. 다른 어디에 있는 정보도 인정되지 않는다.

**콜드 스타트 테스트(Cold-Start Test)** `후보:cold-start-test`. 저장소 내용만 가진 새 세션이 다섯 질문에 답할 수 있는지 본다.

| 질문 | 답이 있어야 할 곳 |
| --- | --- |
| 이 시스템이 무엇인가? | `AGENTS.md`, `README` |
| 어떻게 구성되어 있는가? | `ARCHITECTURE.md`, 모듈 문서 |
| 어떻게 실행하는가? | `Makefile`, `init.sh`, 패키지 스크립트 |
| 어떻게 검증하는가? | 테스트·린트·검사 명령([[concepts/verification-commands]]) |
| 지금 어디에 있는가? | `PROGRESS.md`, 기능 목록, git 기록 |

다섯 개 모두 답하면 새 세션이 사람에게 묻지 않고 일을 시작할 수 있다.

**측정 용어**

- **지식 가시성 격차**: 전체 프로젝트 지식 중 저장소 밖에 있는 비율. 연습 2는 이걸 10% 미만으로 낮추라고 한다.
- **발견 비용(Discovery Cost)**: 에이전트가 핵심 정보를 찾느라 쓰는 컨텍스트. 깊이 숨길수록 실제 작업에 쓸 예산이 준다. 지하실 금고에 잠긴 소화기와 같다.
- **지식 부패율**: 단위 시간당 낡아지는 지식 비율.

**좋은 지도를 그리는 네 원칙**

1. 지식은 코드 옆에: API 인증 규칙은 API 코드 옆 문서에.
2. 표준 진입 파일: `AGENTS.md`(또는 `CLAUDE.md`)가 "무엇·실행·검증" 세 질문에 답한다. 50~100줄([[concepts/map-not-manual]]).
3. 최소하되 완전하게: 지워도 결정 품질이 안 변하는 규칙은 없어야 하지만, 콜드 스타트 다섯 질문엔 모두 답이 있어야 한다.
4. 코드와 함께 갱신: 문서를 모듈 디렉터리에 두면 코드를 고칠 때 자연히 보게 된다. CI로 문서 갱신 여부를 알릴 수도 있다.

```
project/
├── AGENTS.md              # 진입점: 개요, 실행 명령, HARD 제약
├── src/api/ARCHITECTURE.md  # API 레이어 결정
├── src/db/CONSTRAINTS.md    # DB 작업 HARD 제약
├── PROGRESS.md             # 완료 / 진행 중 / 차단
└── Makefile                # setup, test, lint, check
```

**ACID로 에이전트 상태 관리하기**

| 원칙 | 에이전트 작업에 적용 | 비유 |
| --- | --- | --- |
| 원자성 | 논리적 작업 하나 = git 커밋 하나. 중간 실패는 롤백. "반쯤 완료" 없음 | 전부 아니면 아무것도 |
| 일관성 | "일관된 상태"(테스트 통과, 린트 오류 없음)를 정의하고 매 작업 후 검증. 깨진 상태는 커밋 안 함 | 입금 없이 출금 불가 |
| 격리성 | 여러 에이전트가 동시에 일하면 진행 파일을 따로 쓰거나 브랜치로 나눔 | 두 주방장이 한 냄비 간을 동시에 안 봄 |
| 내구성 | 세션을 넘는 지식은 git으로 추적되는 파일에 | 머릿속 말고 종이에 |

**OpenAI 사례.** Codex로 내부 제품을 만들며 아키텍처, 설계 결정, 실행 계획, 진행 상황을 버전 관리 파일에 두고, docs 디렉터리의 최신성과 링크를 검사했다. 강의는 이것이 공개 사례일 뿐 문서의 효과를 분리 측정한 대조 실험은 아니라고 명시한다.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-02]]: "저장소는 단일 진실 원천"을 핵심 개념 하나로 짧게 소개했고, 이 강의가 그것을 펼친다.
- [[lectures/lecture-01]]: 다섯 방어 레이어 중 "컨텍스트 제공"과 "상태 관리"를 다룬다.
- 실습: [[projects/project-02]]. 문서와 세션 인계 파일로 새 세션이 이어받을 수 있는지 시험한다.
- 다음: [[lectures/lecture-04]]. 진입 파일 하나에 다 넣으면 왜 실패하는지.
- 상태 심화: [[lectures/lecture-05]](장기 작업 연속성), [[lectures/lecture-12]](세션 종료 시 깨끗한 상태).
- 하네스 실례: [[harness/gajae-code]]는 계획·목표·증거를 `.gjc/` 아래 파일로, [[harness/oh-my-claudecode]]는 `.omc/state/`의 `prd.json`과 `progress.txt`로 남긴다. 둘 다 "세션을 넘는 상태는 파일에" 원칙을 따른다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md`
- 표기 차이: ko 강의 제목은 "저장소를 단일 진실 원천으로 만들어라", ko project-02 페이지는 "저장소를 단일 진실 원천으로 만들기"로 적는다.
