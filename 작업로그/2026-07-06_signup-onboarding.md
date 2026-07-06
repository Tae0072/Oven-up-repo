# 회원가입 페이지 + 소셜 첫 로그인 온보딩 (2026-07-06, PR #50)

## 요구사항 (작업자)
1. 카카오/네이버 첫 로그인 → 닉네임 설정 → 주소 설정 순서로 진행
2. 회원가입은 아이디·비밀번호·전화번호·이메일·주소 입력
3. 회원가입은 전용 페이지에서 작성

## 구현
### 앱
- `signup_page.dart` 신설: 아이디/비밀번호(+확인)/전화번호/이메일/주소 입력 → 가입 성공 시 자동 로그인 → 홈
- 로그인 화면: 인라인 가입 폼 제거, "회원가입" 링크로 전용 페이지 이동. 로그인 입력칸은 "아이디 또는 이메일"
- `profile_setup_page.dart` 신설(온보딩 1/2 닉네임 → 2/2 주소, 뒤로가기 차단). 소셜 로그인 응답의 needsProfile=true면 자동 진입
- 닉네임 저장 시 홈 인사말 이름도 즉시 갱신

### 서버
- users 테이블: loginId(unique)·nickname·address 컬럼 추가 (ddl-auto로 자동 반영)
- 회원가입: 새 필드 수용. loginId 없으면 이메일을 아이디로(하위호환 — 기존 테스트·데모계정 무영향)
- 로그인: 아이디/이메일 모두 허용
- 소셜 로그인 응답에 needsProfile 추가 (nickname 또는 address 미설정 시 true)
- PATCH /api/users/me: nickname/address 부분 수정. 닉네임 설정 시 표시 이름(name)도 동기화
- 테스트: updateNicknameAndAddress 추가, 기존 테스트 전부 통과

## 검증
- 서버(운영): 새 형식 가입 → 아이디 로그인 → 주소 저장 → 닉네임 PATCH 전부 200 확인
- flutter test / 서버 관련 테스트 로컬 통과
- 웹 재배포 + 태블릿 APK 재설치 완료

## 참고
- 기존 카카오/네이버 계정도 닉네임·주소가 없으므로 **다음 로그인 때 온보딩이 한 번 뜬다** (정상 동작)
- 서버 배포 절차: D:\dev\ovenup-server-build에서 bootJar → scp → systemctl restart ovenup (기동 약 50초)
