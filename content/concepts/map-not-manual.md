---
type: "concept"
title: "지도를 줘라, 매뉴얼이 아니라(라우팅 진입 파일)"
aliases:
  - "라우팅 파일"
  - "진입 파일은 라우터"
  - "map not manual"
sources:
  - "raw/learn-harness-engineering/docs/ko/harness-designs/codex/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md"
related:
  - "[[concepts/repo-system-of-record]]"
  - "[[concepts/harness-subsystems]]"
tags:
  - "type/concept"
  - "topic/instructions"
updated: "2026-11-06"
---

# 지도를 줘라, 매뉴얼이 아니라(라우팅 진입 파일)

## 왕초보 설명

- **이게 뭐야?** `AGENTS.md`(AI가 제일 먼저 읽는 파일)를 두꺼운 설명서가 아니라 **건물 안내판**처럼 짧게 쓰라는 원칙이다. "API 일이면 3층(`docs/api-patterns.md`)으로 가세요"처럼 길만 알려 준다.
- **왜 필요해?** AI는 한 번에 볼 수 있는 양이 정해져 있고, 긴 글의 가운데를 잘 잊는다. 설명서가 두꺼울수록 정작 중요한 규칙을 놓친다.
- **나는 뭘 하면 돼?** `AGENTS.md`가 200줄을 넘으면, 주제별로 `docs/` 아래 파일로 나누고 진입 파일엔 링크와 "언제 읽어야 하는지"만 남긴다. 꼭 지켜야 할 규칙은 맨 위에 둔다.

## 정의

진입 지시 파일을 **모든 지식을 담는 매뉴얼이 아니라, 상세 문서로 안내하는 라우터**로 설계하는 것. 진입 파일엔 개요, 실행·검증 명령, 소수의 전역 하드 제약, 주제 문서 링크(적용 조건 포함)만 두고 나머지는 필요할 때 읽게 한다.

## 왜 중요한가

- 컨텍스트 예산: 600줄 파일은 10,000~20,000 토큰을 시작 전에 소비한다.
- 중간 손실: 긴 파일 가운데 규칙은 처음·끝보다 덜 활용된다(Liu et al., 2023).
- 우선순위 신호: 하드 제약과 권장이 한 파일에 섞이면 구별되지 않는다. 진입 파일에 하드 제약만 두면 그 자체가 우선순위 신호가 된다.
- 단, 효과는 작업과 내용에 따라 다르다. ETH Zurich 연구는 컨텍스트 파일이 성공률을 일반적으로 높이지 않았다고 보고한다. 실제 작업으로 확인해야 한다.

## 등장하는 곳

- [[lectures/lecture-02]]: "`AGENTS.md`는 백과사전이 아니라 목차, 100줄 안팎"이라고 한 줄로 제시.
- [[projects/project-02]]: solution `AGENTS.md`(68줄)가 5단계 시작 규칙으로 `docs/ARCHITECTURE.md`·`docs/PRODUCT.md`를 가리키는 안내판 역할을 한다.
- [[lectures/lecture-04]]: 거대 파일의 다섯 피해와 분산 방법(진입 50~200줄, 주제 문서 50~150줄, 하드 제약 최대 15개)으로 펼친다.

- [[harness/codex]]: 코스의 Codex 분석은 이 원칙의 "권위 있는 출처"로 OpenAI 《Harness Engineering》의 "AGENTS.md should be a directory page"를 든다(약 100줄, 넘치면 `docs/`). [[harness/claude-code]]는 같은 문제를 `CLAUDE.md` 범위 계층과 하위 디렉터리 필요 시 로드로 푼다.

## 관련 개념

- [[concepts/progressive-disclosure]]: 지도가 가리키는 상세를 필요할 때만 여는 원칙.

- [[concepts/repo-system-of-record]]: 지식을 저장소에 두라는 원칙. 이 개념은 그 지식을 저장소 안에서 어떻게 배치할지다.
- [[concepts/harness-subsystems]]: 지시 하위 시스템의 설계 지침.

## 상충·열린 질문

- 적정 길이가 강의마다 조금 다르다. [[lectures/lecture-02]]는 "100줄 안팎", [[lectures/lecture-03]]은 진입 파일 "50~100줄", [[lectures/lecture-04]]는 "50~200줄"이라고 한다. 셋 다 "짧게"라는 방향은 같고, 정확한 상한은 작업으로 검증하라는 것이 lecture-04의 결론이다.
