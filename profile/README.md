![JoJoPay](https://raw.githubusercontent.com/116lv-jojopay/.github/main/profile/assets/header.svg)

<div align="center">

**일반 상품 구매와 정기 구독을 지원하는 결제 중심 커머스 플랫폼**

[Backend](https://github.com/116lv-jojopay/backend) · [Frontend](https://github.com/116lv-jojopay/frontend) · [Original Project](https://github.com/pcb2002/Commerce-payment-system-JoJoPay) · [116Lv](https://github.com/116Lv)

</div>

## 프로젝트 소개

JoJoPay는 상품을 장바구니에 담아 구매하거나 정기 구독으로 이용할 수 있는 **결제 중심 커머스 플랫폼**입니다. 카드 결제에 포인트를 함께 사용하고, 주문한 상품의 일부 또는 전체를 환불하는 흐름까지 구현했습니다.

팀은 주문 기능을 만드는 데서 더 나아가, 결제가 실패하거나 요청이 겹쳤을 때 데이터가 어떻게 남아야 하는지를 함께 다뤘습니다. 주문 당시의 상품 정보, 포인트 변경 이력, 환불 대상 품목과 구독 결제 이력을 나누어 관리하여 처리 과정을 추적할 수 있도록 설계했습니다.

### 서비스에서 할 수 있는 일

| 영역 | 주요 기능 |
| --- | --- |
| 회원·상품 | 회원가입과 JWT 로그인, 상품 검색·필터·정렬·페이지 조회 |
| 장바구니·주문 | 상품 수량 관리, 재고 검증, 주문 당시 상품명·가격·수량 보관 |
| 결제 | PortOne V2 연동, 일반 결제와 포인트 복합 결제, 결제 결과 검증 |
| 포인트 | 적립·사용 내역 조회, 환불에 따른 사용 포인트 복구와 적립 포인트 회수 |
| 환불 | 부분·전체 환불, 환불 원장과 품목별 내역 관리, PG 취소 |
| 정기 구독 | Billing Key 관리, 구독 상태 관리, 스케줄러 결제와 성공·실패 이력 |

### 주문에서 환불까지

상품 선택 → 장바구니 → 주문 생성·재고 확인 → PortOne 결제 → 결제 결과 검증 → 주문·포인트 이력 반영으로 이어집니다. 환불할 때는 대상 상품과 수량을 확인하고, 환불 내역을 만든 뒤 포인트 복구와 PG 취소 결과를 반영합니다.

정기 구독은 결제수단을 등록한 뒤 구독 정보에 따라 주기적으로 결제를 진행하고, 각 회차의 결과를 별도 이력으로 관리합니다.

## 내가 맡은 역할

**포인트 API, 환불 도메인, PortOne V2 인프라 클라이언트**를 담당했습니다. 결제 수단을 연결하는 것뿐 아니라 부분·전체 환불 시 어떤 금액과 포인트를 되돌려야 하는지, 외부 취소가 실패했을 때 어떤 상태를 남겨야 하는지를 구현했습니다. 관련 단위 테스트와 프론트엔드 구현에도 참여했습니다.

### 포인트 잔액과 변경 이력을 함께 관리했습니다

잔액만으로는 적립·사용·환불에 따른 변경 이유를 확인하기 어려워 PointHistory에 변경 내역을 남겼습니다. 환불에서는 사용한 포인트를 돌려주는 처리와 구매로 적립된 포인트를 회수하는 처리를 구분했습니다.

동시에 들어온 변경 요청이 서로의 잔액 갱신을 덮어쓰지 않도록 회원 조회에 **비관적 잠금**을 적용하고, 잔액과 이력을 같은 트랜잭션에서 저장하도록 구성했습니다.

### 환불 대상과 외부 결제 취소를 연결했습니다

환불 요청의 상품과 수량을 검증하고, Refund와 RefundItem으로 환불 전체 내역과 품목별 정보를 나누었습니다. 포인트로 결제한 금액과 PG로 결제한 금액을 구분하여 포인트 복구·회수와 외부 취소가 이어지도록 처리했습니다.

### 외부 API 호출과 DB 트랜잭션을 분리했습니다

외부 결제 서비스의 응답을 기다리는 동안 DB 트랜잭션이 이어지는 구조를 개선했습니다. 내부 처리를 먼저 저장한 뒤 트랜잭션 밖에서 PG 취소를 호출하고, 결과에 따라 완료 또는 실패 상태를 기록하도록 나누었습니다.

장시간 미완료로 남은 환불은 스케줄러에서 외부 상태를 재확인하도록 했습니다. 실패 상태는 별도 확인 대상으로 남겨, 모든 실패를 자동 재시도하는 방식과 구분했습니다.

### 정상 처리와 실패 상황을 나누어 확인했습니다

PG 취소 성공, 외부 API 예외, 전액 포인트 환불 등의 분기를 서비스 단위 테스트로 확인하도록 작성했습니다. 테스트 범위와 실행 기록은 [검증 문서](https://github.com/116lv-jojopay/backend/blob/main/VALIDATION.md)에 정리되어 있습니다.

[포인트 API #29](https://github.com/pcb2002/Commerce-payment-system-JoJoPay/pull/29) · [환불 도메인 #83](https://github.com/pcb2002/Commerce-payment-system-JoJoPay/pull/83) · [트랜잭션·동시성 개선 #95](https://github.com/pcb2002/Commerce-payment-system-JoJoPay/pull/95)

## 기술과 구조

Java 17 · Spring Boot · Spring Security · JPA · MySQL · PortOne V2 · Docker · AWS EC2

위 기술과 서비스 기능은 팀 프로젝트 전체 기준입니다. 개인 담당은 위의 역할과 연결된 PR에서 확인할 수 있습니다.

### 시스템 아키텍처

![JoJoPay 시스템 아키텍처](https://github.com/user-attachments/assets/574973c8-93f0-4579-8452-6a07f2ace33b)

<details>
<summary>ERD와 원본 설계 자료</summary>

![JoJoPay ERD](https://github.com/user-attachments/assets/2d085b21-4889-49ad-94ab-455023d15d84)

주문 스냅샷, 장바구니 중복 제약, 구독 도메인과 개선 전후 자료는 [원본 프로젝트 README](https://github.com/pcb2002/Commerce-payment-system-JoJoPay#readme)에 정리되어 있습니다.

</details>

## 저장소와 참고 자료

| 저장소 | 내용 |
| --- | --- |
| [backend](https://github.com/116lv-jojopay/backend) | 서버 코드, 테스트, 원본 이력과 실행 안내 |
| [frontend](https://github.com/116lv-jojopay/frontend) | 서비스 화면, 클라이언트 코드와 실행 안내 |
| [원본 프로젝트](https://github.com/pcb2002/Commerce-payment-system-JoJoPay) | 전체 기능, 설계 자료, 개선 전후와 팀 협업 기록 |

---

<sub>팀 프로젝트 종료 후 116Lv가 정리한 개인 포트폴리오입니다. 팀의 공식 조직을 대체하지 않습니다. 팀 전체 기능과 개인 담당을 구분하며, 협업 기록은 원본 저장소에 보존되어 있습니다.</sub>
