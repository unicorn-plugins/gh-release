---
name: create-release
description: Release 문서 생성
user-invocable: true
type: orchestrator
---

# Create Release

[CREATE-RELEASE 활성화]

## 목표

템플릿 기반으로 마지막 Release 이후 변경사항(커밋 로그 + PR)을 취합하여
새로운 GitHub Release 문서를 생성하고 `gh release create`로 등록함.

## 활성화 조건

"Release 생성", "릴리즈 만들어" 키워드 감지 시 또는 `/gh-release:create-release` 호출 시.

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN |
|----------|-----|
| explorer | `gh-release:explorer:explorer` |
| executor | `gh-release:executor:executor` |

### 프롬프트 조립

1. `agents/{agent-name}/` 에서 3파일 로드 (AGENT.md + agentcard.yaml + tools.yaml)
2. `gateway/runtime-mapping.yaml` 참조하여 구체화:
   - **모델 구체화**: agentcard.yaml의 `tier` → `tier_mapping`에서 모델 결정
   - **툴 구체화**: tools.yaml의 추상 도구 → `tool_mapping`에서 실제 도구 결정
   - **금지액션 구체화**: agentcard.yaml의 `forbidden_actions` → `action_mapping`에서 제외할 실제 도구 결정
   - **최종 도구** = (구체화된 도구) - (제외 도구)
3. 프롬프트 조립: AGENT.md + agentcard.yaml + tools.yaml을 합쳐 하나의 프롬프트로 구성
   - **구성 순서**: 공통 정적(runtime-mapping) → 에이전트별 정적(3파일) → 동적(작업 지시)
4. `Task(subagent_type=FQN, model=구체화된 모델, prompt=조립된 프롬프트)` 호출

## 워크플로우

### Phase 1: 템플릿 확인 → Agent: explorer (`ulw` 활용)

- **TASK**: templates/ 디렉토리에 Release 템플릿이 존재하는지 확인
- **EXPECTED OUTCOME**: 템플릿 존재 여부와 파일 경로 보고
- **MUST DO**: templates/ 디렉토리 전체 탐색
- **MUST NOT DO**: 파일 수정 금지
- **CONTEXT**: templates/ 디렉토리 대상

### Phase 1-1: 템플릿 없음 분기 → Skill: recommend-template

Phase 1에서 템플릿 미존재 시에만 실행.

- **INTENT**: Release 템플릿이 없으므로 구성 추천 워크플로우 실행
- **ARGS**: 사용자 요청 원문 전달
- **RETURN**: templates/ 디렉토리에 템플릿이 등록된 상태

### Phase 2: 버전 문의 (ulw 활용)

AskUserQuestion으로 사용자에게 Release 버전을 문의함.
사용자가 직접 입력하도록 안내 (예: v1.0.0, v1.1.0).

### Phase 3: 변경사항 취합 → Agent: explorer (`/oh-my-claudecode:deepsearch` 활용)

- **TASK**: 마지막 Release 이후의 Git 커밋 로그와 병합된 PR 정보를 취합
- **EXPECTED OUTCOME**: 변경사항 보고서 (커밋 목록, PR 목록, 카테고리 분류)
- **MUST DO**: 마지막 Release 태그 이후 모든 커밋과 병합된 PR 수집, 카테고리별 분류 (Features, Bug Fixes, Improvements 등)
- **MUST NOT DO**: 파일 수정 금지, Release 등록 금지
- **CONTEXT**: 프로젝트 Git 저장소, gh CLI로 PR 정보 조회

### Phase 4: Release 생성 → Agent: executor (`/oh-my-claudecode:ralph` 활용)

- **TASK**: 템플릿과 변경사항을 조합하여 Release 문서를 작성하고 gh release create로 GitHub에 등록
- **EXPECTED OUTCOME**: GitHub에 새 Release가 등록되고 URL이 반환됨
- **MUST DO**: 템플릿 구조를 유지하며 변경사항 반영, `gh release create {버전} --title "{버전}" --notes-file {임시파일}` 형식으로 실행
- **MUST NOT DO**: 사용자가 지정하지 않은 버전 사용 금지, 템플릿 구조 변형 금지
- **CONTEXT**: Phase 2의 버전, Phase 3의 변경사항 보고서, templates/ 디렉토리의 템플릿 참조

