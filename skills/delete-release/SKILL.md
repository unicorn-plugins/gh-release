---
name: delete-release
description: Release 문서 삭제
user-invocable: true
type: orchestrator
---

# Delete Release

[DELETE-RELEASE 활성화]

## 목표

특정 버전의 GitHub Release 문서를 사용자 확인 후 삭제하고
`gh release delete`로 GitHub에서 제거함.

## 활성화 조건

"Release 삭제", "릴리즈 삭제" 키워드 감지 시 또는 `/github-release-manager:delete-release` 호출 시.

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN |
|----------|-----|
| executor | `github-release-manager:executor:executor` |

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

### Phase 1: 버전 확인 (ulw 활용)

AskUserQuestion으로 사용자에게 삭제할 Release 버전을 문의함.
삭제할 Release 버전 (예: v1.0.0)을 사용자가 직접 입력하도록 안내.

### Phase 2: 삭제 확인 (ulw 활용)

AskUserQuestion으로 사용자에게 삭제 최종 확인을 요청함.
- "정말로 {버전} Release를 삭제하시겠습니까?" 형식으로 문의
- 사용자가 명시적으로 승인해야만 다음 단계 진행

### Phase 3: Release 삭제 → Agent: executor (`/oh-my-claudecode:ralph` 활용)

- **TASK**: 사용자가 확인한 버전의 Release를 gh release delete 명령으로 삭제
- **EXPECTED OUTCOME**: GitHub에서 해당 Release가 삭제됨
- **MUST DO**: `gh release delete {버전} --yes` 형식으로 실행
- **MUST NOT DO**: 사용자가 확인하지 않은 버전 삭제 금지, Release 수정 금지
- **CONTEXT**: Phase 1에서 수집한 버전, Phase 2에서 사용자 최종 확인 완료

### Phase 4: 검증 (`ulw` 활용)

`gh release list`로 삭제된 버전이 목록에 없는지 확인 후 사용자에게 보고함.
추가로 `gh release view {버전}` 명령이 실패하는지 확인 (Release not found 에러 예상).

## 완료 조건

- [ ] 사용자가 삭제할 버전을 명확히 지정함
- [ ] 사용자가 삭제를 최종 확인함
- [ ] `gh release delete` 명령으로 Release 삭제 완료
- [ ] `gh release list`로 삭제 확인됨
- [ ] 삭제 결과가 사용자에게 보고됨

## 검증 프로토콜

완료 선언 전 반드시 `gh release list` 명령을 실행하여 삭제를 확인함.
검증 없이 완료 선언 불가 (Iron Law).

검증 항목:

| 항목 | 검증 방법 | 성공 기준 |
|------|----------|----------|
| Release 삭제 | `gh release list` | 삭제된 버전이 목록에 없음 |
| 태그 확인 | `gh release view {버전}` | 명령 실패 (Release not found) |

## 상태 정리

완료 시 임시 파일 없음.
상태 파일 미사용.

## 취소

사용자가 "cancelomc" 또는 "stopomc" 요청 시 즉시 중단.
진행 중인 Phase에서 중단하며, GitHub Release는 삭제되지 않은 상태로 유지됨.

## 재개

마지막 완료된 Phase부터 재시작 가능.
Phase 3 실패 시 동일한 버전으로 재시도 (단, 사용자 확인 재수행 필요).

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 삭제 전 사용자에게 반드시 최종 확인 요청 |
| 2 | `gh release delete {버전} --yes` 명령으로 Release 삭제 |
| 3 | 삭제 완료 후 `gh release list`로 검증 수행 |
| 4 | 모든 워크플로우 단계에서 오케스트레이션 스킬 활용 필수 (매핑 없으면 `ulw` 폴백) |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | 사용자 최종 확인 없이 삭제 실행 금지 |
| 2 | 사용자가 지정하지 않은 버전 삭제 금지 |
| 3 | 검증 없이 완료 선언 금지 |
| 4 | 에이전트 위임 시 HOW(방법)를 상세히 기술 금지 (WHAT+제약만 명시) |

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
