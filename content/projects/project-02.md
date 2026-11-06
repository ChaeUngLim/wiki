---
type: "project"
title: "프로젝트 02. 프로젝트를 읽기 쉽게 만들고 중단된 곳에서 재개하기"
aliases:
  - "에이전트가 읽을 수 있는 작업 공간"
  - "Agent-readable workspace"
  - "세션 핸드오프 실습"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-02-agent-readable-workspace/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-02-agent-readable-workspace/index.md"
  - "raw/learn-harness-engineering/projects/project-02/starter/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-02/solution/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-02/solution/session-handoff.md"
  - "raw/learn-harness-engineering/projects/project-02/solution/feature_list.json"
related:
  - "[[lectures/lecture-03]]"
  - "[[lectures/lecture-04]]"
  - "[[projects/project-01]]"
  - "[[concepts/repo-system-of-record]]"
  - "[[concepts/definition-of-done]]"
  - "[[concepts/continuity-artifacts]]"
tags:
  - "type/project"
  - "topic/system-of-record"
  - "topic/context-continuity"
  - "topic/feature-list"
updated: "2026-11-06"
---

# 프로젝트 02. 프로젝트를 읽기 쉽게 만들고 중단된 곳에서 재개하기

## 왕초보 설명

- **이게 뭐야?** AI가 **두 번에 나눠** 일하게 하고, 두 번째 AI가 첫 번째가 어디까지 했는지 얼마나 빨리 알아채는지 보는 실습이다.
- **비유:** 교대 근무. 앞 근무자가 인수인계 메모 없이 퇴근하면 다음 사람은 처음부터 다시 파악해야 한다. 메모(`session-handoff.md`)가 있으면 바로 이어서 일한다.
- **왜 중요해?** ([[concepts/continuity-artifacts]]) AI는 대화가 새로 시작되면 이전 일을 전혀 기억하지 못한다. 기억은 파일에만 남는다([[concepts/repo-system-of-record]]).
- **나는 뭘 하면 돼?** 작업을 끝낼 때마다 AI에게 "한 일 / 남은 일 / 막힌 것과 결정한 것 / 고친 파일"을 `session-handoff.md`에 쓰게 하고, 새로 시작할 때는 "그 파일부터 읽어"라고 `AGENTS.md`에 적어 둔다.

## 목표

- 제품 기능 세 가지를 **두 세션에 걸쳐** 구현한다: 문서 가져오기(import), 문서 상세 보기(전체 내용 로딩), 재시작 후에도 유지되는 로컬 저장(persistence).
- 하네스 기능은 "인수인계가 읽히는 작업 공간"이다. 비교 질문: 두 번째 세션이 얼마나 많이 다시 탐색하는가, 그리고 말로 설명해 주지 않아도 저장소 상태만으로 이어갈 수 있는가.
- 두 번 실행한다: 1차는 얇은 문서에 `session-handoff.md`가 없는 starter, 2차는 `ARCHITECTURE.md`·`PRODUCT.md`가 보강되고 `session-handoff.md`가 있는 solution 형태.
- 도구: Claude Code 또는 Codex, Git, Node.js + Electron.

## 연결 강의

- [[lectures/lecture-03]]: 저장소를 기록의 원천으로. 이 실습은 그 강의의 콜드 스타트 다섯 질문 중 "지금 어디에 있는가?"를 `session-handoff.md`와 `feature_list.json`으로 답하게 만든다.
- [[lectures/lecture-04]]: 지시를 여러 파일로 나누기. solution의 `AGENTS.md`는 상세를 `docs/ARCHITECTURE.md`·`docs/PRODUCT.md`로 보내는 안내판 역할을 한다([[concepts/map-not-manual]]).
- 앞 실습: [[projects/project-01]]의 코드 위에서 시작한다.

## starter→solution 핵심 변경점

**하네스 파일**

| 파일 | starter | solution |
| --- | --- | --- |
| `AGENTS.md` | 20줄. 빠른 시작 3줄, 계층 4줄, 컨벤션 3줄 | 68줄. 코드 전 5단계 시작 규칙(이 파일 → `docs/ARCHITECTURE.md` → `docs/PRODUCT.md` → `npm install && npm run check` → `feature_list.json`), docs 계층 설명, Electron 계층 경계, 완료 기준 5개, 세션 핸드오프 규칙 |
| `session-handoff.md` | 없음 | 52줄. 한 일 / 남은 일 / 결정 / 고친 파일 / 차단 / 다음 단계 |
| `feature_list.json` | 7개 중 4개 `pass`, `document-import`·`document-detail`·`basic-persistence`는 `not-started` | 7개 모두 `pass` |
| `docs/PRODUCT.md` | 기술 요구사항 절 포함 | 파일 선택기 가져오기, 내용 보기, 삭제 범위, "Persistence" 절 추가. 기술 요구사항 절은 빠짐 |
| `docs/ARCHITECTURE.md` | 114줄 | 121줄. 가져오기 흐름 추가 |

solution `AGENTS.md`의 **[[concepts/definition-of-done|완료 기준]]**: ① `npm run check` 통과 ② 앱이 뜨고 창이 보임 ③ `feature_list.json`에 `"pass"`와 증거 ④ Electron 계층 경계 준수 ⑤ `docs/ARCHITECTURE.md` 또는 `docs/PRODUCT.md` 갱신.

**세션 핸드오프 규칙**: 재개할 때 `session-handoff.md`를 읽고, 끝낼 때 한 일, 남은 일, 차단·결정, 고친 파일을 갱신한다.

**제품 코드**

- 새 IPC 채널 `GET_DOCUMENT_CONTENT: 'documents:get-content'`(`src/shared/types.ts`), 핸들러 등록, preload에 `documents.getContent()` 노출. 핸드오프 기록상 이유: 목록 화면의 응답을 작게 유지하려고 내용을 `GET_DOCUMENT`에 묶지 않았다.
- `App.tsx`: 마운트 시 `useEffect`로 `refreshDocuments()` 호출(재시작 후 목록 복원), 가져오기 버튼이 `Cancel`/`+ Import`로 토글.
- `DocumentDetail.tsx`: 내용 보기와 삭제 버튼. 삭제는 저장된 내용 파일과 원본 사본을 함께 지운다.
- [[projects/project-01]]과 같은 정리 패턴도 반복된다: 쓰지 않는 `React` 기본 import 제거, `'../../shared/types'` → `'../shared/types'` 경로 수정, `package.json`에 `@types/node` 추가.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-02-agent-readable-workspace/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-02-agent-readable-workspace/index.md`
- 코드: `raw/learn-harness-engineering/projects/project-02/{starter,solution}`

**원문 간 차이**

- ko는 2차 실행을 "`ARCHITECTURE.md`, `PRODUCT.md`, `session-handoff.md`를 저장소에 미리 배치한 상태"로 설명하고, en은 "체크인된 solution 형태(보강된 문서 + `session-handoff.md`)에 대해" 실행한다고 설명한다. en만 "제품 기능은 가져오기·상세·지속성, 하네스 기능은 인수인계가 읽히는 작업 공간"이라고 둘을 구분해 적는다.
- solution `docs/PRODUCT.md`의 화면 배치도에는 "Show Chunks toggle", "Index Document button"이 있다. 이 실습의 7개 기능 목록에는 색인 기능이 없고, 핸드오프의 다음 단계가 "Project 03에서 색인·메타데이터·근거 있는 Q&A 추가"다. 다음 실습 범위가 문서에 먼저 들어간 것으로 보인다.
