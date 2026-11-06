---
type: "project"
title: "프로젝트 03. 세션 재시작 후에도 에이전트가 계속 작업하도록 만들기"
aliases:
  - "멀티 세션 연속성"
  - "Multi-session continuity"
  - "한 번에 기능 하나"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/projects/project-03/starter/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/init.sh"
  - "raw/learn-harness-engineering/projects/project-03/solution/session-handoff.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/claude-progress.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/clean-state-checklist.md"
  - "raw/learn-harness-engineering/projects/project-03/solution/feature_list.json"
related:
  - "[[lectures/lecture-05]]"
  - "[[lectures/lecture-06]]"
  - "[[projects/project-02]]"
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/initialization-phase]]"
  - "[[concepts/definition-of-done]]"
  - "[[concepts/one-feature-at-a-time]]"
  - "[[lectures/lecture-07]]"
  - "[[concepts/feature-list]]"
tags:
  - "type/project"
  - "topic/context-continuity"
  - "topic/scope-control"
  - "topic/feature-list"
updated: "2026-11-06"
---

# 프로젝트 03. 세션 재시작 후에도 에이전트가 계속 작업하도록 만들기

## 왕초보 설명

- **이게 뭐야?** AI에게 기능 4개를 맡기되, **한 번에 하나씩만** 하고 **증거 없이는 "완료" 표시를 못 하게** 규칙을 거는 실습이다. 중간에 대화가 끊겨도 다음 AI가 정확히 이어받는지 본다.
- **비유:** 체크리스트 들고 하는 공사. 한 칸을 끝내고 검사 도장을 받아야 다음 칸으로 간다. 교대할 땐 체크리스트와 작업 일지를 넘긴다.
- **왜 중요해?** AI는 여러 기능을 한꺼번에 건드리다 서로 망가뜨리기 쉽고, "다 했어요"를 말로만 하기 쉽다.
- **나는 뭘 하면 돼?** `AGENTS.md`에 "`feature_list.json`에서 하나만 골라라 → 그것만 고쳐라 → 검사해라 → pass와 증거를 적어라 → 커밋해라 → 그다음에만 다음 기능" 6단계를 적는다. 세션 시작엔 `init.sh`, 끝엔 `clean-state-checklist.md`를 돌린다.

## 목표

- 제품 기능 4개: 문서 청킹, 메타데이터 추출, 인덱싱 상태 표시, 인용 기반 Q&A.
- 하네스 기능: 범위 통제와 검증 게이트. `feature_list.json`으로 상태를 추적하고, 한 번에 기능 하나, 검증 증거 없이 `"pass"` 금지. 진행 로그 + 세션 핸드오프로 재시작 후에도 정확한 다음 단계에서 재개.
- 비교 질문: starter에선 에이전트가 여러 기능 사이에서 드리프트하거나 재시작 후 상태를 잃는가. solution에선 각 기능이 pass 전에 구체적 검증 증거를 갖는가.
- 도구: Claude Code 또는 Codex, Git, Node.js + Electron.

## 연결 강의

- [[lectures/lecture-05]]: 진행 로그와 핸드오프로 세션을 잇기([[concepts/continuity-artifacts]]). solution의 `claude-progress.md`, `session-handoff.md`가 그 구현이다.
- [[lectures/lecture-06]]: 세션 전에 초기화하기([[concepts/initialization-phase]]). solution의 `init.sh`와 `AGENTS.md` 5단계 시작 규칙이 그 구현이다.
- 앞 실습: [[projects/project-02]]의 코드와 7개 기능(모두 pass) 위에서 시작한다.

## starter→solution 핵심 변경점

**하네스 파일**

