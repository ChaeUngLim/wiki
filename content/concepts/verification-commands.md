---
type: "concept"
title: "검증 명령어"
aliases:
  - "검증 명령어 명시"
  - "피드백 하위 시스템"
  - "verification commands"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-04-incremental-indexing/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-01-why-capable-agents-still-fail/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/ko/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-01-baseline-vs-minimal-harness/index.md"
  - "raw/oh-my-claudecode/README.ko.md"
  - "raw/oh-my-claudecode/README.md"
  - "raw/gajae-code/README.ko.md"
  - "raw/gajae-code/README.md"
related:
  - "[[concepts/harness-subsystems]]"
  - "[[lectures/lecture-01]]"
tags:
  - "type/concept"
  - "topic/verification"
updated: "2026-11-06"
---

# 검증 명령어

## 왕초보 설명

- **이게 뭐야?** "다 됐어요"를 AI 말이 아니라 **기계로 확인하는 명령**이다. 예: `npm test`(테스트), `npm run build`(빌드). 통과하면 진짜 된 것이고, 실패하면 아직 안 된 것이다.
- **왜 필요해?** AI는 자기가 만든 결과를 과하게 칭찬한다. 요리사가 맛도 안 보고 "완벽해요"라고 하는 것과 같다. 확인 명령이 있으면 AI가 스스로 맛을 보고 고친다.
- **나는 뭘 하면 돼?** 규칙 파일(`AGENTS.md`)에 "작업이 끝나면 이 명령을 돌려라"라고 명령 이름을 그대로 적어 둔다. 한 줄로도 효과가 크다.

## 정의

에이전트가 자기 결과가 맞는지 스스로 확인할 수 있도록 테스트·타입 검사·린트·빌드 명령을 지시 파일에 **명시적으로** 적어 두는 것. [[concepts/harness-subsystems]] 중 피드백 하위 시스템(품질 확인 창)의 최소 구현이다.

강의의 예:

```
검증 명령어:
- 테스트: pytest tests/ -x
- 타입 검사: mypy src/ --strict
- 린트: ruff check src/
- 전체 검증: make check (위 모두 포함)
```

## 왜 중요한가

- 다섯 하위 시스템 중 투자 대비 효과가 보통 가장 크다. 강의 사례에서 `AGENTS.md`에 `yarn test && yarn lint && yarn build` 한 줄을 더하자 성공률이 60%에서 80%로 올랐다.
- 검증 수단이 없으면 "다 익었는지" 아무도 모른다. Claude Code가 테스트를 돌릴 수 있어도 방법을 알려주지 않으면 품질 확인 창이 고장 난 것과 같다.
- 완료 판정의 근거가 된다. 명령이 통과해야 완료라는 기준이 있으면 에이전트의 자기 칭찬 대신 기계적 증거로 판정한다.

## 등장하는 곳

- [[lectures/lecture-01]]: 다섯 방어 레이어 중 "검증 피드백". 에이전트의 완료 자신감과 실제 정확성의 차이(검증 격차)를 줄이는 수단이다. FastAPI 사례에서 `pytest tests/api/v2/ && python -m mypy src/`를 명시하자 같은 모델이 스스로 확인했다.
- [[lectures/lecture-02]]: 피드백 하위 시스템을 "가장 먼저 맞출 것"으로 꼽는다.
- [[projects/project-01]]: `init.sh`가 `npm install` → `npm run check` → `npm run build`를 순서대로 돌리고, `AGENTS.md`의 완료 정의가 `npm run check` 통과와 `npm run dev` 실행 확인을 요구한다. 2차 실행이 끝나면 `./init.sh` 결과를 기록한다.
- [[harness/oh-my-claudecode]]: `ralph`의 `feedbackCommands`(예: `npm run build`, `npm test`, `npm run lint`)를 `omc ralph verify`가 실행한다. 첫 바퀴에 기준선을 기록하고, 이후에는 새로 생긴 실패만 문제로 본다. AI가 결과를 눈으로 보고 판단하는 것은 금지다.
- [[harness/gajae-code]]: `ultragoal`은 목표마다 실행한 검증 명령과 결과를 quality-gate JSON으로 기록해야 완료 checkpoint를 남길 수 있다.

- [[projects/project-04]]: `scripts/check-architecture.sh`가 계층 경계 위반(renderer의 Node 모듈 등)을 grep으로 찾아 exit 1로 막는다. 검증 명령을 코드 구조 규칙까지 넓힌 예다.

## 관련 개념

- [[concepts/executable-architecture-rules]]: 구조 규칙을 검사 스크립트로 바꾼 검증 명령.
- [[concepts/definition-of-done]]: 검증 명령은 완료 기준 중 기계로 확인하는 부분이다.
- [[concepts/harness-subsystems]]: 이 개념이 속한 상위 모델.

## 상충·열린 질문

없음.
