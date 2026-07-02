# github-setup — 오븐업 코드 레포용 파일 모음

이 폴더의 파일들은 **문서 레포가 아니라 코드 레포(`Oven-up`)** 에 넣는 것들이다.
자세한 세팅 순서는 상위 폴더의 **`07_자동PR_세팅_가이드.md`** 를 따르세요.

## 코드 레포에 복사할 위치

```
Oven-up/
├── .github/
│   ├── CODEOWNERS
│   ├── pull_request_template.md
│   └── workflows/
│       ├── ci.yml
│       ├── auto-merge.yml
│       └── claude-review.yml   (선택)
└── .gitignore
```

- `.github/workflows/ci.yml` — PR마다 서버/앱 빌드 + 비밀정보 검사
- `.github/workflows/auto-merge.yml` — 검사 통과 시 dev로 자동 머지
- `.github/workflows/claude-review.yml` — (선택) Claude 자동 리뷰. API 키 필요
- `.github/pull_request_template.md` — PR 작성 양식
- `.github/CODEOWNERS` — 검토자 지정
- `.gitignore` — 빌드물·비밀파일 제외

> 파일 이름이 점(.)으로 시작해서 탐색기에서 숨김일 수 있어요. 보기 설정에서 "숨긴 항목 표시"를 켜세요.
