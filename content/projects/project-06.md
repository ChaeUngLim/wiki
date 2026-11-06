---
type: "project"
title: "프로젝트 06. 완전한 에이전트 하네스 구축하기 (캡스톤)"
aliases:
  - "완전한 에이전트 하네스 구축하기"
  - "Build a Complete Agent Harness (Capstone)"
  - "캡스톤"
sources:
  - "raw/learn-harness-engineering/docs/ko/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/docs/en/projects/project-06-runtime-observability-and-debugging/index.md"
  - "raw/learn-harness-engineering/projects/project-06/starter/quality-document.md"
  - "raw/learn-harness-engineering/projects/project-06/solution/quality-document.md"
  - "raw/learn-harness-engineering/projects/project-06/solution/scripts/benchmark.sh"
  - "raw/learn-harness-engineering/projects/project-06/solution/scripts/cleanup-scanner.sh"
  - "raw/learn-harness-engineering/projects/project-06/solution/clean-state-checklist.md"
related:
  - "[[lectures/lecture-11]]"
  - "[[lectures/lecture-12]]"
  - "[[concepts/quality-document]]"
  - "[[concepts/clean-state]]"
  - "[[concepts/component-ablation]]"
tags:
  - "type/project"
  - "topic/observability"
  - "topic/clean-state"
updated: "2026-11-06"
---

# 프로젝트 06. 완전한 에이전트 하네스 구축하기 (캡스톤)

## 왕초보 설명

- **이게 뭐야?** 앞의 다섯 프로젝트에서 배운 하네스 장치를 **전부 합쳐** 같은 앱에 씌워 보는 졸업 과제(캡스톤)다. 약한 하네스로 한 번, 최강 하네스로 한 번, 정리 후 한 번 더 돌리고, 마지막에 장치를 하나씩 빼 보며 무엇이 진짜 중요한지 잰다.
- **비유:** 요리 수업 마지막 날 코스 요리. 배운 기술을 한 상에 다 올리고, 소스 하나씩 빼서 맛이 얼마나 달라지는지 본다.
- **나는 뭘 하면 돼?** 내 프로젝트에도 `AGENTS.md`·`feature_list.json`·`init.sh`·`session-handoff.md`·`clean-state-checklist.md`·품질 문서를 모두 두고, 측정 스크립트(벤치마크)를 만들어 **실제로 돌려 본다**. 문서에 "통과"라고 적힌 것도 직접 실행해 확인한다(아래 "직접 실행 결과" 참고).

## 목표

- 앞선 다섯 프로젝트의 모든 것을 조립하고, 전체 벤치마크를 실행하고, 품질이 유지되는지 정리 패스(cleanup pass)로 확인한다.
- 고정된 멀티 기능 작업 세트: 문서 임포트, 인덱싱, 인용 기반 Q&A, 런타임 관찰 가능성, 읽기 쉽고 재시작 가능한 저장소 상태.
- 실행 순서: 약한 하네스 베이스라인 → 가장 강한 하네스 → 정리 후 재실행 → **하네스 절제 실험**(구성 요소를 하나씩 제거, [[concepts/component-ablation]]).
- 하네스 메커니즘: 모든 메커니즘 + 관찰 가능성 + 절제 연구.
- 도구: Claude Code 또는 Codex, Git, Node.js + Electron, 품질 문서 템플릿, 평가자 루브릭, 앞선 프로젝트의 모든 하네스 구성 요소.

## 연결 강의

- [[lectures/lecture-11]]: 런타임 관측 가능성(구조화 로그)과 프로세스 관측 가능성(평가자 루브릭).
- [[lectures/lecture-12]]: 클린 상태, 품질 문서, 정리 스크립트, 하네스 단순화.

## starter→solution 핵심 변경점

starter는 "제품 기능은 거의 완성, 하네스 표면은 의도적으로 약함"이다. 기본 `AGENTS.md`(36줄)와 `quality-document.md`, `scripts/dev.js`뿐이다.

