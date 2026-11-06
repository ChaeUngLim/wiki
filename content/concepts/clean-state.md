---
type: "concept"
title: "클린 상태"
aliases:
  - "Clean state"
  - "클린 핸드오프"
  - "clean-state-checklist.md"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/projects/project-04/solution/clean-state-checklist.md"
related:
  - "[[concepts/definition-of-done]]"
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/verification-commands]]"
tags:
  - "type/concept"
  - "topic/clean-state"
updated: "2026-11-06"
---

# 클린 상태

## 왕초보 설명

- **이게 뭐야?** 세션을 끝낼 때 "다음 사람이 아무것도 고치지 않고 바로 일을 시작할 수 있는 상태"로 두는 것이다.
- **비유:** 공용 주방. 설거지 끝내고, 불 끄고, 다 쓴 재료는 메모해 두고 나간다.
- **나는 뭘 하면 돼?** `clean-state-checklist.md`를 만들어 빌드·테스트·진행 기록·임시 파일·앱 실행 다섯 가지를 체크하게 하고, `AGENTS.md`에 "이 체크리스트를 통과해야 세션 완료"라고 적는다.

## 정의

세션 종료 시 다섯 조건을 모두 만족하는 상태: 빌드 통과, 테스트 통과(기존 테스트 포함), 진행 상황 기록됨, 오래된 산출물·디버그 코드 없음, 표준 시작 경로 사용 가능. 하나라도 빠지면 세션은 완료가 아니다.

## 왜 중요한가

- 엔트로피 증가가 기본값이라, 정리하지 않으면 매 세션 엉망이 쌓이고 다음 세션은 진단부터 한다.
- 세션 종료 상태가 다음 세션의 시작 비용을 정한다.
- DB 트랜잭션처럼 "완전히 커밋하거나 롤백"하는 세션 무결성의 실천이다.

## 등장하는 곳

- [[lectures/lecture-12]]: 다섯 차원, 세션 종료 체크리스트, 즉각 정리 + 주간 클린업 루프, 멱등 정리.
- [[projects/project-03]]: `clean-state-checklist.md`의 "범위 통제 검증" 묶음(모든 기능 pass, 기능별 증거 등).
- [[projects/project-04]]: `clean-state-checklist.md`를 Build / Architecture / Runtime / Data Integrity / Repository로 나누고, "커밋 전과 세션 끝에 실행"하라고 적는다. `scripts/check-architecture.sh`와 구조화 로그 확인이 체크 항목에 들어간다.
- [[projects/project-05]]: 세 변형 디렉터리 모두 `clean-state-checklist.md`를 둔다.
- [[projects/project-06]]: solution에 `clean-state-checklist.md`(`- [ ]` 41개)와 데이터 디렉터리 불일치를 찾는 `scripts/cleanup-scanner.sh`, 멱등 "클린 상태 리셋" 기능이 있다.

## 관련 개념

- [[concepts/quality-document]]: 주기적 정리 때 갱신하는 모듈별 성적표.

- [[concepts/definition-of-done]]: 클린 상태는 세션 단위 완료 정의의 일부다.
- [[concepts/continuity-artifacts]]: "진행 상황 기록" 차원이 연속성 산출물을 최신으로 유지한다.
- [[concepts/verification-commands]]: 빌드·테스트 차원은 검증 명령 실행으로 확인한다.

## 상충·열린 질문

- lecture-12의 다섯 차원 중 "기능 목록 업데이트"는 project-04 체크리스트에 없다(project-04에는 `feature_list.json` 자체가 없다). 대신 "최종 요약에 현재 상태·검증·남은 위험 기록"이 있다.
