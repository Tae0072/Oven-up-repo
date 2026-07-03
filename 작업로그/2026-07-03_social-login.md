# 작업로그 · 2026-07-03 — 소셜 로그인(카카오·네이버)

> 카카오·네이버 소셜 로그인. 실제 앱 등록 전이라 mock 검증을 기본으로, 실제 검증 코드도 함께. (PR #18)

## 1. 워크플로우
1. [x] 서버 소셜 로그인(social_account + 검증기 mock/real + 엔드포인트) + 테스트
2. [x] 서버 빌드검증(ASCII,H2) + MySQL 실측
3. [x] 앱 카카오/네이버 버튼 + AuthApi.socialLogin + analyze/test
4. [x] 커밋/PR(#18) → dev 머지 + 문서(작업로그 + 실제 연결 가이드)

## 2. 리포트 (PR #18)

### 서버 (05_API §2.3)
- `POST /api/auth/social/{provider}` (kakao|naver) → {accessToken, user, isNew}.
- `social_account` 테이블 + 연동/자동가입: 연결된 소셜계정이면 그 회원, 같은 이메일 자체회원 있으면 연결, 없으면 자동가입(isNew=true).
- 검증기 분리: `SocialProfileVerifier` — Mock(기본, app.social.mock=true) / Real(카카오·네이버 사용자 API, false).

### 앱
- 로그인 화면에 [카카오로 시작하기]/[네이버로 시작하기] + `AuthApi.socialLogin`. 지금은 dev mock 토큰(안내 문구 표시). 실제 SDK는 이후.

## 3. 검증
- 서버 gradlew build(H2) 성공 — 테스트 22건(social 4 + cart 4 + order 5 + auth 5 + menu 3 + contextLoads).
- 앱 flutter analyze 통과 + test 4건.
- MySQL 실측: 카카오 mock 로그인 → 첫 로그인 isNew=true, 재로그인 isNew=false(동일 회원). social_account(KAKAO/9001) 저장 확인.

## 4. 실제 카카오/네이버 연결
- 가이드: `가이드/소셜로그인_연결.md`. 요약: 카카오/네이버 developers 앱 등록 → 앱 SDK로 사용자 토큰 획득 → 서버 `app.social.mock=false`.

## 5. 다음 단계
결제 연동(§5, PortOne), 쿠폰·적립(§10), 푸시 알림(§9).
