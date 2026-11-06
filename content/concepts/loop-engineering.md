---
type: "concept"
title: "루프 엔지니어링"
aliases:
  - "Loop engineering"
  - "목표 루프"
  - "/goal"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-07-loop-engineering-first-loop/index.md"
related:
  - "[[concepts/ralph-loop]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/clean-state]]"
tags:
  - "type/concept"
  - "topic/loop-engineering"
updated: "2026-11-06"
---

# 루프 엔지니어링

## 왕초보 설명

- **이게 뭐야?** AI에게 내가 한 줄씩 지시하는 대신, **AI에게 지시하는 시스템(루프)을 설계**하는 일이다.
- **비유:** 식물에 매일 물 주기 vs 자동 급수기 설치. 급수기에는 물 양(목표), 흙 습도 센서(확인), 물통이 비면 멈춤(중단 조건)을 정해 둔다.
- **나는 뭘 하면 돼?** 가장 작은 루프부터: 목표 + 기계로 확인할 수 있는 검증 방법 + 중단 조건을 적고 `/goal`(또는 하네스의 같은 기능)로 돌린다. 끝났는지는 다른 AI나 테스트가 판정하게 하고, 진행은 마크다운 파일에 남긴다.

## 정의

수동 단계별 입력을 대체해 **에이전트에게 자동으로 프롬프트하는 시스템을 설계하는 것**(Addy Osmani: "에이전트에게 프롬프트하는 사람으로서의 자신을 대체하는 것"). 사람은 루프 안에서 밖으로 나가고, 레버리지는 "좋은 프롬프트"에서 "좋은 루프 설계"로 옮겨 간다. 최소 단위는 **목표 + 검증 방법 + 중단 조건**이다.

## 왜 중요한가

- 하네스는 한 번의 실행을 믿을 만하게 하고, 루프는 그 실행을 사람 없이 이어 간다. 루프는 하네스를 대체하지 않고 그 위에 쌓인다.
- 대신 위험도 가속한다: 검증 부채, 이해 부패, 인지 항복, 토큰 폭발.
- 루프의 신뢰성은 중단 판단을 작업자 밖에 두는 데([[concepts/maker-checker-separation]]) 달려 있다.

## 등장하는 곳

- [[lectures/lecture-13]]: 루프 유형(턴·목표·시간·이벤트), `/goal` vs `/loop`, 여섯 프리미티브, Karpathy autoresearch 래칫, 네 조용한 비용, 첫 루프 5단계, 성숙도 사다리.
- [[projects/project-07]]: 목표 루프(`goal.md`) → 타이머 루프(`/loop`) → 메이커-체커 루프(`loop-state.md`, 최소 5라운드) 세 실험.
- 하네스 구현: [[concepts/ralph-loop]](OMC `ralph`, gjc `ultragoal`).

## 관련 개념

- [[concepts/ralph-loop]]: 목표 기반 루프를 하네스 기능으로 내장한 형태.
- [[concepts/maker-checker-separation]]: 루프의 핵심 신뢰성 장치.
- [[concepts/continuity-artifacts]]: 루프의 외부 상태(메모리).
- [[concepts/clean-state]]: 각 라운드가 클린 상태를 남겨야 다음 라운드가 자동으로 시작된다.
- [[concepts/graph-engineering]]: 루프가 분업·병렬·롤백·인계를 필요로 하면 그래프가 된다. 루프는 그래프의 한 노드.

## 상충·열린 질문

- 중단 조건으로 강의는 "독립 평가자 확인 또는 최대 턴"을, project-07 실험 3은 "N회 연속 통과 또는 최대 라운드"를 쓴다. N을 얼마로 할지는 정하지 않는다.
- [[projects/project-07]]의 참고 구현(solution)이 저장소에 없어, 강의가 권하는 루프 구조의 실제 예는 이 강의 저장소 안에서는 확인할 수 없다.
