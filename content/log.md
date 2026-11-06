---
type: "log"
title: "로그"
aliases:
  - "작업 기록"
sources: []
related: []
tags:
  - "type/meta"
updated: "2026-11-06"
---

# 로그

시간순 append-only 기록이다. 목차는 [[index]], 종합은 [[overview]]를 본다.

## [2026-11-05] lint | 골격 기준선 · 오류 0

- 골격 점검: [[index]], [[overview]]. 소스 페이지는 아직 없다.

## [2026-11-06] ingest | lecture-02 · 하네스란 실제로 무엇인가

- 생성: [[lectures/lecture-02]]
- 갱신: [[index]], [[overview]]
- 단일 소스 개념 후보: harness-subsystems, verification-commands, repo-system-of-record, map-not-manual, maker-checker-separation, component-ablation

## [2026-11-06] ingest | project-01 · 프롬프트 단독 vs. 규칙 우선

- 생성: [[projects/project-01]]
- 개념 승격(lecture-02 + project-01): [[concepts/harness-subsystems]], [[concepts/verification-commands]]
- 갱신: [[lectures/lecture-02]](미수집 마커·후보 마커를 링크로 교체), [[index]], [[overview]]
- 기록한 불일치: ko 페이지가 en보다 짧고 최소 하네스 구성(3개 vs 6개)과 git 브랜치 사용 여부가 반대

## [2026-11-06] lint | 도구 변경: 코스 밖 하네스 2개와 왕초보 기준 추가 · 오류 0

- 사용자 요청: oh-my-claudecode, gajae-code 하네스를 왕초보 기준으로 분석해 위키에 포함. 왕초보 가이드(`raw/gjc-beginner-guide.html`)를 gajae 하네스 페이지의 본보기로 받음.
- raw 추가(커밋·해시 고정): `raw/oh-my-claudecode`(0564555), `raw/gajae-code`(d4ecb35), `raw/gjc-beginner-guide.html`.
- `wiki-config`: `rawBaseline`을 저장소 목록(`repos`)·파일 목록(`files`)으로 일반화, ID별 `paths` 재정의, ID 2개 추가(26 → 28), 소스·개념 템플릿 첫 헤딩에 왕초보 설명 추가.
- `wiki-lint.mjs`: 여러 원문 저장소 검사, `paths` 재정의 지원. `--self-test` 79/79 PASS(신규 5건), `--check-skills` 위반 0.
- 왕초보 설명 섹션 추가: [[lectures/lecture-02]], [[projects/project-01]], [[concepts/harness-subsystems]], [[concepts/verification-commands]].

## [2026-11-06] ingest | gajae-code · 계획 먼저, 증거로 끝내는 가벼운 랄프톤 하네스

- 생성: [[harness/gajae-code]]
- 개념 승격(lecture-02 + gajae-code): [[concepts/maker-checker-separation]]
- 갱신: [[lectures/lecture-02]](후보 마커를 개념 링크로), [[index]], [[overview]]
- 기록한 불일치: README 설치 예시 v0.15.3 vs 스냅숏 0.19.3, 가이드의 역할 5개 vs 번들 역할 파일 4개. "하루 랄프톤"은 원문 근거 없는 사용자 전제로 표시.

## [2026-11-06] ingest | oh-my-claudecode · Claude Code 위에 얹는 무거운 멀티 에이전트 랄프톤 하네스

- 생성: [[harness/oh-my-claudecode]]
- 개념 승격(oh-my-claudecode + gajae-code): [[concepts/ralph-loop]]
- 갱신: [[harness/gajae-code]](미수집 마커를 링크로), [[concepts/maker-checker-separation]], [[concepts/verification-commands]], [[concepts/harness-subsystems]], [[index]], [[overview]]
- 기록한 불일치: README의 "32개 전문 에이전트" vs `agents/` 정의 19개, v4.4.0 미래형 서술 vs 스냅숏 5.7.31. "2~3일 랄프톤"은 원문 근거 없는 사용자 전제로 표시.

## [2026-11-06] lint | 사용자 정정 반영: 기간 표현 제거, 장단점·특징 상세화 · 오류 0

