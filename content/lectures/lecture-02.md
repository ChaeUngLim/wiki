---
type: "lecture"
title: "강의 02. 하네스란 실제로 무엇인가"
aliases:
  - "하네스란 실제로 무엇인가"
  - "What a Harness Actually Is"
  - "5개 하위 시스템 하네스 모델"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md"
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/code/harness-components.md"
related:
  - "[[lectures/lecture-01]]"
  - "[[projects/project-01]]"
  - "[[concepts/harness-subsystems]]"
  - "[[concepts/verification-commands]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/repo-system-of-record]]"
  - "[[concepts/map-not-manual]]"
tags:
  - "type/lecture"
  - "topic/harness-model"
  - "topic/verification"
updated: "2026-11-06"
---

# 강의 02. 하네스란 실제로 무엇인가

## 왕초보 설명

- **이게 뭐야?** AI 코딩 에이전트(코드를 대신 짜 주는 AI)는 똑똑한 요리사다. 하네스(harness)는 그 요리사가 일하는 **주방 전체**다. 레시피, 칼, 가스레인지, 재료 정리대, 맛보기 창구까지 다 포함한다.
- **왜 필요해?** 같은 요리사도 주방이 엉망이면 요리를 망친다. 실제로 AI 모델은 그대로 두고 주방(규칙 파일, 검사 명령, 진행 기록)만 정리했더니 성공률이 20%에서 거의 100%로 올랐다.
- **나는 뭘 하면 돼?** 내 프로젝트에 다음 두 줄부터 갖춘다. ① AI에게 읽힐 규칙 파일(`AGENTS.md` 또는 `CLAUDE.md`). ② 그 파일 안에 "다 됐는지 확인하는 명령"(예: `npm test`). 이 두 개가 가장 싸고 효과가 크다.

## 핵심 요약

- 하네스는 "프롬프트 파일"이 아니다. **모델 가중치 바깥의 엔지니어링 인프라 전체**가 하네스이고, 이것이 모델 역량이 실제로 얼마나 발휘되는지를 정한다.
- 강의는 하네스를 다섯 하위 시스템으로 나눈다: **지시 + 도구 + 환경 + 상태 + 피드백**. 주방 비유로는 레시피 선반, 칼 랙, 가스레인지, 준비 스테이션, 품질 확인 창이다. 하나라도 빠지면 일은 되지만 늘 불편하다.
- 다섯 중 **피드백**(검증 명령어)이 보통 가장 적은 투자로 가장 큰 효과를 낸다. 먼저 손볼 곳이다.
- 컴포넌트 가치는 같은 모델을 고정하고 하나씩 빼 보는 제거 실험으로 잰다. 다만 가장 큰 하락이 곧 병목이라는 뜻은 아니다. 병목은 실패 기록과 귀인으로 찾고, 제거 실험은 보조 증거로 쓴다.
- 하네스는 코드처럼 부패한다. 기술 부채처럼 정기적으로 감사하고 갚는다.

사례: TypeScript + React 앱(약 20,000줄)에 GPT-4o를 고정하고 하위 시스템을 하나씩 더했다.

| 단계 | 추가한 것 | 성공률 |
| --- | --- | --- |
| 1. 빈 주방 | README의 기본 설명만 | 20% (5회 중 1회) |
| 2. 레시피 선반 | 스택 버전·명명 규칙·아키텍처 결정을 담은 `AGENTS.md` | 60% |
| 3. 품질 확인 창 | `AGENTS.md`에 `yarn test && yarn lint && yarn build` 명시 | 80% |
| 4. 준비 스테이션 | 완료·미완료를 기록하는 진행 파일 템플릿 | 80~100%로 안정 |

모델은 한 번도 바꾸지 않았다. 바꾼 것은 주방 정리뿐이다.

## 핵심 개념

**하네스의 정의.** 모델 가중치 밖의 모든 것이다. OpenAI는 엔지니어의 일을 환경 설계, 의도 표현, 피드백 루프 구축으로 정리하고, Anthropic은 Claude Agent SDK를 "범용 에이전트 하네스"라 부른다. 보조 코드 예시(`code/harness-components.md`)는 시스템 프롬프트, `AGENTS.md`, bash·파일 도구, git, 시작 스크립트, 테스트 명령, 중지 훅, 린트, 평가자 루프를 모두 하네스로 나열하고 "이 중 하나만 바꿔도 실효 에이전트가 바뀐다"고 말한다.

**5개 하위 시스템** ([[concepts/harness-subsystems]])

| 하위 시스템 | 주방 비유 | 강의가 권하는 최소 구현 |
| --- | --- | --- |
| 지시 | 레시피 선반 | `AGENTS.md`/`CLAUDE.md`: 한 문장 개요, 스택과 버전, 첫 실행 명령, 협상 불가 HARD 제약, 상세 문서 링크 |
| 도구 | 칼 랙 | 셸을 막지 않되 최소 권한으로 연다 |
| 환경 | 가스레인지 | 의존성 잠금(`package.json` 등), 런타임 버전 파일(`.nvmrc` 등), Docker·devcontainer로 재현성 확보 |
| 상태 | 준비 스테이션 | `PROGRESS.md`에 완료·진행 중·차단을 적고, 세션 끝에 갱신, 시작 때 읽기 |
| 피드백 | 품질 확인 창 | `AGENTS.md`에 테스트·타입 검사·린트·전체 검증 명령을 명시 |

