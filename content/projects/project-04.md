---
type: "project"
title: "프로젝트 04. 런타임 피드백을 사용하여 에이전트 동작 수정하기"
aliases:
  - "런타임 피드백과 범위 제어"
  - "Incremental indexing"
  - "심어 둔 버그 고치기"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/projects/project-04/starter/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-04/solution/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-04/solution/scripts/check-architecture.sh"
  - "raw/learn-harness-engineering/projects/project-04/solution/src/services/logger.ts"
  - "raw/learn-harness-engineering/projects/project-04/solution/src/services/indexing-service.ts"
  - "raw/learn-harness-engineering/projects/project-04/solution/clean-state-checklist.md"
related:
  - "[[lectures/lecture-07]]"
  - "[[lectures/lecture-08]]"
  - "[[projects/project-03]]"
  - "[[concepts/verification-commands]]"
  - "[[concepts/definition-of-done]]"
  - "[[concepts/executable-architecture-rules]]"
tags:
  - "type/project"
  - "topic/observability"
  - "topic/scope-control"
  - "topic/verification"
updated: "2026-11-06"
---

# 프로젝트 04. 런타임 피드백을 사용하여 에이전트 동작 수정하기

## 왕초보 설명

- **이게 뭐야?** 앱에 **일부러 버그를 숨겨 두고**, AI가 그걸 찾게 하는 실습이다. 한 번은 로그(프로그램이 남기는 작업 일기) 없이, 한 번은 로그와 규칙 검사 스크립트를 갖춘 채로.
- **비유:** 자동차 계기판. 경고등이 없으면 정비사는 엔진을 다 뜯어봐야 하고, 경고등이 있으면 "오일 부족"을 바로 안다.
- **왜 중요해?** AI는 프로그램 안에서 무슨 일이 일어나는지 직접 못 본다. 로그와 오류 메시지가 없으면 고장을 모르고 "구현됐다"고 착각한다.
- **나는 뭘 하면 돼?** 서비스마다 "무엇을 했고 결과가 몇 개였는지"를 로그로 남기게 하고, "화면 코드는 파일 시스템을 못 쓴다" 같은 구조 규칙을 스크립트로 검사하게 한 뒤, 그 스크립트를 커밋 전 필수 단계로 `AGENTS.md`에 적는다.

## 목표

- 런타임 관측성 추가: 시작 로그, 가져오기·색인 로그, 오류 상태.
- 아키텍처 제약 도입: 계층 간 위반(renderer가 Node 모듈을 import하는 등)을 스크립트로 막는다.
- 심어 둔 런타임 버그를 에이전트가 고친다. 비교 질문: 런타임 신호 없이 근본 원인을 찾는 데 얼마나 걸리는가, 로그와 경계 검사가 수정을 더 빠르고 덜 침습적으로 만드는가.
- 도구: Claude Code 또는 Codex, Git, Node.js + Electron.

## 연결 강의

- [[lectures/lecture-07]]: 범위 통제. solution `AGENTS.md`의 "선택한 기능 범위 안에서만 수정, 막히면 좁은 보조 수정만"이 그 적용이다.
- [[lectures/lecture-08]]: 기능 목록. 다만 이 실습에는 `feature_list.json`이 없다(아래 의문점).
- 관측성 주제 자체는 [[lectures/lecture-11]](관측성은 하네스 안에)이 다룬다. ko·en 원문 모두 관련 강의로는 07·08만 든다.
- 앞 실습: [[projects/project-03]]의 코드에서 시작한다.

## starter→solution 핵심 변경점

**심어 둔 버그** (`src/services/indexing-service.ts`): starter의 `chunkDocument()`는 문서 전체가 1000자를 넘으면 모든 청크 내용을 빈 문자열로 만든다(`content.length > 1000 ? '' : buffer.trim()`, 주석 `// BUG:`). 결과는 긴 파일의 Q&A 검색이 조용히 깨지는 것. solution은 조건을 지우고 `buffer.trim()`을 그대로 쓴다.

