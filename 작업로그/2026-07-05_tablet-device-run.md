# 작업로그 · 실기기(태블릿) 실행 지원

- 날짜: 2026-07-05
- 브랜치/PR: `feature/tablet-device-run` → dev, PR #36 (머지 완료)
- 기기: 삼성 SM X930 (갤럭시 탭, Android 16), USB 디버깅 연결

## 무엇을 했나

앱을 처음으로 **실제 안드로이드 태블릿**에서 실행했다.

## 변경 내용

- `api_config.dart`: 서버 주소를 `--dart-define=API_BASE_URL`로 주입 가능하게 변경 (기본 localhost 유지)
  - 실기기에서 localhost는 "기기 자신"이라 PC IP가 필요함
- `AndroidManifest.xml`: INTERNET 권한, 개발용 cleartext(http) 허용, 앱 이름 '오븐업'
- `android/gradle.properties`: `kotlin.incremental=false`
  - 프로젝트(D:)와 pub 캐시(C:)가 다른 드라이브면 Kotlin 증분 컴파일이 깨지는 문제 해결

## 실행 방법 (재현)

1. 태블릿 USB 연결 + USB 디버깅 허용 (개발자 옵션)
2. 태블릿과 PC가 **같은 와이파이**에 있어야 함
3. 서버 실행 (8080) — 방화벽 인바운드 8080 허용 규칙 'OvenUp Server 8080' 추가돼 있음
4. 앱 실행:
   ```powershell
   cd D:\dev\appcheck   # 최신 코드 동기화 후
   D:\dev\flutter\bin\flutter.bat run -d <기기ID> --dart-define=API_BASE_URL=http://<PC IP>:8080
   ```
   이번 실행: PC IP `192.168.50.164`, 기기ID `R5KL3064NWM`

## 트러블슈팅 기록

| 증상 | 원인/해결 |
| --- | --- |
| device not authorized | 태블릿 화면의 USB 디버깅 허용 창에서 '허용' |
| Kotlin Daemon compilation failed (different roots) | D:/C: 드라이브 분리 → `kotlin.incremental=false` |
| 실기기에서 서버 연결 안 됨 | localhost 대신 PC IP 주입 + 방화벽 8080 인바운드 허용 |

## 참고

- 태블릿에서 소셜로그인은 키 미주입 시 dev mock으로 동작(정상). 실제 모바일 소셜로그인은 SDK 방식으로 추후 추가.
- 배포 시 cleartext(http) 허용은 제거하고 https로 전환해야 함.
