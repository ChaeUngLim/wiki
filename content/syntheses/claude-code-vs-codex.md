---
type: "synthesis"
title: "Claude Code와 Codex 하네스 비교"
aliases:
  - "Claude Code vs Codex"
  - "클로드 코드와 코덱스 비교"
sources:
  - "raw/learn-harness-engineering/docs/ko/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/claude-code/index.md"
  - "raw/learn-harness-engineering/docs/ko/harness-designs/codex/index.md"
  - "raw/learn-harness-engineering/docs/en/harness-designs/codex/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-13-loop-engineering/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/index.md"
related:
  - "[[harness/claude-code]]"
  - "[[harness/codex]]"
  - "[[concepts/map-not-manual]]"
  - "[[concepts/progressive-disclosure]]"
tags:
  - "type/synthesis"
  - "topic/harness-model"
updated: "2026-11-06"
---

# Claude Code와 Codex 하네스 비교

## 질문

Claude Code와 Codex는 하네스를 어떻게 다르게 설계했나?

## 답변

**한 줄 대비**: 코스 원문은 [[harness/claude-code]]를 메모리·permissions·hooks·subagent를 커널에 넣는 "덧셈"으로, [[harness/codex]]를 커널을 절제하고 책임을 저장소 규약과 컨텍스트 엔지니어링에 두는 "뺄셈"으로 대비한다.

| 하위 시스템 | Claude Code | Codex |
| --- | --- | --- |
| 지시 | `CLAUDE.md` 4범위(조직/사용자/프로젝트/로컬) + 하위 디렉터리 필요 시 로드 + auto memory | `AGENTS.md` = 약 100줄 디렉터리 페이지, 세부는 `docs/`, 불변 조건과 검증 명령만 |
| 상태 | append-only `history.jsonl`, 5단계 compaction, `/resume`·fork | Write 전략(상태를 파일·문서에), `/compact`와 `compact_prompt` |
| 경계 | permissions 7모드 + ML 분류기 | 작업마다 git worktree 격리, 승인 정책, plan mode |
| 피드백 | `PostToolUse` hook 강제 검사, `Stop` hook이 완료 선언에 개입 | 검증 명령을 `AGENTS.md` 규약에 포함, worktree마다 로그·메트릭·트레이스 |
| subagent | 대화를 독립 sidechain 파일에 저장 | `spawn_agent`/`wait_agent`, `.codex/agents/*.toml` |

- **같은 문제, 다른 해법**: 둘 다 거대한 지시 파일을 피한다. Claude Code는 범위 계층과 필요 시 로드로, Codex는 짧은 목차 + `docs/` 분할로 푼다([[concepts/map-not-manual]], [[concepts/progressive-disclosure]]).
- **완료 판정을 모델 밖으로**: Claude Code는 hook으로 결정론적 검사를 주입하고, Codex는 검증 경로를 저장소의 기본 구성 요소로 만든다([[concepts/verification-commands]], [[concepts/maker-checker-separation]]).
- **연속성**: Claude Code는 저장 계층(추가 지향 기록)으로, Codex는 규약(상태를 파일로 쓰기)으로 보장한다([[concepts/continuity-artifacts]]).
- **고르는 감각**: 기본 장치를 바로 쓰고 싶으면 Claude Code 쪽, 저장소를 잘 정리해 두고 규약으로 굴리고 싶으면 Codex 쪽이다. Claude Code 위에 워크플로를 더 얹은 것이 [[harness/oh-my-claudecode]]다.

**이 위키가 확인하지 않은 것**

- 5단계 compaction, permissions 7모드, sidechain은 원문이 인용한 VILA Lab 소스 분석 기준이다. Codex의 `build_environment_update_item` 등도 커뮤니티 분석 인용이다. 외부 문서는 직접 확인하지 않았다.
- 같은 OpenAI 사례의 기간이 원문끼리 다르다: Codex 분석은 "몇 주 만에 100만 줄 넘게", [[lectures/lecture-13]]은 "다섯 달 동안 약 100만 줄, 약 1,500 PR".

## 작성일

2026-11-06. `wiki-query` 질문 "Claude Code와 Codex 하네스 비교"의 답변을 사용자 승인으로 저장했다.
