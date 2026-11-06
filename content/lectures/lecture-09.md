---
type: "lecture"
title: "강의 09. 에이전트가 너무 일찍 완료를 선언하지 못하도록 방지하기"
aliases:
  - "에이전트가 너무 일찍 완료를 선언하지 못하도록 방지하기"
  - "Why Agents Declare Victory Too Early"
  - "조기 완료 선언"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-09-why-agents-declare-victory-too-early/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-09-why-agents-declare-victory-too-early/index.md"
related:
  - "[[concepts/definition-of-done]]"
  - "[[concepts/maker-checker-separation]]"
  - "[[concepts/verification-commands]]"
tags:
  - "type/lecture"
  - "topic/verification"
  - "topic/role-separation"
updated: "2026-11-06"
---

# 강의 09. 에이전트가 너무 일찍 완료를 선언하지 못하도록 방지하기

## 왕초보 설명

- **이게 뭐야?** AI가 "다 됐습니다!"라고 하는데 실제로 돌려 보면 안 되는 문제다. 단위 테스트(부품 하나씩 하는 검사)는 다 통과했는데 전체 흐름은 한 번도 안 돌려 본 경우가 대부분이다.
- **비유:** 시험지를 빽빽이 채우고 제일 먼저 냈는데 낙제. 채운 것과 맞은 것은 다르다. 그리고 학생이 자기 시험을 채점하면 안 된다.
- **왜 그래?** AI는 원래 자기 답을 실제보다 더 확신한다(신경망의 과신, Guo et al. 2017).
- **나는 뭘 하면 돼?** `AGENTS.md`에 "완료 = ① 단위 테스트 ② 통합 테스트 ③ 실제 흐름 검증을 차례로 통과. 앞 단계가 실패하면 다음으로 안 감"을 적는다. 가능하면 확인은 다른 AI(또는 다른 역할)에게 맡긴다. 실패 메시지에는 "무엇이 틀렸고 어떻게 고치는지"를 담게 한다.

## 핵심 요약

- **조기 완료 선언**은 우연이 아니라 구조적이다. 에이전트는 코드 수준의 국소적 확신으로 판단하지만, 정확성은 시스템 수준 검증이 필요하다.
- 미끄러운 경사면: 코드가 괜찮아 보임 → 실행 검증 생략 또는 부분 실행 → "괜찮아 보인다"가 "완료"의 증거가 됨.
- 해법은 **완료 판단의 외부화**. 하네스가 런타임 신호로 독립적으로 판정한다. 에이전트의 확신은 입력이 아니다.
- **3단계 종료 검사**: ① 문법·정적 분석 ② 런타임 동작(테스트 실행, 앱 시작, 핵심 경로) ③ 시스템 수준(엔드투엔드, 사용자 시나리오).
- 같은 모델이 자기를 평가하면 관대하다. **작업자와 검사자를 분리**한다([[concepts/maker-checker-separation]]).

## 핵심 개념

- **신뢰도 보정 편향** `후보:confidence-calibration`: 자체 보고 완료 신뢰도와 실제 품질의 체계적 차이. 복잡한 다중 파일 작업일수록 크게 양(+)이다.
- **종료 기준(Termination Criteria)**: 하네스가 정한, 실행 가능한 판단 조건 집합. "완료"를 주관에서 객관으로 바꾼다([[concepts/definition-of-done]]).
- **검증-유효성 검사 이중 게이트**: "명세대로 구현됐나"(검증)와 "시스템 수준 요구를 충족하나"(유효성 검사)를 둘 다 통과해야 완료.
- **런타임 피드백 신호**: 앱이 준비 상태가 됐나, 핵심 경로가 실행됐나, DB 쓰기·파일 작업 같은 부작용이 맞나, 임시 리소스가 정리됐나.
- **완료 우선순위 제약**: 기능 정확성 → 성능 → 스타일 순. 핵심 기능 검증 전에는 리팩터링 금지.
- **에이전트용 "빨간 펜"**(OpenAI): 오류 메시지에 수정 지침을 넣는다. "테스트 실패" 대신 "POST /api/reset-password가 500. 환경 변수에 이메일 서비스 설정이 있는지 확인. 템플릿은 templates/reset-email.html".

**단위 테스트 통과 ≠ 완료**: 단위 테스트는 의존성을 모킹해 격리하므로 컴포넌트 사이 문제(경로 형식 불일치 등)를 구조적으로 못 잡는다. 자세한 유형은 [[lectures/lecture-10]] 참고.

**Anthropic 실험** (같은 Opus 4.5, 같은 프롬프트 "2D 레트로 게임 에디터")

| 아키텍처 | 시간 | 비용 | 핵심 기능 |
| --- | --- | --- | --- |
| 단일 에이전트 | 20분 | $9 | 작동 안 함(입력 무반응) |
| 플래너 + 생성기 + 평가기(Playwright로 실제 클릭 테스트) | 6시간 | $200 | 핵심 게임플레이 작동, 물리·레벨 문제 남음 |

[[lectures/lecture-01]]에 같은 실험이 나온다.

**사례: 비밀번호 재설정**: 스키마·엔드포인트·이메일 템플릿·단위 테스트 통과 후 완료 선언. 실제로는 ① 엔드투엔드 흐름 미실행 ② 마이그레이션 중간 실패로 스키마 불일치 ③ 대상 환경 이메일 설정 누락. 하네스가 앱 실행 → 전체 재설정 흐름 → DB 상태 확인을 강제하자 세 결함 모두 세션 안에서 발견됐다.

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-01]]: "검증 격차"를 가장 흔한 실패로 정의했고, 이 강의가 그 대응을 다룬다.
- [[lectures/lecture-02]]: 일하는 쪽과 확인하는 쪽의 분리를 처음 소개.
- [[lectures/lecture-07]]·[[lectures/lecture-08]]: 실행 가능한 완료 증거와 패스 상태 게이팅. 이 강의는 그 증거가 "어느 수준까지" 가야 하는지(3단계)를 정한다.
- 다음: [[lectures/lecture-10]](엔드투엔드 테스트). 실습: [[projects/project-05]].
- 하네스 실례: [[harness/gajae-code]]의 `ultragoal`은 마지막 목표에서 정리·`architect` 리뷰·`executor` QA를 같은 스냅숏에 돌리고, 끝낼 때 읽기 전용 `critic`의 `OKAY`가 있어야 한다. [[harness/oh-my-claudecode]]의 `ralph`는 `omc ralph verify`로 기준선 대비 새 실패만 판정하고, AI가 결과를 눈으로 보고 판단하는 것을 금지한다. 둘 다 "완료 판단 외부화"의 구현이다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-09-why-agents-declare-victory-too-early/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-09-why-agents-declare-victory-too-early/index.md`
- 표기 차이: ko 강의 제목은 "강의 9. 에이전트가 너무 일찍 완료를 선언하지 못하도록 방지하기"(두 자리 번호 아님), ko project-05 페이지는 "에이전트가 섣불리 완료를 선언하지 않도록 막기"로 적는다.
