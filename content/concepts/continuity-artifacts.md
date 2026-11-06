---
type: "concept"
title: "연속성 산출물(세션 인계 파일)"
aliases:
  - "세션 핸드오프"
  - "진행 파일"
  - "continuity artifacts"
  - "session handoff"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-02-agent-readable-workspace/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-02-agent-readable-workspace/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-03-multi-session-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-03-multi-session-continuity/index.md"
related:
  - "[[concepts/repo-system-of-record]]"
  - "[[concepts/context-anxiety]]"
tags:
  - "type/concept"
  - "topic/context-continuity"
updated: "2026-11-06"
---

# 연속성 산출물(세션 인계 파일)

## 왕초보 설명

- **이게 뭐야?** AI가 대화를 끝내기 전에 남기는 **인수인계 메모 파일**이다. 다음 대화의 AI는 이 파일만 읽고 바로 이어서 일한다.
- **왜 필요해?** AI는 새 대화에서 이전 일을 하나도 기억하지 못한다. 메모가 없으면 매번 처음부터 다시 파악한다.
- **나는 뭘 하면 돼?** `AGENTS.md`에 두 줄을 적는다. "시작할 때 `session-handoff.md`(또는 `PROGRESS.md`)를 읽어라", "끝낼 때 한 일 / 남은 일 / 막힌 것·결정 / 고친 파일을 적고 커밋해라".

## 정의

새 세션이 이전 세션이 멈춘 곳에서 **구두 설명 없이** 재개하도록 저장소에 남기는 구조화된 상태 파일. 기본형은 진행 로그 + 검증 기록 + 다음 행동이고, 결정 로그와 git 체크포인트가 함께 쓰인다.

## 왜 중요한가

- 컨텍스트 창은 유한하고, 세션이 바뀌면 결정의 "왜"가 가장 먼저 사라진다.
- 핵심 지표인 재구축 비용(새 세션이 일할 수 있게 되기까지의 시간)을 직접 줄인다.
- 컨텍스트 리셋 전략은 이 산출물이 완전할 때만 작동한다.

## 등장하는 곳

- [[lectures/lecture-05]]: `PROGRESS.md`, `DECISIONS.md`, git 체크포인트, 출근·퇴근 루틴의 네 도구로 정의한다.
- [[projects/project-02]]: `session-handoff.md`(한 일 / 남은 일 / 결정 / 고친 파일 / 차단 / 다음 단계)로 두 번째 세션이 이어받는지 시험한다.
- [[projects/project-03]]: `session-handoff.md`에 더해 기능별 시간대 로그 `claude-progress.md`, 시작 점검 `init.sh`, 종료 점검 `clean-state-checklist.md`를 둔다. 단, 핸드오프의 "변경 없음" 기록이 실제 diff와 한 줄 어긋난다.
- [[harness/gajae-code]], [[harness/oh-my-claudecode]]: 하네스가 이 파일들(`goals.json`·`ledger.jsonl`, `prd.json`·`progress.txt`)을 자동으로 쓰고 읽는다.

## 관련 개념

- [[concepts/initialization-phase]]: 첫 세션에서 인계 체계(진행 파일, 작업 목록)를 깔아 두는 단계.
- [[concepts/repo-system-of-record]]: 인계 파일도 저장소 안에 있어야 존재한다.
- [[concepts/context-anxiety]]: 리셋으로 불안을 피하려면 인계 파일이 완전해야 한다.
- [[concepts/definition-of-done]]: 인계 파일의 "완료" 항목은 완료 기준과 증거로 뒷받침돼야 한다.

## 상충·열린 질문

없음.
