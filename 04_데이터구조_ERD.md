# 04. 데이터 구조 (ERD)

> 데이터를 어떤 표(테이블)로 저장할지 정리. 개발 2단계에서 이 구조대로 DB를 만든다.
> 버전: v0.2 · 작성일: 2026-07-02 (개정: 소셜계정·예약·단체주문·고객의소리·쿠폰·적립·권한 추가)

---

## 1. 표(테이블) 관계 한눈에 보기

```
[회원 user] 1──< N [소셜계정 social_account]
[회원 user] 1──< N [장바구니항목 cart_item] >──1 [메뉴 menu]
[회원 user] 1──< N [주문 order] 1──< N [주문항목 order_item] >──1 [메뉴 menu] 1──< N [메뉴옵션 menu_option]
[주문 order] 1──1 [결제 payment]
[회원 user] 1──< N [단체주문 group_order]
[회원 user] 1──< N [고객의소리 inquiry] 1──< N [답변 inquiry_reply]
[쿠폰 coupon] 1──< N [보유쿠폰 user_coupon] >──1 [회원 user]
[회원 user] 1──< N [적립내역 point_history]
```

- 회원 1명 → 소셜계정·장바구니·주문·단체주문·문의를 여러 개 가질 수 있다.
- 회원에 **권한(role)** 컬럼으로 손님/사장님을 구분한다.

---

## 2. 테이블 상세

### user (회원)
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 회원 고유번호 |
| email | VARCHAR | 이메일 (중복불가) |
| password | VARCHAR | 암호화된 비밀번호 (소셜만 쓰면 비어있을 수 있음) |
| name | VARCHAR | 이름 |
| phone | VARCHAR | 전화번호 |
| role | VARCHAR | 권한: USER(손님) / ADMIN(사장님) |
| point_balance | INT | 현재 적립금 잔액 |
| created_at | DATETIME | 가입일시 |

### social_account (소셜 계정) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 고유번호 |
| user_id | BIGINT (FK) | 연결된 회원 |
| provider | VARCHAR | KAKAO / NAVER |
| provider_user_id | VARCHAR | 카카오·네이버가 준 고유 사용자 ID |
| created_at | DATETIME | 연결일시 |

> 한 회원이 카카오·네이버를 모두 연결할 수 있어 별도 테이블로 분리.

### menu (메뉴)
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 메뉴 고유번호 |
| name | VARCHAR | 메뉴 이름 |
| description | TEXT | 설명 |
| price | INT | 기본 가격(원) |
| category | VARCHAR | 카테고리(커피/티/디저트) |
| image_url | VARCHAR | 사진 주소 |
| status | VARCHAR | ON_SALE / SOLD_OUT |

### menu_option (메뉴 옵션)
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 옵션 고유번호 |
| menu_id | BIGINT (FK) | 소속 메뉴 |
| name | VARCHAR | 옵션 이름(예: 샷 추가) |
| extra_price | INT | 추가 금액 |

### cart_item (장바구니 항목) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 장바구니 항목 고유번호 |
| user_id | BIGINT (FK) | 담은 회원 (회원별 장바구니) |
| menu_id | BIGINT (FK) | 담은 메뉴 |
| quantity | INT | 수량 |
| option_ids | VARCHAR | 선택한 옵션 id 목록 (예: "100,101") |
| created_at | DATETIME | 담은 시각 |

> 장바구니를 **서버에 저장**하므로 웹·태블릿 어디서 담아도 유지된다. 주문이 완료되면 해당 회원의 cart_item을 비운다.
> (옵션을 더 엄격히 관리하려면 `cart_item_option` 연결 테이블로 분리할 수 있으나, 초기엔 option_ids 문자열로 단순화.)

### orders (주문)
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 주문 고유번호 |
| user_id | BIGINT (FK) | 주문한 회원 |
| order_no | VARCHAR | 사람이 보는 주문번호 |
| total_price | INT | 최종 결제금액 |
| discount_price | INT | 쿠폰+적립 할인 합계 |
| fulfillment_type | VARCHAR | DINE_IN(매장) / TAKEOUT(포장) / DELIVERY(배달) |
| scheduled_at | DATETIME | 예약 주문 시각 (null이면 지금 주문) 🆕 |
| delivery_address | VARCHAR | 배달 주소 (배달일 때만, 명지에코펠리스 건물 내 호실 등) 🆕 |
| delivery_fee | INT | 배달비 (배달일 때, 없으면 0) 🆕 |
| request_msg | TEXT | 요청사항 |
| status | VARCHAR | 결제대기/결제완료/준비중/준비완료/픽업완료/배달중/배달완료 |
| created_at | DATETIME | 주문일시 |

> `order`는 예약어라 테이블명은 `orders` 사용.

### order_item (주문 항목)
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 항목 고유번호 |
| order_id | BIGINT (FK) | 소속 주문 |
| menu_id | BIGINT (FK) | 메뉴 |
| menu_name | VARCHAR | 주문 당시 메뉴 이름(기록용) |
| unit_price | INT | 주문 당시 단가(옵션 포함) |
| quantity | INT | 수량 |
| options_desc | VARCHAR | 선택 옵션 요약 |

