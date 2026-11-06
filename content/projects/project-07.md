---
type: "project"
title: "프로젝트 07. 첫 번째 자동 루프 구축하기"
aliases:
  - "첫 번째 자동 루프 구축하기"
  - "Build Your First Automated Loop"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-07-loop-engineering-first-loop/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/code/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/code/goal-template.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-13-loop-engineering/code/checker-prompt.md"
related:
  - "[[lectures/lecture-13]]"
  - "[[concepts/loop-engineering]]"
  - "[[concepts/maker-checker-separation]]"
tags:
  - "type/project"
  - "topic/loop-engineering"
updated: "2026-11-06"
---

# 프로젝트 07. 첫 번째 자동 루프 구축하기

## 왕초보 설명

- **이게 뭐야?** 지금까지 만든 하네스를 **스스로 도는 루프**로 바꿔 보는 실습이다. 세 단계로 한다: ① 할 일을 `/goal`로 맡겨 보기 ② 정기 점검을 `/loop` 타이머로 맡겨 보기 ③ 만드는 AI와 검사하는 AI를 짝지어 내가 빠진 채로 5라운드 이상 돌려 보기.
- **비유:** 자전거 보조 바퀴 떼기. 처음엔 손으로 잡아 주고(수동), 다음엔 옆에서 뛰어 주고(타이머), 마지막엔 손을 놓는다(메이커-체커 루프).
- **나는 뭘 하면 돼?** `goal.md`에 목표·확인 방법·멈출 조건·건드리면 안 되는 것을 적고, 루프 상태는 `loop-state.md`에 라운드마다 적게 한다. 내가 몇 번 끼어들었는지 세어 본다. 주의: 강의 저장소에는 이 프로젝트의 코드(starter/solution)가 **실제로 없다**(아래 참고).

## 목표

- "하네스"에서 "루프"로 넘어가는 전환 프로젝트. 이미 갖춘 환경·지침·피드백 설정을 스스로 도는 루프로 바꾼다.
- 준비: P06을 끝낸 같은 커밋에서 시작, 브랜치 `p07-goal-loop`·`p07-timer-loop`·`p07-maker-checker` 생성, `init.sh`로 하네스 작동 확인, 완료 기준이 분명한 중간 규모 대상 작업 선택(예: "모든 모듈에 단위 테스트, 커버리지 80%").
- 도구: Claude Code 또는 Codex, Git, P06의 완전한 하네스, tmux 또는 screen(오래 도는 루프 관찰), 선택으로 GitHub Actions 또는 cron.

**세 실험**

| 실험 | 브랜치 | 할 일 | 기록할 것 |
| --- | --- | --- | --- |
| 1 목표 루프 | `p07-goal-loop` | `goal.md`(목표·검증 방법·중단 조건·제약) 작성 → 수동 실행 기준선 → 같은 `goal.md`로 `/goal` 실행 → 비교 → `goal.md` 반복 수정 | 턴 수, 개입 수, 품질, 시간 |
| 2 타이머 루프 | `p07-timer-loop` | 모니터링 작업(매시간 테스트, 의존성 보안 등)을 `/loop`(또는 Codex 스레드 자동화)로 10~30분 간격, 최소 2시간 | 발견·자체 수정·거짓 양성·악화 수, 후속 조치 시간 |
| 3 메이커-체커 | `p07-maker-checker` | 메이커·체커·루프 제어 세 프롬프트, 상태 파일 `loop-state.md`, 중단 조건 "N회 연속 통과 또는 최대 라운드", 최소 5라운드 | 라운드별 한 일·체커 지적·통과/실패·개입 여부와 이유 |

실험 3 회고 질문: 몇 번 개입했나, 체커가 놓친 문제는, 메이커가 같은 실수를 반복하나, 품질 상한선은 메이커 역량인가 체커 역량인가.

**제출물**: `goal.md`(최소 2회 반복), 실험 1 비교 노트, 실험 2 프롬프트 + 2시간 실행 로그, 실험 3 세 프롬프트, `loop-state.md`(최소 5라운드), 최종 회고.

## 연결 강의

- [[lectures/lecture-13]]: 루프 유형, `/goal` vs `/loop`, 메이커·체커 분리, 외부 상태. 이 실습의 세 실험은 강의의 성숙도 사다리 1~3단계와 대응한다.
- [[lectures/lecture-12]]: 루프의 모든 라운드에 클린 상태가 필요하다([[concepts/clean-state]]).
- [[lectures/lecture-11]]: 루프 안을 볼 수 있어야 한다.
- [[lectures/lecture-05]]: 루프 상태 파일은 상태 파일의 확장이다([[concepts/continuity-artifacts]]).

## 핵심 실습 내용(문서 기반)

실습 내용은 위 "목표"의 세 실험·제출물이 전부다. 측정표(원문 "결과 측정 방법")는 완료율·인간 개입·결과 품질·절약 시간·신뢰성 다섯 지표를 세 실험에 각각 다르게 묻는다(예: 실험 2는 "거짓 양성률", 실험 3은 "체커가 내가 못 찾았을 문제 수").

**비교할 코드가 없다.** ko·en 원문 모두 `projects/project-07/`에 starter(P06 최종 하네스가 있는 지식 베이스 프로젝트)와 solution(세 루프의 완전한 구현, 루프 상태 파일, 검증 스크립트)이 있다고 적지만, 수집한 스냅숏의 `projects/`에는 `project-01`~`project-06`과 `shared`만 있다. `git log --all -- 'projects/project-07*'`도 비어 있어 이 저장소 이력에 추적된 적이 없다(HEAD `a9342ff` 기준).

따라서 문서가 약속한 "세 루프의 참고 구현"은 확인할 수 없다. 가장 가까운 자료는 [[lectures/lecture-13]]의 `code/` 폴더(ko·en 모두 있음)에 있는 템플릿 네 개다: `goal-template.md`(Goal / Acceptance / Scope: fair game·hands off / Verification / Stop Conditions), `loop-state-template.md`, `maker-prompt.md`, `checker-prompt.md`(체크리스트, 전체 판정, 심각도별 이슈, 검증 명령 결과). 이 폴더의 `index.md`가 "Loop control script examples"도 담는다고 적지만 스크립트 파일은 없다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-07-loop-engineering-first-loop/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-07-loop-engineering-first-loop/index.md`
- 코드: 없음(위 참고). 템플릿: `raw/learn-harness-engineering/docs/{ko,en}/lectures/lecture-13-loop-engineering/code/`

**원문 간 차이·의문점**

- 관련 강의 링크 제목이 실제 강의 제목과 다르다: L13 "왜 에이전트에게 프롬프트하는 것을 그만둬야 하는가"(실제 "수동 프롬프팅에서 자율 루프로"), 5강 "왜 상태 파일은 연속성의 중심인가".
- 실험 2의 "최소 2시간 실행" 로그를 요구하지만, [[lectures/lecture-13]]은 `/loop`가 세션에 묶이고 세션이 닫히면 끝난다고 설명한다. 터미널을 계속 열어 둬야 한다(그래서 tmux·screen을 도구로 든다).
