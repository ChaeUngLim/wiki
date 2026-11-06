---
type: "lecture"
title: "강의 06. 모든 에이전트 세션 전에 초기화하라"
aliases:
  - "모든 에이전트 세션 전에 초기화하라"
  - "Why Initialization Needs Its Own Phase"
  - "부트스트랩 계약"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md"
related:
  - "[[lectures/lecture-05]]"
  - "[[concepts/initialization-phase]]"
  - "[[concepts/continuity-artifacts]]"
tags:
  - "type/lecture"
  - "topic/initialization"
  - "topic/context-continuity"
updated: "2026-11-06"
---

# 강의 06. 모든 에이전트 세션 전에 초기화하라

## 왕초보 설명

- **이게 뭐야?** AI에게 기능을 만들게 하기 **전에**, 첫 대화 한 번은 "일할 준비"만 시키라는 이야기다. 설치가 되는지, 테스트가 한 개라도 통과하는지, 할 일 목록이 있는지.
- **비유:** 집 짓기. 기초 콘크리트를 붓고 굳힌 다음에 벽을 세운다. 둘을 동시에 하면 기초가 굳기 전에 벽이 올라가 다 허물게 된다.
- **왜 중요해?** 섞어서 시키면 AI는 눈에 보이는 기능 코드부터 쓰고, 테스트 설정 같은 기초는 대충 넘긴다. 그 대가는 다음 대화에서 터진다.
- **나는 뭘 하면 돼?** 새 프로젝트의 첫 지시는 "기능은 만들지 말고, 설치·테스트·실행 명령이 돌아가게 하고, 그 명령과 할 일 목록을 문서로 남기고, 커밋해"로 한다. 가능하면 빈 폴더 대신 템플릿(예: create-react-app)에서 시작한다.

## 핵심 요약

- 초기화와 구현은 최적화 목표가 다르다. 구현은 검증된 기능의 양과 질, 초기화는 이후 모든 구현의 신뢰성과 효율. 섞으면 에이전트는 보이는 출력(기능 코드)을 택하고 인프라를 희생한다.
- 초기화의 산출물은 코드가 아니라 **인프라**: 실행 가능한 환경, 검증되는 테스트 프레임워크, 부트스트랩 계약 문서, 작업 분해, 깨끗한 체크포인트 커밋.
- 초기화 완료는 "코드를 얼마나 썼나"가 아니라 **부트스트랩 계약 네 조건**으로 판정한다.
- 웜 스타트(템플릿)가 콜드 스타트(빈 디렉터리)보다 낫다.
- Anthropic은 장기 실행 에이전트에서 초기화를 별도 첫 세션으로 분리했다. initializer가 `init.sh`, 진행 파일, 기능 목록, 첫 git 커밋을 만들고, 이후 세션은 그 상태를 읽어 기능 하나씩 끝까지 테스트·커밋한다.

## 핵심 개념

**섞었을 때 생기는 일**

| 문제 | 설명 | 비유 |
| --- | --- | --- |
| 부실한 기초 | 테스트 프레임워크는 설정만 되고 검증 안 됨, 린트 느슨, 진행 파일 없음. 세션 1에선 안 보이고 세션 2에서 터짐 | 덜 굳은 기초 |
| 미검증 누적 | 테스트 전에 쓴 코드는 검증 없는 코드. 나중에 테스트를 붙이면 설계부터 잘못됐음을 발견 | 젖은 콘크리트 위 타일 |
| 예산 낭비 | 초기화가 예산을 먹어 기능은 절반만, 기초도 단단하지 않음 | — |
| 암묵적 가정 지뢰밭 | 테스트 프레임워크·디렉터리 구성 선택이 기록 안 돼 다음 세션이 상충되는 선택을 함 | 콘크리트 기초 위에 나무 말뚝 |

**부트스트랩 계약(Bootstrap Contract)** ([[concepts/initialization-phase]]): 새 세션이 프로젝트를 운영할 수 있는 네 조건. **시작할 수 있다, 테스트할 수 있다, 진행 상황을 볼 수 있다, 다음 단계를 안다.** 넷 다 필요하다.

**용어**

- **콜드 스타트 vs 웜 스타트**: 빈 디렉터리에서 구조를 추측하며 시작 vs 인프라가 갖춰진 템플릿·기존 프로젝트에서 시작. 주의: [[lectures/lecture-03]]의 "콜드 스타트 테스트"(새 세션이 저장소만으로 다섯 질문에 답하는지)와 이름은 같지만 뜻이 다르다.
- **핸드오프 준비성**: 언제든 새 에이전트가 저장소만으로 인계받을 수 있는 상태.
- **첫 번째 검증까지의 시간(Time to First Verification)**: 시작부터 첫 기능이 검증을 통과하기까지. 초기화 효율의 핵심 지표.
- **다운스트림 사용성**: 암묵적 지식 없이 작업에 성공하는 이후 세션의 비율. 초기화 품질의 가장 좋은 척도.

**초기화 인수 체크리스트**

```markdown
- [ ] `make setup`이 처음부터 성공함
- [ ] `make test`에 최소한 하나의 통과하는 테스트가 있음
- [ ] 새 에이전트 세션이 저장소 내용만으로 "실행 방법"과 "테스트 방법"을 답할 수 있음
- [ ] 최소 3개의 작업이 있는 작업 분해 파일이 존재함
- [ ] 모든 것이 git에 커밋됨
```

작업 분해의 각 작업에는 인수 기준(예: `pytest tests/test_auth.py 모두 통과`)을 단다([[concepts/definition-of-done]]).

**React 프로젝트 사례** (강의 본문 사례, 출처 표기 없음)

- 혼합: 세션 1에서 스캐폴딩과 첫 기능을 함께 → 시작·테스트 명령 문서, 진행 파일, 작업 분해 없음 → 세션 2가 구조 파악에 약 20분.
- 전용 초기화: 세션 1은 템플릿, Vitest + React Testing Library, 예제 테스트 1개, 부트스트랩 계약, 작업 분해, 커밋만 → 세션 2 재구축 3분 미만.
- 전체 주기에서 혼합 쪽 재구축 시간 합계가 약 60% 더 많았고, 초기화에 더 쓴 20분은 이후 3~4세션에서 회수됐다고 한다. 강의는 이 사례의 측정 출처를 밝히지 않는다.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-05]]: 세션 사이의 인계를 다뤘고, 이 강의는 첫 세션에서 그 인계 체계를 깔아 두는 법이다([[concepts/continuity-artifacts]]).
- [[projects/project-01]]: solution의 `init.sh`(`npm install` → `npm run check` → `npm run build`)가 이 주제의 첫 등장이다.
- 실습: [[projects/project-03]]. solution의 `init.sh`와 5단계 시작 규칙이 이 강의를 구현한다.
- 하네스 실례: [[harness/gajae-code]]의 `deep-interview`·`ralplan`은 코드를 고치지 않는 별도 단계로 명세, 계획, 목표 분해(`goals.json`)를 먼저 만든다. [[harness/oh-my-claudecode]]의 `ralph`도 첫 바퀴에 `prd.json` 정비와 검증 기준선 기록(`omc ralph verify --write-baseline`)을 먼저 한다. 둘 다 "구현 전 준비 단계"를 하네스가 강제한다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-06-why-initialization-needs-its-own-phase/index.md`
