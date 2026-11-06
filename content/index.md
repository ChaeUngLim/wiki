---
type: "index"
title: "목차"
aliases:
  - "인덱스"
  - "카탈로그"
sources: []
related: []
tags:
  - "type/meta"
updated: "2026-11-06"
---

# 목차

## 메타

- [[log]] — 수집·질의·점검 이력(append-only)
- [[overview]] — 코스 종합과 학습 경로

## 강의

- [[lectures/lecture-01|강의 01. 강력한 모델도 실행 신뢰성을 보장하지 않는다]] — 모델 역량 ≠ 실행 신뢰성, 실패는 다섯 방어 레이어에 귀속, 하네스부터 점검
- [[lectures/lecture-02|강의 02. 하네스란 실제로 무엇인가]] — 하네스 = 지시·도구·환경·상태·피드백 5개 하위 시스템, 피드백 우선, 제거 실험
- [[lectures/lecture-03|강의 03. 저장소를 단일 진실 원천으로 만들어라]] — 저장소 밖 지식은 에이전트에게 없다, 콜드 스타트 테스트, ACID로 상태 관리
- [[lectures/lecture-04|강의 04. 명령 파일을 여러 파일로 분산하라]] — 거대한 AGENTS.md의 다섯 피해, 짧은 라우팅 파일 + 주제 문서, 근거의 한계
- [[lectures/lecture-05|강의 05. 세션을 넘어 컨텍스트를 살아있게 유지하라]] — 컨텍스트는 유한, 진행 파일·결정 로그·git 체크포인트로 재구축 비용 줄이기
- [[lectures/lecture-06|강의 06. 모든 에이전트 세션 전에 초기화하라]] — 초기화를 전용 단계로, 부트스트랩 계약 네 조건, 웜 스타트
- [[lectures/lecture-07|강의 07. 에이전트에게 명확한 작업 경계를 그어 주어야 합니다]] — 과욕·미완성의 악순환, WIP=1, 실행 가능한 완료 증거
- [[lectures/lecture-08|강의 08. 기능 목록으로 에이전트의 행동을 제약하십시오]] — 기능 목록은 하네스 프리미티브, 삼중 구조, 패스 상태 게이팅
- [[lectures/lecture-09|강의 09. 에이전트가 너무 일찍 완료를 선언하지 못하도록 방지하기]] — 과신 편향, 완료 판단 외부화, 3단계 종료 검사, 작업자·검사자 분리
- [[lectures/lecture-10|강의 10. 엔드투엔드 테스트만이 진정한 검증이다]] — 단위 테스트의 경계 맹점, 실행 가능한 아키텍처 규칙, 리뷰 피드백 승격
- [[lectures/lecture-11|강의 11. 에이전트의 런타임을 관측 가능하게 만들어라]] — 런타임·프로세스 두 계층 관측 가능성, 스프린트 계약, 평가자 루브릭
- [[lectures/lecture-12|강의 12. 모든 세션은 클린 상태(clean state)로 끝나야 한다]] — 클린 상태 다섯 차원, 이중 모드 정리, 품질 문서, 하네스 단순화
- [[lectures/lecture-13|13강. 수동 프롬프팅에서 자율 루프로]] — 루프 엔지니어링, `/goal` vs `/loop`, 여섯 프리미티브, autoresearch 래칫, 네 조용한 비용
- [[lectures/lecture-14|14강. 단일 루프에서 그래프 엔지니어링으로]] — 4층 겹침, 노드·엣지·상태·라우팅, graph ≠ workflow, 오케스트레이션 세금

## 프로젝트

- [[projects/project-01|프로젝트 01. 프롬프트 단독 vs. 규칙 우선]] — 같은 Electron 작업을 프롬프트만 vs. 최소 하네스로 두 번 실행해 비교
- [[projects/project-02|프로젝트 02. 읽기 쉬운 작업 공간과 세션 재개]] — 두 세션에 나눠 구현, session-handoff.md와 feature_list.json으로 이어받기
- [[projects/project-03|프로젝트 03. 세션 재시작 후에도 계속 작업하기]] — 한 번에 기능 하나, 증거 없이 pass 금지, init.sh·진행 로그·핸드오프·클린 상태 체크리스트
- [[projects/project-04|프로젝트 04. 런타임 피드백으로 에이전트 동작 수정하기]] — 심어 둔 청킹 버그, 구조화 로그, 아키텍처 경계 검사 스크립트
- [[projects/project-05|프로젝트 05. 에이전트가 자신의 작업을 검증하도록 만들기]] — 혼자 / 생성+평가 / 계획+생성+평가 세 변형, rubric 1.6→3.3→4.9
- [[projects/project-06|프로젝트 06. 완전한 에이전트 하네스 구축하기 (캡스톤)]] — 모든 장치 조립, 품질 문서 D+→A, 벤치마크·정리 스캐너, 절제 실험 과제
- [[projects/project-07|프로젝트 07. 첫 번째 자동 루프 구축하기]] — 목표·타이머·메이커-체커 세 루프 실험(저장소에 코드 없음)
- [[projects/project-08|프로젝트 08. 내 워크플로우를 그래프로 그리기]] — P07 루프를 graph.md로, fan-out/fan-in, 롤백·사람 승인(저장소에 코드 없음)

