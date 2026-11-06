---
type: "project"
title: "프로젝트 01. 프롬프트 단독 vs. 규칙 우선(Rules-First): 차이는 얼마나 클까"
aliases:
  - "프롬프트 단독 vs. 규칙 우선"
  - "Prompt-Only vs. Rules-First"
  - "baseline-vs-minimal-harness"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/learn-harness-engineering/projects/project-01/starter/task-prompt.md"
  - "raw/learn-harness-engineering/projects/project-01/solution/AGENTS.md"
  - "raw/learn-harness-engineering/projects/project-01/solution/init.sh"
  - "raw/learn-harness-engineering/projects/project-01/solution/feature_list.json"
  - "raw/learn-harness-engineering/projects/project-01/solution/claude-progress.md"
related:
  - "[[lectures/lecture-01]]"
  - "[[lectures/lecture-02]]"
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/verification-commands]]"
tags:
  - "type/project"
  - "topic/instructions"
  - "topic/initialization"
  - "topic/feature-list"
  - "topic/verification"
updated: "2026-11-06"
---

# 프로젝트 01. 프롬프트 단독 vs. 규칙 우선(Rules-First): 차이는 얼마나 클까

## 왕초보 설명

- **이게 뭐야?** AI에게 같은 일을 두 번 시켜 보는 실험이다. 한 번은 "문서 보여주고 질문에 답하는 앱 만들어" 한 줄만 준다. 다른 한 번은 규칙 파일, 준비 스크립트, 할 일 체크리스트를 미리 깔아 두고 시킨다.
- **왜 해?** "AI가 똑똑하면 설명 없이도 잘하겠지"라는 생각이 맞는지 직접 확인하려고. 신입에게 업무 설명서 없이 일을 맡기는 것과, 설명서와 체크리스트를 주고 맡기는 것의 차이를 숫자로 본다.
- **나는 뭘 하면 돼?** 두 실행을 **서로 다른 빈 폴더**에서 하나씩 돌리고, 걸린 시간, 사람이 끼어든 횟수, 완성된 기능 수를 적어 비교한다. 첫 실행 결과는 손으로 고치지 않는다.

## 목표

같은 작업을 두 번 시켜 **최소 하네스가 결과를 얼마나 바꾸는지** 직접 잰다. 작업 자체는 단순하다: 왼쪽 문서 목록, Q&A 패널, 로컬 데이터 디렉터리를 가진 최소 Electron 지식 베이스 앱 쉘. 어려운 건 에이전트가 이것을 끝내게 만드는 방법이다.

- 1차(약한 하네스): `starter/task-prompt.md`의 한 줄 프롬프트("Build an Electron app that can show documents and answer questions.")만 준다.
- 2차(강한 하네스): 같은 프롬프트에 `AGENTS.md`, `init.sh`, `feature_list.json` 등을 미리 둔 저장소를 준다.
- 두 실행은 같은 에이전트(Claude Code 또는 Codex)로 하고 각각 시간을 잰다.
- 측정할 4개 기능: 창 실행, 문서 목록, 질문 패널, 로컬 데이터 디렉터리 생성. 기대 증거는 `solution/feature_list.json`에 있다.
- 실험이지 제품 요구가 아니다. 부분 완성이나 실패도 유효한 증거다. 기능 목록은 "무엇을 잴지"를 정할 뿐 1차 실행이 다 통과해야 한다는 뜻이 아니다.

en 원문의 측정 지표와 제출물:

| 지표 | 내용 |
| --- | --- |
| 완료도 | 완료 / 부분 / 실패 |
| 첫 성공 실행 | 시작부터 첫 `npm start`(또는 에이전트가 만든 실행 명령) 성공까지 걸린 시간 |
| 재시도 | 실행에 성공하기까지 필요했던 사람의 개입 횟수 |
| 누락 항목 | 에이전트가 완료를 선언했을 때 아직 구현되지 않은 기능 |
| 이른 중단 | 앱이 아직 안 뜨는데 완료를 선언했는지 |

제출: 실행별 기록(프롬프트, 로그, 최종 diff, 실행 증거), 2차는 준비한 하네스 파일 포함, 1~2쪽 비교 노트.

## 연결 강의

- [[lectures/lecture-01|강의 01. 강력한 모델도 실행 신뢰성을 보장하지 않는다]]: 강력한 모델이 곧 신뢰할 수 있는 실행은 아니라는 문제 제기. 1차 실행이 그 실패를 재현하는 자리다.
- [[lectures/lecture-02|강의 02. 하네스란 실제로 무엇인가]]: 이 프로젝트가 직접 실습하는 강의다. 2차 실행에 넣는 파일들은 강의의 [[concepts/harness-subsystems|5개 하위 시스템]]에 그대로 대응한다.

| 하위 시스템 | 2차 실행의 산출물 |
| --- | --- |
| 지시 | `AGENTS.md`(시작 순서, Electron 4계층 경계, 컨벤션, 완료 정의), `CLAUDE.md`(빠른 참조), `docs/ARCHITECTURE.md`·`docs/PRODUCT.md` |
| 환경 | `init.sh`(`npm install` → `npm run check` → `npm run build`) |
| 상태 | `feature_list.json`(기능별 status·evidence·testedAt), `claude-progress.md`(세션 기록) |
| 피드백 | `init.sh`와 완료 정의의 `npm run check`·`npm run dev`([[concepts/verification-commands\|검증 명령어]]) |
| 도구 | 별도 산출물 없음. 두 실행 모두 같은 에이전트의 기본 도구를 쓴다 |

