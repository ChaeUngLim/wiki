---
type: "project"
title: "프로젝트 05. 에이전트가 자신의 작업을 검증하도록 만들기"
aliases:
  - "에이전트가 자신의 작업을 검증하도록 만들기"
  - "Make the Agent Verify Its Own Work"
  - "생성자·평가자·플래너 역할 분리 실습"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-05-grounded-qa-verification/index.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/single-role/evaluator-rubric.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/gen-eval/evaluator-rubric.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/plan-gen-eval/evaluator-rubric.md"
  - "raw/learn-harness-engineering/projects/project-05/solution/plan-gen-eval/sprint-contract.md"
related:
  - "[[lectures/lecture-09]]"
  - "[[lectures/lecture-10]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/definition-of-done]]"
  - "[[concepts/sprint-contract]]"
  - "[[concepts/evaluator-rubric]]"
tags:
  - "type/project"
  - "topic/role-separation"
  - "topic/verification"
updated: "2026-11-06"
---

# 프로젝트 05. 에이전트가 자신의 작업을 검증하도록 만들기

## 왕초보 설명

- **이게 뭐야?** 같은 기능(질문·답변 대화 기록 화면)을 **세 가지 팀 구성**으로 만들어 비교하는 실습이다. ① AI 혼자 계획·구현·자기 채점 ② 만드는 AI + 검사하는 AI ③ 계획하는 AI + 만드는 AI + 검사하는 AI.
- **비유:** 혼자 요리하고 혼자 맛보는 사람 vs 요리사와 시식 담당 vs 메뉴 기획자까지 있는 주방.
- **결과(체크인된 기록):** 평가표 점수가 1.6 → 3.3 → 4.9(5점 만점)로 올랐다. 역할을 하나씩 더할 때마다 좋아졌다.
- **나는 뭘 하면 돼?** 큰 기능은 ① 먼저 "완료 기준과 범위"를 정하는 계획 단계(스프린트 계약)를 두고 ② 만든 뒤에는 **다른** AI(또는 새 대화)에게 그 기준으로 채점하게 하고 ③ 지적받은 걸 고쳐 다시 채점받는다.

## 목표

- 역할 분리를 구현한다: 구현하는 **생성자(generator)**, 검토하는 **평가자(evaluator)**, 선택적으로 **플래너(planner)**.
- 세 번 실행해 역할이 하나 추가될 때마다의 효과를 잰다. 기능은 세 실행에서 같게 둔다(체크인된 기능은 멀티턴 Q&A 대화 기록, `ConversationHistory` 컴포넌트).
- 이유(ko 원문): 구현과 검증을 한 에이전트가 하면 자기 오류를 찾기 어렵다. 독립 평가자는 환각과 섣부른 완료 선언을 막는다.
- 하네스 메커니즘: 자기 검증 + 근거 기반 Q&A + 증거 기반 완료.
- 도구: Claude Code 또는 Codex, Git, Node.js + Electron.

## 연결 강의

- [[lectures/lecture-09]]: 조기 완료 선언과 작업자·검사자 분리([[concepts/maker-checker-separation]]). 세 변형이 그 분리의 효과를 직접 보여 준다.
- [[lectures/lecture-10]]: 엔드투엔드 검증. 단, 이 실습의 평가는 rubric 점수이고 E2E 테스트 결과는 산출물에 없다.
- [[lectures/lecture-11]]: 스프린트 계약과 평가자 루브릭을 프로세스 관측 가능성의 도구로 정의한다(이 실습의 ko·en 원문은 관련 강의로 09·10만 든다).
- [[lectures/lecture-02]]: Anthropic의 플래너 + 생성기 + 평가기 3에이전트 구조를 처음 소개했다.

## starter→solution 핵심 변경점

solution이 하나가 아니라 **세 변형 디렉터리**다. starter는 project-04 기반의 앱(대화 기록 업그레이드 전)이다.

점수는 각 변형의 [[concepts/evaluator-rubric|평가자 루브릭]] `evaluator-rubric.md`에서 가져왔다.

| 변형 | 역할 | 평가자 | 점수 | 수정 기록 | `ConversationHistory.tsx` |
| --- | --- | --- | --- | --- | --- |
| `solution/single-role/` | 한 에이전트가 계획·구현·자기 평가 | "Self" | 1.6/5 | 없음 | 45줄 |
| `solution/gen-eval/` | 생성자 + 평가자 | 별도 평가 에이전트(생성 후 리뷰) | 3.3/5 | 2.8 → 3.1 → 3.3 | 111줄 |
| `solution/plan-gen-eval/` | 플래너 + 생성자 + 평가자 | 수용 기준·rubric 대비 리뷰 | 4.9/5 | 4.2 → 4.6 → 4.8 → 4.9 | 277줄 |

**기능 비교** (plan-gen-eval rubric의 표)

| 기능 | Single-Role | Gen-Eval | Plan-Gen-Eval |
| --- | --- | --- | --- |
| 채팅 말풍선 | 없음 | 있음 | 있음 |
| 타임스탬프 | 전체 | 간결 | 간결 + 날짜 구분선 |
| 인용 | 없음 | 개수만 | 펼칠 수 있는 전체 인용 |
| 복사·후속 질문·신뢰도 표시 | 없음 | 없음 | 있음 |
| 빈 상태 | 기본 | 아이콘 | 안내 포함 |

**[[concepts/sprint-contract|스프린트 계약]]** (`plan-gen-eval/sprint-contract.md`, 플래너 산출물): 범위 안(말풍선, 타임스탬프·날짜 구분선, 인용 펼치기, 복사, 후속 질문 제안, 빈 상태 안내)과 범위 밖(재시작 간 대화 유지, 대화 검색·필터, 내보내기)을 먼저 나누고, 역할 표와 번호 붙은 수용 기준(예: "사용자 메시지는 오른쪽 정렬 보라색")을 둔다. 구현 전에 [[concepts/definition-of-done|완료 기준]]을 고정하는 장치다.

각 변형의 `AGENTS.md`는 80줄대로 비슷하고, plan-gen-eval만 `sprint-contract.md`와 `evaluator-rubric.md`를 읽으라는 시작 단계가 더 있다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-05-grounded-qa-verification/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-05-grounded-qa-verification/index.md`
- 코드: `raw/learn-harness-engineering/projects/project-05/{starter,solution/single-role,solution/gen-eval,solution/plan-gen-eval}`

**원문 간 차이·해석 주의**

- en 표는 각 변형의 점수(1.6/5, 3.3/5, 4.9/5)를 직접 적고, ko 표는 "점수와 결함", "높은 평가 증거"라고만 쓴다. en만 "체크인된 기능은 멀티턴 Q&A 대화 기록"이라고 명시한다.
- 디렉터리 이름은 `grounded-qa-verification`이지만 실제 변경 대상은 대화 기록 UI다.
- 세 점수는 **서로 다른 평가자**가 매겼다(single-role은 자기 평가, 나머지는 별도 평가자). 같은 채점자가 같은 rubric으로 세 결과를 채점한 대조 실험이 아니므로, 점수 차이를 역할 분리만의 효과로 단정할 수 없다. 기능 비교표가 더 직접적인 증거다.
