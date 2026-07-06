# 휴대폰 본인인증(PASS식) 전환 (2026-07-06, PR #51에 포함)

## 배경
- Firebase SMS 인증이 동작하지 않았고(문자 미수신), 작업자가 배민/쿠팡이츠식 본인인증(PASS) 요청
- PortOne V2 본인인증(KG이니시스 통합인증)으로 교체 — 통신사 PASS 인증 그대로, 테스트 채널 무료

## 구현
### PortOne 콘솔
- 본인인증 채널 `5ven-Up-Identity` 추가 (KG이니시스 통합인증 V2, 테스트 모드, 공용 MID MIIiasTest)
- 채널 키는 secrets의 PORTONE_CHANNEL_KEY_IDENTITY (dart-define으로 주입, 빌드 스크립트 반영)

### 앱
- 회원가입: 전화번호 입력칸 → **[휴대폰 본인인증] 버튼**. 인증창(통신사 선택→PASS/SMS) 완료 시
  서버 preview API로 이름·전화번호 자동 입력 + "본인인증 완료 · 이름(번호)" 표시
- data/identity_verify*.dart 신설 (웹: 브라우저 SDK requestIdentityVerification 팝업 / 모바일: portone_flutter SDK 웹뷰)
- 채널 키 없으면(개발 모드) 기존처럼 번호 직접 입력으로 동작
- Firebase 전화인증 코드·의존성 제거 (firebase_auth, phone_verify.dart, firebase_setup.dart)

### 서버
- auth/IdentityVerifier: GET api.portone.io/identity-verifications/{id} 로 status=VERIFIED 확인,
  통신사 확인 이름·전화번호 추출 (PORTONE_API_SECRET 재사용)
- POST /api/auth/identity/preview: 인증 직후 이름·전화 미리보기
- 회원가입: identityVerificationId 있으면 서버가 재검증 후 **통신사 확인 이름·번호를 신뢰해 저장**

## 트러블슈팅
- Firebase SMS 미수신 원인은 미해결(웹 예외, 네트워크 요청 자체가 안 나감) — PASS 전환으로 대체
- **Firebase Hosting 캐시**: JS가 1시간 캐시돼 새 배포가 반영 안 되던 문제 → firebase.json에 Cache-Control: no-cache 추가 (이후 배포는 새로고침 1번이면 반영)
- **gitleaks**: 히스토리에 남은 firebase 웹 apiKey(공개값)가 secret-scan을 막음 → .gitleaksignore로 해당 fingerprint만 예외

## 확인 필요 (실기기)
- 웹/태블릿에서 회원가입 → [휴대폰 본인인증] → 통신사 인증 → 이름·번호 자동입력 → 가입
- ⚠️ 웹은 팝업 차단 시 인증창이 안 뜰 수 있음(브라우저가 물어보면 허용)
- 실오픈 시: 이니시스와 본인인증 실계약 후 실연동 채널로 교체 (콘솔에서 채널만 추가하면 됨)
