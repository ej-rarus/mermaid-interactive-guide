# 쇼핑몰 시스템 - 초간단 예시 가이드

이 문서는 Mermaid 다이어그램을 처음 배우는 분들을 위한 아주 쉬운 예시입니다.

---

## 1. 시스템 구조도 (System Architecture)

온라인 쇼핑몰의 전체 구조를 보여줍니다.

```mermaid
---
title: 쇼핑몰 시스템 구조
---
graph TB
    subgraph "사용자"
        Customer[고객]
    end

    subgraph "프론트엔드"
        Web[웹사이트]
        Mobile[모바일 앱]
    end

    subgraph "백엔드"
        Server[서버]
        DB[(데이터베이스)]
    end

    subgraph "외부 시스템"
        Payment[결제 시스템]
        Delivery[배송 시스템]
    end

    Customer --> Web
    Customer --> Mobile
    Web --> Server
    Mobile --> Server
    Server --> DB
    Server --> Payment
    Server --> Delivery

    style Customer fill:#FFE5E5
    style Web fill:#E5F3FF
    style Mobile fill:#E5F3FF
    style Server fill:#E5FFE5
    style DB fill:#FFF9E5
    style Payment fill:#F0E5FF
    style Delivery fill:#F0E5FF
```

**설명:**
- **고객**: 웹사이트나 모바일 앱으로 쇼핑
- **서버**: 주문을 처리하고 데이터베이스에 저장
- **결제/배송**: 외부 시스템과 연동

---

## 2. 주문 프로세스 (Order Process)

고객이 상품을 주문하는 전체 과정입니다.

```mermaid
---
title: 주문 프로세스
---
flowchart TD
    Start([쇼핑 시작]) --> Browse[상품 둘러보기]
    Browse --> Select{마음에 드는<br/>상품 있나요?}

    Select -->|아니오| Browse
    Select -->|예| AddCart[장바구니에 담기]

    AddCart --> MoreShopping{더 쇼핑할까요?}
    MoreShopping -->|예| Browse
    MoreShopping -->|아니오| Checkout[주문하기]

    Checkout --> Login{로그인<br/>했나요?}
    Login -->|아니오| DoLogin[로그인하기]
    Login -->|예| EnterAddress
    DoLogin --> EnterAddress[배송지 입력]

    EnterAddress --> SelectPayment[결제 방법 선택]
    SelectPayment --> PaymentType{결제 방법}

    PaymentType -->|신용카드| CardPay[카드 결제]
    PaymentType -->|계좌이체| BankPay[계좌이체]
    PaymentType -->|무통장입금| DepositPay[입금 대기]

    CardPay --> Complete[주문 완료]
    BankPay --> Complete
    DepositPay --> WaitDeposit[입금 확인 대기]
    WaitDeposit --> Complete

    Complete --> SendEmail[주문 확인 이메일 발송]
    SendEmail --> PrepareDelivery[배송 준비]
    PrepareDelivery --> End([끝])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style Complete fill:#87CEEB
    style AddCart fill:#FFD700
```

**설명:**
1. **상품 선택**: 원하는 상품을 장바구니에 담기
2. **로그인**: 주문하려면 로그인 필요
3. **배송지**: 받을 주소 입력
4. **결제**: 원하는 방법으로 결제
5. **완료**: 이메일로 주문 확인

---

## 3. 데이터 모델 (Data Model)

쇼핑몰에서 저장하는 데이터들의 관계입니다.

```mermaid
---
title: 쇼핑몰 데이터 모델
---
erDiagram
    CUSTOMER ||--o{ ORDER : "주문한다"
    ORDER ||--|{ ORDER_ITEM : "포함한다"
    PRODUCT ||--o{ ORDER_ITEM : "주문된다"
    ORDER ||--|| PAYMENT : "결제한다"
    ORDER ||--|| DELIVERY : "배송한다"

    CUSTOMER {
        int customer_id PK "고객ID"
        string name "이름"
        string email "이메일"
        string phone "전화번호"
        string address "주소"
    }

    PRODUCT {
        int product_id PK "상품ID"
        string name "상품명"
        int price "가격"
        int stock "재고"
        string category "카테고리"
    }

    ORDER {
        int order_id PK "주문ID"
        int customer_id FK "고객ID"
        datetime order_date "주문일시"
        int total_amount "총금액"
        string status "상태"
    }

    ORDER_ITEM {
        int order_item_id PK "주문상품ID"
        int order_id FK "주문ID"
        int product_id FK "상품ID"
        int quantity "수량"
        int price "가격"
    }

    PAYMENT {
        int payment_id PK "결제ID"
        int order_id FK "주문ID"
        string method "결제방법"
        int amount "금액"
        datetime payment_date "결제일시"
        string status "상태"
    }

    DELIVERY {
        int delivery_id PK "배송ID"
        int order_id FK "주문ID"
        string address "배송지"
        string status "배송상태"
        datetime delivery_date "배송일시"
    }
```

