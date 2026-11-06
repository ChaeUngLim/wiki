---
type: "concept"
title: "일하는 쪽과 확인하는 쪽 분리"
aliases:
  - "메이커-체커 분리"
  - "생성기·평가기 분리"
  - "maker-checker separation"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-14-graph-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-14-graph-engineering/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-08-graph-engineering-first-graph/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-08-graph-engineering-first-graph/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-09-why-agents-declare-victory-too-early/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-09-why-agents-declare-victory-too-early/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/gajae-code/README.ko.md"
  - "raw/gajae-code/README.md"
  - "raw/oh-my-claudecode/README.ko.md"
  - "raw/oh-my-claudecode/README.md"
related:
  - "[[concepts/verification-commands]]"
tags:
  - "type/concept"
  - "topic/role-separation"
  - "topic/verification"
updated: "2026-11-06"
---

# 일하는 쪽과 확인하는 쪽 분리

## 왕초보 설명

- **이게 뭐야?** 숙제를 한 학생이 자기 숙제를 채점하지 않게 하는 것이다. AI 하나가 만들고, **다른 AI(또는 다른 역할)**가 확인한다.
- **왜 필요해?** AI는 자기가 만든 결과를 너무 후하게 칭찬한다. 같은 AI에게 "잘했니?"라고 물으면 대부분 "네"라고 답한다.
- **나는 뭘 하면 돼?** 하네스를 고를 때 "확인 담당이 따로 있고, 그 담당은 코드를 고칠 수 없는지"를 본다. 직접 쓸 때는 구현을 끝낸 뒤 새 대화나 다른 역할에게 "이 변경을 비판해 줘"라고 시킨다.

## 정의

결과를 만드는 주체(maker)와 그 결과를 판정하는 주체(checker)를 서로 다른 에이전트·역할·권한으로 나누는 하네스 설계. 판정자는 보통 **읽기 전용**이고, 정해진 기준(완료 기준, 검증 명령 결과)에 비춰 승인·거절만 한다.

## 왜 중요한가

- [[lectures/lecture-02]]: Anthropic은 에이전트가 자기 작업을 자신만만하게 칭찬한다는 것을 발견했고, 해법으로 "일하는 사람"과 "확인하는 사람"을 나눴다.
- 확인 쪽이 고칠 권한까지 가지면 "확인하다가 슬쩍 고쳐서 통과"시키는 일이 생긴다. 권한을 나누면 판정이 독립적인 증거가 된다.
- [[concepts/verification-commands|검증 명령어]]가 기계적 판정이라면, 이 분리는 사람이 아닌 두 번째 눈으로 하는 판정이다. 둘은 함께 쓴다.

## 등장하는 곳

- [[lectures/lecture-02]]: "제약하되 마이크로매니지하지 않는다"의 일부로, 일하는 쪽과 확인하는 쪽의 분리를 든다.
- [[harness/gajae-code]]: 계획 단계에서 PLANNER가 쓰고 ARCHITECT·CRITIC이 검토한다. 세 역할은 읽기 도구와 일부 `git` 조회 명령만 쓸 수 있고, 코드를 고치는 건 `executor`뿐이다. 실행이 끝날 때도 읽기 전용 `critic`의 `OKAY`가 있어야 한다.
- [[harness/oh-my-claudecode]]: `ralph`는 구현이 끝나면 별도 리뷰어(기본 `architect`, 선택 `critic`·`codex`)가 완료 기준에 비춰 검증하고, 거절되면 고친 뒤 같은 리뷰어에게 다시 받는다. `team`은 실행(`team-exec`)과 검증(`team-verify`)을 다른 에이전트에게 맡긴다.

- [[lectures/lecture-09]]: "같은 모델이 생성과 평가를 하면 자기에게 관대하다. 해결책은 작업자와 검사자를 분리하는 것"이라고 명시하고, Anthropic의 플래너+생성기+평가기(Playwright 실제 클릭 테스트) 실험을 근거로 든다.

- [[projects/project-05]]: 같은 기능을 혼자(자기 평가 1.6/5), 생성자+평가자(3.3/5), 플래너+생성자+평가자(4.9/5)로 만든 체크인 기록. 점수는 서로 다른 평가자가 매겼다.
- [[lectures/lecture-13]]: 루프 설계에서 "단일로 가장 중요한 신뢰성 보장 장치". 모델은 자기 출력의 최고 변호사이므로, 다른 지시·때로는 다른 모델의 검증자를 둔다. 커뮤니티 관행 "적대적 검증"(발견마다 N명의 회의론자, 다수 거부 시 폐기)도 소개.
- [[projects/project-07]]: 실험 3에서 메이커·체커·루프 제어 세 프롬프트로 최소 5라운드를 돌린다.
- [[lectures/lecture-14]]·[[projects/project-08]]: 그래프에서 verify 노드는 fresh context(구현자 기억을 상속하지 않음)로 분리한다. 루프 안 자기 검토가 못 잡는 구조적 실패를 독립 노드로 옮긴다.

## 관련 개념

- [[concepts/sprint-contract]]·[[concepts/evaluator-rubric]]: 검사자가 무엇을, 어떤 기준으로 볼지 파일로 고정하는 도구.

- [[concepts/verification-commands]]: 기계 판정. 이 개념의 확인 쪽이 근거로 삼는다.
- [[concepts/harness-subsystems]]: 피드백 하위 시스템을 사람·에이전트 역할 쪽에서 구현한 것.

## 상충·열린 질문

없음.
