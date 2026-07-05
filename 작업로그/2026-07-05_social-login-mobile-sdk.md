# 작업로그 · 모바일(태블릿) 카카오·네이버 SDK 실제 로그인

- 날짜: 2026-07-05
- 브랜치/PR: `feature/social-login-mobile-sdk` → dev, PR #37 (머지), `fix/android-resvalues` PR #38
- **E2E 확인: 태블릿(SM X930)에서 실제 카카오/네이버 로그인 성공** ✅

## 무엇을 했나

웹 리다이렉트 방식에 이어, 실제 안드로이드 기기에서도 카카오/네이버 공식 SDK로 로그인되게 했다.
서버는 기존 accessToken 검증 경로를 그대로 재사용(변경 없음).

## 변경 내용 (앱)

- `kakao_flutter_sdk_user ^2.0.0`, `flutter_naver_login ^2.1.1` 추가
- 신규 `data/social_auth_mobile.dart`: SDK 로그인 → 액세스 토큰 → 서버 전달
  - 카카오: 카카오톡 설치 시 앱 로그인, 아니면 계정(브라우저) 로그인
  - 네이버: `FlutterNaverLogin.logIn()` (타입은 `interface/types/` 하위에서 직접 import 필요)
- `main.dart`: `KakaoSdk.init(nativeAppKey:)` — 키 주입 시에만
- `MainActivity`: `FlutterFragmentActivity`로 전환 (네이버 SDK 요구)
- 매니페스트: 카카오 커스텀 스킴 activity(`kakao{키}://oauth`), 네이버 meta-data
- 키 주입: 카카오 네이티브 키는 `manifestPlaceholder`+`--dart-define`, 네이버는 `resValue`(환경변수) — 커밋되는 파일에 키 없음
- `build.gradle.kts`: `buildFeatures { resValues = true }` (최신 AGP 기본 비활성)

## 콘솔 등록 (완료)

- 카카오: Android 플랫폼 — 패키지명 `com.ovenup.oven_up_app`, 키 해시 `3cokq5ladLA45kTNW7hsJ+iOyCk=` (디버그)
- 네이버: Android 환경 — 패키지명 동일
- ⚠️ 배포(릴리즈 서명) 시 릴리즈 키스토어의 키 해시를 카카오에 추가 등록해야 함

## 트러블슈팅 기록

| 증상 | 원인/해결 |
| --- | --- |
| `defaultConfig contains custom resource values, but the feature is disabled` | 최신 AGP는 resValues 기본 꺼짐 → `buildFeatures { resValues = true }` |
| Kotlin `different roots` 컴파일 실패 (D:/C: 드라이브) | pub 캐시를 D:로 이동 (`PUB_CACHE=D:\dev\pubcache`) |
| `Storage ... is already registered` | 이전 실패 빌드의 Kotlin 데몬이 캐시 점유 → java(kotlin/gradle) 데몬 종료 후 build 삭제 재빌드 |
| PowerShell .ps1에서 한글 경로 깨짐 | 스크립트는 ASCII만 사용, 한글 경로는 `Resolve-Path 'D:\ai*\...'` 와일드카드로 해석 |

## 실행 방법

`D:\dev\run-ovenup-tablet.ps1` 하나로: secrets 로드 → appcheck 동기화 → PC IP 감지 → 태블릿 실행.
서버는 `D:\dev\srvchkN`에서 `--app.social.mock=false`로 실행 (secrets의 키를 환경변수로).

## 남은 것

- 릴리즈 서명 시 카카오 키 해시 추가 등록
- iOS는 앱 배포 단계에서 별도 설정 (Info.plist 스킴 등)
