# 06. Git / PR 규칙

> 코드를 안전하게 관리하고 협업(또는 나중의 나)과 충돌하지 않도록 하는 규칙.
> QT-AI-2nd-Team-Project의 `.github` 규칙을 참고하되, **오븐업(모놀리식·입문자)에 맞게 단순화**했다.
> 버전: v0.1 · 작성일: 2026-07-02

---

## 0. 왜 규칙이 필요해요? (쉽게)

- Git은 **코드의 저장/되돌리기 기록장**. 규칙 없이 쓰면 나중에 뭐가 왜 바뀌었는지 알 수 없다.
- **브랜치**: 원본을 건드리지 않고 따로 작업하는 "복사본 작업 공간". 다 되면 원본에 합친다(머지).
- **PR(Pull Request)**: "이 브랜치 내용을 합쳐도 될까요?" 하고 올리는 **검토 요청**. 합치기 전에 확인하는 안전장치.
- 혼자 해도 이 습관을 들이면 실수 복구가 쉽고, 나중에 협업할 때 그대로 쓸 수 있다.

---

## 1. 레포 두 개 (헷갈리지 않기)

| 레포 | 용도 |
| --- | --- |
| `Oven-up` | **실제 코드**(Flutter 앱 + Spring Boot 서버) |
| `Oven-up-repo` | **문서**(이 docs 폴더의 기획·설계 문서) |

- 이 Git/PR 규칙과 `.github` 템플릿 파일들은 **코드 레포(Oven-up)** 에 둔다.
- **작업 시작 전 항상 최신화**: `git pull` 로 최신 상태를 받은 뒤 작업한다. (지침)

---

## 2. 브랜치 전략

입문자용으로 **3종류**만 쓴다.

| 브랜치 | 역할 | 규칙 |
| --- | --- | --- |
| `main` | 배포용(항상 정상 동작하는 안정 버전) | 직접 커밋 금지. `dev`에서만 병합 |
| `dev` | 개발 통합(기능들이 모이는 곳) | PR로만 병합 |
| `feature/*` | 기능 하나를 만드는 작업 브랜치 | `dev`에서 갈라져 나와 `dev`로 PR |

### 작업 브랜치 이름 규칙
```
feature/{영역}-{짧은설명}
예)
feature/menu-list          # 메뉴 목록 화면
feature/cart-api           # 장바구니 API
feature/order-payment      # 주문·결제
fix/login-error            # 버그 수정
```

### 흐름 그림
```
main  ←──(안정될 때만)── dev  ←──(PR)── feature/menu-list
                              ←──(PR)── feature/cart-api
```

---

## 3. 커밋 메시지 규칙 (Conventional Commits)

형식: `type(scope): 설명`

| type | 언제 |
| --- | --- |
| feat | 새 기능 |
| fix | 버그 수정 |
| docs | 문서 |
| refactor | 기능 변화 없는 코드 정리 |
| test | 테스트 추가/수정 |
| chore | 설정·빌드·기타 잡일 |

`scope`는 작업 영역(menu, cart, order, payment, auth, admin …).

**예시**
```
feat(menu): 메뉴 목록 화면 추가
feat(cart): 장바구니 담기 API 구현
fix(payment): 결제 금액 재계산 오류 수정
docs(api): 05 API 명세서 장바구니 항목 갱신
```

**규칙**
- 한 커밋에는 **한 가지 일**만. (너무 많은 변경을 한 커밋에 담지 않기)
- 설명은 "무엇을 했는지" 한 줄로 명확하게 (한글 OK).

---

## 4. 작업 흐름 (한 사이클)

```bash
# 1. 최신화
git checkout dev
git pull

# 2. 작업 브랜치 만들기
git checkout -b feature/menu-list

# 3. 작업 후 커밋
git add .
git commit -m "feat(menu): 메뉴 목록 화면 추가"

# 4. 원격에 올리기
git push -u origin feature/menu-list

# 5. GitHub에서 dev로 PR 생성 → 검토 후 머지
# 6. 머지 후 정리
git checkout dev
git pull
```

---

## 5. PR(Pull Request) 규칙

