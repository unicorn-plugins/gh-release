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
- "Release 구성 추천", "템플릿 추천" → /gh-release:recommend-template
- "Release 생성", "릴리즈 만들어" → /gh-release:create-release
- "Release 수정", "릴리즈 수정" → /gh-release:edit-release
- "Release 삭제", "릴리즈 삭제" → /gh-release:delete-release
- "설정", "setup" → /gh-release:setup
- "도움말", "help" → /gh-release:help

## 운영 규칙

- **요구사항 변형 금지**: 원래의 요구사항을 변형해야 할 때는 반드시 사용자에게 확인 후 진행
