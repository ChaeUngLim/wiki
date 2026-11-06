---
type: "project"
title: "프로젝트 08. 내 워크플로우를 그래프로 그리기"
aliases:
  - "내 워크플로우를 그래프로 그리기"
  - "Draw Your Workflow as a Graph"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-08-graph-engineering-first-graph/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-08-graph-engineering-first-graph/index.md"
related:
  - "[[lectures/lecture-14]]"
  - "[[concepts/graph-engineering]]"
  - "[[projects/project-07]]"
tags:
  - "type/project"
  - "topic/graph-engineering"
updated: "2026-11-06"
---

# 프로젝트 08. 내 워크플로우를 그래프로 그리기

## 왕초보 설명

- **이게 뭐야?** 프로젝트 07에서 만든 "만드는 AI ↔ 검사하는 AI" 루프를 **그림(그래프)으로 펼쳐 적는** 실습이다. 그다음 ① 일을 둘로 나눠 동시에 돌리는 갈래(fan-out)와 합치기(fan-in) ② 문제가 생긴 단계로 정확히 되돌아가는 선과 사람 승인 단계를 덧붙인다.
- **비유:** 머릿속으로만 하던 이삿짐 순서를 체크리스트와 동선도로 적는 것. 적어 보면 "아, 여기서 늘 되돌아갔었구나"가 보인다.
- **나는 뭘 하면 돼?** `graph.md`에 노드 표·화살표 그림(mermaid)·"통과면 병합, 실패면 구현으로" 규칙을 적고, 공유 메모 `state.md`에 요구사항·진행·검증 결과를 모은다. 병합 전에는 내가 `state.md`에 "승인"을 써야 넘어가게 한다.

## 목표

- "Loop"에서 "Graph"로 넘어가는 프로젝트. 루프 안에 숨어 있던 구조(노드, 엣지, 공유 상태, 라우팅 규칙)를 명시적으로 적는다.
- 끝나면 체감할 것(원문): 그래프는 새 발명이 아니라 루프가 어느 정도 복잡해지면 스스로 변하는 모습이다.
- 준비: P07 완료 저장소(또는 아무 agent 워크플로)에서 출발, 브랜치 `p08-explicit-graph`·`p08-parallel`·`p08-human-in-the-loop`, 공유 상태 파일 `state.md`.
- 도구: Claude Code 또는 Codex, Git, P07의 maker-checker 루프, mermaid 또는 손으로 쓴 `graph.md`.

## 연결 강의

- [[lectures/lecture-14]]: 그래프의 네 부품, 첫 그래프 6단계, 언제 그래프를 쓰나.
- [[lectures/lecture-13]]: 루프는 그래프의 한 노드다. 이 프로젝트는 그 노드의 내부를 펼친다.
- [[lectures/lecture-09]]: 검증 노드가 구현 노드와 독립이어야 하는 이유.
- [[lectures/lecture-11]]: 그래프가 복잡할수록 각 노드가 무엇을 하는지 보여야 한다.

## 핵심 실습 내용(문서 기반)

| 실험 | 브랜치 | 할 일 | 측정 |
| --- | --- | --- | --- |
| 1 명시적 그래프 | `p08-explicit-graph` | P07 루프의 단계마다 노드(책임·입력·출력·agent/결정적 코드), 모든 엣지(조건 엣지, 롤백 엣지 표시), 공유 상태 필드와 읽기/쓰기 주체, if-then 라우팅 → `graph.md`. 원래 **암묵적이던 엣지**를 하나 이상 찾기 | 찾은 암묵적 엣지 수, 실패 시 잘못된 엣지를 바로 지적할 수 있나 |
| 2 병렬 fan-out/fan-in | `p08-parallel` | 독립 분할 지점 선택(두 모듈 구현, 테스트·린트 검토 + 코드 리뷰, 두 방향 조사), fan-out 규칙, fan-in 병합 기준, 서브태스크마다 git worktree | 병렬 전후 wall-clock·토큰·품질, 조정 오버헤드 |
| 3 롤백·사람 승인 | `p08-human-in-the-loop` | "부분 통과" 시 **문제가 생긴 노드**로 롤백(예: 요구 이해 오류면 연구 노드로, 공유 상태에 문제 층 기록), 병합 앞 사람 승인 노드(`state.md`에 승인/반려, 타임아웃 시 자동 반려 또는 상위 승격), interrupt 형식(무슨 일·무엇이 바뀜·왜 사람·결과), 최소 2라운드(한 번 승인, 한 번 반려) | 롤백이 문제 층을 정확히 찾나, 승인이 검증 노드가 못 막은 것을 막았나 |

**제출물**: `graph.md`(mermaid + 노드 표 + 엣지 표 + 공유 상태 필드 + 라우팅 규칙), 암묵적 엣지 목록, 실험 2 규칙과 실행 기록, 실험 3 롤백 규칙·승인 형식·2라운드 기록, 최종 회고.

**코드**: ko·en 원문 모두 저장소 경로를 안내하지 않는다(project-07과 달리 starter/solution 표가 없다). 스냅숏의 `projects/`에도 `project-08`은 없다. 가장 가까운 자료는 [[lectures/lecture-14]]의 `code/maker_checker_graph.py`(en에만 있음, 자리표시자가 있는 뼈대)다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-08-graph-engineering-first-graph/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-08-graph-engineering-first-graph/index.md`

**원문 간 차이·의문점**

- 첫 문단은 "이번 강의에서 할 일은"이라고 쓰지만 이 문서는 프로젝트다(ko·en 공통 문체).
- 실험 3은 이 그래프의 롤백 엣지를 "문제가 생긴 노드로" 보내라고 하지만, [[lectures/lecture-14]]의 참조 코드는 verify 실패를 항상 implement로만 보낸다. 실험 3이 참조 구현을 넘어서는 과제다.
