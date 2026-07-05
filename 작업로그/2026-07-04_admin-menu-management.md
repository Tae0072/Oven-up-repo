# 작업로그 · 2026-07-04 — 관리자 메뉴 관리 (A3)

> 사장님이 메뉴를 등록·수정·삭제하고 품절을 토글. 품절 메뉴는 주문 불가. (PR #27)

## 1. 워크플로우
1. [x] 서버 관리자 메뉴 CRUD + 품절 토글 + 품절 담기 거부 + 테스트
2. [x] 서버 빌드검증(ASCII,H2) + MySQL 실측
3. [x] 앱 관리자 메뉴 관리 화면 + 손님 품절 표시 + 진입점 + analyze/test
4. [x] 커밋/PR(#27) → dev 머지 + 작업로그

## 2. 리포트 (PR #27)

### 서버 (05_API §11, 화면 A3)
- `AdminMenuController`(ADMIN): `GET/POST/PUT/DELETE /api/admin/menus`, `PATCH /{id}/soldout`.
- `MenuService`: adminList/create/update/setSoldOut/delete. `MenuEntity` update/changeStatus.
- 기존 `status` 필드 재사용("판매중"/"품절"). 품절 메뉴는 장바구니 담기 거부(`MENU_SOLD_OUT`).
- 테스트: 등록/품절토글/품절담기 거부/판매중 복구/수정/삭제, 비관리자 403.

### 앱
- `MenuItem`에 status/soldOut 추가. 손님 메뉴카드 '품절' 배지 + 담기 비활성.
- 관리자 메뉴 관리 화면(목록·품절 스위치·수정/삭제·등록 폼). 마이페이지 '메뉴 관리' 진입점.

## 3. 검증
- 서버 gradlew build(H2) 41 테스트 통과. MySQL 실측: 메뉴 등록(불고기 치아바타) → 품절 토글(status "품절") → 손님 품절 담기 거부(MENU_SOLD_OUT) → 삭제.
- 앱 flutter analyze 무결점 + test 6건.

## 4. 삽질 메모
- MySQL 실측 시 PowerShell이 인라인 JSON을 curl로 넘길 때 한글/따옴표를 깨뜨려 오검출 → 요청 본문은 write_file로 만든 파일을 `curl --data-binary @파일`로 보내야 정확. (서버·DB는 정상, H2 통합테스트가 로직 보증)

## 5. 다음 단계
- 메뉴 옵션 관리, 실제 메뉴 사진 업로드, 예약 주문 전용 화면(S9), 실제 외부 연동(PortOne/소셜/FCM).