## 하네스

- [[harness/pi|Pi]] — 코스 분석. 극도로 단순한 커널 + 프로그래밍 가능한 확장, session 트리, 교체 가능한 compaction
- [[harness/claude-code|Claude Code]] — 코스 분석. 계층형 `CLAUDE.md`·5단계 compaction·permissions·hooks·subagent를 갖춘 완전한 실행 환경
- [[harness/codex|Codex]] — 코스 분석. 저장소가 사실의 원천, `AGENTS.md`는 약 100줄 디렉터리 페이지, worktree 격리
- [[harness/deepseek|DeepSeek Harness]] — 코스 분석. 모든 것이 plugin(루프까지), 능력 접합부, 이벤트 파이프라인, "Model-visible means logged"
- [[harness/oh-my-claudecode|oh-my-claudecode(OMC)]] — Claude Code 플러그인. ralph·autopilot·team과 Stop 훅으로 오래 도는 무거운 멀티 에이전트 랄프톤 하네스(왕초보 설명 포함)
- [[harness/gajae-code|Gajae-Code(gjc)]] — deep-interview → ralplan → ultragoal로 계획 먼저, 목표별 증거로 끝내는 가벼운 랄프톤 하네스(왕초보 설명 포함)

## 개념

- [[concepts/context-anxiety|컨텍스트 불안]] — 컨텍스트가 찰수록 서둘러 대충 끝내는 에이전트의 버릇
- [[concepts/clean-state|클린 상태]] — 빌드·테스트·기록·산출물·시작 경로 다섯 조건으로 세션 끝내기
- [[concepts/component-ablation|컴포넌트 제거 실험과 하네스 단순화]] — 구성 요소를 하나씩 꺼서 기여도를 재고 필요 없는 것은 지우기
- [[concepts/continuity-artifacts|연속성 산출물]] — 새 세션이 이어받게 하는 인계 파일(진행·결정·검증·다음 행동)
- [[concepts/definition-of-done|완료 기준]] — "다 했다"를 기계로 확인 가능한 조건 목록으로 미리 적기
- [[concepts/evaluator-rubric|평가자 루브릭]] — 항목별 점수표로 평가를 재현 가능하게
- [[concepts/executable-architecture-rules|실행 가능한 아키텍처 규칙]] — 구조 규칙을 문서가 아닌 실패하는 검사로
- [[concepts/feature-list|기능 목록]] — 동작·검증 명령·상태 삼중 구조로 하네스가 읽는 작업 목록
- [[concepts/harness-subsystems|하네스 5개 하위 시스템]] — 지시·도구·환경·상태·피드백으로 하네스를 나누는 모델
- [[concepts/maker-checker-separation|일하는 쪽과 확인하는 쪽 분리]] — 만드는 에이전트와 판정하는 에이전트를 권한까지 나누기
- [[concepts/graph-engineering|그래프 엔지니어링]] — 노드·엣지·공유 상태·라우팅으로 여러 루프와 agent를 조직
- [[concepts/initialization-phase|초기화 단계와 부트스트랩 계약]] — 기능 전에 시작·테스트·진행·다음 단계를 갖추기
- [[concepts/loop-engineering|루프 엔지니어링]] — 에이전트에게 지시하는 시스템을 설계하기: 목표 + 검증 + 중단 조건
- [[concepts/map-not-manual|지도를 줘라, 매뉴얼이 아니라]] — 진입 지시 파일은 짧은 안내판, 상세는 주제 문서로
- [[concepts/one-feature-at-a-time|한 번에 기능 하나(WIP=1)]] — 하나를 검증·기록·커밋한 뒤에만 다음 기능
- [[concepts/ralph-loop|Ralph 루프(랄프톤식 자동 실행)]] — 완료 증거가 나올 때까지 하네스가 에이전트를 계속 돌리는 구조
- [[concepts/progressive-disclosure|점진적 공개]] — 개요는 늘, 상세는 필요할 때만 컨텍스트에
- [[concepts/quality-document|품질 문서]] — 모듈별 성적표로 코드베이스 건강을 추적
- [[concepts/repo-system-of-record|저장소가 기록의 원천]] — 에이전트가 알아야 할 결정·제약·상태를 모두 저장소 파일에 두기
- [[concepts/sprint-contract|스프린트 계약]] — 코딩 전에 범위·검증 기준·제외 사항을 합의
- [[concepts/verification-commands|검증 명령어]] — 테스트·타입 검사·린트·빌드 명령을 지시 파일에 명시하기

## 종합

- [[syntheses/claude-code-vs-codex|Claude Code와 Codex 하네스 비교]] — "덧셈"(커널에 메모리·권한·hook) vs "뺄셈"(저장소 규약·worktree) 하위 시스템별 대비
