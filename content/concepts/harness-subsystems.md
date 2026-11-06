---
type: "concept"
title: "하네스 5개 하위 시스템"
aliases:
  - "5개 하위 시스템 하네스 모델"
  - "지시·도구·환경·상태·피드백"
  - "harness subsystems"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/oh-my-claudecode/README.ko.md"
  - "raw/oh-my-claudecode/README.md"
  - "raw/gajae-code/README.ko.md"
  - "raw/gajae-code/README.md"
related:
  - "[[concepts/verification-commands]]"
tags:
  - "type/concept"
  - "topic/harness-model"
updated: "2026-11-06"
---

# 하네스 5개 하위 시스템

## 왕초보 설명

주방에 꼭 있어야 할 구역 다섯 개라고 생각하면 된다.

- **지시** = 레시피: AI가 읽는 규칙 파일. "이 프로젝트는 이렇게 만든다."
- **도구** = 칼: AI가 쓸 수 있는 명령(터미널, 파일 편집).
- **환경** = 가스레인지: 매번 똑같이 돌아가는 실행 환경(설치 목록, 버전 고정).
- **상태** = 메모판: 어디까지 했는지 적어 두는 파일. 다음에 이어서 할 수 있다.
- **피드백** = 맛보기: "제대로 됐나?"를 확인하는 테스트 명령.

AI가 자꾸 실패하면 이 다섯 개 중 어느 칸이 비었는지부터 본다.

## 정의

하네스(모델 가중치 밖의 엔지니어링 인프라 전체)를 다섯 하위 시스템으로 나눈 모델이다: **지시 + 도구 + 환경 + 상태 + 피드백**. 강의는 주방의 기능 구역에 빗댄다.

| 하위 시스템 | 주방 비유 | 담당 | 대표 산출물 |
| --- | --- | --- | --- |
| 지시 | 레시피 선반 | 무엇을, 어떤 규칙으로 할지 | `AGENTS.md`, `CLAUDE.md`, `docs/` |
| 도구 | 칼 랙 | 무엇을 실행할 수 있는지(최소 권한) | 셸, 파일 도구, 테스트 러너 |
| 환경 | 가스레인지 | 재현 가능한 실행 조건 | 의존성 잠금, 런타임 버전 파일, 컨테이너, `init.sh` |
| 상태 | 준비 스테이션 | 세션을 넘는 진행 기록 | `PROGRESS.md`, `feature_list.json`, git 커밋 |
| 피드백 | 품질 확인 창 | 결과가 맞는지 판정 | 테스트·타입 검사·린트·빌드 명령 |

## 왜 중요한가

- 하네스를 "프롬프트 파일" 하나로 보는 오해를 깬다. 지시만 있고 나머지가 없으면 재료만 있는 냉장고다.
- 진단 틀이 된다. 에이전트가 실패하면 어느 하위 시스템이 비었는지 묻는다. [[lectures/lecture-02]]의 연습 1은 하위 시스템마다 1~5점을 매겨 가장 낮은 곳부터 고치게 한다.
- 개선 순서를 정한다. 강의의 사례에서 지시 → 피드백 → 상태를 차례로 더하자 같은 모델의 성공률이 20% → 60% → 80% → 80~100%가 됐다. 그중 피드백이 가장 싼 투자다([[concepts/verification-commands]]).

## 등장하는 곳

- [[lectures/lecture-01]]: 같은 틀을 "고장 진단" 쪽에서 본 다섯 방어 레이어(작업 명세, 컨텍스트 제공, 실행 환경, 검증 피드백, 상태 관리)를 제시한다. 실패는 모델 탓 전에 이 중 하나에 귀속시킨다.
- [[lectures/lecture-02]]: 모델을 정의하고, Claude Code·Cursor·Codex·AutoGPT를 이 틀로 읽는다. 하위 시스템별 가치는 같은 모델을 고정한 제거 실험으로 잰다.
- [[projects/project-01]]: 2차(강한 하네스) 실행에 넣는 파일이 하위 시스템에 대응한다. 지시는 `AGENTS.md`·`CLAUDE.md`·`docs/`, 환경과 피드백은 `init.sh`, 상태는 `feature_list.json`·`claude-progress.md`. 도구는 두 실행에서 같게 두어 변수에서 뺀다.
- [[harness/oh-my-claudecode]], [[harness/gajae-code]]: 코스 밖 실제 하네스를 이 틀로 분해한 표가 각 페이지에 있다.

## 관련 개념

- [[concepts/initialization-phase]]: 환경·피드백에 걸치는 `init.sh`를 별도 단계로 다루는 개념.
- [[concepts/verification-commands]]: 피드백 하위 시스템의 최소 구현.

## 상충·열린 질문

- `init.sh`처럼 한 산출물이 여러 하위 시스템(환경 준비 + 빌드·타입 검사 피드백)에 걸치는 경우가 있다. 강의는 하위 시스템 간 경계를 따로 정의하지 않는다.