| 하네스 파일 | starter | solution |
| --- | --- | --- |
| `AGENTS.md` | 36줄 | 104줄. 시작 절차 7단계(AGENTS → CLAUDE → docs 3종 → `bash init.sh` → `feature_list.json`) |
| `CLAUDE.md` | 없음 | 86줄. Claude Code용 빠른 참조(빌드·실행 명령 등) |
| `feature_list.json` | 없음 | 15개 기능 모두 `pass` |
| `init.sh`, `session-handoff.md`, `claude-progress.md` | 없음 | 있음 |
| `clean-state-checklist.md` | 없음 | `- [ ]` 항목 41개 |
| `docs/` | 없음 | `ARCHITECTURE.md`, `PRODUCT.md`, `RELIABILITY.md` |
| `scripts/` | `dev.js` | + `benchmark.sh`(가져오기·인덱싱·질의·검증 4작업), `check-architecture.sh`, `cleanup-scanner.sh`(고아 내용 파일, 매달린 청크, 내용 없는 메타데이터, 청크 없는 인덱스 문서, 삭제된 문서를 가리키는 Q&A 기록 검사) |
| `evaluator-rubric.md` | 없음 | 종합 5.0/5 |

**[[concepts/quality-document|품질 문서]] 변화** (`quality-document.md`)

| 차원(일부) | starter | solution |
| --- | --- | --- |
| 기능 완성도 | D | A (15/15) |
| 구조화 로깅 | C (일부 서비스만) | A (JSON, 레벨, 서비스 태그) |
| 테스트 커버리지 | F (테스트 없음) | B (빌드 시 검사 + 벤치마크 스크립트로 런타임 확인) |
| 클린 상태 리셋 | F | A (확인 후 전체 초기화, 멱등) |
| 벤치마킹 | F | A |
| **종합** | **D+** | **A** |

starter 품질 문서에는 "치명적 격차"와 "행동 항목" 목록이 있어, 다음 세션이 무엇부터 할지 바로 보인다.

## 원문 경로

- ko: `raw/learn-harness-engineering/docs/ko/projects/project-06-runtime-observability-and-debugging/index.md`
- en: `raw/learn-harness-engineering/docs/en/projects/project-06-runtime-observability-and-debugging/index.md`
- 코드: `raw/learn-harness-engineering/projects/project-06/{starter,solution}`

**원문 간 차이·의문점**

- 디렉터리 이름은 `runtime-observability-and-debugging`, 제목은 "완전한 에이전트 하네스 (캡스톤)".
- ko만 "저장소에 포함된 프로젝트 사용하기" 표(starter·solution 내용과 비교 방법)와 "시너지·가장 큰 저하 요소 파악이 핵심"이라는 문단이 있다.
- 절제 실험은 과제로만 있고, 결과 파일은 저장소에 없다.
- solution 품질 문서는 "`clean-state-checklist.md`: All 30 checks pass"라고 하지만 체크리스트의 `- [ ]` 항목은 41개다.
- solution 품질 문서는 "`bash scripts/benchmark.sh`: All tasks complete"라고 하지만, `benchmark.sh`는 앱 서비스를 호출하지 않고 임시 디렉터리에 파일을 복사하고 awk로 청크 수를 **추정**하는 시뮬레이션이다.

**직접 실행 결과** (2026-11-06, macOS, 이 위키 작업 중 실행)

`bash raw/learn-harness-engineering/projects/project-06/solution/scripts/benchmark.sh`는 [1/4] 가져오기에서 파일 3개를 복사한 뒤 41행에서 `syntax error: invalid arithmetic operator`로 멈췄다(exit 1). `python3`로 얻은 소수 초를 bash 정수 산술 `$(( ... ))`에 넣기 때문이다. `python3`가 있는 환경에서는 "All tasks complete"가 재현되지 않는다. [[lectures/lecture-09]]가 경고한 "검증하지 않은 완료 선언"의 실례다.
