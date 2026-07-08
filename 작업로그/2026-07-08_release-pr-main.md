# 2026-07-08 main 릴리스 PR 생성

## 무엇을 했나
지금까지 모든 기능 작업은 `dev` 브랜치에 모여 있었고, `main`에는 반영되지 않은 상태였습니다.
`dev`에 쌓인 커밋 57개(PR #1~#57)를 `main`으로 옮기는 릴리스 PR을 만들었습니다.

- PR: https://github.com/Tae0072/Oven-up/pull/58
- 방향: `dev` → `main`
- 제목: release: dev -> main (PR #1~#57 누적 반영)

## 왜 이렇게 했나 (입문자 설명)
- `main`은 "실제 서비스에 내보내는 완성본" 브랜치, `dev`는 "개발 중 작업을 모으는" 브랜치입니다.
- 기능 하나하나는 feature 브랜치 → dev로 PR을 통해 합쳐 왔습니다.
- 이제 dev가 안정된 상태이므로, dev 전체를 main으로 올리는 PR(릴리스 PR)을 만든 것입니다.

## 확인한 것
- 열린 PR 없음, 마지막 기능 PR(#57 PWA iOS)은 dev에 머지 완료
- `main`이 `dev`보다 앞선 커밋 0개 → 충돌 없이 fast-forward 가능한 상태

## 다음 할 일
- PR #58의 자동 리뷰/CI 통과 확인 후 머지
- 머지 후 `main` 기준으로 배포(Firebase Hosting / 서버) 재확인

## 결과
- 2026-07-08 11:51 (KST) PR #58 머지 완료 (merge commit 방식, dev 이력 보존)
- 머지 후 확인: main 대비 dev 미반영 커밋 0개 → main 최신 상태
