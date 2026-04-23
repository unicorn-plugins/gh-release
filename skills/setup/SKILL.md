---
name: setup
description: 플러그인 초기 설정
user-invocable: true
type: setup
---

# Setup

[SETUP 활성화]

## 목표

GitHub Release Manager 플러그인의 초기 환경을 설정함.
MCP 서버 설치, gh CLI 인증 확인, CLAUDE.md 라우팅 등록을 수행하여
플러그인이 정상 동작할 수 있는 기반을 구성함.

## 활성화 조건

사용자가 `/gh-release:setup` 호출 시 또는 "설정", "setup", "초기 설정" 키워드 감지 시.

## 사용자 상호작용

- Step 4에서 CLAUDE.md 적용 범위를 AskUserQuestion으로 질문
- 선택지: "모든 프로젝트 (전역)" / "이 프로젝트만 (로컬)"

## 워크플로우

### Step 1: install.yaml 읽기 (`ulw` 활용)

`gateway/install.yaml` 파일을 읽어 설치 대상 MCP 서버 목록을 확인함.

- 파일 경로: 플러그인 루트의 `gateway/install.yaml`
- 확인 대상: `mcp_servers` 목록의 name, config, scope, required 필드

### Step 2: MCP 서버 설치 (`ulw` 활용)

install.yaml에 정의된 MCP 서버를 `claude mcp add-json` 명령으로 등록함.

- 설치 전 `claude mcp list` 명령으로 기존 등록 여부를 확인하여 중복 설치 방지
- 이미 등록된 서버는 스킵하고 안내
- MCP 설정 파일 경로: `gateway/mcp/context7.json`
- 등록 명령 예시:
  ```
  claude mcp add-json context7 --scope user < gateway/mcp/context7.json
  ```

### Step 3: gh CLI 인증 확인 (`ulw` 활용)

`gh auth status` 명령으로 GitHub CLI 인증 상태를 확인함.

- 인증 완료 시: 상태 확인 결과 출력
- 미인증 시: `gh auth login` 실행 안내 메시지 제공
  ```
  gh CLI 인증이 필요합니다.
  터미널에서 다음 명령을 실행하세요: gh auth login
  ```

### Step 4: CLAUDE.md 등록 (`ulw` 활용)

사용자에게 플러그인 라우팅 테이블의 적용 범위를 질문함 (AskUserQuestion).

**질문**: "플러그인 라우팅을 어디에 등록할까요?"

| 선택지 | 대상 파일 |
|--------|----------|
| 모든 프로젝트 (전역) | `~/.claude/CLAUDE.md` |
| 이 프로젝트만 (로컬) | `./CLAUDE.md` |

선택된 파일에 다음 라우팅 테이블을 추가함:

```markdown
# gh-release 플러그인

## 사용 가능한 명령

| 명령 | 설명 |
|------|------|
| `/gh-release:setup` | 플러그인 초기 설정 |
| `/gh-release:recommend-template` | Release 문서 구성 추천 |
| `/gh-release:create-release` | Release 문서 생성 |
| `/gh-release:edit-release` | Release 문서 수정 |
| `/gh-release:delete-release` | Release 문서 삭제 |
| `/gh-release:help` | 사용 안내 |

## 자동 라우팅

다음과 같은 요청은 자동으로 gh-release 플러그인이 처리합니다:
- "Release 구성 추천", "템플릿 추천" → recommend-template
- "Release 생성", "릴리즈 만들어" → create-release
- "Release 수정", "릴리즈 수정" → edit-release
- "Release 삭제", "릴리즈 삭제" → delete-release
```

- 기존 CLAUDE.md 파일이 있으면 내용을 보존하고 끝에 추가
- 이미 동일한 라우팅 테이블이 존재하면 중복 추가하지 않음

### Step 5: 검증 (`ulw` 활용)

설치 결과를 확인하고 요약 보고함.

검증 항목:

| # | 항목 | 확인 방법 |
|---|------|----------|
| 1 | MCP 서버 등록 | `claude mcp list`에서 context7 확인 |
| 2 | gh CLI 인증 | `gh auth status` 성공 여부 |
| 3 | CLAUDE.md 등록 | 대상 파일에 라우팅 테이블 존재 확인 |

결과를 사용자에게 요약 형태로 보고:

```
## 설치 결과

| 항목 | 상태 |
|------|------|
| context7 MCP 서버 | 설치 완료 / 이미 설치됨 / 실패 |
| gh CLI 인증 | 인증됨 / 미인증 (수동 설정 필요) |
| CLAUDE.md 등록 | 등록 완료 / 이미 등록됨 |
```

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | install.yaml에 정의된 MCP 서버만 설치 (임의 추가 금지) |
| 2 | 기존 등록 여부를 확인하여 중복 설치 방지 |
| 3 | CLAUDE.md 적용 범위는 반드시 사용자에게 질문 후 결정 |
| 4 | 모든 Step 완료 후 검증 결과를 요약 보고 |
| 5 | gh CLI 미인증 시 수동 설정 안내만 제공 (자동 로그인 시도 금지) |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | Agent에 위임하지 않음 (직결형 — Gateway 도구 직접 사용) |
| 2 | install.yaml에 없는 서버나 도구를 설치하지 않음 |
| 3 | 사용자 확인 없이 CLAUDE.md를 수정하지 않음 |
| 4 | 애플리케이션 코드를 작성하거나 수정하지 않음 (설정 파일만 허용) |

## 검증 체크리스트

- [ ] Frontmatter에 name, description, user-invocable 포함
- [ ] 워크플로우가 번호 기반 순차(Step 1~5)로 구성되어 있는가
- [ ] 모든 Step에 스킬 부스팅(`ulw`)이 명시되어 있는가
- [ ] 사용자 상호작용 섹션에 AskUserQuestion 사용이 정의되어 있는가
- [ ] Agent 위임 없이 Gateway 도구를 직접 사용하는가 (직결형)
- [ ] 중복 설치 방지 로직이 포함되어 있는가
- [ ] 검증 단계(Step 5)에서 설치 결과를 요약 보고하는가
- [ ] `disable-model-invocation: true`를 사용하지 않았는가