### payment (결제)
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 결제 고유번호 |
| order_id | BIGINT (FK) | 대상 주문 |
| amount | INT | 결제 금액 |
| method | VARCHAR | CARD / KAKAOPAY / NAVERPAY / TOSSPAY / SAMSUNGPAY 🆕 |
| pg_tx_id | VARCHAR | 대행사 거래번호 |
| status | VARCHAR | PAID/FAILED/CANCELLED |
| paid_at | DATETIME | 결제일시 |

### 예약 주문은 별도 테이블이 아니다 🆕
예약은 좌석 예약이 아니라 **"시간을 지정한 선결제 주문"** 이므로, 별도 `reservation` 테이블을 두지 않고 **`orders` 테이블로 처리**한다.
- `scheduled_at` 이 있으면 예약 주문, 없으면(null) 지금 주문.
- `fulfillment_type` 으로 매장/포장/배달 구분, `delivery_address` 로 배달 주소 저장.
- 즉, 예약 주문도 일반 주문과 **같은 결제·조회 흐름**을 그대로 탄다.

### group_order (단체 주문) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 단체주문 고유번호 |
| user_id | BIGINT (FK) | 신청자 |
| desired_at | DATETIME | 희망 일시 |
| headcount | INT | 인원/수량 |
| detail | TEXT | 요청 메뉴·상세 |
| contact | VARCHAR | 연락처 |
| status | VARCHAR | 접수/협의중/확정/취소 |
| admin_memo | TEXT | 사장님 답변·견적 메모 |
| created_at | DATETIME | 신청일시 |

### inquiry (고객의 소리) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 문의 고유번호 |
| user_id | BIGINT (FK) | 작성자 |
| title | VARCHAR | 제목 |
| content | TEXT | 내용 |
| image_url | VARCHAR | (선택) 첨부 사진 |
| status | VARCHAR | 접수/답변완료 |
| created_at | DATETIME | 작성일시 |

### inquiry_reply (문의 답변) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 답변 고유번호 |
| inquiry_id | BIGINT (FK) | 대상 문의 |
| content | TEXT | 답변 내용 |
| created_at | DATETIME | 답변일시 |

### coupon (쿠폰) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 쿠폰 고유번호 |
| name | VARCHAR | 쿠폰 이름 |
| discount_type | VARCHAR | AMOUNT(정액) / RATE(정률) |
| discount_value | INT | 할인 값(원 또는 %) |
| min_order_price | INT | 최소 주문금액 |
| valid_until | DATETIME | 유효기간 |

### user_coupon (보유 쿠폰) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 고유번호 |
| user_id | BIGINT (FK) | 보유 회원 |
| coupon_id | BIGINT (FK) | 쿠폰 |
| is_used | BOOLEAN | 사용 여부 |
| used_at | DATETIME | 사용일시 |

### point_history (적립 내역) 🆕
| 컬럼 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT (PK) | 고유번호 |
| user_id | BIGINT (FK) | 회원 |
| change_amount | INT | 증감(+적립 / -사용) |
| reason | VARCHAR | 사유(주문적립/주문사용 등) |
| order_id | BIGINT (FK) | 관련 주문(있으면) |
| created_at | DATETIME | 발생일시 |

---

## 3. 용어 쉽게 이해하기
- **테이블**: 엑셀 시트 한 장. 행 하나 = 데이터 하나.
- **PK(기본키)**: 각 줄을 구분하는 고유번호.
- **FK(외래키)**: 다른 테이블을 가리키는 연결고리.
- **role(권한)**: 이 회원이 손님인지 사장님인지 구분하는 값.
- **1:N**: 하나가 여러 개를 가짐.

---

## 4. 배달(직배송) 조건 데이터 처리 🆕
- 메뉴 `category` 에 **"샌드위치"** 값을 사용해 샌드위치를 구분한다.
- 배달 가능 판정(서버):
  1. 배달지가 **명지에코펠리스 건물**인가?
  2. 주문 항목 중 `category = '샌드위치'` 인 항목의 **수량 합계 ≥ 2** 인가?
  - 둘 다 만족해야 `fulfillment_type = DELIVERY` 허용. 아니면 픽업(TAKEOUT)으로 유도.
- 픽업은 최소 주문 금액 제한 없음.
- (선택) 매장 설정용 상수: 배달가능건물명="명지에코펠리스", 직배송_최소_샌드위치수=2, 배달비=0.

## 5. 시작용 예시 데이터 (테스트용)
- 커피/음료: 아메리카노(4,000), 카페라떼(4,500), 카푸치노(5,000), 아이스티(4,000)
- 디저트: 치즈케이크(6,000)
- **샌드위치**(category="샌드위치"): 햄치즈 샌드위치(6,500), BLT 샌드위치(7,000), 에그마요 샌드위치(6,000)
- 옵션: 샷 추가(+500), 사이즈업(+500), 시럽 추가(+0)
- 관리자 계정 1개: role=ADMIN
- 쿠폰 예시: "신규가입 2,000원 할인"(AMOUNT, 2000, 최소 5,000원)
