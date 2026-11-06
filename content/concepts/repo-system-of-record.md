---
type: "concept"
title: "저장소가 기록의 원천(System of Record)"
aliases:
  - "저장소는 단일 진실 원천"
  - "repo is the spec"
  - "SoR"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-03-why-the-repository-must-become-the-system-of-record/index.md"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/verification-commands]]"
tags:
  - "type/concept"
  - "topic/system-of-record"
updated: "2026-11-06"
---

# 저장소가 기록의 원천(System of Record)

## 왕초보 설명

- **이게 뭐야?** "AI가 알아야 할 건 전부 프로젝트 폴더 안 파일에 적는다"는 원칙이다.
- **왜 필요해?** AI는 슬랙도, 회의록도, 내 머릿속도 못 본다. 폴더 밖 정보는 AI에게 없는 것과 같다.
- **나는 뭘 하면 돼?** "이건 다들 아는 건데…" 싶은 규칙(쓰는 라이브러리 버전, 꼭 지켜야 할 보안 규칙, 실행 명령)을 `AGENTS.md`나 모듈 옆 문서에 한 줄씩 적는다. 코드를 바꾸면 그 문서도 같이 고친다.

## 정의

프로젝트 결정, 아키텍처 제약, 실행 상태, 검증 기준의 **권위 있는 출처를 코드 저장소 하나로 정하는 것**. 저장소가 최종 발언권을 갖고, 저장소 밖 지식은 에이전트 작업에서 인정되지 않는다. OpenAI는 이를 "저장소가 명세다(repo is the spec)"로 표현한다.

## 왜 중요한가

- 에이전트 입력은 프롬프트, 저장소 파일, 도구 출력뿐이다. 저장소 밖 규칙은 "따르기 싫어서"가 아니라 "존재를 몰라서" 어긴다.
- 영속적 상태는 장기 작업 연속성의 필요 조건이다(Anthropic). 세션 간에 복구 가능한 지식의 양이 성공률을 좌우한다.
- 정보가 흩어져 있으면 매 새 세션이 같은 탐색(발견 비용)을 반복한다.

## 등장하는 곳

- [[lectures/lecture-02]]: 핵심 개념 "저장소는 단일 진실 원천"으로 짧게 소개.
- [[lectures/lecture-03]]: 이 원칙을 콜드 스타트 테스트, 지식 가시성 격차, 좋은 지도의 네 원칙, ACID 유추로 펼친다.
- [[projects/project-02]]: 두 세션 실습. `session-handoff.md`와 `feature_list.json`으로 새 세션이 저장소만 보고 이어받게 한다.
- [[harness/gajae-code]]: 인터뷰 명세, 계획 단계 기록, 목표 목록과 증거 일지를 모두 `.gjc/_session-<id>/` 아래 파일로 남긴다.
- [[harness/oh-my-claudecode]]: `prd.json`, `progress.txt`, `.omc/plans/` 등 상태를 파일에 두고, 압축 직전에도 저장하는 훅이 있다.

## 관련 개념

- [[concepts/map-not-manual]]: 저장소에 둔 지식을 진입 파일 + 주제 문서로 배치하는 방법.
- [[concepts/harness-subsystems]]: 지시·상태 하위 시스템을 저장소 파일로 구현하는 원칙.
- [[concepts/verification-commands]]: "어떻게 검증하는가"의 답도 저장소에 적혀 있어야 한다.

## 상충·열린 질문

없음.
