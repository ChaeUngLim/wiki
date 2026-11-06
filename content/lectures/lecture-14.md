---
type: "lecture"
title: "14강. 단일 루프에서 그래프 엔지니어링으로"
aliases:
  - "단일 루프에서 그래프 엔지니어링으로"
  - "From Single Loops to Graph Engineering"
  - "그래프 엔지니어링"
  - "Graph Engineering"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-14-graph-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-14-graph-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-14-graph-engineering/code/maker_checker_graph.py"
  - "raw/oh-my-claudecode/skills/team/SKILL.md"
related:
  - "[[concepts/graph-engineering]]"
  - "[[concepts/loop-engineering]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/lecture"
  - "topic/graph-engineering"
updated: "2026-11-06"
---

# 14강. 단일 루프에서 그래프 엔지니어링으로

## 왕초보 설명

- **이게 뭐야?** AI 하나가 혼자 "만들고 → 검사하고 → 고치는" 루프를 돌리다 보면, 누가 먼저 할지, 실패하면 어디로 돌아갈지 같은 결정이 AI 머릿속에 숨는다. 그 숨은 구조를 **종이에 그려 내는 것**(노드 = 할 일, 엣지 = 다음 단계로 넘기는 선)이 그래프 엔지니어링이다.
- **비유:** 혼자 하는 요리 vs 식당 주방 동선표. 주방에선 "재료 손질 → 조리 → 검식 → 서빙, 검식에서 짜면 조리로 되돌림"을 벽에 붙여 둔다.
- **꼭 필요해?** 아니다. 단계가 많기만 한 일은 순서대로 도는 스크립트면 충분하다. **갈림길·되돌아가기·사람 승인**이 있을 때 그린다.
- **나는 뭘 하면 돼?** 지금 쓰는 루프를 `graph.md`에 노드 표 + 화살표(mermaid) + "통과면 병합, 실패면 구현으로" 같은 규칙으로 적어 본다. 검사 노드는 만든 AI의 대화 기록을 보지 못하게 새 대화에서 돌린다.

## 핵심 요약

- **4층 겹침**(Rohit): Prompt(지시) → Context(정보) → Loop(런타임) → Graph(시스템). 다음 층은 이전 층을 대체하지 않고 자기 노드 안에 담는다. 그래프의 각 노드는 자신의 prompt·context·도구·메모리·루프를 가진다. 이 코스는 하네스를 이 모두의 **기반**으로 둔다(커뮤니티에서는 하네스 위치에 합의가 없다고 강의가 밝힌다).
- **그래프의 네 부품**: 노드(작업 단위: 결정적 코드, 모델 호출, 도구, 완전한 agent) / 엣지(병렬·조건·실패 재시도·롤백) / 공유 상태(공용 작업대) / 라우팅 규칙(다음에 어디로).
- **루프는 결정을 미루고, 그래프는 결정을 앞당긴다.** "루프는 문제를 루프 안에 숨기고, 그래프는 문제를 종이 위에 올려놓는다." 탐색엔 루프, 프로덕션엔 그래프.
- **Graph ≠ Workflow**: 뼈대(노드+엣지+상태+라우팅)는 같지만, 그래프의 노드는 **agent**일 수 있고 엣지는 동적 라우팅일 수 있다. workflow는 완전 결정적인 그래프의 특수한 경우다. 유일하게 새로운 것은 "노드가 함수에서 agent로 바뀐 것".
- **은총알이 아니다**: 형태보다 재생·관측·복구 가능성이 진짜 하중(iii.dev). 그리고 **오케스트레이션 세금**: agent를 켜는 건 싸고 결과를 검토해 끄는 건 비싸다. "당신은 AI agent들의 GIL이다"(Osmani).

## 핵심 개념

**단일 루프의 세 가지 구조적 실패**(eigent.ai). 루프 안 체크포인트로 못 고치는 이유는 "판단하는 자와 실행되는 자가 같은 두뇌·같은 컨텍스트"이기 때문이다.

| 실패 | 뜻 | 예 |
| --- | --- | --- |
| Goodhart | 숫자는 오르는데 실제 결과는 나빠짐 | 티켓 해결율 루프 → bot이 미해결 티켓을 "해결됨"으로, 몇 달 뒤 churn 2배 |
| 위쪽 실명 | "이 목표가 맞나"를 묻지 않음 | 온도 조절기는 68°F가 맞는지 묻지 않음 |
| 충돌 | 독립 루프끼리 서로 깎아냄 | 응답 속도 루프 vs 품질 루프 |

그래프 설계 전 **네 질문**: 어떤 루프가 어떤 루프에 먹이를 주나 / 누가 목표를 소유하나 / 누가 거부·롤백할 수 있나 / 어떤 지표는 움직이고 어떤 것은 얼어 있어야 하나. 그리고 **앵커** `후보:anchors`: 루프 네트워크를 현실(실제 비즈니스 결과, ground truth, 수동 샘플링)에 고정하는 장치. 가장 쉽게 건너뛰지만 생략할 수 없다.

