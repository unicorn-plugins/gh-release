---
name: recommend-template
description: Release 문서 구성 추천
user-invocable: true
---

# Recommend Template

[RECOMMEND-TEMPLATE 활성화]

## 목표

외부 GitHub Release 문서 모범 사례를 검색하여 최적의 Release 문서 구성안을 작성하고,
사용자 승인 후 templates/ 디렉토리에 Release 템플릿으로 등록함.

---

## 활성화 조건

"Release 구성 추천", "템플릿 추천" 키워드 감지 시 또는 `/github-release-manager:recommend-template` 호출 시.

---

## 에이전트 호출 규칙

### 에이전트 FQN

| 에이전트 | FQN |
|----------|-----|
| researcher | `github-release-manager:researcher:researcher` |
| executor | `github-release-manager:executor:executor` |

### 프롬프트 조립 절차

1. `agents/{agent-name}/` 에서 3파일 로드 (AGENT.md + agentcard.yaml + tools.yaml)
2. `gateway/runtime-mapping.yaml` 참조하여 구체화:
   - **모델 구체화**: agentcard.yaml의 `tier` → `tier_mapping`에서 모델 결정
   - **툴 구체화**: tools.yaml의 추상 도구 → `tool_mapping`에서 실제 도구 결정
   - **금지액션 구체화**: agentcard.yaml의 `forbidden_actions` → `action_mapping`에서 제외할 실제 도구 결정
   - **최종 도구** = (구체화된 도구) - (제외 도구)
3. 프롬프트 조립: AGENT.md + agentcard.yaml + tools.yaml을 합쳐 하나의 프롬프트로 구성
   - **구성 순서**: 공통 정적(runtime-mapping) → 에이전트별 정적(3파일) → 동적(작업 지시)
4. `Task(subagent_type=FQN, model=구체화된 모델, prompt=조립된 프롬프트)` 호출

---

## 워크플로우

### Phase 1: 외부 사례 검색 → Agent: researcher (`/oh-my-claudecode:research` 활용)

- **TASK**: 외부 GitHub Release 문서 모범 사례를 검색하여 최적의 Release 문서 구성안을 output/ 디렉토리에 작성
- **EXPECTED OUTCOME**: output/release-template-proposal.md 파일에 구성안이 작성됨 (섹션 목록, 설명, 예시 포함)
- **MUST DO**: 최소 3개 이상 외부 사례 참고, 구성안에 version/date/highlights/changelog/contributors 섹션 포함 여부 검토
- **MUST NOT DO**: Release를 직접 등록하거나 gh CLI를 실행하지 않음
- **CONTEXT**: output/ 디렉토리에 구성안 저장, 외부 GitHub 프로젝트의 Release 페이지 참고

### Phase 2: 사용자 검토 (ulw 활용)

output/ 디렉토리의 구성안을 사용자에게 제시하고 AskUserQuestion으로 승인/수정 요청.

- 승인 시 Phase 3으로 진행
- 수정 요청 시 피드백 반영 후 재제시

### Phase 3: 템플릿 등록 → Agent: executor (`/oh-my-claudecode:ralph` 활용)

- **TASK**: 승인된 구성안을 templates/ 디렉토리에 Release 템플릿으로 등록
- **EXPECTED OUTCOME**: templates/release-template.md 파일이 생성됨
- **MUST DO**: 구성안의 섹션 구조를 유지하며 Markdown 템플릿 형식으로 변환
- **MUST NOT DO**: 사용자가 승인하지 않은 내용을 추가하거나 수정하지 않음
- **CONTEXT**: output/release-template-proposal.md 참조, templates/ 디렉토리에 저장

### Phase 4: 검증 (`ulw` 활용)

templates/ 디렉토리에 템플릿 파일이 존재하는지 확인.

- templates/release-template.md 파일 존재 여부 확인
- 파일 내용이 승인된 구성안의 섹션 구조와 일치하는지 확인
- 검증 실패 시 Phase 3으로 복귀

