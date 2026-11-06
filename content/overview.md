---
type: "overview"
title: "개요"
aliases:
  - "코스 개요"
sources: []
related: []
tags:
  - "type/meta"
updated: "2026-11-06"
---

# 개요

## 코스 종합

전체 목차는 [[index]], 이력은 [[log]]에 있다. 수집한 소스는 28개(강의 14, 프로젝트 8, 하네스 6)다.

**중심 주장**: 모델이 아니라 하네스가 결과를 정한다. [[lectures/lecture-01]]이 "좋은 모델도 실패한다"를 보이고, [[lectures/lecture-02]]가 하네스를 지시·도구·환경·상태·피드백 다섯 하위 시스템으로 정의한다([[concepts/harness-subsystems]]). 이후 강의는 거의 모두 이 다섯 중 하나를 깊게 판다.

**코스의 흐름은 네 덩어리다.**

1. **저장소를 기억으로(강의 03~06)**: 에이전트는 세션마다 기억을 잃는다. 그래서 결정·제약·진행을 저장소 파일에 두고([[concepts/repo-system-of-record]]), 진입 지시 파일은 짧은 지도로 두며([[concepts/map-not-manual]], [[concepts/progressive-disclosure]]), 인계 파일로 다음 세션이 이어받게 하고([[concepts/continuity-artifacts]]), 첫 세션은 초기화에만 쓴다([[concepts/initialization-phase]]).
2. **범위와 완료(강의 07~10)**: 에이전트는 너무 많이 벌이고 너무 일찍 끝냈다고 한다. 한 번에 하나만([[concepts/one-feature-at-a-time]]), 기능 목록으로 범위를 고정하고([[concepts/feature-list]]), 완료 기준을 미리 적고([[concepts/definition-of-done]]), 완료 판단은 에이전트 밖 검증 명령과 E2E로([[concepts/verification-commands]], [[concepts/executable-architecture-rules]]), 만드는 쪽과 판정하는 쪽을 나눈다([[concepts/maker-checker-separation]]).
3. **증거와 정리(강의 11~12)**: 무슨 일이 있었는지(로그)와 왜 그랬는지(계약·채점표)를 남기고([[concepts/sprint-contract]], [[concepts/evaluator-rubric]]), 세션은 클린 상태로 끝내고([[concepts/clean-state]], [[concepts/quality-document]]), 하네스 자체도 주기적으로 덜어 낸다([[concepts/component-ablation]]).
4. **사람이 루프 밖으로(강의 13~14)**: 하네스 위에 목표·검증·중단 조건으로 도는 루프를 쌓고([[concepts/loop-engineering]]), 루프가 분업·병렬·롤백을 필요로 하면 그래프로 펼친다([[concepts/graph-engineering]]).

**실습은 강의와 짝을 이룬다.**

- [[projects/project-01]] — lecture-02의 실습. 같은 작업을 프롬프트만으로 한 번, 최소 하네스(`AGENTS.md`·`init.sh`·`feature_list.json`)로 한 번 돌려 차이를 잰다.
- [[projects/project-02]] — lecture-03·04의 실습. 세션 인계 파일로 새 세션이 이어받는지 본다.
- [[projects/project-03]] — lecture-05·06의 실습. 한 번에 기능 하나와 증거 기반 pass로 범위를 통제한다.
- [[projects/project-04]] — lecture-07·08의 실습. 구조화 로그와 경계 검사 스크립트로 심어 둔 버그를 찾는다.
- [[projects/project-05]] — lecture-09·10의 실습. 혼자 / 생성+평가 / 계획+생성+평가로 rubric 1.6 → 3.3 → 4.9.
- [[projects/project-06]] — lecture-11·12의 캡스톤. 모든 장치를 합치고 하나씩 빼 본다. 직접 실행한 `benchmark.sh`는 품질 문서의 "모든 작업 완료"와 달리 첫 작업에서 실패했다.
- [[projects/project-07]] — lecture-13의 실습. 목표·타이머·메이커-체커 세 루프.
- [[projects/project-08]] — lecture-14의 실습. 루프를 그래프로 펼치고 병렬·롤백·사람 승인을 더한다.

**원문이 스스로 어긋나는 곳**(각 페이지에 기록): 진입 파일 권장 길이(약 100줄 / 50~100 / 50~200, [[concepts/map-not-manual]]), 프로젝트 문서가 가리키는 코드와 실제 코드의 차이(project-04에 `feature_list.json` 없음, project-07·08은 코드 자체가 없음), 수치의 성격(lecture-11의 45분/15분과 lecture-12의 12주 수치는 가정 시나리오), 같은 OpenAI 사례의 기간("몇 주" vs "다섯 달").

