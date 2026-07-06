# 주소 검색창 + 회원가입 휴대폰 인증 (2026-07-06, PR #51)

## 요구사항 (작업자)
1. 주소 입력은 주소 검색창으로
2. 회원가입에서 휴대폰 번호 인증 필수 (Firebase 전화 인증 선택)

## 구현
### 주소 검색 — 다음(카카오) 우편번호 서비스 (무료, 키 불필요)
- 회원가입·소셜 온보딩 주소칸: 누르면 검색창 → 도로명주소 선택 → 상세주소(동/호수) 별도 입력
- 웹: index.html에 postcode.v2.js + openDaumPostcode() 추가, dart:js_interop로 호출 (팝업)
- 모바일: flutter_inappwebview 전체화면 embed (data/address_search_mobile.dart)

### 휴대폰 인증 — Firebase 전화 인증
- 흐름: 번호 입력 → [인증번호 받기] → 문자 6자리 → [확인] → 인증완료 표시. 미인증 시 가입 불가
- 웹: signInWithPhoneNumber(보이지 않는 reCAPTCHA), 모바일: verifyPhoneNumber(자동 인증 지원)
- 인증 후 Firebase 세션 즉시 로그아웃 (우리 JWT 로그인과 별개)
- 새 파일: data/firebase_setup.dart(웹 config 포함), data/phone_verify.dart

### Firebase 설정 (완료)
- 콘솔 Authentication > 전화 제공업체 활성화
- 웹 앱 ovenup-web 생성 (appId 1:574847401193:web:910d…)
- 디버그 키스토어 SHA-1/256 등록 (firebase apps:android:sha:create)

## ⚠️ 운영 참고
- **무료(Spark) 플랜은 인증 SMS 하루 10건 제한.** 테스트엔 충분, 실 오픈 전 Blaze(종량제) 전환 필요
  (전환해도 월 1만 건까지 무료, 카드 등록만 필요)
- 안드로이드 실기기에서 첫 인증 시 브라우저로 reCAPTCHA가 잠깐 열릴 수 있음(정상)
- 테스트 번호 없이 실제 문자가 발송되므로 본인 번호로 테스트할 것

## 검증
- flutter test 통과, 웹·APK 빌드 성공, 배포·설치 완료
- 실기기 확인 필요: 웹/태블릿에서 회원가입 → 문자 수신 → 인증 → 가입, 주소 검색창 동작