**검증 명령어를 먼저** ([[concepts/verification-commands]]). 피드백 하위 시스템이 투자 대비 효과가 가장 크다. 에이전트가 "다 익었는지" 확인할 방법이 없으면 나머지가 좋아도 결과를 믿을 수 없다. 위 사례에서도 검증 명령 한 줄이 60%→80%를 만들었다.

**저장소가 기록의 원천(SoR)** ([[concepts/repo-system-of-record]]). 에이전트는 앞에 놓인 파일과 실행 가능한 명령만 본다. 동료에게 물어볼 수 없으니, 필요한 맥락은 구조화된 파일과 디렉터리로 저장소 안에 있어야 한다. OpenAI의 "repo가 spec이다"와 같은 말이다.

**지도를 줘라, 매뉴얼이 아니라** ([[concepts/map-not-manual]]). `AGENTS.md`는 백과사전이 아니라 목차 페이지여야 하고 100줄 안팎이 적당하다. 넘치면 `docs/`로 나누고 필요할 때 읽게 한다.

**제약하되 마이크로매니지하지 않는다.** 실행 가능한 규칙으로 불변식을 강제하고, 구현 단계를 일일이 지시하지 않는다. Anthropic은 에이전트가 자기 결과를 과신한다는 점을 발견했고, 해법으로 "일하는 쪽"과 "확인하는 쪽"을 나눈다([[concepts/maker-checker-separation]]).

**[[concepts/component-ablation|컴포넌트 제거 실험]]**. 모델을 고정하고 하위 시스템을 하나씩 빼서 하락폭으로 한계 기여도를 잰다. 하락이 거의 없으면 중복, 약한 설계, 해당 작업에서 덜 쓰임 중 무엇인지 해석해야 한다. 병목 진단은 실패 원인(불명확한 의도, 맥락 부족, 재현 불가 환경, 검증 피드백 부재, 끊긴 상태 관리)의 귀인이 먼저다. Anthropic은 모델이 강해질수록 일부 컴포넌트는 덜 중요해지지만 새 컴포넌트가 늘 생긴다고 보고했다.

**도구별 하네스 읽기.** 강의는 익숙한 도구를 같은 틀로 읽는다.

- Claude Code: `CLAUDE.md`(지시), 셸(도구), 로컬 실행(환경), 세션 기록(상태), 테스트 실행(피드백). 테스트 방법을 안 알려주면 피드백이 고장 난다.
- Cursor: `.cursorrules`가 지시. IDE를 다시 열면 맥락이 사라져 상태가 약하다.
- Codex: git worktree로 작업마다 환경을 격리하고 로그·메트릭·트레이스로 검증한다. `AGENTS.md`와 검증 명령이 있는 저장소에서 훨씬 낫다.
- AutoGPT: 구조화된 상태와 정밀한 피드백이 없어 장시간 작업에서 맥락이 쌓이고 루프에 빠졌다. 실패한 건 모델이 아니라 하네스다.

**어포던스 분석(연습 3).** 에이전트가 "하고 싶지만 못 하는" 상황을 실행의 간극(Gulf of Execution, 방법을 모름)과 평가의 간극(Gulf of Evaluation, 맞는지 모름)으로 나눠 진단하고, 각각을 메우는 하네스 개선을 설계한다.

## 다른 강의·프로젝트와의 연결

- 실습: [[projects/project-01|프로젝트 01. 프롬프트 단독 vs. 규칙 우선]]. 프롬프트만 준 실행과 `AGENTS.md`·`init.sh`·`feature_list.json`을 미리 둔 실행을 비교해 하네스 효과를 직접 잰다.
- 앞 강의: [[lectures/lecture-01|강의 01. 강력한 모델도 실행 신뢰성을 보장하지 않는다]]. 강력한 모델이 왜 여전히 실패하는지를 다루고, 이 강의가 그 답으로 하네스를 정의한다.
- 지시 하위 시스템 심화: [[lectures/lecture-03|강의 03. 저장소를 단일 진실 원천으로]](저장소를 기록 원천으로), [[lectures/lecture-04]](거대한 지시 파일 하나가 실패하는 이유).
- 상태 하위 시스템 심화: [[lectures/lecture-05]](장기 작업의 연속성), [[lectures/lecture-06]](초기화 단계 분리).
- 피드백 하위 시스템 심화: [[lectures/lecture-09]](너무 이른 완료 선언), [[lectures/lecture-10]](종단 간 테스트).

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-02-what-a-harness-actually-is/index.md`
- 보조: `raw/learn-harness-engineering/docs/ko/lectures/lecture-02-what-a-harness-actually-is/code/harness-components.md`
- 표기 차이: ko 강의 제목은 "하네스란 실제로 무엇인가"인데, ko project-01 페이지는 같은 강의를 "하네스가 실제로 의미하는 것"으로 적는다.
