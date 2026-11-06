---
type: "lecture"
title: "강의 07. 에이전트에게 명확한 작업 경계를 그어 주어야 합니다"
aliases:
  - "에이전트에게 명확한 작업 경계를 그어 주어야 합니다"
  - "Why Agents Overreach and Under-finish"
  - "WIP=1"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md"
related:
  - "[[concepts/one-feature-at-a-time]]"
  - "[[concepts/definition-of-done]]"
  - "[[projects/project-03]]"
tags:
  - "type/lecture"
  - "topic/scope-control"
updated: "2026-11-06"
---

# 강의 07. 에이전트에게 명확한 작업 경계를 그어 주어야 합니다

## 왕초보 설명

- **이게 뭐야?** AI에게 "인증 추가해"라고 하면 DB, 라우트, 화면, 오류 처리까지 **김에 다 손대다가** 하나도 제대로 안 끝내는 문제와 그 해법이다.
- **비유:** 간장 한 병 사러 갔다가 카트를 가득 채워 나오는 사람. 또는 요리 여섯 개를 동시에 하다 다 태우는 사람.
- **왜 그래?** AI에게 "여기도 고쳐볼게"라고 쓰는 비용은 거의 0이다. 하지만 주의력은 한정돼 있어서, 일을 동시에 많이 벌일수록 하나당 쓸 수 있는 생각이 줄어든다.
- **나는 뭘 하면 돼?** `AGENTS.md`에 "한 번에 기능 하나만. 그 기능이 검사를 통과해야 다음으로. 기능 A를 하면서 B를 리팩터링하지 마라"를 적는다([[concepts/one-feature-at-a-time]]). 각 기능엔 "어떤 명령이 통과하면 끝"을 적어 둔다.

## 핵심 요약

- **과욕(overreach)**과 **미완성(under-finish)**은 서로를 키운다. 과욕 → 주의력 희석 → 미완성 → 반쯤 된 코드가 복잡도를 높임 → 더 큰 과욕.
- 주의력 수학: 컨텍스트 용량 C를 k개 작업에 나누면 작업당 C/k. 이것이 최소 임계값 아래로 떨어지면 아무것도 완성되지 않는다.
- 해법은 **WIP=1**(동시에 active인 작업은 하나)과 **실행 가능한 완료 증거**, 그리고 작업 상태를 파일로 외부화하는 것.
- "적게 하되 끝내는 것"이 "많이 하되 반만 남기는 것"보다 항상 낫다.

## 핵심 개념

- **WIP 제한**: 칸반에서 온 개념. 진행 중 작업 수를 제한한다. 에이전트의 안전한 기본값은 WIP=1. 리틀의 법칙 L = λ × W에 따라 진행 중 작업(L)이 많으면 작업당 리드 타임(W)이 늘어난다.
- **완료 증거(Completion Evidence)**: "진행 중 → 완료"에 필요한 검증 가능한 조건. "코드가 멀쩡해 보인다"는 해당 안 되고 "curl이 201을 반환한다"가 해당된다([[concepts/definition-of-done]]).
- **범위 표면(Scope Surface)**: 작업 단위를 노드, 의존성을 엣지로 하는 DAG. 상태는 `not_started`, `active`, `blocked`, `passing` 넷.
- **완료 압력**: WIP 제한과 완료 증거로 하네스가 "새로 시작하기 전에 지금 것을 끝내라"고 거는 힘.
- **검증 완료율(VCR)** `후보:verified-completion-rate`: 검증된 작업 / 활성화된 작업. VCR < 1.0이면 새 작업 활성화를 막는다.

**올바른 방법 네 가지**

1. WIP=1 강제: `CLAUDE.md`/`AGENTS.md`에 "한 번에 기능 하나, 엔드투엔드 검증 통과 후에만 다음, A 구현 중 B 리팩터링 금지".
2. 모든 작업에 실행 가능한 완료 증거: 예 `F01: 사용자 등록 / 검증: curl -X POST /api/register ... | jq .status == 201 / 상태: passing`.
3. 범위 표면을 기계 판독 파일(JSON·Markdown)로 외부화.
4. VCR 모니터링.

**사례: 기능 8개 REST API** (강의 본문 사례, 측정 출처 표기 없음)

| | 제약 없음(뷔페 모드) | WIP=1(단일 접시 모드) |
| --- | --- | --- |
| 세션 1 | 기능 5개 동시 활성화, 12개 파일 ~800줄, 엔드투엔드 통과 20%(등록만 작동) | 등록 하나만, 4개 파일 ~200줄, 100% 통과 후 커밋 |
| 결과 | 세션 3 끝 8개 중 3개(37.5%) | 세션 4 끝 8개 중 7개(87.5%), 1개는 외부 의존성으로 차단 |

강의 결론 문장 "총 코드는 적지만(800 vs 1200줄)"은 어느 쪽이 800줄인지 명시하지 않는다(en도 같은 문장). 표의 800줄은 제약 없음 쪽 세션 1 수치다.

## 다른 강의·프로젝트와의 연결

- [[projects/project-03]]: solution `AGENTS.md`의 "한 번에 기능 하나 정책"과 기능 의존 그래프가 이 강의의 WIP=1과 범위 표면을 그대로 구현한다.
- 다음: [[lectures/lecture-08]](기능 목록을 하네스 프리미티브로). 실습: [[projects/project-04]].
- [[lectures/lecture-01]]: 다섯 방어 레이어 중 "작업 명세". [[lectures/lecture-05]]: 반쯤 된 일이 세션 경계를 넘으면 드리프트가 된다.
- 하네스 실례: [[harness/oh-my-claudecode]]의 `ralph`는 `prd.json`의 가장 앞 미통과 이야기 하나를 골라 구현·검증한 뒤 다음으로 간다. [[harness/gajae-code]]의 `ultragoal`도 목표(`G001`, `G002` …)를 하나씩 checkpoint로 닫는다. 두 하네스 모두 WIP=1을 루프 구조로 강제한다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-07-why-agents-overreach-and-under-finish/index.md`
- 표기 차이: ko 강의 제목은 "에이전트에게 명확한 작업 경계를 그어 주어야 합니다", ko project-04 페이지는 "에이전트를 위한 명확한 작업 경계 설정하기"로 적는다.
