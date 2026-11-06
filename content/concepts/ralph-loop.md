---
type: "concept"
title: "Ralph 루프(랄프톤식 자동 실행)"
aliases:
  - "랄프 루프"
  - "랄프톤"
  - "Ralph-thon"
  - "지속 모드"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/oh-my-claudecode/README.ko.md"
  - "raw/oh-my-claudecode/README.md"
  - "raw/gajae-code/README.ko.md"
  - "raw/gajae-code/README.md"
  - "raw/oh-my-claudecode/skills/ralph/SKILL.md"
  - "raw/gajae-code/packages/coding-agent/src/defaults/gjc/skills/ultragoal/SKILL.md"
  - "raw/gjc-beginner-guide.html"
related:
  - "[[concepts/verification-commands]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/concept"
  - "topic/loop-engineering"
  - "topic/verification"
updated: "2026-11-06"
---

# Ralph 루프(랄프톤식 자동 실행)

## 왕초보 설명

- **이게 뭐야?** AI에게 목표 하나를 주고, **끝났다는 증거가 나올 때까지 알아서 계속 돌게** 하는 방식이다. 만들기 → 확인하기 → 틀린 곳 고치기 → 다시 확인하기를 사람 없이 반복한다.
- **랄프톤은?** 이 Ralph 루프를 쓰는 해커톤을 "랄프톤(Ralph-thon)"이라고 부른다(사용자 가이드 기준). 사람은 처음에 무엇을 만들지 정하고, 나머지는 루프가 밤새 돌린다.
- **왜 필요해?** AI는 일을 반쯤 하고 "다 했어요"라며 멈추는 버릇이 있다. 루프가 "증거 없으면 못 멈춘다"를 강제한다.
- **조심할 점?** 목표가 흐릿하면 엉뚱한 방향으로 끝까지 달린다. 그래서 두 하네스 모두 루프 **앞에** 질문(deep-interview)과 계획 검토(ralplan)를 둔다. 원본 코드는 분리된 작업 폴더(worktree)에서 돌린다.

## 정의

에이전트가 스스로 멈추는 판단을 믿지 않고, **하네스가 완료 조건을 판정**해 조건이 충족될 때까지 같은 목표로 반복 실행시키는 구조. 세 부품으로 이루어진다.

1. **완료 조건**: 확인 가능한 기준 목록(이야기·목표별 완료 기준).
2. **계속 돌리는 장치**: 턴이 끝나도 다시 일을 시키는 메커니즘.
3. **판정**: 기계 검증([[concepts/verification-commands]])과 별도 리뷰어([[concepts/maker-checker-separation]]).

## 왜 중요한가

- 긴 작업에서 "이른 완료 선언"을 막는다. 사람이 자리를 비워도 진행된다.
- 반대로 판정이 약하면 루프는 같은 실수를 수백 번 반복하는 비싼 기계가 된다. 그래서 상한(최대 횟수)과 "사람만 풀 수 있는 막힘"의 구분이 함께 필요하다.

## 등장하는 곳

| | [[harness/oh-my-claudecode]] | [[harness/gajae-code]] |
| --- | --- | --- |
| 이름 | `ralph`(지속 모드), `autopilot`이 내부에서 사용 | `ralph` 명령 없음. `ultragoal` + goal 모드로 같은 목적 |
| 완료 조건 | `prd.json`의 이야기별 완료 기준(`passes: true`) | `goals.json`의 목표별 objective + checkpoint 증거 |
| 계속 돌리는 장치 | Stop 훅 `persistent-mode`가 턴 종료를 차단(`decision: "block"`) | goal 모드가 턴 종료 뒤 다음 작업으로 이어 줌 |
| 상한 | 기본 500회(strict 200회) | 목표당 `nudgeBudget` 기본 10회 |
| 판정 | `omc ralph verify` 기준선 비교 + `architect`(기본) 리뷰 | quality-gate + 마지막 경계 검토 + 종료 `critic` `OKAY` |
| 자동 시작 | 키워드(`ralph`, "don't stop" 등)로 바로 | `gjc.ralplan.autoHandoff = ultragoal`이면 계획 확정 뒤 질문 없이 |

코스 쪽에서는 [[lectures/lecture-13]]이 같은 구조를 "목표 기반 루프"(`/goal` = 목표 + 검증 방법 + 중단 조건, 완료 판단은 독립 심판)로 일반화해 설명하고, [[projects/project-07]] 실험 1·3이 직접 만들어 보게 한다([[concepts/loop-engineering]]).

## 관련 개념

- [[concepts/loop-engineering]]: Ralph 루프는 루프 엔지니어링의 "목표 기반 루프"를 하네스에 내장한 형태다.
- [[concepts/verification-commands]]: 루프가 "끝났다"를 판정하는 기계 근거.
- [[concepts/maker-checker-separation]]: 루프 마지막의 독립 리뷰.
- [[concepts/harness-subsystems]]: 루프는 상태(진행 파일)와 피드백(검증)을 묶어 돌리는 장치다.

## 상충·열린 질문

없음.