**설명:**
- **CUSTOMER**: 고객 정보 저장
- **PRODUCT**: 판매하는 상품 정보
- **ORDER**: 주문 내역
- **ORDER_ITEM**: 주문한 상품들 (한 주문에 여러 상품 가능)
- **PAYMENT**: 결제 정보
- **DELIVERY**: 배송 정보

**관계:**
- 한 고객은 여러 번 주문 가능
- 한 주문은 여러 상품 포함 가능
- 한 주문은 하나의 결제와 하나의 배송

---

## 4. 반품 프로세스 (Return Process)

고객이 상품을 반품하는 과정입니다.

```mermaid
---
title: 반품 프로세스
---
flowchart TD
    Start([반품 시작]) --> Login[로그인]
    Login --> ViewOrders[내 주문 보기]
    ViewOrders --> SelectOrder[반품할 주문 선택]

    SelectOrder --> CheckPeriod{배송 후<br/>7일 이내?}
    CheckPeriod -->|아니오| Reject[반품 불가]
    CheckPeriod -->|예| SelectReason[반품 사유 선택]

    SelectReason --> ReasonType{반품 사유}
    ReasonType -->|단순 변심| CheckCondition{상품 상태<br/>양호?}
    ReasonType -->|불량/오배송| Approve

    CheckCondition -->|훼손됨| Reject
    CheckCondition -->|양호| Approve[반품 승인]

    Approve --> RequestPickup[수거 요청]
    RequestPickup --> PickupSchedule[수거 일정 안내]
    PickupSchedule --> Pickup[택배 수거]

    Pickup --> Inspect[상품 검수]
    Inspect --> InspectResult{검수 결과}

    InspectResult -->|불합격| RejectRefund[환불 거부]
    InspectResult -->|합격| ProcessRefund[환불 처리]

    ProcessRefund --> RefundMethod{원결제<br/>수단}
    RefundMethod -->|카드| CardRefund[카드 취소]
    RefundMethod -->|계좌이체| BankRefund[계좌 환불]

    CardRefund --> Complete[반품 완료]
    BankRefund --> Complete
    Complete --> SendNotification[완료 알림 발송]
    SendNotification --> End([끝])

    Reject --> End
    RejectRefund --> End

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style Approve fill:#87CEEB
    style Complete fill:#98FB98
    style Reject fill:#FF6B6B
    style RejectRefund fill:#FF6B6B
```

**설명:**
1. **반품 가능 여부 확인**: 배송 후 7일 이내인지 체크
2. **반품 사유**: 불량/오배송은 즉시 승인, 단순 변심은 상품 상태 확인
3. **수거**: 택배로 상품 회수
4. **검수**: 상품 상태 재확인
5. **환불**: 원래 결제한 방법으로 환불

---

## Mermaid 다이어그램 기본 문법 정리

### 1. 시스템 구조도 (graph)
```
graph TB
    A[박스] --> B[박스]  (TB = 위에서 아래로)
    A --> C[박스]
```

### 2. 플로우차트 (flowchart)
```
flowchart TD
    Start([시작]) --> Process[처리]  (TD = 위에서 아래로)
    Process --> End([끝])
```

**모양 종류:**
- `[텍스트]` : 사각형
- `([텍스트])` : 둥근 사각형 (시작/끝)
- `{텍스트}` : 다이아몬드 (조건 분기)

**화살표:**
- `-->` : 실선 화살표
- `-.->` : 점선 화살표
- `==>` : 굵은 화살표
- `-->|설명|` : 화살표에 설명 추가

### 3. ER 다이어그램 (erDiagram)
```
erDiagram
    A ||--o{ B : "관계명"
    A {
        int id PK
        string name
    }
```

**관계 표시:**
- `||--||` : 일대일 (1:1)
- `||--o{` : 일대다 (1:N)
- `}o--o{` : 다대다 (N:M)

---

## 다음 단계

이제 기본을 익혔으니 `MISTO POS` 같은 복잡한 시스템도 이해하기 쉬울 거예요!

1. **01-system-architecture.md** - 시스템 전체 구조 보기
2. **02-sales-process-flowchart.md** - 판매 프로세스 상세히 보기
3. **05-data-model-er-diagram.md** - 데이터베이스 구조 보기

화이팅!