- 사용자 정정: "하루 / 2~3일"은 예시였다. gjc는 [[harness/gajae-code]]를 말한다. 비교 대상은 [[harness/oh-my-claudecode]]와 gjc 두 하네스다.
- 두 하네스 페이지에서 기간 전제 단락을 제거하고 "특징 한눈에", "장점", "단점과 주의할 점"(왕초보 관점, 원문 근거 열 포함)을 추가했다. gjc 페이지에 "무엇을 고를까" 선택 표를 추가했다.
- [[concepts/ralph-loop]]의 기간 관련 열린 질문을 삭제했다.

## [2026-11-06] ingest | lecture-01 · 강력한 모델도 실행 신뢰성을 보장하지 않는다

- 생성: [[lectures/lecture-01]]
- 갱신: [[lectures/lecture-02]], [[projects/project-01]](미수집 마커를 링크로), [[concepts/verification-commands]], [[concepts/harness-subsystems]](lecture-01 근거 추가), [[index]], [[overview]]
- 단일 소스 개념 후보: diagnostic-loop, definition-of-done

## [2026-11-06] ingest | lecture-03 · 저장소를 단일 진실 원천으로 만들어라

- 생성: [[lectures/lecture-03]]
- 개념 승격(lecture-02 + lecture-03): [[concepts/repo-system-of-record]]
- 갱신: [[lectures/lecture-01]], [[lectures/lecture-02]](후보 마커와 미수집 마커를 링크로), [[index]], [[overview]]
- 단일 소스 개념 후보: cold-start-test

## [2026-11-06] ingest | lecture-04 · 명령 파일을 여러 파일로 분산하라

- 생성: [[lectures/lecture-04]]
- 개념 승격(lecture-02 + lecture-04): [[concepts/map-not-manual]]
- 갱신: [[lectures/lecture-01]], [[lectures/lecture-02]], [[lectures/lecture-03]], [[concepts/repo-system-of-record]], [[index]], [[overview]]
- 단일 소스 개념 후보: progressive-disclosure
- 기록한 상충: 진입 파일 적정 길이(lecture-02 100줄 안팎 / lecture-03 50~100줄 / lecture-04 50~200줄)

## [2026-11-06] ingest | project-02 · 프로젝트를 읽기 쉽게 만들고 중단된 곳에서 재개하기

- 생성: [[projects/project-02]]
- 갱신: [[lectures/lecture-03]], [[lectures/lecture-04]](미수집 마커를 링크로), [[concepts/repo-system-of-record]], [[concepts/map-not-manual]], [[index]], [[overview]]
- 기록한 불일치: 2차 실행 설명(ko "문서를 미리 배치" / en "solution 형태에 대해 실행"), solution `docs/PRODUCT.md`에 project-03 범위(청크 보기·색인 버튼)가 먼저 들어가 있음

## [2026-11-06] lint | 개념 승격: definition-of-done · 오류 0

- 개념 승격(lecture-01 + project-02): [[concepts/definition-of-done]]
- 갱신: [[lectures/lecture-01]](후보 마커를 링크로), [[projects/project-02]], [[concepts/verification-commands]], [[index]], [[overview]]

## [2026-11-06] ingest | lecture-05 · 세션을 넘어 컨텍스트를 살아있게 유지하라

- 생성: [[lectures/lecture-05]]
- 개념 승격: [[concepts/continuity-artifacts]](lecture-05 + project-02), [[concepts/context-anxiety]](lecture-01 + lecture-05)
- 갱신: [[lectures/lecture-01]], [[lectures/lecture-03]], [[projects/project-02]], [[index]], [[overview]]
- 단일 소스 개념 후보: compaction-vs-reset

## [2026-11-06] ingest | lecture-06 · 모든 에이전트 세션 전에 초기화하라

- 생성: [[lectures/lecture-06]]
- 개념 승격(lecture-06 + project-01): [[concepts/initialization-phase]]
- 갱신: [[lectures/lecture-01]], [[lectures/lecture-02]], [[lectures/lecture-05]], [[projects/project-01]](미수집 마커를 링크로), [[concepts/harness-subsystems]], [[concepts/continuity-artifacts]], [[index]], [[overview]]
- 기록한 표기 충돌: lecture-06의 "콜드 스타트"(빈 디렉터리에서 시작)와 lecture-03의 "콜드 스타트 테스트"는 이름만 같고 뜻이 다르다.

## [2026-11-06] ingest | project-03 · 세션 재시작 후에도 에이전트가 계속 작업하도록 만들기

