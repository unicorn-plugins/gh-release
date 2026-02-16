---
name: help
description: 사용 안내
user-invocable: true
type: utility
---

# Help

[HELP 활성화]

## 목표

GitHub Release Manager 플러그인의 사용 가능한 명령, 자동 라우팅 규칙, 주요 기능을 안내함.
런타임 상주 파일(CLAUDE.md)에 라우팅 테이블을 등록하는 대신,
이 스킬이 호출 시에만 토큰을 사용하여 사용자 발견성을 제공함.

## 활성화 조건

사용자가 `/github-release-manager:help` 호출 시 또는 "도움말", "help", "뭘 할 수 있어", "사용법" 키워드 감지 시.

## 명령어

| 명령 | 설명 |
|------|------|
| `/github-release-manager:help` | 이 도움말 표시 |

## 워크플로우

**중요: 추가적인 파일 탐색이나 에이전트 위임 없이, 아래 내용을 즉시 사용자에게 출력하세요.**

### 사용 가능한 명령

| 명령 | 설명 |
|------|------|
| `/github-release-manager:setup` | 플러그인 초기 설정 |
| `/github-release-manager:recommend-template` | Release 문서 구성 추천 |
| `/github-release-manager:create-release` | Release 문서 생성 |
| `/github-release-manager:edit-release` | Release 문서 수정 |
| `/github-release-manager:delete-release` | Release 문서 삭제 |
| `/github-release-manager:add-ext-skill` | 외부호출 스킬(ext-{대상플러그인}) 추가 |
| `/github-release-manager:remove-ext-skill` | 외부호출 스킬(ext-{대상플러그인}) 제거 |
| `/github-release-manager:help` | 사용 안내 |

### 자동 라우팅

다음과 같은 요청은 자동으로 적절한 스킬로 라우팅됩니다:

- "Release 구성 추천", "템플릿 추천", "Release 구조" → recommend-template
- "Release 생성", "릴리즈 만들어", "새 릴리즈" → create-release
- "Release 수정", "릴리즈 수정", "릴리즈 편집" → edit-release
- "Release 삭제", "릴리즈 삭제", "릴리즈 제거" → delete-release
- "설정", "setup", "초기 설정" → setup

### 시작하기

1. `/github-release-manager:setup`으로 초기 설정 수행
2. `/github-release-manager:recommend-template`로 Release 문서 구성 추천 받기
3. `/github-release-manager:create-release`로 Release 생성

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 워크플로우에 정의된 내용을 즉시 출력 (추가 파일 탐색 없음) |
| 2 | 모든 사용 가능한 명령과 자동 라우팅 규칙을 빠짐없이 안내 |
| 3 | 시작하기 섹션으로 신규 사용자에게 순서를 안내 |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | Agent에 위임하지 않음 (직결형 — 즉시 출력) |
| 2 | 추가적인 파일 탐색이나 코드베이스 조사를 수행하지 않음 |
| 3 | 워크플로우에 정의되지 않은 명령이나 기능을 안내하지 않음 |

## 검증 체크리스트

- [ ] Frontmatter에 name, description, user-invocable 포함
- [ ] 명령어 섹션에 help 명령이 정의되어 있는가
- [ ] 모든 플러그인 명령이 사용 가능한 명령 테이블에 포함되어 있는가
- [ ] 자동 라우팅 규칙이 core 스킬의 라우팅 테이블과 일치하는가
- [ ] Agent 위임 없이 즉시 출력하는 직결형인가
- [ ] 시작하기 안내가 포함되어 있는가
