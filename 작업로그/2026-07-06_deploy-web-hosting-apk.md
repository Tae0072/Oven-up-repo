# 배포4: Flutter 웹 호스팅 + 콘솔 주소 등록 + 태블릿 APK (2026-07-06)

## 결과
- **손님용 웹: https://ven-up.web.app** (Firebase Hosting, 프로젝트 ven-up)
- **API 서버: https://ovenup.duckdns.org** (웹·앱 공통, --dart-define으로 주입)
- 웹에서 카카오 소셜로그인 E2E 성공 (실제 계정 로그인 → 홈 화면 진입 확인)
- 태블릿(R5KL3064NWM)에 release APK 설치 완료 — 화면 동작은 실기기에서 확인 필요

## 한 일
1. Firebase CLI 설치(npm) + 로그인(rkdxodh41@gmail.com), `firebase.json`/`.firebaserc` 생성
2. `flutter build web --release` (API_BASE_URL + KAKAO_REST_API_KEY + NAVER_CLIENT_ID 주입) → `firebase deploy`
   - ⚠️ 처음에 API_BASE_URL만 주입해서 소셜로그인이 mock 모드로 배포됨 → 키 포함 재빌드로 해결
3. 카카오 콘솔: REST API 키의 카카오 로그인 리다이렉트 URI에 `https://ven-up.web.app` 추가 (기존 localhost:8091 유지)
4. PortOne 콘솔: 변경 불필요 확인 (도메인 화이트리스트 없음, 웹훅 미사용)
5. 네이버 콘솔: 서비스 URL·Callback URL에 `https://ven-up.web.app` 등록 (작업자 직접 진행)
6. release APK 빌드 (PR #47, dev 머지됨):
   - `android.overridePathCheck=true` — 한글 경로 체크 우회 (AOT는 여전히 실패해서 D:\dev\ovenup-apk 복사 빌드)
   - R8 minify 비활성 — okhttp/conscrypt missing class로 R8 실패, 태블릿용이라 용량 무관
   - 소셜 키는 secrets에서 환경변수로 로드 후 빌드 (빌드 스크립트: D:\dev\build-ovenup-apk.ps1, 웹: build-ovenup-web.ps1)

## 트러블슈팅 기록 (다음에 또 만날 것들)
- **한글 경로**: `flutter build apk`는 gradle 체크 우회해도 AOT 스냅샷터가 한글 경로 못 읽어 실패 → ASCII 경로(D:\dev\ovenup-apk) 복사 빌드가 정답
- **BOM 없는 .ps1 + 한글 리터럴**: PS 5.1이 CP949로 읽어 한글 경로가 깨짐 → 스크립트는 ASCII-only로 쓰고 한글 폴더는 와일드카드로 해석
- **Flutter 웹 캐시**: 재배포 후 브라우저가 옛 빌드를 보여줌 → 강력 새로고침(Ctrl+Shift+R) 두 번이면 갱신
- **DuckDNS reCaptcha**: 자동화 브라우저는 점수 미달로 통과 불가, 카톡 인앱 브라우저는 Google 로그인 403 → 폰 크롬에서 직접
- **adb 원격 캡처**: release 앱 화면이 screencap에 검게 나옴 — 실기기 확인 필요

## 남은 것 / 확인 필요
- [ ] 태블릿에서 오븐업 앱 열어 메뉴 로드·주문 흐름 확인 (사장님 확인)
- [ ] 웹에서 네이버 로그인 실측 (콘솔 등록 반영 후)
- [ ] 웹에서 결제(카카오페이 테스트) 실측
- [ ] (선택) ven-up.web.app 대신 커스텀 도메인 연결
