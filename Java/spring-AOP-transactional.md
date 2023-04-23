## @Transactional의 동작 원리

```
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    @Transactional
    public void save(User user){
        userRepository.save(user);
    }
}
```

### `@Transactional` 을 붙이면 일어나는 일

1. Spring Configuration 에 `@EnableTransactionManagement` 어노테이션 추가 (Spring boot 에서는 auto configuration 됨)
2. Spring Configuration 에서 Transaction manager 지정
3. `@Transactional` 이 붙은 public 메서드에 대해 내부적으로 JDBC 트랜잭션 코드를 실행함
   - 메서드 실행 전 doBegin() 실행: JDBC 커넥션 가져오기, auto commit 설정
   - 메서드 실행 후 doCommit() 실행: 해당 메서드가 정상 종료 시 commit(), 예외 발생 시 rollback()

**_스프링은 어떻게 Transaction 관련 코드를 넣어 동작하게 하는걸까?_**

@Transactional 을 살펴보면, Transactional 은 런타임 시점에 동작하며, ElementType.METHOD 에 대하여 @Target 으로 등록되도록 설정되어 있다.

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {
  @AliasFor("transactionManager")
  String value() default "";

  @AliasFor("value")
  String transactionManager() default "";

  String[] label() default {};

  Propagation propagation() default Propagation.REQUIRED;

  Isolation isolation() default Isolation.DEFAULT;

  ....
}
```

```
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;

}
```

Transactional 에는 트랜잭션 기능이 담긴 Advice 는 이미 등록되어 있으며(before `doBegin()`, after `doCommit()`), @Transactional 에 Target(=`UserService`) 을 명시하면 Pointcut 정보로 등록된다. Advice 와 Pointcut 을 가지는 어드바이저(=`Transaction Manager`) 는 Bean 으로 등록된다.

Target이 빈으로 생성된 후, 어드바이저 빈을 조회 후 생성된 Target 빈에 Advice 가 적용될지 Pointcut 으로 판단한 뒤, 판단 결과에 따라 Target 빈에 프록시 객체(=`Proxy Transaction Manager`)로 치환된다. 모든 트랜잭션은 프록시 객체로 생성한 트랜잭션 매니저에게 위임하여 처리하게 한다.

### @Transactional 이 붙은 메서드의 동작 순서

1. 스프링은 `@Transactional` 어노테이션을 발견하면, 그 빈의 dynamic proxy 를 생성한다.
2. Target(=`UserService`) 에 대한 호출이 들어오면, AOP proxy 는 이를 intercept 하여 가져온다.
3. 프록시 객체는 `Transaction manager`(= Advisor) 에 접근하고, 커넥션을 열고 닫도록 요청한다.
4. Transaction manager는 commit, rollback 등의 트랜잭션을 처리한다.
5. 트랜잭션 처리 외에 다른 부가 기능이 있을 경우, 해당 Advisor 가 기능을 처리한다.
6. 각 Advisor 가 부가 기능 처리를 마치면 Target 메서드를 수행한다.
7. interceptor chain을 따라 caller 에게 결과를 다시 전달한다.

## 유의사항

Spring AOP는 프록시 방식으로 동작하기 때문에, 아래 유의사항을 숙지하여야 한다.

1. private 은 트랜잭션 처리를 할 수 없다.

프록시 객체는 Target 클래스/인터페이스를 상속 받아 구현하는데, 접근 제어가 private 으로 되어 있을 경우 자식인 프록시 객체에서 호출 할 수 없다.
때문에 AOP 가 적용되는 메서드, 클래스는 프록시 객체에서 접근 가능한 레벨로 지정하여야 한다.

2. 트랜잭션은 객체 외부에서 처음 진입하는 메서드를 기준으로 동작한다.

객체 외부에서 처음으로 진입하는 메서드에 트랜잭션 처리가 되어 있어야, 해당 요청을 프록시 객체가 대신 처리할 수 있다.

```
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final PointService pointService;

    @Transactional
    public void order(Order order){
        orderRepository.save(order);
        pointService.addPoint();

    }
}
```

주문을 완료하고 포인트를 지급하는 서비스가 있다. pointService.addPoint() 시에 오류가 발생했다면, 어디까지 롤백될까?
호출하는 부분에 @Transactional 처리가 되어 있다면, 해당 메서드 호출 시에 생성된 트랜잭션 프록시 객체를 사용한다. 때문에 이 경우 order() 에 대한 모든 트랜잭션이 하나로 관리되어, 주문 저장과 포인트 지급이 모두 롤백된다.

## Reference

- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html
- https://jeong-pro.tistory.com/228
