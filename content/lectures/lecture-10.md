---
type: "lecture"
title: "강의 10. 엔드투엔드 테스트만이 진정한 검증이다"
aliases:
  - "엔드투엔드 테스트만이 진정한 검증이다"
  - "Why End-to-End Testing Changes Results"
  - "리뷰 피드백 승격"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md"
  - "raw/oh-my-claudecode/skills/team/SKILL.md"
related:
  - "[[lectures/lecture-09]]"
  - "[[concepts/verification-commands]]"
  - "[[concepts/executable-architecture-rules]]"
tags:
  - "type/lecture"
  - "topic/verification"
updated: "2026-11-06"
---

# 강의 10. 엔드투엔드 테스트만이 진정한 검증이다

## 왕초보 설명

- **이게 뭐야?** 부품마다 따로 하는 검사(단위 테스트)는 다 통과해도, 부품을 **붙였을 때** 생기는 고장은 못 잡는다. 그래서 사용자가 실제로 버튼을 누르는 흐름 전체를 돌려 보는 검사(엔드투엔드, E2E)가 필요하다는 이야기다.
- **비유:** 합창. 각 성부가 헤드폰 끼고 따로 연습하면 다 완벽한데, 같이 부르면 한쪽은 반 박자 빠르고 반주는 반음 낮다.
- **왜 중요해?** AI는 가장 빠른 검사만 돌리고 "완료"라고 한다. 그런데 AI가 "E2E 검사를 받는다"는 걸 알면 처음부터 부품 연결을 신경 써서 짠다.
- **나는 뭘 하면 돼?** ① 여러 부분을 건드리는 작업은 E2E 테스트 통과를 완료 조건으로 적는다. ② "화면 코드는 파일을 직접 못 연다" 같은 구조 규칙을 문서가 아니라 **검사 스크립트**로 만든다. ③ 리뷰에서 같은 지적이 두 번 나오면 그걸 자동 검사로 바꾼다.

## 핵심 요약

- 단위 테스트는 격리(모킹)가 설계 철학이라 **컴포넌트 경계 결함에 체계적으로 눈이 멀었다**. 엔드투엔드 테스트만이 시스템 수준 결함이 없음을 보여 줄 수 있다.
- E2E는 결과만이 아니라 **에이전트의 코딩 행동을 바꾼다**: 컴포넌트 상호작용을 고려하고, 아키텍처 경계를 지키고, 오류 경로를 처리하게 된다.
- E2E의 전제는 명확한 아키텍처 경계다. 경계 규칙은 **실행 가능한 검사**가 되어야 한다([[concepts/executable-architecture-rules]]).
- 오류 메시지는 에이전트용으로: 무엇이, 왜, 어떻게 고치나.
- **리뷰 피드백 승격**으로 하네스가 시간이 갈수록 강해진다.

## 핵심 개념

**단위 테스트의 맹점 네 가지**

| 유형 | 예 | 비유 |
| --- | --- | --- |
| 인터페이스 불일치 | renderer는 상대 경로, preload는 절대 경로를 기대 | 4/4박과 3/4박 |
| 상태 전파 오류 | 마이그레이션이 스키마를 바꿨는데 ORM 캐시는 옛 스키마 | 바뀐 가사를 모르는 사람 |
| 리소스 수명 주기 | 파일 핸들·DB 연결 경합·누수 | 무대에서 마이크 부족 |
| 환경 의존성 | 모킹 환경에선 되고 실제 설정·네트워크에선 실패 | 리허설실 vs 야외 페스티벌 |

**용어**

- **컴포넌트 경계 결함**: A와 B가 각자 단위 테스트는 통과하지만 상호작용이 틀린 경우.
- **테스팅 적합성 기울기**: 단위 ≤ 통합 ≤ E2E 순으로 잡는 결함이 늘어난다.
- **아키텍처 경계 강제 규칙**: 문서의 규칙을 CI에서 도는 검사로.
- **리뷰 피드백 승격** `후보:review-feedback-promotion`: 반복되는 리뷰 코멘트를 자동 테스트로 바꾸기. 결함 유형 하나가 영구 방어선이 된다.
- **에이전트 지향 오류 메시지**:

```
ERROR: Found direct import of 'fs' in src/renderer/App.tsx:12
WHY: Renderer process has no access to Node.js APIs for security
FIX: Move file operations to src/preload/file-ops.ts and call via window.api.readFile()
```

**OpenAI 경험**: 에이전트 코드베이스에서 아키텍처 제약은 팀이 커질 때가 아니라 **1일차 전제 조건**이다. 에이전트는 저장소의 기존 패턴을 그대로 복사하기 때문이다. OpenAI는 도메인마다 Types → Config → Repo → Service → Runtime → UI 고정 계층을 두고 의존성을 앞으로만 흐르게 하고, 커스텀 린트로 기계 강제했다. 원칙: **불변식은 강제하되 구현은 마이크로매니지하지 않는다**(예: "데이터는 경계에서 파싱"은 요구하되 라이브러리는 지정 안 함).

**검증 계층 구조 예시**

```
## 검증 계층 구조
- 레벨 1: 단위 테스트 (반드시 통과)
- 레벨 2: 통합 테스트 (반드시 통과)
- 레벨 3: 엔드투엔드 테스트 (크로스 컴포넌트 변경 시 반드시 통과)
- 필수 레벨 건너뛰기 = 완료 아님
```

**사례: Electron 파일 내보내기** (강의 본문 사례): renderer·preload·서비스 단위 테스트 모두 통과. E2E에서 다섯 결함(경로 형식, IPC 진행률 전달, 대용량 파일 핸들 누수, 패키징 환경 권한, 서비스 예외가 UI로 안 옴)이 나왔고 단위 테스트는 하나도 못 잡았다.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-09]]: 3단계 종료 검사의 3단계(시스템 수준)를 이 강의가 구체화한다.
- [[lectures/lecture-02]]: "제약하되 마이크로매니지하지 않는다"를 처음 제시.
- [[projects/project-04]]: `scripts/check-architecture.sh`가 이 강의의 "아키텍처 규칙을 실행 가능한 검사로"를 이미 구현한다(grep 기반, renderer의 Node 모듈 금지 등).
- 실습: [[projects/project-05]].
- 하네스 실례: [[harness/oh-my-claudecode]]의 `team`은 `team-verify` 단계에서 `verifier`를 항상 돌리고, 변경 성격에 따라 `test-engineer`·`security-reviewer`·`code-reviewer`를 더한다(보안 민감 또는 20개 초과 파일 변경이면 `security-reviewer` + `code-reviewer` 필수, `skills/team/SKILL.md`). [[harness/gajae-code]]의 `ultragoal` 경계 검토는 `executor` QA·레드팀 레인을 포함한다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md`
- 표기 차이: ko 강의 제목은 "엔드투엔드 테스트만이 진정한 검증이다", ko project-05 페이지는 "완전한 파이프라인 실행만이 진정한 검증으로 인정됩니다"로 적는다.