**로그가 버그를 드러내는 방식**: solution의 `chunkDocument complete` 로그는 `totalChunks`와 함께 `totalChars`(청크 내용 길이 합)를 남긴다. 빈 청크면 청크 수는 있는데 글자 수가 0으로 찍혀 바로 보인다. starter는 흩어진 `console.log`로 청크 수만 출력했다.

**하네스 파일**

| 파일 | starter | solution |
| --- | --- | --- |
| `AGENTS.md` | 16줄. 명령 3개, 규칙 2개("한 번에 기능 하나", 커밋 전 `npm run check`) | 85줄. 6단계 시작 절차(`pwd` → `docs/ARCHITECTURE.md` → `git log --oneline -5` → `npm install` → `npm run check` → `bash scripts/check-architecture.sh`), "기준선이 이미 깨져 있으면 그것부터 고친다", 작업 규칙, 런타임 관측성, 아키텍처 제약, 필수 산출물, 완료 정의, 세션 종료 절차 |
| `docs/ARCHITECTURE.md` | 없음(`docs/` 자체가 없음) | 계층 경계와 데이터 흐름 |
| `scripts/check-architecture.sh` | 없음 | renderer의 Node 코어 모듈, services의 Electron IPC, services·main의 React import를 grep으로 찾아 위반 시 exit 1 |
| `src/services/logger.ts` | 없음 | 128줄. JSON 한 줄 로그(timestamp, level, service, message, data), 레벨 DEBUG/INFO/WARN/ERROR, `logger.forService('이름')` |
| `clean-state-checklist.md`([[concepts/clean-state]]) | 없음 | 빌드, 아키텍처, 런타임(로그에서 이벤트 확인), 데이터 무결성, 저장소 다섯 묶음. 마지막 항목은 "`AGENTS.md`·`docs/ARCHITECTURE.md`·이 체크리스트가 실제 존재하는 파일과 일치하는가" |

**solution `AGENTS.md` 작업 규칙** (요약): 코드를 추가했다고 완료 표시하지 않는다. 구현 중 검증 규칙을 몰래 바꾸지 않는다. 채팅 요약보다 저장소 산출물을 선호한다. [[concepts/definition-of-done|완료 정의]]에 "필요한 검증이 실제로 돌았다", "증거가 기록됐다", "표준 시작 경로로 다시 시작할 수 있다", "`check-architecture.sh` 위반 0"이 들어간다. 아키텍처 검사는 [[concepts/verification-commands|검증 명령]]을 코드 구조 규칙까지 넓힌 것이다([[concepts/executable-architecture-rules]]).

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-04-incremental-indexing/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-04-incremental-indexing/index.md`
- 코드: `raw/learn-harness-engineering/projects/project-04/{starter,solution}`

**원문 간 차이·의문점**

- ko는 "첫 번째는 로그나 제약 없이, 두 번째는 적절한 도구와 규칙을 갖춘 상태로" 두 번 실행한다고 하고, en은 체크인된 starter와 solution을 비교한다고 한다(project-02·03과 같은 패턴).
- 디렉터리 이름은 `incremental-indexing`(증분 색인)이고 하네스 메커니즘에도 "증분 인덱싱"이 있지만, 실제 변경은 로그·경계 검사·버그 수정이다. 증분 색인 기능은 산출물에서 확인되지 않는다.
- 관련 강의는 07(범위)·08(기능 목록)인데, 이 실습에는 `feature_list.json`이 없다. solution `AGENTS.md`는 "`feature_list.json`, `claude-progress.md`, `init.sh`, `session-handoff.md`는 다른 단계에서 도입되니 있다고 가정하지 말라"고 쓰지만, 이 파일들은 이미 [[projects/project-03]] solution에 있었다.
- starter에는 project-03 solution에 있던 `docs/`, `feature_list.json`, `init.sh` 등이 없다. 실습마다 starter가 앞 실습의 solution을 그대로 잇지는 않는다.