---

## 완료 조건

- [ ] output/release-template-proposal.md에 구성안이 작성됨
- [ ] 사용자가 구성안을 승인함
- [ ] templates/release-template.md에 템플릿이 등록됨
- [ ] 검증 통과 (템플릿 파일 존재 및 구조 일치)

---

## 검증 프로토콜

완료 선언 전 아래 항목을 반드시 확인:

| # | 검증 항목 | 방법 |
|---|----------|------|
| 1 | 구성안 파일 존재 | output/release-template-proposal.md 파일 읽기 |
| 2 | 사용자 승인 | AskUserQuestion으로 승인 응답 확인 |
| 3 | 템플릿 파일 존재 | templates/release-template.md 파일 읽기 |
| 4 | 구조 일치 | 구성안의 섹션 구조와 템플릿의 섹션 구조 비교 |

**Iron Law**: 검증 증거 없이 완료 선언 불가.

---

## 상태 정리

완료 시 임시 상태 파일 없음 (상태 파일 미사용).
output/release-template-proposal.md는 참고 자료로 보존.

---

## 취소

사용자가 "cancelomc" 또는 "stopomc" 요청 시 즉시 중단.
진행 중인 Phase를 기록하여 재개 시 참조.

---

## 재개

마지막 완료된 Phase부터 재시작 가능:

| 중단 시점 | 재개 방법 |
|----------|----------|
| Phase 1 완료 후 | output/release-template-proposal.md 확인 후 Phase 2부터 재개 |
| Phase 2 완료 후 | 승인된 구성안으로 Phase 3부터 재개 |
| Phase 3 완료 후 | Phase 4 검증부터 재개 |

---

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 외부 사례를 최소 3개 이상 참고하여 구성안 작성 |
| 2 | 구성안에 version/date/highlights/changelog/contributors 섹션 포함 여부를 반드시 검토 |
| 3 | 사용자 승인 없이 템플릿을 등록하지 않음 — 반드시 AskUserQuestion으로 승인 확인 |
| 4 | 템플릿 등록 후 검증 Phase를 반드시 수행 |
| 5 | 모든 워크플로우 단계에서 대응하는 오케스트레이션 스킬을 활용 |

---

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | Release를 직접 등록하거나 gh CLI를 실행하지 않음 (이 스킬은 템플릿 관리만 수행) |
| 2 | 사용자가 승인하지 않은 내용을 템플릿에 추가하거나 수정하지 않음 |
| 3 | 에이전트 위임 시 내부 사고 방식이나 단계별 절차를 기술하지 않음 |
| 4 | 직접 애플리케이션 코드를 작성하거나 수정하지 않음 |

---

## 검증 체크리스트

- [ ] Frontmatter에 name, description 포함
- [ ] H1 타이틀 존재
- [ ] 목표 섹션 존재
- [ ] 에이전트 호출 규칙 섹션에 FQN 테이블과 프롬프트 조립 절차 포함
- [ ] 워크플로우의 Agent 위임 단계에 5항목(TASK, EXPECTED OUTCOME, MUST DO, MUST NOT DO, CONTEXT) 포함
- [ ] 위임 프롬프트에 HOW(방법) 없이 WHAT(목표)+제약만 기술
- [ ] 모든 워크플로우 단계에 오케스트레이션 스킬 활용이 명시됨
- [ ] 매핑 스킬이 없는 단계에 ulw 폴백이 적용됨
- [ ] 완료 조건, 검증 프로토콜, 상태 정리, 취소/재개 섹션 포함
- [ ] 프롬프트 구성 순서가 공통 정적 → 에이전트별 정적 → 동적 순서
- [ ] MUST 규칙 섹션이 파일 마지막 3개 섹션 중 첫 번째에 위치
- [ ] MUST NOT 규칙 섹션이 MUST 규칙 바로 다음에 위치
- [ ] 검증 체크리스트 섹션이 파일의 최종 섹션
