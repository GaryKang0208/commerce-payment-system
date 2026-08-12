## commerce-payment-system
커머스 결제 시스템 프로젝트

### 주요 기술 스택

| 구분 | 기술                 |
| :--- |:-------------------|
| **Language** | Java 17            |
| **Framework** | Spring Boot 4.1.0  |
| **ORM** | Spring Data JPA    |
| **Database** | MySQL              |
| **Library** | Lombok, Spring Validation |
| **Frontend** | Thymeleaf, Bootstrap |

---

### 디렉토리 구조
```text
src
└── main
    └── java
        └── com.example.commercepaymentsystem
            ├── common          # 공통 기능 (설정, BaseEntity, 글로벌 예외 처리 등)
            ├── customer        # 고객 도메인 (인증, 프로필 관리 등)
            │   ├── controller  # API 요청 처리 및 응답 반환
            │   ├── dto         # 계층 간 데이터 교환 객체 (Request/Response 분리)
            │   ├── entity      # DB 테이블 매핑 도메인 객체
            │   ├── error       # 도메인 커스텀 에러
            │   ├── repository  # Spring Data JPA 기반 DB 접근 계층
            │   └── service     # 핵심 비즈니스 로직 및 트랜잭션 처리
            ├── payments        # 결제 도메인 (결제, 결제취소)
            ├── order           # 주문 도메인 (주문 생성, 취소, 재고 차감/복구)
            ├── cart            # 장바구니 도메인 (장바구니 생성, 취소, 일괄결제)
            └── product         # 상품 도메인 (조회, 정렬, 검색)
```

---

### 비즈니스 로직 및 핵심 기능

#### <span style="color:lightgreen">고객 (Customer) </span>
- **고객 데이터 제어 및 도메인 검증**: 
  - 신규 등록 및 정보 수정 시 **이메일 중복 여부**를 엄격하게 검증하여 중복 시 예외(`EMAIL_DUPLICATION`)를 발생시킵니다.
  - 이미 탈퇴 처리(`INACTIVE`)된 고객의 정보 수정이나 상태 변경을 시도할 경우 도메인 예외(`ALREADY_INACTIVE_CUSTOMER`)로 접근을 차단합니다.
  - 다건 조회 시 페이징 처리와 함께 상태/키워드 기반 검색을 지원하며, 단건 상세 조회 시에는 해당 고객의 **총 주문 횟수와 누적 구매 금액**을 별도로 집계하여 함께 반환합니다.
- **상태 관리 흐름**: 
  - 고객 상태는 `ACTIVE`(활성), `SUSPENDED`(정지), `INACTIVE`(탈퇴)로 구분됩니다. 
  - 일반 상태 변경 API를 통해서는 `ACTIVE`와 `SUSPENDED` 간의 변경만 허용되며, 상태 변경 API로 강제 탈퇴(`INACTIVE`) 처리를 시도할 경우 예외를 발생시킵니다. 회원 탈퇴는 전용 API(Soft Delete)를 통해서만 안전하게 수행되도록 역할이 분리되어 있습니다.

#### <span style="color:lightgreen">결제 (Payments) </span>
- **상품 결제기능**
    - 장바구니에 담긴 상품 정보를 바탕으로 결제 시스템을 구현합니다
    - 상품 결제 시 **무결성 및 멱등성 여부**를 엄격하게 검증하여 오류 발생시 예외(`PAYMENT_ILLEGAL_EXCEPTION`)을 발생시킵니다.
    - 결제시 포인트 제도를 도입하여 결제 금액을 조정합니다.
- **결제 취소기능**
    - 결제 취소 시스템을 구현합니다.
    - 결제 취소 시 **포인트 사용 유무에 따른 환급조치**를 최우선으로 취소 절차를 진행합니다.

---

### 개발 규칙 및 코드 컨벤션

#### 네이밍 규칙 (Naming Conventions)
- **클래스명**: `PascalCase` (예: `AdminService`)
- **메서드 및 변수명**: `camelCase` (예: `getAdmins`, `adminId`)
- **상수명**: `UPPER_SNAKE_CASE` (예: `MAX_PAGE_SIZE`)
- **DB 테이블 및 컬럼**: `snake_case` (예: `admin_role`, `created_at`)

#### 아키텍처 및 DTO
- **계층 분리**: `Controller` -> `Service` -> `Repository`의 3계층 구조를 준수합니다.
- **DTO 분리**: Entity를 뷰나 API에 직접 노출하지 않고, 별도의 `Request / Response DTO` 객체로 변환하여 사용합니다.
- **예외 처리**: 비즈니스 예외는 도메인별 커스텀 에러로 정의하고 `GlobalExceptionHandler`에서 `ApiResponse` 형식으로 일괄 처리합니다.

#### RESTful API 설계
- **URI 표기**: 소문자와 하이픈(`-`) 위주로 사용하며, 자원(Resource)은 복수형 명사로 표현합니다. (예: `/admins/{adminId}`)
- **HTTP 메서드**: 의미에 맞는 표준 메서드(`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)를 엄격히 사용합니다.

#### 커밋 메시지 규칙 (Commit Convention)
- `feat` : 새로운 기능 추가
- `fix` : 버그 수정
- `docs` : 문서 수정
- `style` : 코드 포멧팅, 세미콜론 누락, 코드 변경이 없는 경우
- `refactor` : 코드 리펙토링
- `test` : 테스트 코드, 리펙토링 테스트 코드 추가
- `chore` : 빌드 업무 수정, 패키지 매니저 수정
- 
---

### ERD

<img width="1126" height="711" alt="image" src="<img width="1112" height="702" alt="image" src="https://github.com/user-attachments/assets/87067ed7-5268-4814-b3cc-8d1c871a127a" />
" />

---

### API 명세서

🔗 [API 명세서 바로가기](https://app.notion.com/p/teamsparta/3a42dc3ef51480a597bdc872391adda9)

---
