---
type: "concept"
title: "그래프 엔지니어링"
aliases:
  - "Graph engineering"
  - "graph.md"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-14-graph-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-14-graph-engineering/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-08-graph-engineering-first-graph/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-08-graph-engineering-first-graph/index.md"
related:
  - "[[concepts/loop-engineering]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/concept"
  - "topic/graph-engineering"
updated: "2026-11-06"
---

# 그래프 엔지니어링

## 왕초보 설명

- **이게 뭐야?** 여러 AI·루프·도구·검사자가 함께 일할 때, **누가 무엇을 하고(노드), 다음은 어디로 넘기고(엣지), 어떤 메모를 같이 보고(공유 상태), 실패하면 어디로 되돌아가는지(라우팅)**를 그림과 표로 미리 정하는 것이다.
- **비유:** 회사 결재 라인. "작성 → 팀장 검토 → 반려면 작성자에게, 승인이면 부장에게"를 미리 정해 둔다.
- **나는 뭘 하면 돼?** 갈림길이나 되돌아가기가 있는 작업이면 `graph.md`를 쓴다. 단계가 많기만 하면 그냥 순서대로 도는 스크립트로 충분하다.

## 정의

여러 agent, 루프, 도구, 평가자를 **명시적 그래프**(노드 + 엣지 + 공유 상태 + 라우팅 규칙)로 조직해, 작업 단위 간 연결·공유 상태·경로 선택을 설계·관측·부분 수리 가능하게 만드는 실천. 노드는 결정적 코드, 모델 호출, 도구, 또는 자기 루프를 가진 완전한 agent일 수 있다.

## 왜 중요한가

- 단일 루프는 결정(분업·병렬·롤백·인계)을 한 agent의 컨텍스트 안에 숨긴다. 그래프는 그것을 종이 위에 올려 읽고 감사하고 고칠 수 있게 한다.
- 단일 루프의 구조적 실패(Goodhart, 위쪽 실명, 루프 간 충돌)는 루프 안 체크포인트로 못 고친다. 검사를 **독립 노드**로 옮겨야 한다.
- 반대로 남용하면 오케스트레이션 세금(사람의 검토 대역폭)만 늘어난다.

## 등장하는 곳

- [[lectures/lecture-14]]: 4층 겹침(prompt → context → loop → graph), 네 부품, graph ≠ workflow, 첫 그래프 6단계, 다섯 판단 기준(셋 이상), LangGraph 참조 구현.
- [[projects/project-08]]: P07 루프를 `graph.md`로 그리기 → fan-out/fan-in → 문제 층으로의 롤백과 사람 승인 노드.

## 관련 개념

- [[concepts/loop-engineering]]: 루프는 그래프의 한 노드. 그래프는 루프를 대체하지 않고 그 위에 쌓인다.
- [[concepts/maker-checker-separation]]: 그래프에서 verify 노드는 fresh context로 구현 노드와 분리된다. 프롬프트 문제가 아니라 구조 문제.

## 상충·열린 질문

- 하네스의 위치: 이 코스는 하네스를 loop·graph 모두의 기반으로 두지만, 강의 스스로 explainx는 loop 위에, Buildrix 논문은 loop 아래에 둔다고 소개한다. 합의된 정의가 없다.
- [[lectures/lecture-14]]의 6단계 설명(implement 내부 최대 3회 루프, 병합 전 사람 승인)과 참조 코드 `maker_checker_graph.py`(횟수 상한 없음, `attempts` 미증가, 승인 정지 없음)가 다르다.