- 생성: [[projects/project-03]]
- 갱신: [[lectures/lecture-05]], [[lectures/lecture-06]](미수집 마커를 링크로), [[concepts/continuity-artifacts]], [[concepts/initialization-phase]], [[concepts/definition-of-done]], [[index]], [[overview]]
- 단일 소스 개념 후보: one-feature-at-a-time
- 기록한 불일치: 실행 방식(ko 제약 없음/엄격 두 번 실행, en starter·solution 비교), 진행 로그가 기능 4개를 한 세션에서 완료, 핸드오프의 `ipc-handlers.ts` "변경 없음" vs 실제 한 줄 변경

## [2026-11-06] ingest | lecture-07 · 에이전트에게 명확한 작업 경계를 그어 주어야 합니다

- 생성: [[lectures/lecture-07]]
- 개념 승격(lecture-07 + project-03): [[concepts/one-feature-at-a-time]]
- 갱신: [[projects/project-03]](후보 마커를 링크로), [[concepts/definition-of-done]], [[index]], [[overview]]
- 단일 소스 개념 후보: verified-completion-rate
- 기록한 모호함: 사례 결론 "총 코드는 적지만(800 vs 1200줄)"이 어느 전략의 수치인지 불명확(ko·en 동일)

## [2026-11-06] ingest | lecture-08 · 기능 목록으로 에이전트의 행동을 제약하십시오

- 생성: [[lectures/lecture-08]]
- 개념 승격(lecture-08 + project-03): [[concepts/feature-list]]
- 갱신: [[lectures/lecture-07]], [[projects/project-03]], [[concepts/one-feature-at-a-time]], [[concepts/definition-of-done]], [[index]], [[overview]]
- 기록한 상충: 강의의 삼중 구조·4상태·스크립트 전환 vs 코스 `feature_list.json`의 검증 명령 없는 형식·2상태·에이전트 직접 수정

## [2026-11-06] ingest | project-04 · 런타임 피드백을 사용하여 에이전트 동작 수정하기

- 생성: [[projects/project-04]]
- 갱신: [[lectures/lecture-07]], [[lectures/lecture-08]](미수집 마커를 링크로), [[concepts/verification-commands]], [[index]], [[overview]]
- 기록한 불일치: 실행 방식(ko 두 번 실행 / en starter·solution 비교), 이름의 "증분 인덱싱"이 산출물에 없음, 관련 강의 08인데 `feature_list.json` 없음, starter가 project-03 solution의 하네스 파일을 잇지 않음

## [2026-11-06] ingest | lecture-09 · 에이전트가 너무 일찍 완료를 선언하지 못하도록 방지하기

- 생성: [[lectures/lecture-09]]
- 갱신: [[lectures/lecture-01]], [[lectures/lecture-02]](미수집 마커를 링크로), [[concepts/maker-checker-separation]], [[concepts/definition-of-done]], [[index]], [[overview]]
- 단일 소스 개념 후보: confidence-calibration

## [2026-11-06] ingest | lecture-10 · 엔드투엔드 테스트만이 진정한 검증이다

- 생성: [[lectures/lecture-10]]
- 개념 승격(lecture-10 + project-04): [[concepts/executable-architecture-rules]]
- 갱신: [[lectures/lecture-01]], [[lectures/lecture-02]], [[lectures/lecture-09]](미수집 마커를 링크로), [[projects/project-04]], [[concepts/verification-commands]], [[index]], [[overview]]
- 단일 소스 개념 후보: review-feedback-promotion
- 기록한 상충: project-04 스크립트 실패 메시지에 lecture-10이 권하는 WHY·FIX가 없음

## [2026-11-06] ingest | project-05 · 에이전트가 자신의 작업을 검증하도록 만들기

- 생성: [[projects/project-05]]
- 갱신: [[lectures/lecture-09]], [[lectures/lecture-10]](미수집 마커를 링크로), [[concepts/maker-checker-separation]], [[index]], [[overview]]
- 기록한 주의: 세 변형 점수는 서로 다른 평가자가 매긴 것(대조 실험 아님), 디렉터리 이름(grounded-qa)과 실제 기능(대화 기록 UI) 불일치, ko 표에는 점수 수치 없음

## [2026-11-06] ingest | lecture-11 · 에이전트의 런타임을 관측 가능하게 만들어라

- 생성: [[lectures/lecture-11]]
- 개념 승격(lecture-11 + project-05): [[concepts/sprint-contract]], [[concepts/evaluator-rubric]]
- 갱신: [[projects/project-04]](미수집 마커를 링크로), [[projects/project-05]], [[concepts/maker-checker-separation]], [[index]], [[overview]]
- 단일 소스 개념 후보: task-trace
- 기록한 상충: 계약 "협상"(lecture-11) vs 플래너 단독 작성(project-05), 루브릭 재현성 주장은 project-05 기록으로 검증되지 않음, 다크 모드 45분/15분은 가정 시나리오

