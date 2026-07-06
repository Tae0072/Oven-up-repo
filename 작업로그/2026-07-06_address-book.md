# 주소 선택 화면(주소록) + 상단바 재배치 (2026-07-06, PR #54)

## 요구사항 (작업자)
1. 주소 탭 → 주소 "선택 화면": 등록한 주소 목록 표시, [주소 추가] 시 주소 검색창
2. 좌측 상단에 로고/화면이름, 주소 선택은 상단 중앙

## 구현
### 서버 — 주소록 API
- user_addresses 테이블 신설 (userId, address). 현재 선택 주소는 기존 users.address 유지
- GET/POST /api/users/me/addresses, PATCH .../{id}/select, DELETE .../{id}
- 과거 가입자의 단일 주소는 첫 목록 조회 때 자동으로 목록에 이관
- 추가 시 자동 선택, 선택 주소 삭제 시 남은 주소 중 최신 것으로 자동 선택

### 앱
- screens/address_select_page.dart: 주소 목록(선택 체크·삭제), [새 주소 추가] → 주소 검색창 → 상세주소 입력 → 추가+선택
- data/address_api.dart 신설
- AppBar: 홈=좌측 로고 이미지, 메뉴=좌측 '메뉴' 텍스트, 주소 선택은 상단 중앙(centerTitle)
- 비회원은 목록 없이 바로 주소 검색창 (앱 내 임시 기억)

## 검증
- flutter test / UserProfileControllerTest 통과, 서버·웹·APK 배포 완료
- 웹 확인: 상단 중앙 주소 표시 → 클릭 → 주소 선택 화면(기존 주소 이관·선택 표시) 정상