### Phase 5: 검증 (`ulw` 활용)

`gh release view {버전}`으로 등록 확인 후 Release URL을 사용자에게 보고함.

## 완료 조건

- [ ] templates/ 디렉토리에 템플릿 존재 확인 (없으면 recommend-template 완료)
- [ ] 사용자가 지정한 버전으로 Release 생성됨
- [ ] 변경사항(커밋 + PR)이 Release 문서에 반영됨
- [ ] `gh release view {버전}` 명령으로 등록 확인됨
- [ ] Release URL이 사용자에게 보고됨

## 검증 프로토콜

완료 선언 전 반드시 `gh release view {버전}` 명령을 실행하여 등록을 확인함.
검증 없이 완료 선언 불가 (Iron Law).

검증 항목:

| 항목 | 검증 방법 | 성공 기준 |
|------|----------|----------|
| Release 등록 | `gh release view {버전}` | 명령 성공 + Release 정보 출력 |
| 버전 일치 | 출력된 Release 태그 확인 | 사용자 지정 버전과 일치 |
| 문서 내용 | Release body 확인 | 변경사항이 포함되어 있음 |

## 상태 정리

완료 시 임시 파일(Release notes 임시 파일) 삭제.
상태 파일 미사용.

## 취소

사용자가 "cancelomc" 또는 "stopomc" 요청 시 즉시 중단.
진행 중인 Phase에서 중단하며, GitHub에 미등록 상태로 종료됨.

## 재개

마지막 완료된 Phase부터 재시작 가능.
Phase 4 실패 시 동일한 버전과 변경사항으로 재시도.

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 템플릿 미존재 시 반드시 recommend-template 스킬을 호출하여 템플릿 등록 후 진행 |
| 2 | 사용자가 지정한 버전을 정확히 사용하여 Release 생성 |
| 3 | 마지막 Release 태그 이후의 모든 커밋과 병합된 PR을 취합 |
| 4 | 템플릿 구조를 유지하며 변경사항 반영 |
| 5 | `gh release create` 명령으로 GitHub에 등록 |
| 6 | 등록 완료 후 `gh release view`로 검증 수행 |
| 7 | 모든 워크플로우 단계에서 오케스트레이션 스킬 활용 필수 (매핑 없으면 `ulw` 폴백) |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 사용자가 지정하지 않은 버전으로 Release 생성 금지 |
| 2 | 템플릿 구조를 임의로 변형 금지 |
| 3 | 검증 없이 완료 선언 금지 |
| 4 | 변경사항 취합 없이 Release 문서 생성 금지 |
| 5 | explorer 에이전트가 파일을 수정하는 것을 허용 금지 |
| 6 | 에이전트 위임 시 HOW(방법)를 상세히 기술 금지 (WHAT+제약만 명시) |

## 검증 체크리스트

- [ ] Frontmatter에 name, description 포함
- [ ] H1 타이틀 존재
- [ ] 목표 섹션 존재
- [ ] 에이전트 호출 규칙 섹션에 FQN 테이블 포함
- [ ] 프롬프트 조립 절차가 4단계로 기술됨
- [ ] `→ Agent:` 마커가 있는 워크플로우 단계에 5항목이 포함되는가
- [ ] `→ Skill:` 마커가 있는 워크플로우 단계에 3항목이 포함되는가
- [ ] 모든 워크플로우 단계에 오케스트레이션 스킬 활용이 명시되었는가
- [ ] 매핑 스킬이 없는 단계에 `ulw` 폴백이 적용되었는가
- [ ] 완료 조건, 검증 프로토콜, 상태 정리, 취소/재개 섹션 포함
- [ ] 프롬프트 구성 순서가 공통 정적 → 에이전트별 정적 → 동적 순서인가
- [ ] `## MUST 규칙` 섹션이 파일 마지막 3개 섹션 중 첫 번째에 위치하는가
- [ ] `## MUST NOT 규칙` 섹션이 `## MUST 규칙` 바로 다음에 위치하는가
- [ ] `## 검증 체크리스트` 섹션이 파일의 최종 섹션인가
