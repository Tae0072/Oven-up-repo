# 메뉴 사진 적용 (2026-07-06, PR #52)

## 결과
- 메뉴 7종 전부 실제 사진 적용 — 목록(64px 썸네일) + 상세(240px 대형) 확인 완료
- 원본: `오븐업\로고, 이미지\` 의 아이폰 촬영본(4284px, ~2.5MB) → 웹용 900px JPEG(~110KB)로 최적화

## 구성
- 이미지 서빙: VM `/opt/ovenup/images/` + nginx `location /images/` (7일 캐시, CORS 허용)
  - 주소 형식: https://ovenup.duckdns.org/images/menu-la-galbi.jpg
- DB: menu.image_url 7건 UPDATE (이름 LIKE 매칭)
- 앱: MenuItem.imageUrl 추가, menu_card·menu_detail_page에서 사진 표시 (사진 없으면 기존 이모지 유지)

## 트러블슈팅
- **Flutter 웹(CanvasKit)은 다른 도메인 이미지에 CORS 필수** — 처음에 사진이 안 뜨고 이모지 fallback으로 표시됨
  → nginx images에 `add_header Access-Control-Allow-Origin *;` 추가로 해결
- APK 빌드 AccessDeniedException(merged_native_libs) → 고아 java 정리 + build 폴더 삭제 후 재빌드
- PowerShell로 VM에 명령 보낼 때 $변수·CRLF 깨짐 → 스크립트 파일을 `ssh "tr -d '\r' | bash"` 로 전송하는 패턴 정착

## 새 메뉴 사진 추가 방법 (사장님용 메모)
1. 사진을 `오븐업\로고, 이미지\`에 넣고 요청 → 최적화 후 서버 업로드 + DB 연결
2. (추후) 관리자 화면에서 직접 업로드하는 기능 추가 고려
