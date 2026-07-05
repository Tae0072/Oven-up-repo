# 작업로그 · 소셜로그인(카카오·네이버) real 전환 (외부 연동 1/3)

- 날짜: 2026-07-05
- 브랜치/PR: `feature/social-login-real` → dev, PR #35
- 관련 가이드: `가이드/소셜로그인_연결.md`, `가이드/외부연동_인수인계.md`

## 무엇을 했나

외부 연동 3종 중 첫 번째인 소셜로그인을 mock → real 전환 가능한 구조로 완성.
**웹 OAuth 인가코드(redirect) 방식**을 채택했다.

> 왜 이 방식? 앱이 현재 웹으로 돌아가는데, 네이버는 웹용 Flutter SDK가 없다.
> 표준 OAuth 리다이렉트 방식이면 카카오/네이버를 같은 구조로 처리할 수 있고,
> 네이버 Client Secret이 **서버에만** 있어 보안상도 올바르다.

## 로그인 흐름 (real 모드)

1. 앱: "카카오로 시작하기" 클릭 → 카카오/네이버 로그인 페이지로 이동
2. 사용자 로그인 → 우리 앱 주소(`http://localhost:8091`)로 `?code=...&state=...` 반환
3. 앱: 시작 시 code를 감지(state 검증 포함) → 서버 `POST /api/auth/social/{provider}` 에 code 전달
4. 서버: code → 액세스 토큰 교환(RealSocialAuthCodeExchanger) → 사용자 조회(RealSocialProfileVerifier) → 회원 연결/자동가입 → JWT 발급

## 변경 내용

### 서버
- `SocialLoginRequest`: `accessToken` 외에 `code`/`redirectUri`/`state` 추가 (두 방식 병행)
- 신규 `social/SocialAuthCodeExchanger` 인터페이스
  - `MockSocialAuthCodeExchanger` (기본): code를 그대로 토큰으로 → 기존 mock 흐름·테스트 유지
  - `RealSocialAuthCodeExchanger` (`app.social.mock=false`): 카카오 `kauth.kakao.com/oauth/token`, 네이버 `nid.naver.com/oauth2.0/token` 호출
- `AuthService`: accessToken이 없고 code가 오면 먼저 토큰으로 교환 후 기존 검증 재사용
- `application.properties`: 키 자리 추가 — **값은 전부 환경변수** (`KAKAO_REST_API_KEY`, `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`)

### 앱 (Flutter 웹)
- 신규 `data/social_auth.dart`: 리다이렉트 시작, state(CSRF) 생성·검증, 복귀 콜백 캡처
- `data/social_auth_platform_web.dart` / `_stub.dart`: 웹 전용 브라우저 API 분리(조건부 import)
- `main.dart`: 시작 시 `SocialAuth.captureRedirectCallback()` 1회 호출
- `login_page.dart`: 키가 주입돼 있으면 실제 로그인, 복귀 시 자동으로 code를 서버에 전달
- 키 주입: `--dart-define=KAKAO_REST_API_KEY=... --dart-define=NAVER_CLIENT_ID=...`
  - **키가 없으면 자동으로 기존 dev mock 로그인** → 테스트/CI 영향 없음
- `pubspec.yaml`: `web` 패키지 추가

## 검증

- 서버: `D:\dev\srvchkN` 에서 `gradlew build` 통과 (기존 소셜 테스트 4건 포함)
- 앱: `D:\dev\appcheck` 에서 `flutter analyze` 0건, `flutter test` 6/6 통과
- PR CI: secret-scan 통과(키 커밋 없음 확인)

## 실행 방법 (real 모드)

로컬 실행 스크립트(키 포함, **레포 밖**): `D:\dev\run-ovenup-server-real.ps1`, `D:\dev\run-ovenup-app-real.ps1`

수동 실행 시:
```powershell
# 서버 (ASCII 복사본에서)
$env:KAKAO_REST_API_KEY='(키)'; $env:NAVER_CLIENT_ID='(키)'; $env:NAVER_CLIENT_SECRET='(키)'
$env:GRADLE_USER_HOME='D:\dev\gradlehome'
cd D:\dev\srvchkN; .\gradlew.bat bootRun --args='--app.social.mock=false'

# 앱 (웹, 8091)
cd D:\dev\appcheck
D:\dev\flutter\bin\flutter.bat run -d web-server --web-port 8091 `
  --dart-define=KAKAO_REST_API_KEY=(키) --dart-define=NAVER_CLIENT_ID=(키)
```

## 개발자 콘솔 등록값 (사용자 확인 필요)

- 카카오: 플랫폼 Web 도메인 `http://localhost:8091`, **Redirect URI `http://localhost:8091`**, 카카오 로그인 ON, 동의항목 닉네임(+이메일)
- 네이버: 네아로 사용, 서비스 URL `http://localhost:8091`, **콜백 URL `http://localhost:8091`**

## 남은 것 / 다음 단계

- [ ] 실제 카카오/네이버 계정으로 브라우저에서 로그인 E2E 확인 (콘솔 등록값 위와 일치해야 함)
- [ ] 외부 연동 2/3: 결제(PortOne) — `가이드/결제_연동.md`
- [ ] 외부 연동 3/3: 푸시(FCM) — `가이드/푸시_알림_FCM.md`
- 모바일(안드로이드/iOS) 소셜 로그인은 앱 배포 단계에서 SDK 방식(accessToken 직접 전달)으로 추가하면 됨 — 서버는 이미 두 방식 모두 지원