## 하네스 종합

코스 분석 4개와 코스 밖 하네스 2개를 같은 다섯 하위 시스템 틀로 읽었다. 6열 비교표는 각 하네스 페이지의 "다른 하네스와의 비교"에 같은 내용으로 들어 있다.

- [[harness/pi]] — 가장 작은 커널, 나머지는 확장으로. 아무것도 대신 정하지 않는다.
- [[harness/claude-code]] — 루프는 단순하고, 루프를 둘러싼 계층형 `CLAUDE.md`·5단계 compaction·permissions·hooks·subagent가 신뢰성을 만든다.
- [[harness/codex]] — 기능을 더하기보다 저장소 규약(약 100줄 디렉터리 페이지 `AGENTS.md`, 검증 명령)과 worktree 격리로 승부하는 "뺄셈".
- [[harness/deepseek]] — 하네스를 모델과 독립된 OS로: 루프까지 plugin, 모델이 본 것은 모두 기록.
- [[harness/oh-my-claudecode]] — Claude Code의 hooks·스킬·subagent 위에 `ralph`·`team`·`autopilot` 워크플로와 키워드 자동 감지를 얹은 무거운 쪽. `Stop` hook으로 턴 종료를 막아 오래 돈다.
- [[harness/gajae-code]] — 독립 CLI. 스킬 4개·역할 4개로 deep-interview → ralplan → ultragoal 한 흐름(계획 → 승인 → 목표별 실행·증거). 처음 쓰는 사람은 이 페이지의 왕초보 설명부터.

**축 두 개로 보면**

| 축 | 한쪽 끝 | 다른 쪽 끝 |
| --- | --- | --- |
| 무엇을 기본으로 넣나 | [[harness/pi]], [[harness/codex]]: 규약·확장 지점만, 나머지는 사용자 몫 | [[harness/claude-code]], [[harness/oh-my-claudecode]]: 메모리·권한·워크플로까지 내장 |
| 루프를 누가 돌리나 | 코스 분석 4개: 하네스 분석 글은 루프 워크플로를 거의 다루지 않음(루프는 [[lectures/lecture-13]]에서 `/goal`·`/loop`로) | [[harness/oh-my-claudecode]], [[harness/gajae-code]]: [[concepts/ralph-loop]]를 하네스 기능으로 내장 |

**가벼운 랄프톤 vs 무거운 랄프톤**: 이 위키의 출발 질문은 "gjc의 deep-interview → ralplan → ultragoal(`gjc.ralplan.autoHandoff`)로 하루짜리 가벼운 랄프톤, OMC의 ralph·team·autopilot로 며칠짜리 무거운 랄프톤"이었다. 원문으로 확인되는 차이는 **구성의 무게**다: 기본 기능 수(스킬 4·역할 4 vs 스킬 48·에이전트 19), 계속 돌리는 장치(goal 모드 + 목표당 `nudgeBudget` 기본 10 vs `Stop` hook + 안전 상한 기본 500회), 사람 개입(계획 승인 1회, `autoHandoff=ultragoal`이면 0회 vs 매직 키워드). **작업 기간(하루 vs 며칠)은 README·SKILL·가이드 어디에도 없는 사용자 전제**이고 이 위키는 검증하지 않았다. 자세한 비교는 [[concepts/ralph-loop]]와 [[harness/gajae-code]]의 "무엇을 고를까" 표.

## 강의 순서 학습 경로

각 줄: 강의 — 한 줄 요지 · 실습 · 핵심 개념.

