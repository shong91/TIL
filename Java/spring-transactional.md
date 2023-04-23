## @Transactional

![](https://velog.velcdn.com/images/ann0905/post/56a48b12-b2d0-4071-b09e-959e585551bb/image.png)

동작 순서

1. target에 대한 호출이 들어오면 AOP proxy가 이를 가로채서(intercept) 가져온다.
2. AOP proxy에서 Transaction Advisor가 commit 또는 rollback 등의 트랜잭션 처리를 한다.
3. 트랜잭션 처리 외에 다른 부가 기능이 있을 경우 해당 Custom Advisor에서 그 처리를 한다.
4. 각 Advisor에서 부가 기능 처리를 마치면 Target Method를 수행한다.
5. interceptor chain을 따라 caller에게 결과를 다시 전달한다.

유의사항

1. 원본 클래스/인터페이스를 상속 받아 프록시를 생성하기 때문에 접근 제어가 private 인 경우 트랜잭션 처리를 할 수 없다.
2. 트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다. 객체 외부에서 처음으로 진입하는 메서드에 트랜잭션 처리가 되어 있어야, 해당 요청을 프록시 객체가 대신 처리할 수 있다.

## customized AOP

https://cobbybb.tistory.com/25

## @Async
