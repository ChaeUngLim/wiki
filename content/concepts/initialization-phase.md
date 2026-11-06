---
type: "concept"
title: "초기화 단계와 부트스트랩 계약"
aliases:
  - "초기화 단계"
  - "부트스트랩 계약"
  - "init.sh"
  - "initialization phase"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/learn-harness-engineering/projects/project-01/solution/init.sh"
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
related:
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/verification-commands]]"
tags:
  - "type/concept"
  - "topic/initialization"
updated: "2026-11-06"
---

# 초기화 단계와 부트스트랩 계약

## 왕초보 설명

- **이게 뭐야?** 기능을 만들기 전에 "일할 준비가 됐는지"부터 확인하는 단계다. 프로젝트에선 보통 `init.sh` 같은 스크립트 하나로 설치 → 검사 → 빌드를 돌린다.
- **왜 필요해?** 준비가 안 된 채 일을 시작하면 AI는 기능을 만들다가 설치 문제, 테스트 설정 문제에 계속 걸려 시간을 버린다.
- **나는 뭘 하면 돼?** 새 대화를 시작할 때마다 `bash init.sh`(또는 `make check`)를 먼저 돌리라고 `AGENTS.md`에 적는다. 새 프로젝트라면 첫 대화는 이 스크립트와 할 일 목록 만들기만 시킨다.

## 정의

기능 구현 없이, 이후 모든 구현의 전제 조건만 확립하는 에이전트 생명주기의 첫 단계. 완료 판정 기준은 **부트스트랩 계약** 네 조건이다: 시작할 수 있다, 테스트할 수 있다, 진행 상황을 볼 수 있다, 다음 단계를 안다. 매 세션 시작에 돌리는 `init.sh` 같은 점검 스크립트가 이 계약을 기계로 확인한다.

## 왜 중요한가

- 초기화와 구현을 섞으면 에이전트는 보이는 기능 코드를 택하고 인프라를 희생한다. 그 결함은 다음 세션에서 드러난다.
- 첫 번째 검증까지의 시간과 이후 세션의 재구축 비용을 줄인다.
- 초기화 때 내린 선택(테스트 프레임워크, 디렉터리 구성)을 기록해 이후 세션의 상충 선택을 막는다.

## 등장하는 곳

- [[lectures/lecture-06]]: 초기화를 전용 첫 세션으로 분리하고, 부트스트랩 계약·작업 분해·인수 체크리스트·웜 스타트를 제시한다.
- [[projects/project-01]]: solution `init.sh`가 `npm install` → `npm run check` → `npm run build`로 "작업 전에 깨끗하게 빌드되는지"를 확인하고, `AGENTS.md`의 시작 규칙이 이를 실행하게 한다. en 원문은 lecture-06을 관련 강의로 든다.

- [[projects/project-03]]: solution에 같은 `init.sh`(`set -euo pipefail`)를 다시 두고, `AGENTS.md` 시작 규칙 4단계에서 `npm install && npm run check`를 요구한다.

## 관련 개념

- [[concepts/continuity-artifacts]]: 초기화가 처음 만들어 두는 인계 체계(진행 파일, 작업 목록).
- [[concepts/verification-commands]]: `init.sh` 안에서 돌리는 검사 명령.
- [[concepts/harness-subsystems]]: 초기화 스크립트는 환경과 피드백 하위 시스템에 함께 걸친다.

## 상충·열린 질문

없음.
