---
type: "concept"
title: "점진적 공개"
aliases:
  - "Progressive disclosure"
  - "필요할 때 로드"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-04-why-one-giant-instruction-file-fails/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/pi/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/pi/index.md"
related:
  - "[[concepts/map-not-manual]]"
  - "[[harness/claude-code]]"
  - "[[harness/pi]]"
tags:
  - "type/concept"
  - "topic/instructions"
  - "topic/tools-extensibility"
updated: "2026-11-06"
---

# 점진적 공개

## 왕초보 설명

- **이게 뭐야?** AI에게 모든 설명을 처음부터 다 주지 않고, **개요만 먼저 주고 필요해지는 순간에 자세한 내용을 연다**는 원칙이다.
- **비유:** 식당 메뉴판. 처음엔 메뉴 이름만 보이고, 궁금한 요리를 고르면 그때 재료·조리법 설명을 본다.
- **나는 뭘 하면 돼?** `AGENTS.md`(또는 `CLAUDE.md`)에는 "어디에 무엇이 있다"만 적고, 긴 절차는 스킬(`SKILL.md`)이나 `docs/` 문서로 나눈다. 스킬 설명 첫 줄은 "언제 쓰는지"가 드러나게 써야 AI가 제때 꺼낸다.

## 정의

지시·지식을 **개요(항상 로드)**와 **상세(필요할 때 로드)**로 나눠, 상세는 관련 작업이 시작되거나 트리거 조건이 맞을 때만 컨텍스트에 넣는 설계.

## 왜 중요한가

- 컨텍스트 창은 유한하고, 관계없는 지시가 많을수록 중요한 지시가 묻힌다([[lectures/lecture-04]]의 거대 지시 파일 문제).
- 비용: 컨텍스트에 token이 하나 늘 때마다 추론마다 비용을 낸다. 필요할 때만 로드하면 프롬프트 캐시도 깨지 않는다([[harness/pi]] 분석).
- 지시 파일이 짧아져 유지·검사가 쉬워진다([[concepts/map-not-manual]]).

## 등장하는 곳

- [[lectures/lecture-04]]: 진입 파일은 라우터, 상세는 주제 문서로 나눠 필요할 때만 읽힌다. "좋은 UI가 옵션을 한 번에 쏟아내지 않는 것과 같다."
- [[harness/claude-code]]: 스킬(`SKILL.md`)은 트리거 단어에 따라 자동 로드되고 점진적 공개를 쓴다. 하위 디렉터리 `CLAUDE.md`도 그 디렉터리 파일을 읽을 때만 로드된다.
- [[harness/pi]]: 스킬은 "self-contained capability packages that the agent loads on-demand"이고, 트리거될 때만 상세가 컨텍스트에 들어간다. 시스템 프롬프트를 극도로 단순하게 두는 이유.
- 하네스 실례: [[harness/gajae-code]]는 긴 단계 규칙을 별도 fragment 파일로 빼서 그 단계에서만 읽게 한다([[lectures/lecture-04]] 페이지 참고).

## 관련 개념

- [[concepts/map-not-manual]]: 진입 파일을 지도로 두는 원칙. 점진적 공개는 그 지도가 가리키는 상세를 "언제" 여는지에 대한 원칙이다.

## 상충·열린 질문

없음.