- en 원문은 [[lectures/lecture-06]](매 작업 세션 전에 초기화)도 관련 강의로 든다. `init.sh`가 그 주제([[concepts/initialization-phase]])의 첫 등장이다.

## starter→solution 핵심 변경점

앱 코드는 거의 같다. 차이의 대부분은 **하네스 파일의 유무**다.

solution에만 있는 하네스 파일:

- `AGENTS.md`(62줄): 코드 작성 전 5단계 시작 규칙(이 파일 → `docs/ARCHITECTURE.md` → `docs/PRODUCT.md` → `bash init.sh` → `feature_list.json`), Main·Preload·Renderer·Services 4계층의 import 경계, 컨벤션(strict TS, 이름 있는 export, IPC 채널은 `src/shared/types.ts`의 `IPC_CHANNELS` 한 곳), 5개 항목의 완료 정의, 기능 목록 운영 규칙(`pass`/`fail`/`not-started`, 삭제 금지).
- `CLAUDE.md`, `docs/ARCHITECTURE.md`, `docs/PRODUCT.md`.
- `init.sh`: 의존성 설치, 타입 검사, 빌드를 순서대로 돌려 "작업 시작 전에 깨끗하게 빌드되는지"를 확인한다.
- `feature_list.json`: 4개 기능(`window-launch`, `document-list`, `question-panel`, `data-directory`)이 모두 `pass`이고, 기능마다 구체적 증거가 붙어 있다. 예: "1200x800 창, contextIsolation=true, nodeIntegration=false".
- `claude-progress.md`: 한 세션의 작업, 결정(PersistenceService 생성자 주입, IPC 채널 단일 객체), 이슈, 다음 세션 안내(Project 02로 진행).

starter에만 있는 파일: `task-prompt.md`(한 줄 작업 설명).

코드 차이(소수):

- Renderer의 `App.tsx`·컴포넌트들에서 쓰지 않는 `React` 기본 import 제거.
- Renderer의 공유 타입 import 경로를 한 단계씩 줄임: `App.tsx`는 `'../../shared/types'` → `'../shared/types'`, 컴포넌트는 `'../../../shared/types'` → `'../../shared/types'`. starter 경로는 `src/shared/`가 아니라 존재하지 않는 프로젝트 루트의 `shared/`를 가리켜 타입 검사에서 깨진다.
- `package.json`에 `@types/node` 추가.

즉 solution의 코드는 `npm run check`를 통과하도록 정리된 상태이고, 하네스가 "통과"를 판정할 수단(`init.sh`, 완료 정의, 기능 목록)을 함께 제공한다.

실습 시 주의(en 원문):

- `starter/`에도 참조 구현이 들어 있다. 1차 실행 전에 `src/`, `package.json`, 설정, `scripts/`를 지워 에이전트가 처음부터 만들게 한다. `data/` 샘플 문서는 두 실행에서 똑같이 두거나 똑같이 뺀다.
- 2차 실행 전에 `feature_list.json`의 모든 status를 `not-started`로 바꾸고 evidence·testedAt 값을 비운다(필드는 유지). `claude-progress.md`의 세션 기록도 비운다. 그러지 않으면 에이전트가 4개 기능이 이미 통과한 것으로 본다.
- 두 실행은 git 브랜치가 아니라 **격리된 작업 디렉터리 두 개**(`p01-baseline/`, `p01-improved/`)에서 하나씩 한다. 에이전트가 형제 디렉터리와 브랜치 ref까지 탐색해 강한 하네스 파일을 보면 실험이 오염된다.
- 1차 실행 결과는 손으로 고치지 않는다. 안 뜨면 그대로 기록한다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-01-baseline-vs-minimal-harness/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-01-baseline-vs-minimal-harness/index.md`
- 코드: `raw/learn-harness-engineering/projects/project-01/starter/`, `raw/learn-harness-engineering/projects/project-01/solution/`
- 공통 기반(비고): `raw/learn-harness-engineering/projects/shared/`

**ko·en 불일치.** ko 페이지(약 3.9KB)는 en(약 8KB)보다 내용이 적고, 갱신 전 버전으로 추정된다. 학습 시 en을 기준으로 한다.

- 최소 하네스 구성: ko는 `AGENTS.md` + `init.sh` + `feature_list.json` 3개, en은 여기에 `CLAUDE.md` + `claude-progress.md` + `docs/`를 더한 6개.
- 도구: ko는 "Git (브랜치 관리 및 비교)"를 권하지만, en은 브랜치로 두 실행을 나누지 말고 격리 디렉터리를 쓰라고 명시한다. 정반대다.
- ko에는 실행 프로토콜, 측정 지표, 제출물, 실험 성격 설명, starter 참조 구현 삭제와 증거 초기화 안내, lecture-06 연결이 없다.
- 강의 제목 표기: ko 페이지는 lecture-02를 "하네스가 실제로 의미하는 것"으로 적는다(강의 페이지 제목은 "하네스란 실제로 무엇인가").
