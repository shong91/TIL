# 도메인 주도 설계 (Domain Driven Developement)

## 등장배경

| 기존의 개발                    | 도메인 주도 개발                                                                         |
| ------------------------------ | ---------------------------------------------------------------------------------------- |
| 데이터에 종속                  | 문제 영역을 개념적으로 표현                                                              |
| 모델링과 개발 간의 불일치 발생 | 이해관계자(개발, 기획, 사용자 등) 이 공통적으로 의미를 이해할 수 있음 => 효과적인 모델링 |

## 도메인

- 소프트웨어 프로그램에 대한 기능성을 정의하는 연구의 한 영역
- 소프트웨어로 해결하고자 하는 문제 영역 ex) 광고회사에서 광고와 관련된 지식 = 도메인

### 도메인 모델

- 문제 영역을 개념적으로 표현한 것
- 도메인 모델을 여러 이해 당사자가 이해할 수 있는 개념적 모델링 가능
- 하위 도메인으로 개념 구체화 가능 ex) 상품 주문 도메인 => 주문자, 주문상품, 배송 (하위 도메인)
- 클래스 다이어그램, 상태 다이어그램, 시퀀스 다이어그램 등의 방식으로 표현 가능
- 도메인은 고정적이지 않기 때문에, 도메인이 변경되면 이에 따라 도메인 용어 및 모델도 변경 반영되어야 한다.

## DDD 아키텍쳐

### Layered Architecture

![img_01](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbbeJCW%2FbtqvsZLoVQu%2FnqAjktecGoh8bkW2L4kS61%2Fimg.png)

- PRESENTATION LAYER : 표현 영역 또는 UI 영역. 사용자의 요청을 받아 응용 영역에 전달하고, 응용 영역의 처리 결과를 다시 사용자에게 보여주는 역할을 한다.
  - Controller 영역, DispatcherServlet에게 요청과 응답을 전달하는 역할
  - session 관리
- APPLICATION LAYER : 응용 영역. 시스템이 사용자에게 제공해야 할 기능을 구현한다.
  - Service, Repository 영역
  - 트랜잭션 관리
- DOMAIN LAYER : 도메인 영역. 도메인 모델을 구현한다. (이름, 주소, 상품, 주문서 등)
- INFRASTRUCTURE LAYER : 구현 기술에 대한 것을 다룬다. (외부 API, 데이터베이스, 외부 라이브러리 사용 등)

### DIP (Dependency Inversion Principle)

- 의존관계를 역전시켜 **저수준 모듈이 고수준 모듈에 의존하도록** 구현하는 것
- 인터페이스 도출 시 저수준 모듈의 관점에서 도출하지 않도록 주의
  - 고수준 모듈: 의미 있는 단일 기능을 제공하는 모듈 (ex 배송알림 기능)
  - 저수준 모듈: 고수준 모듈의 기능을 구현하기 위한 하위 기능의 구현 모듈 (ex 배송 알림 메일 발송)
  - 고수준에서의 "배송 알림" 자체에 변경이 없어도, 저수준 모듈의 기능이 "배송 알림 메일" -> "배송 알림 문자"로 바뀐다면 고수준 모듈에서도 변경이 발생하는 것

## 기본 요소

### 1. Entity/Value

- Entity

  - 고유의 식별자와 라이프사이클을 가짐
  - 각 엔티티는 서로 다른 식별자를 가짐 (특정 규칙, UUID, 직접 입력값, 일련번호 등)
  - **setter 메서드 사용하지 않기**
    - 세터를 사용하면 도메인의 핵심 개념/의도를 명확히 표현하기 어려워진다!
    - 도메인 객체 생성 시점에 완벽한 상태가 아닐 수 있음 => 잠정적 에러 가능성
    - 생성자/팩토리 메서드를 활용하여 **객체를 생성하는 시점에 완벽한 상태 값을 주입**해 주어야 함.

- Value

  - 의미를 명확하게 표현하거나, 두 개 이상의 데이터가 개념적으로 하나인 경우 사용 (ex 배송지 주소 + 우편번호 + 수령인 이름 = '배소정보' value object)
  - 불변(Immutable) 원칙: 밸류 객체 값을 변경하려면 새로운 밸류 객체를 할당하는 방법 뿐
  - 식별자 존재 X

## 2. Aggregate

- 도메인 객체들의 묶음. (독립된 객체군)
- 도메인 객체를 상위 수준으로 추상화하여 도메인 모델의 복잡도를 관리하기 위함
- 애그리거트 밖에서 에그리거트에 직접 접근하기 위해서는 **애그리거트 루트(Aggregate Root)** 를 통하여야 함 => 애그리거트에 포함된 객체들의 일관성 유지

![img_02](https://miro.medium.com/max/1400/1*QkwiZLj_czgs01k8lK75hw.png)

## References.

0. DDD start!
1. https://medium.com/myrealtrip-product/what-is-domain-driven-design-f6fd54051590
2. https://stylishc.tistory.com/143?category=565468
3. https://ppiyo5.tistory.com/21
