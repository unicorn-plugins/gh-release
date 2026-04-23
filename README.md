# GitHub Release Manager

> GitHub Release 문서를 자동으로 생성·수정·삭제하고 구성을 추천하는 Claude Code 플러그인

---

## 개요

GitHub Release Manager는 GitHub Repository의 Release 문서 관리를 자동화하는 DMAP 플러그인임.
릴리스 노트 구성 추천부터 생성·수정·삭제까지 전 과정을 멀티에이전트가 수행하며, Context7 MCP를 통해 최신 GitHub Release 문서 작성 가이드를 자동으로 참조함.

**주요 기능:**
- 프로젝트 특성 분석 기반 Release 문서 구성 자동 추천
- 커밋 히스토리·PR·이슈 분석을 통한 Release 문서 자동 생성
- 기존 Release 문서 분석 및 수정 (버전 업데이트, 내용 추가·삭제)
- Release 문서 삭제 및 태그 관리 자동화

---

## 설치

### 사전 요구사항

- [Claude Code](https://claude.com/claude-code) CLI 설치
- [GitHub CLI (gh)](https://cli.github.com/) 설치 및 인증 완료
- Context7 MCP 서버 설치 (공식 문서 조회용)

### 플러그인 설치

**방법 1: 마켓플레이스 — GitHub (권장)**

```bash
# 1. GitHub 저장소를 마켓플레이스로 등록
claude plugin marketplace add unicorn-plugins/gh-release

# 2. 플러그인 설치 (형식: {플러그인명}@{마켓플레이스명})
claude plugin install gh-release@gh-release

# 3. 설치 확인
claude plugin list
```

**방법 2: 마켓플레이스 — 로컬**

```bash
# 1. 로컬 경로를 마켓플레이스로 등록
claude plugin marketplace add ./gh-release

# 2. 플러그인 설치
claude plugin install gh-release@gh-release

# 3. 설치 확인
claude plugin list
```

> **설치 후 setup 스킬 실행:**
> ```
> /gh-release:setup
> ```
> - `gateway/install.yaml`을 읽어 필수 MCP/LSP 서버 자동 설치
> - 설치 결과 검증 (`required: true` 항목 실패 시 중단)
> - 플러그인 활성화 확인 (스킬 자동 탐색)
> - 적용 범위 선택 (모든 프로젝트 / 현재 프로젝트만)

### 처음 GitHub을 사용하시나요?

다음 가이드를 참고하세요:

- [GitHub 계정 생성 가이드](https://github.com/unicorn-plugins/gen-ma-plugin/blob/main/resources/guides/github/github-account-setup.md)
- [Personal Access Token 생성 가이드](https://github.com/unicorn-plugins/gen-ma-plugin/blob/main/resources/guides/github/github-token-guide.md)
- [GitHub Organization 생성 가이드](https://github.com/unicorn-plugins/gen-ma-plugin/blob/main/resources/guides/github/github-organization-guide.md)

---

## 업그레이드

### Git Repository 마켓플레이스

저장소의 최신 커밋을 가져와 플러그인을 업데이트함.

```bash
# 마켓플레이스 업데이트 (최신 커밋 반영)
claude plugin marketplace update gh-release

# 플러그인 재설치
claude plugin install gh-release@gh-release

# 설치 확인
claude plugin list
```

> **버전 고정**: `marketplace.json`에 특정 `ref`/`sha`가 지정된 경우,
> 저장소 관리자가 해당 값을 업데이트해야 새 버전이 반영됨.

> **갱신이 반영되지 않는 경우**: 플러그인을 삭제 후 재설치함.
> ```bash
> claude plugin remove gh-release@gh-release
> claude plugin marketplace update gh-release
> claude plugin install gh-release@gh-release
> ```

### 로컬 마켓플레이스

로컬 경로의 파일을 직접 갱신한 뒤 마켓플레이스를 업데이트함.

```bash
# 1. 로컬 플러그인 소스 갱신 (예: git pull 또는 파일 복사)
cd ./gh-release
git pull origin main

# 2. 마켓플레이스 업데이트
claude plugin marketplace update gh-release

# 3. 플러그인 재설치
claude plugin install gh-release@gh-release
```

> **갱신이 반영되지 않는 경우**: 플러그인을 삭제 후 재설치함.
> ```bash
> claude plugin remove gh-release@gh-release
> claude plugin marketplace update gh-release
> claude plugin install gh-release@gh-release
> ```

> **setup 재실행**: 업그레이드 후 `gateway/install.yaml`에 새 도구가 추가된 경우
> `/gh-release:setup`을 재실행하여 누락된 도구를 설치할 것.

---

## 사용법

### 슬래시 명령

| 명령 | 설명 |
|------|------|
| `/gh-release:setup` | 플러그인 초기 설정 |
| `/gh-release:recommend-template` | Release 문서 구성 추천 |
| `/gh-release:create-release` | Release 문서 생성 |
| `/gh-release:edit-release` | Release 문서 수정 |
| `/gh-release:delete-release` | Release 문서 삭제 |
| `/gh-release:help` | 사용 안내 |

### 사용 예시

```
사용자: Release 문서 구성 추천해줘
→ recommend-template 스킬이 프로젝트 분석 후 최적 구성 제안
  (버전 네이밍 규칙, 섹션 구조, 작성 가이드 포함)
```

```
사용자: v1.2.0 릴리스 노트 작성해줘
→ create-release 스킬이 커밋·PR·이슈 분석 후 자동 생성
  (변경사항, 신규 기능, 버그 수정, Breaking Changes 자동 분류)
```

```
사용자: v1.1.0 릴리스 노트에 보안 패치 내용 추가해줘
→ edit-release 스킬이 기존 Release 분석 후 내용 추가
```

```
사용자: v0.9.0-beta 릴리스 삭제해줘
→ delete-release 스킬이 Release 및 태그 삭제 (선택적)
```

---

## 에이전트 구성

| 에이전트 | 티어 | 역할 |
|----------|------|------|
| researcher | MEDIUM | Context7 통해 GitHub Release 작성 가이드 조회 |
| executor | MEDIUM | gh CLI로 Release 생성·수정·삭제 실행 |
| explorer | LOW | 프로젝트 구조·커밋·PR·이슈 분석 |

---

## 요구사항

### 필수 도구

| 도구 | 유형 | 용도 |
|------|------|------|
| GitHub CLI (gh) | Custom | Release 생성·수정·삭제 명령 실행 |
| Context7 | MCP | GitHub Release 공식 문서 조회 |

### 런타임 호환성

| 런타임 | 지원 |
|--------|:----:|
| Claude Code | ✅ |
| Codex CLI | 미검증 |
| Gemini CLI | 미검증 |

---

## 디렉토리 구조

```
gh-release/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── core/
│   │   └── SKILL.md
│   ├── setup/
│   │   └── SKILL.md
│   ├── help/
│   │   └── SKILL.md
│   ├── recommend-template/
│   │   └── SKILL.md
│   ├── create-release/
│   │   └── SKILL.md
│   ├── edit-release/
│   │   └── SKILL.md
│   └── delete-release/
│       └── SKILL.md
├── agents/
│   ├── researcher/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   ├── executor/
│   │   ├── AGENT.md
│   │   ├── agentcard.yaml
│   │   └── tools.yaml
│   └── explorer/
│       ├── AGENT.md
│       ├── agentcard.yaml
│       └── tools.yaml
├── gateway/
│   ├── install.yaml
│   └── runtime-mapping.yaml
├── commands/
│   ├── setup.md
│   ├── help.md
│   ├── recommend-template.md
│   ├── create-release.md
│   ├── edit-release.md
│   └── delete-release.md
└── README.md
```

---

## 라이선스

MIT License - Unicorn Inc.
