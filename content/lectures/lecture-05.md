---
type: "lecture"
title: "강의 05. 세션을 넘어 컨텍스트를 살아있게 유지하라"
aliases:
  - "세션을 넘어 컨텍스트를 살아있게 유지하라"
  - "Why Long-Running Tasks Lose Continuity"
  - "건망증 장인의 일지"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md"
related:
  - "[[lectures/lecture-03]]"
  - "[[concepts/continuity-artifacts]]"
  - "[[concepts/context-anxiety]]"
tags:
  - "type/lecture"
  - "topic/context-continuity"
updated: "2026-11-06"
---

# 강의 05. 세션을 넘어 컨텍스트를 살아있게 유지하라

## 왕초보 설명

- **이게 뭐야?** 긴 작업을 하다 대화가 꽉 차서 새 대화를 열면, AI는 **방금까지 한 일과 그 이유를 전부 잊는다**. 이걸 어떻게 버티느냐의 이야기다.
- **비유:** 매일 아침 기억을 잃는 뛰어난 장인. 해결책은 기억력을 키우는 게 아니라 **퇴근 전에 일지를 쓰게 하는 것**이다.
- **왜 중요해?** 일지가 없으면 새 대화는 15분씩 다시 탐색하고, 어제 일부러 고른 방법(빨간 벽돌)을 모르고 뒤집기도 한다.
- **나는 뭘 하면 돼?** AI에게 작업을 마칠 때마다 `PROGRESS.md`(한 일 / 하는 중 / 알려진 문제 / 다음 단계)와 `DECISIONS.md`(무엇을, 왜, 버린 대안)를 갱신하고 git 커밋하게 한다. 새 대화는 "그 두 파일부터 읽어"로 시작한다.

## 핵심 요약

- 컨텍스트 창은 유한하다. 1M 토큰이 돼도 긴 작업은 결국 넘친다. 이건 모델 업그레이드로 못 푼다.
- 잃어버리는 정보는 균등하지 않다. 코드("무엇")는 남지만 결정의 이유("왜")가 사라진다. 다음 세션은 의도적인 설계를 "최적화"해 망가뜨릴 수 있다.
- 해법은 더 큰 창이 아니라 **더 나은 상태 영속화**: 진행 파일 + 결정 로그 + git 체크포인트([[concepts/continuity-artifacts]]).
- 핵심 지표는 **재구축 비용**: 새 세션이 일할 수 있는 상태가 되기까지 걸리는 시간.
- 하네스 설계는 모델마다 다르다. Sonnet 4.5는 [[concepts/context-anxiety|컨텍스트 불안]]이 심해 컨텍스트 리셋이 필요했고, Opus 4.5는 컴팩션만으로 관리할 수 있었다.

## 핵심 개념

**연속성이 깨지면 생기는 일**

| 문제 | 설명 | 비유 |
| --- | --- | --- |
| 결정 번복 | 이전 세션이 A·B·C를 분석해 B를 고른 이유를 모르고 다시 결정 | 빨간 벽돌을 고른 이유를 잊고 파란 벽돌로 다시 쌓기 |
| 중복 작업 | 이미 끝난 일인지 몰라 다시 하거나, 절반 하다 충돌 발견 | 두 팀이 같은 벽을 쌓기 |
| 요구 드리프트 | 세션마다 목표 이해가 조금씩 달라져 원래 요구에서 벗어남 | 전화 전달 게임 |
| 검증 격차 | 이전 검증 결과가 기록 안 돼 매번 처음부터 진단 | — |

**용어**

- **연속성 산출물**: 새 세션이 이어받게 하는 영속 상태 파일. 기본형은 진행 로그 + 검증 기록 + 다음 행동.
- **재구축 비용(Rebuild cost)**: 새 세션이 실행 가능한 상태에 이르는 시간.
- **드리프트(Drift)**: 에이전트의 이해와 저장소 실제 상태의 차이. 세션 경계마다 생기고 누적된다.
- **컴팩션 vs 리셋** `후보:compaction-vs-reset`: 컴팩션은 같은 세션에서 앞 대화를 요약한다("무엇"은 남고 "왜"를 잃기 쉬움, 불안도 안 사라짐). 리셋은 새 세션을 열어 파일에서 재구축한다(깨끗하지만 산출물이 완전해야 함).

**일지를 구성하는 네 도구**

1. `PROGRESS.md`: 현재 상태(최신 커밋, 테스트 42/43 등), 완료, 진행 중, 알려진 이슈, 다음 단계.
2. `DECISIONS.md`: 날짜·결정·이유·거부된 대안·제약. 상세 설계 문서가 아니라 메모.
3. git 커밋을 체크포인트로: 원자적 작업 단위마다, 무엇과 왜를 적은 메시지로.
4. `AGENTS.md`의 출근·퇴근 루틴:

```markdown
## 세션 시작 시 (출근)
1. PROGRESS.md를 읽어 현재 상태 파악
2. DECISIONS.md를 읽어 중요 결정 파악
3. make check로 저장소가 일관된 상태인지 확인
4. PROGRESS.md의 "다음 단계"에서 이어서 진행
## 세션 종료 전 (퇴근)
1. PROGRESS.md 업데이트
2. make check로 일관된 상태 확인
3. 완료된 작업 커밋
```

**혼합 전략**: 30분 이하 작업은 한 세션에서 끝낸다. 세션을 넘는 작업은 산출물로 잇는다. 기준: 창의 60% 이상이 필요해 보이면 인계 준비를 시작한다.

**근거와 한계**

- Anthropic(Claude 웹사이트 복제): 초기화 에이전트가 기능 200개 이상을 정의해 모두 미통과로 표시하고, `init.sh`·진행 파일·첫 커밋을 만들었다. 후속 세션이 진행 파일과 git 기록을 읽고 기능을 하나씩 구현·검증했다. 강의는 이 사례가 완료율이나 복구 시간 수치를 보고하지 않으며, 동봉 코드는 교육용 시뮬레이션이라고 단서를 단다.
- LangChain: 같은 gpt-5.2-codex로 Terminal Bench 2.0(89개 작업) 52.8% → 66.5%(+13.7%p). 검증 지침·미들웨어·컨텍스트 관리를 함께 바꾼 결과라 진행 파일만의 효과는 아니다.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-03]]: ACID의 "내구성"(세션 간 지식은 파일에)과 콜드 스타트 질문 "지금 어디에 있는가?"를 이 강의가 구체 도구로 만든다.
- [[lectures/lecture-01]]: 다섯 방어 레이어 중 "상태 관리"와 컨텍스트 불안을 처음 언급했다.
- 다음: [[lectures/lecture-06]](초기화를 별도 단계로). 실습: [[projects/project-03]].
- 하네스 실례: [[harness/gajae-code]]의 `ultragoal`은 `goals.json`(상태)과 `ledger.jsonl`(증거 일지), [[harness/oh-my-claudecode]]의 `ralph`는 `prd.json`과 `progress.txt`로 같은 일지를 자동으로 쓴다. OMC는 압축(compaction) 직전에 상태를 저장하는 훅도 둔다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-05-why-long-running-tasks-lose-continuity/index.md`
- 표기 차이: ko 강의 제목은 "세션을 넘어 컨텍스트를 살아있게 유지하라", ko project-03 페이지는 "세션 간에 컨텍스트를 유지하기"로 적는다.
