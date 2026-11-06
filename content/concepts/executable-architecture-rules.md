---
type: "concept"
title: "실행 가능한 아키텍처 규칙"
aliases:
  - "아키텍처 경계 검사"
  - "check-architecture.sh"
  - "아키텍처 경계 강제 규칙"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-10-why-end-to-end-testing-changes-results/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/projects/project-04/solution/scripts/check-architecture.sh"
related:
  - "[[concepts/verification-commands]]"
  - "[[concepts/map-not-manual]]"
tags:
  - "type/concept"
  - "topic/verification"
  - "topic/instructions"
updated: "2026-11-06"
---

# 실행 가능한 아키텍처 규칙

## 왕초보 설명

- **이게 뭐야?** "화면 코드는 파일을 직접 열면 안 된다" 같은 **설계 규칙을 문서에만 적지 않고, 위반하면 실패하는 검사 스크립트로 만드는 것**이다.
- **왜 필요해?** AI는 문서를 놓치거나 잊지만, 실패하는 검사는 못 넘어간다. 그리고 AI는 저장소에 있는 기존 코드 패턴을 그대로 따라 하기 때문에, 한 번 생긴 위반이 계속 복제된다.
- **나는 뭘 하면 돼?** 꼭 지켜야 할 구조 규칙마다 grep 한 줄이라도 검사를 만들고, 실패 메시지에 "무엇이 / 왜 / 어떻게 고치나"를 적고, `AGENTS.md`에 "커밋 전에 이 스크립트를 돌려라"를 적는다.

## 정의

아키텍처 문서의 경계 규칙(계층 간 import 금지, 의존 방향 등)을 **CI나 커밋 전에 자동으로 도는 검사**(린트 규칙, 스크립트)로 바꾸고, 실패 메시지에 수정 지침을 넣어 에이전트가 스스로 고치게 하는 것.

## 왜 중요한가

- 문서는 무시될 수 있지만 실패하는 검사는 우회할 수 없다.
- 에이전트는 기존 패턴을 복사하므로, 제약은 팀이 커질 때가 아니라 1일차 전제 조건이어야 한다(OpenAI).
- 명확한 경계가 있어야 엔드투엔드 테스트도 "어디서 설계 의도가 깨졌는지"를 알려준다.

## 등장하는 곳

- [[lectures/lecture-10]]: 원칙과 예(`grep -r "require('fs')" src/renderer/ && exit 1`), 에이전트 지향 오류 메시지(ERROR/WHY/FIX), OpenAI의 고정 계층 + 커스텀 린트.
- [[projects/project-04]]: `scripts/check-architecture.sh`가 renderer의 Node 코어 모듈, services의 Electron IPC, services·main의 React import를 찾아 위반 시 exit 1. `AGENTS.md` 시작 절차·완료 정의·세션 종료 절차 모두에 이 스크립트가 들어간다.

## 관련 개념

- [[concepts/verification-commands]]: 검증 명령의 범위를 동작에서 코드 구조까지 넓힌 것.
- [[concepts/map-not-manual]]: 규칙을 진입 파일에 쌓는 대신 실행 가능한 검사로 옮기면 지시 파일이 짧아진다.

## 상충·열린 질문

- [[projects/project-04]]의 스크립트 실패 메시지는 `VIOLATION: <파일> imports React`처럼 "무엇이"만 있고, [[lectures/lecture-10]]이 권하는 "왜 / 어떻게 고치나"는 없다.
