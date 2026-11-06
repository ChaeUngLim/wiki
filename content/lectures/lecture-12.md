---
type: "lecture"
title: "강의 12. 모든 세션은 클린 상태(clean state)로 끝나야 한다"
aliases:
  - "모든 세션은 클린 상태로 끝나야 한다"
  - "Leave a Clean Handoff at the End of Every Session"
  - "Why Every Session Must Leave a Clean State"
sources:
  - "raw/learn-harness-engineering/docs/ko/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
  - "raw/learn-harness-engineering/docs/en/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md"
related:
  - "[[concepts/clean-state]]"
  - "[[concepts/component-ablation]]"
  - "[[concepts/definition-of-done]]"
  - "[[concepts/quality-document]]"
tags:
  - "type/lecture"
  - "topic/clean-state"
updated: "2026-11-06"
---

# 강의 12. 모든 세션은 클린 상태(clean state)로 끝나야 한다

## 왕초보 설명

- **이게 뭐야?** AI 작업 세션이 끝날 때 **다음 사람이 바로 이어서 일할 수 있게 정리하고 끝내라**는 규칙이다. 빌드 통과, 테스트 통과, 진행 상황 기록, 임시 파일 삭제, 앱 실행 가능, 이 다섯 가지.
- **비유:** 캠핑장 규칙 "왔을 때보다 깨끗하게". 안 지키면 다음 팀이 쓰레기 치우는 데 첫 30분을 쓴다.
- **왜 중요해?** 정리를 안 하면 엉망이 매 세션 쌓인다. "나중에 정리"는 절대 안 한다는 뜻이다.
- **나는 뭘 하면 돼?** `AGENTS.md`(또는 `CLAUDE.md`)에 아래 체크리스트를 넣고 "이게 다 체크돼야 세션 완료"라고 적는다. 한 달에 한 번은 하네스 규칙 하나를 꺼 보고 결과가 나빠지지 않으면 지운다.

```
## 세션 종료 체크리스트
- [ ] 빌드 통과 (npm run build)
- [ ] 모든 테스트 통과 (npm test)
- [ ] 기능 목록 업데이트됨
- [ ] 디버그 코드 없음 (console.log, debugger, TODO)
- [ ] 표준 시작 경로 사용 가능 (npm run dev)
```

## 핵심 요약

- OpenAI와 Anthropic: 장기 신뢰성은 단일 실행의 성공이 아니라 **운영 규율**에 달렸다. 세션 종료 상태의 품질이 다음 세션의 효율을 결정한다.
- **세션 완료 = 작업 검증 통과 AND 클린 상태 검사 통과**([[concepts/clean-state]]). 클린 상태는 [[concepts/definition-of-done]]의 일부다.
- 엔트로피 증가가 기본값이다(Lehman의 소프트웨어 진화 법칙). 능동적 정리만이 막는다.
- **이중 모드 정리**: 매 세션 종료 시 즉각 정리("참조 카운팅") + 주간 전체 스캔("추적" 정리, 품질 문서 갱신, 드리프트 탐지 벤치마크).
- **하네스 단순화**: 모든 하네스 구성 요소는 모델이 혼자 못 하는 것 때문에 존재하고, 모델이 좋아지면 그 가정은 낡는다([[concepts/component-ablation]]).

## 핵심 개념

- **클린 상태의 다섯 차원**: 빌드 / 테스트(세션 전부터 있던 테스트 포함, CI에서 검증) / 진행 상황 기록 / 오래된 산출물 없음 / 표준 시작 경로 작동.
- **세션 무결성**: DB 트랜잭션처럼 완전히 커밋해 클린 상태를 남기거나, 마지막 일관 상태로 롤백한다. 중간은 없다. [[lectures/lecture-03]]의 ACID 관점과 같은 비유다.
- **[[concepts/quality-document|품질 문서]]**: 모듈별 품질 등급(검증 통과, 에이전트 이해 가능성, 테스트 안정성, 아키텍처 경계, 코드 컨벤션)을 계속 기록하는 활성 문서. 새 세션은 가장 낮은 등급부터 고친다.
- **클린업 루프**: 엔트로피를 줄이는 정기 유지보수 세션. 긴급 수정이 아닌 일상 운영.
- **멱등 정리**: 몇 번 실행해도 같은 결과. 예: `rm -f /tmp/debug-*.log`, `git checkout -- .env.local`, 마지막에 `npm run test`.

**12주 수치** (강의는 "가상 프로젝트를 가정하고" 제시)

| 시점 | 정리 전략 | 빌드 통과율 | 테스트 통과율 | 새 세션 시작 | 오래된 산출물 |
| --- | --- | --- | --- | --- | --- |
| 1주차 | 둘 다 | 100% | 100% | 5분 | — |
| 12주차 | 없음 | 68% | 61% | 60분+ | 103개 |
| 12주차 | 매 세션 검사 + 주간 루프 | 97% | 95% | 9분 | 11개 |

## 다른 강의·프로젝트와의 연결

- [[lectures/lecture-05]]·[[concepts/continuity-artifacts]]: 진행 상황 기록 차원은 연속성 산출물을 최신으로 유지하는 일이다.
- [[lectures/lecture-09]]: 완료 판단 외부화. 클린 상태 검사도 완료 조건에 들어간다.
- [[lectures/lecture-02]]: 컴포넌트 제거 실험. 이 강의의 "하네스 단순화"가 그 운영 버전이다.
- [[projects/project-03]]·[[projects/project-04]]: `clean-state-checklist.md`가 이미 실습에 나온다.
- 실습: [[projects/project-06]]. starter·solution의 [[concepts/quality-document|품질 문서]], `cleanup-scanner.sh`, 절제 실험 과제가 이 강의를 그대로 옮긴다.
- 하네스 실례: [[harness/gajae-code]]의 `ultragoal`은 마지막 목표(경계)에서 코드 정리·`architect` 구조 리뷰·`executor` QA를 같은 스냅숏에 돌리고, 목표마다 검증 증거를 `ledger.jsonl` 체크포인트로 남긴다. 단, "매 세션 종료 시 클린 상태 검사"와 같은 것은 아니다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md`
- en: `raw/learn-harness-engineering/docs/en/lectures/lecture-12-why-every-session-must-leave-a-clean-state/index.md`
- 원문 내 불일치: 같은 12주 수치를 앞에서는 "가상 프로젝트를 가정하고"(en "hypothetical"), 뒤 "실제 사례"에서는 대조군·실험군이 있는 Electron 앱 비교로 제시한다. 측정 출처는 없다. 이 위키는 가정 수치로 취급한다.
- 표기 차이: en 본문 제목은 "Leave a Clean Handoff at the End of Every Session".