- [[lectures/lecture-01]] — 좋은 모델도 실패한다: 다섯 방어 레이어와 진단 루프 · 실습: [[projects/project-01]] 1차 실행(실패 재현) · [[concepts/verification-commands]]
- [[lectures/lecture-02]] — 하네스의 정의와 5개 하위 시스템 · 실습: [[projects/project-01]] · [[concepts/harness-subsystems]], [[concepts/verification-commands]]
- [[lectures/lecture-03]] — 저장소를 기록의 원천으로: 콜드 스타트 테스트와 ACID · 실습: [[projects/project-02]] · [[concepts/repo-system-of-record]]
- [[lectures/lecture-04]] — 짧은 진입 파일과 주제 문서로 나눠 두기 · 실습: [[projects/project-02]] · [[concepts/map-not-manual]], [[concepts/progressive-disclosure]]
- [[lectures/lecture-05]] — 세션이 바뀌어도 이어지게: 진행 파일·결정 로그·git 체크포인트 · 실습: [[projects/project-03]] · [[concepts/continuity-artifacts]], [[concepts/context-anxiety]]
- [[lectures/lecture-06]] — 첫 세션은 초기화만: 부트스트랩 계약 · 실습: [[projects/project-03]] · [[concepts/initialization-phase]]
- [[lectures/lecture-07]] — 범위 통제: 한 번에 하나만 끝까지 · 실습: [[projects/project-04]] · [[concepts/one-feature-at-a-time]], [[concepts/definition-of-done]]
- [[lectures/lecture-08]] — 기능 목록과 패스 상태 게이팅 · 실습: [[projects/project-04]] · [[concepts/feature-list]]
- [[lectures/lecture-09]] — 완료 판단을 에이전트 밖으로: 3단계 종료 검사 · 실습: [[projects/project-05]] · [[concepts/definition-of-done]], [[concepts/maker-checker-separation]]
- [[lectures/lecture-10]] — 엔드투엔드 테스트와 실행 가능한 아키텍처 규칙 · 실습: [[projects/project-05]] · [[concepts/executable-architecture-rules]]
- [[lectures/lecture-11]] — 관측 가능성: 무슨 일이 있었나(로그) + 왜 그랬나(계약·루브릭) · 실습: [[projects/project-06]] · [[concepts/sprint-contract]], [[concepts/evaluator-rubric]]
- [[lectures/lecture-12]] — 세션을 클린 상태로 끝내고, 하네스는 주기적으로 덜어 낸다 · 실습: [[projects/project-06]] · [[concepts/clean-state]], [[concepts/quality-document]], [[concepts/component-ablation]]
- [[lectures/lecture-13]] — 하네스 위에 루프를 쌓아 사람이 루프 밖으로 · 실습: [[projects/project-07]] · [[concepts/loop-engineering]], [[concepts/ralph-loop]]
- [[lectures/lecture-14]] — 루프가 복잡해지면 그래프가 된다: 숨은 결정을 종이 위로 · 실습: [[projects/project-08]] · [[concepts/graph-engineering]]

## 개념 지도

- [[concepts/harness-subsystems]] — 하네스를 읽는 기본 틀. 이후 개념은 대부분 이 중 하나에 속한다.
  - 지시
    - [[concepts/map-not-manual]] — 진입 파일은 지도, 상세는 주제 문서로
    - [[concepts/progressive-disclosure]] — 개요는 늘, 상세는 필요할 때만
    - [[concepts/executable-architecture-rules]] — 구조 규칙을 문서가 아닌 실패하는 검사로
  - 상태
    - [[concepts/repo-system-of-record]] — 지시·상태를 저장소 파일에 두는 원칙
    - [[concepts/continuity-artifacts]] — 인계 파일·진행 로그·체크포인트
    - [[concepts/context-anxiety]] — 상태가 부족할 때 에이전트가 서둘러 끝내는 방식
    - [[concepts/initialization-phase]] — 기능 전에 시작·테스트·진행·다음 단계를 갖추기
    - [[concepts/clean-state]] — 다섯 조건으로 세션 끝내기
    - [[concepts/quality-document]] — 모듈별 성적표로 코드베이스 건강 추적
  - 범위
    - [[concepts/feature-list]] — 범위와 상태를 하나의 파일로
    - [[concepts/one-feature-at-a-time]] — WIP=1
    - [[concepts/sprint-contract]] — 코딩 전에 범위·기준·제외 사항 합의
  - 피드백
    - [[concepts/verification-commands]] — 피드백의 최소 구현
    - [[concepts/definition-of-done]] — 판정할 잣대를 미리 적기
    - [[concepts/evaluator-rubric]] — 항목별 점수로 평가를 재현 가능하게
    - [[concepts/maker-checker-separation]] — 만드는 쪽 ≠ 판정하는 쪽
  - 하네스 자체의 운영
    - [[concepts/component-ablation]] — 구성 요소를 하나씩 꺼서 기여도를 재고 덜어 내기
- 하네스 위의 층
  - [[concepts/loop-engineering]] — 목표 + 검증 + 중단 조건으로 도는 루프
  - [[concepts/ralph-loop]] — 그 루프를 하네스 기능으로 내장한 형태(OMC `ralph`, gjc `ultragoal`)
  - [[concepts/graph-engineering]] — 루프를 노드·엣지·공유 상태·라우팅으로 펼치기
- 개념이 실제 제품에 구현된 모습: [[harness/claude-code]](계층형 지시·hooks), [[harness/codex]](디렉터리 페이지·worktree), [[harness/deepseek]](이벤트 파이프라인·전부 기록), [[harness/pi]](점진적 공개·session 트리), [[harness/oh-my-claudecode]]·[[harness/gajae-code]]([[concepts/ralph-loop]] 내장)