## [2026-11-06] ingest | lecture-12 · 모든 세션은 클린 상태(clean state)로 끝나야 한다

- 생성: [[lectures/lecture-12]]
- 개념 승격: [[concepts/clean-state]](lecture-12 + project-03 + project-04), [[concepts/component-ablation]](lecture-02 + lecture-12, 기존 후보 승격)
- 갱신: [[lectures/lecture-02]](후보 마커를 링크로), [[projects/project-03]], [[projects/project-04]], [[concepts/definition-of-done]], [[index]], [[overview]]
- 단일 소스 개념 후보: quality-document
- 기록한 상충: 12주 수치를 "가상"과 "실제 사례"로 동시에 제시, project-04 체크리스트에 기능 목록 갱신 항목 없음

## [2026-11-06] ingest | project-06 · 완전한 에이전트 하네스 구축하기 (캡스톤)

- 생성: [[projects/project-06]]
- 개념 승격(lecture-12 + project-06): [[concepts/quality-document]]
- 갱신: [[lectures/lecture-11]], [[lectures/lecture-12]](미수집·후보 마커를 링크로), [[concepts/component-ablation]], [[concepts/evaluator-rubric]], [[concepts/clean-state]], [[index]], [[overview]]
- 직접 실행: `bash .../project-06/solution/scripts/benchmark.sh` → 41행 산술 오류로 exit 1. 품질 문서의 "All tasks complete"와 불일치. raw 변경 없음(mktemp 디렉터리만 사용)
- 기록한 상충: 체크리스트 41항목 vs "All 30 checks pass", 절제 실험 결과 파일 없음, ko에만 저장소 사용 표

## [2026-11-06] ingest | lecture-13 · 수동 프롬프팅에서 자율 루프로