**첫 그래프 6단계**

1. 공유 상태 정의: 그래프 층에서 공유되는 건 상태뿐, 노드 컨텍스트는 사유물. 필드마다 병합 방식(덮어쓰기·추가·합산)을 `graph.md`에 적는다.
2. 노드 나열: research(agent) / implement(agent, 내부에 최대 3회 루프) / verify(agent, **fresh context**, 구현자 기억을 상속하지 않음) / merge(결정적 코드).
3. 엣지 연결: research → implement → verify → merge → 끝.
4. 라우팅 규칙(가장 중요): verify에서 pass면 merge, fail이면 implement.
5. 체크포인트: 매 단계 상태를 디스크에 저장, `graph.pause_before("merge")`로 병합 전 사람 승인.
6. 실행: thread id를 넘겨 실행 인스턴스를 구분. `graph.md`(청사진)와 코드가 일대일로 맞아야 한다.

**언제 그래프를 쓰나**(다섯 기준 중 셋 이상): 독립 분할 가능 / 분기·롤백 경로 존재 / 중간 상태 저장 가치 / 결과를 자동 검수 가능 / 협력 이득 > 조정 비용. **"복잡" ≠ "단계가 많다"**: 20단계 선형 파이프라인은 스크립트, 노드 5개라도 롤백·병렬·승인이 있으면 그래프.

**이미 그래프를 그리고 있었다**: LangGraph(2024년 1월 출시, 강의에 따르면 2026년 7월 월 약 6,500만 다운로드), Anthropic "Building Effective Agents"(2024-12)의 다섯 패턴(프롬프트 체인, 라우팅, 병렬화, 오케스트레이터/워커, 평가자/최적화자), Claude Code subagent fan-out, 상태 머신·DAG.

**강의 자신의 사실 확인**: "Graph Engineering"은 2026-07-18 Steinberger 트윗과 Hamel Husain 글(둘 다 농담)로 유행했다. "정확도 +18%, 비용 -85%"는 화학 배관 도면(P&ID) 논문의 서로 다른 기준 두 숫자를 이어 붙인 가짜이고, "MS·Stanford·Anthropic 동시 발견"도 가짜다(goddaehee 사실 확인 인용). 이 위키는 이 외부 사실 확인 자체도 검증하지 않았다.

**참조 구현** `code/maker_checker_graph.py`(en에만 있음, LangGraph): `StateGraph`, `add_conditional_edges("verify", route_after_verify, …)`, `MemorySaver` 체크포인터, `thread_id`. 코드를 읽어 보면 강의 설명과 다른 점이 있다.

- 모델 호출 `call_model`은 `NotImplementedError`를 던지는 자리표시자이고, `tests_pass`는 `"def test" in code`라는 자리표시자다(파일 머리말에 명시).
- `attempts` 필드는 `operator.add` 병합으로 선언됐지만 어느 노드도 값을 더하지 않는다. 강의의 "implement 노드 내부 최대 3회 루프"도 없고, verify→implement 엣지에 횟수 상한이 없다.
- 강의 5단계의 `pause_before("merge")`(사람 승인)는 코드에 없다(LangGraph라면 `interrupt_before`에 해당).

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-13]]: 지난 강의의 루프(목표·검증·중단 조건)는 그래프의 **한 노드의 내부 구조**가 된다([[concepts/loop-engineering]]).
- [[lectures/lecture-09]]: 검증 노드가 구현 노드와 독립이어야 하는 이유. 그래프에서는 프롬프트 문제가 아니라 구조 문제다([[concepts/maker-checker-separation]]).
- [[lectures/lecture-11]]: 관찰할 수 없는 그래프는 블랙박스를 더 큰 블랙박스로 조립한 것.
- [[lectures/lecture-02]]: 하네스를 기반으로 정한 강의.
- 실습: [[projects/project-08]].
- 하네스 실례: [[harness/oh-my-claudecode]]의 `team`은 `team-plan → team-prd → team-exec → team-verify → team-fix` 단계 파이프라인이고, [[harness/gajae-code]]는 deep-interview → ralplan → ultragoal 핸드오프 체인과 `architect`·`critic`·`executor` 역할 레인을 둔다. 둘 다 노드·엣지를 문서와 코드로 고정한 그래프 실례로 읽을 수 있다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-14-graph-engineering/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-14-graph-engineering/index.md`
- 코드: `raw/learn-harness-engineering/docs/en/lectures/lecture-14-graph-engineering/code/maker_checker_graph.py`. ko 강의는 "이 장의 코드 예제: code/"로 링크하지만 ko 디렉터리에는 `code/`가 없다.
- 긴 소스(ko 약 45KB, en 약 37KB). 외부 날짜·수치(2026-07 트윗, LangGraph 다운로드 등)는 강의 주장이고 이 위키는 검증하지 않았다.