| 파일 | starter | solution |
| --- | --- | --- |
| `AGENTS.md` | 47줄 | 91줄. 5단계 시작 규칙, **한 번에 기능 하나 정책**, 기능 의존 그래프, docs 계층, 계층 경계, 컨벤션, 클린 상태 체크리스트·핸드오프 규칙 |
| `feature_list.json`([[concepts/feature-list]]) | 11개 중 7개 pass, 신규 4개 `not-started` | 11개 모두 pass, 기능마다 `evidence` |
| `init.sh` | 없음 | `npm install` → `npm run check` → `npm run build`(22줄, `set -euo pipefail`) |
| `claude-progress.md` | 없음 | 기능별 시간대 로그(무엇을 바꿨고 어떻게 확인했고 언제 pass로 바꿨는지) |
| `session-handoff.md` | 없음 | 한 일 / 남은 일 / 결정 / 고친 파일 / 차단 / 다음 단계 |
| `clean-state-checklist.md` | 없음 | 빌드, 기능, 범위 통제, 코드 품질, 문서 다섯 묶음 체크리스트 |

**[[concepts/one-feature-at-a-time|한 번에 기능 하나]] 정책** (solution `AGENTS.md`, "Project 03의 핵심 규율")

1. `feature_list.json`에서 `"not-started"` 기능을 정확히 하나 고른다.
2. 그 기능만 구현한다. 무관한 코드는 건드리지 않는다.
3. `npm run check`와 동작 확인으로 검증한다.
4. `feature_list.json`을 `"pass"`로 바꾸고 증거를 적는다.
5. 기능 ID를 담은 메시지로 커밋한다.
6. 그다음에만 다음 기능으로 간다.

여러 기능을 한 번에 하거나 범위 밖 파일을 고치는 것을 "이 프로젝트에서 버그와 회귀의 가장 흔한 원인"으로 명시한다.

**기능 의존 그래프**: `metadata-extraction` → `document-chunking` → (`indexing-status-ui`, `grounded-qa`는 순서 무관). 진행 로그의 실제 순서(메타데이터 → 청킹 → 상태 UI → Q&A)가 이를 따른다.

**[[concepts/definition-of-done|완료 기준]]의 확장**: [[concepts/clean-state|클린 상태]] 체크리스트 `clean-state-checklist.md`의 "범위 통제 검증" 묶음이 모든 기능 pass, 기능별 증거, fail·not-started 없음, 정책 존재, 의존 관계 준수를 확인한다.

**제품 코드**

- `DocumentMetadata`(단어·줄·문단·문자 수, 파일 형식)를 가져올 때 추출. 결정 기록: 늦게 계산하지 않고 import 때 추출해 항상 있게 한다.
- 문단 경계(빈 줄) 기준 약 500자 청킹, `chunks/<docId>.json`에 저장, 색인 후 상태 `imported` → `indexed`.
- 상태바: 색 점(회색 idle, 노랑 indexing, 초록 ready, 빨강 error), "2/3 indexed", 전체 청크 수.
- Q&A: 키워드 겹침으로 청크 점수, 상위 2개를 인용(문서 ID, 제목, 청크 번호, 앞 200자)으로 반환, 신뢰도 인용 있음 0.85 / 없음 0.30. 답변 문장은 주제별 모의(mock) 패턴이다.
- starter의 `App.tsx`는 다시 `import React`와 `'../../shared/types'` 경로를 갖고 있고, solution에서 project-01·02와 같은 정리가 반복된다(`@types/node` 추가 포함).

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md`
- 코드: `raw/learn-harness-engineering/projects/project-03/{starter,solution}`

**원문 간 차이·의문점**

- ko는 "첫 번째는 제약 없이, 두 번째는 엄격한 적용 방식으로" 두 번 실행한다고 하고, en은 "체크인된 starter와 solution을 비교"한다고 한다. en의 하네스 메커니즘에만 "one-feature-at-a-time verification"이 들어 있고, en은 "일반적인 멀티 세션 연습이 아니라 네 개의 특정 제품 기능에 대응한다"고 덧붙인다.
- 실습 이름은 "멀티 세션 연속성"인데, solution의 `claude-progress.md`는 기능 4개를 **한 세션**(2026-03-30 10:00–13:00)에서 모두 끝낸 기록이다. 재시작 후 이어받는 장면 자체는 산출물에 없다.
- `session-handoff.md`는 `ipc-handlers.ts`를 "변경 없음"으로 적지만, 실제 diff에는 `indexingService.getStatus()` → `getAppStatus()` 한 줄 변경이 있다. 인계 파일도 검증 대상이라는 [[lectures/lecture-05]]의 경고가 실습 산출물 안에서 재현된 셈이다.