- PR 대상 브랜치는 **`dev`**. (`main` 직접 X)
- PR 제목도 커밋 규칙처럼: `feat(cart): 장바구니 API`
- 아래 **PR 템플릿**을 채워서 올린다.
- 병합 방식은 **Squash merge**(여러 커밋을 하나로 합쳐 깔끔하게) 권장.
- 머지 전 확인: 빌드/실행이 되는가, 문서와 충돌 없는가, 비밀정보가 섞이지 않았는가.

> 팀의 QT-AI 레포는 Claude 자동 리뷰 + CI 통과 시 자동 머지까지 쓰지만, 오븐업은 우선 **본인 확인 후 수동 머지**로 시작하고 익숙해지면 CI를 붙인다(§8).

---

## 6. 오븐업 PR 템플릿 (복사해서 사용)

> 코드 레포에 `.github/pull_request_template.md` 파일로 저장하면, PR 작성 시 자동으로 이 양식이 뜬다.

```markdown
## 구현 내용
<!-- 이 PR에서 무엇을 바꿨는지 간단히 -->

## 관련 문서 / 이슈
<!-- 예: 05_API_명세서 §4 주문, 02_화면_정의서 S5 장바구니 / Closes #00 -->

## 변경 유형
- [ ] 기능 추가 (feat)
- [ ] 버그 수정 (fix)
- [ ] 리팩토링 (refactor)
- [ ] 테스트 (test)
- [ ] 문서 (docs)
- [ ] 설정/기타 (chore)

## 체크리스트
- [ ] 문서(기획·API·ERD)와 충돌 없음
- [ ] 비밀정보(비밀번호·API 키·.env)가 코드에 없음
- [ ] 로컬에서 빌드/실행 확인 (웹 브라우저 / 갤럭시탭)
- [ ] 금액·권한 등 중요한 계산은 서버에서 재검증

## 테스트 방법
<!-- 어떻게 확인했는지: 화면 캡처, 실행 명령 등 -->

## 남은 일 / 다음 PR
<!-- 후속 작업이나 미완성 부분 -->
```

---

## 7. CODEOWNERS (선택)

- 특정 파일이 바뀌면 자동으로 검토자로 지정되는 기능. 혼자면 전부 본인.
- 코드 레포에 `.github/CODEOWNERS` 로 저장.

```
# 모든 파일의 기본 검토자 (본인 GitHub 아이디)
*   @Tae0072
```

> 나중에 팀원이 생기면 `apps/  @프론트담당`, `server/  @백엔드담당` 처럼 경로별로 나눈다.

---

## 8. .gitignore & 보안 (중요)

### .gitignore (코드 레포 루트에 저장)
빌드 결과물·비밀정보가 Git에 올라가지 않게 막는다.
```gitignore
# 환경변수/비밀
.env
*.env

# Java / Spring Boot
build/
.gradle/
*.class

# Flutter / Dart
.dart_tool/
.packages
build/
*.iml

# IDE / OS
.idea/
.vscode/
*.log
.DS_Store
```

### 보안 규칙 (QT-AI 규칙에서 가져온 핵심)
- **비밀번호·API 키·결제 키·소셜 로그인 시크릿을 코드에 직접 쓰지 않는다.** → `.env` 나 환경변수로 관리하고 `.env`는 커밋 금지.
- 실수로 올렸다면 즉시 키를 **재발급**한다(한번 올라간 비밀은 노출된 것으로 간주).
- `.env.example`(값 없이 키 이름만 있는 예시 파일)만 커밋해 팀원이 어떤 값이 필요한지 알게 한다.

---

## 9. (나중에) CI 자동 검사

익숙해지면 GitHub Actions로 PR마다 자동 검사를 붙일 수 있다. 오븐업에 어울리는 최소 구성:
- **빌드 검사**: 서버(`./gradlew build`), 앱(`flutter build`)이 깨지지 않는지
- **비밀 유출 검사**: gitleaks 등으로 키가 섞였는지
- **API 문서 검사**(선택): OpenAPI 스펙 린트

> 처음부터 다 붙이지 말고, 코드가 어느 정도 쌓인 뒤 도입한다.

---

## 10. 적용 방법 요약

코드 레포(`Oven-up`)에 아래 3개 파일을 만든다:
1. `.github/pull_request_template.md` ← §6 내용
2. `.github/CODEOWNERS` ← §7 내용
3. `.gitignore` ← §8 내용

그리고 `main`, `dev` 브랜치를 만들고, 앞으로 모든 작업은 `feature/*` → `dev` PR로 진행한다.