- 생성: [[lectures/lecture-13]] (긴 소스: ko 약 46KB, ## 단위로 나눠 읽음)
- 갱신: [[concepts/ralph-loop]], [[concepts/maker-checker-separation]], [[index]], [[overview]]
- 단일 소스 개념 후보: loop-silent-costs
- 기록한 주의: /goal 출시 시기·Cherny 259 PR·OpenAI 100만 줄 등 외부 수치는 강의 주장(미검증), 도입부의 1~12강 요약이 실제 주제와 일부 다름

## [2026-11-06] ingest | project-07 · 첫 번째 자동 루프 구축하기

- 생성: [[projects/project-07]]
- 개념 승격(lecture-13 + project-07): [[concepts/loop-engineering]]
- 갱신: [[lectures/lecture-13]](미수집 마커를 링크로), [[concepts/ralph-loop]], [[concepts/maker-checker-separation]], [[index]], [[overview]]
- 기록한 상충: 원문이 안내하는 `projects/project-07/{starter,solution}`이 스냅숏(HEAD a9342ff)과 git 이력 어디에도 없음, 관련 강의 링크 제목이 실제 제목과 다름

## [2026-11-06] ingest | lecture-14 · 단일 루프에서 그래프 엔지니어링으로

- 생성: [[lectures/lecture-14]] (긴 소스: ko 약 45KB, ## 단위로 나눠 읽음)
- 갱신: [[lectures/lecture-13]](미수집 마커를 링크로), [[lectures/lecture-10]](OMC team-verify 서술 정정: verifier 상시 + 변경 성격별 추가 리뷰어, skills/team/SKILL.md 인용), [[concepts/maker-checker-separation]], [[concepts/loop-engineering]], [[index]], [[overview]]
- 단일 소스 개념 후보: anchors
- 기록한 상충: 6단계 설명과 en 참조 코드 maker_checker_graph.py 불일치(횟수 상한·attempts 증가·승인 정지 없음), ko에는 code/ 디렉터리 없음, 외부 날짜·수치는 강의 주장

## [2026-11-06] ingest | project-08 · 내 워크플로우를 그래프로 그리기

- 생성: [[projects/project-08]] (문서 기반 템플릿: 저장소 코드 없음)
- 개념 승격(lecture-14 + project-08): [[concepts/graph-engineering]]
- 갱신: [[lectures/lecture-14]](미수집 마커를 링크로), [[concepts/maker-checker-separation]], [[index]], [[overview]]
- 또한: [[projects/project-07]], [[lectures/lecture-13]]에 lecture-13 `code/` 템플릿(goal·loop-state·maker·checker) 기록

## [2026-11-06] ingest | claude-code · Claude Code의 하네스 설계

- 생성: [[harness/claude-code]]
- 갱신: 하네스 비교표 6열을 단일 데이터로 재생성([[harness/claude-code]], [[harness/oh-my-claudecode]], [[harness/gajae-code]]), [[harness/oh-my-claudecode]](미수집 마커를 링크로, "Claude Code가 주는 것 vs OMC가 더하는 것" 보강), [[index]], [[overview]]
- 주의: 5단계 compaction·7 permissions 모드 등은 원문이 인용한 VILA Lab 분석 기준이며 이 위키는 외부 문서를 직접 확인하지 않음

## [2026-11-06] ingest | codex · Codex의 하네스 설계

- 생성: [[harness/codex]]
- 갱신: 하네스 비교표 재생성(4개 하네스 페이지), [[concepts/map-not-manual]](Codex·Claude Code 사례), [[index]], [[overview]]
- 기록한 상충: OpenAI 100만 줄 사례의 기간("몇 주" vs lecture-13 "다섯 달"), gjc 저장소 AGENTS.md 206줄 vs 권장 약 100줄

## [2026-11-06] ingest | deepseek · DeepSeek Harness의 설계

- 생성: [[harness/deepseek]]
- 갱신: 하네스 비교표 재생성(5개 하네스 페이지), [[lectures/lecture-11]](하네스 실례 추가), [[index]], [[overview]]
- 단일 소스 개념 후보: capability-seam

## [2026-11-06] ingest | pi · Pi의 하네스 설계

- 생성: [[harness/pi]]
- 갱신: 하네스 비교표 재생성(6개 하네스 페이지, 미수집 열 없음), [[harness/gajae-code]](oh-my-pi와 Pi 관계 미확인 명시), [[index]], [[overview]]
- 기록한 의문점: 패키지 이름 두 가지(@earendil-works / @mariozechner), 확장 예시 이벤트 이름이 DeepSeek 글과 동일

## [2026-11-06] lint | 개념 승격: progressive-disclosure

- 승격(lecture-04 + claude-code + pi): [[concepts/progressive-disclosure]]
- 갱신: [[lectures/lecture-04]](후보 마커를 링크로), [[concepts/map-not-manual]], [[index]]

## [2026-11-06] lint | 일괄 ingest 종료 후 overview 재종합

- 갱신: [[overview]] 전체 재종합(코스 네 덩어리, 하네스 종합과 두 축, 가벼운/무거운 랄프톤 비교에서 기간 전제는 미검증으로 명시, 개념 지도를 하위 시스템별로 재배치해 신규 개념 8개 반영)

## [2026-11-06] lint | 하네스 개관 글을 aux 출처로 등록

- AGENTS.md sourceMap: claude-code·codex·deepseek·pi의 `aux`에 `docs/{ko,en}/harness-designs/index.md` 추가(소스 ID 수는 28 그대로)
- 갱신: [[harness/claude-code]](개관 글을 `sources`에 정식 인용)

## [2026-11-06] lint | 계획 Step 6 보강·Step 8 최종 게이트 · 오류 0

- Step 6: [[overview]] 학습 경로 14줄에 강의·실습 프로젝트·핵심 개념 링크를 모두 달고, 개념 지도에 하네스 실례 링크 추가
- Step 8: `--final`(전체 범위) 오류 0 · 경고 11(W2 보고만), `--self-test` 79/79, `--check-skills` 위반 0
- raw: 세 저장소 HEAD가 rawBaseline과 같음, porcelain 0, ignored 0, karpathy·가이드 sha256 같음, 최상위 5개 같음. 루트 커밋 없음
- 정리: Obsidian 일일 노트 플러그인이 루트에 만든 빈 `2026-11-06.md`(0바이트, E4)를 지우고 `wiki/.obsidian/core-plugins.json`에서 daily-notes를 끔

## [2026-11-06] query | Claude Code와 Codex 하네스 비교

- 생성: [[syntheses/claude-code-vs-codex]] (사용자 승인으로 저장)
- 갱신: [[harness/claude-code]]·[[harness/codex]] `related` 역링크, [[index]] 종합 섹션
