# Builder Pattern 사용하기

빌더 패턴은 객체를 생성할 때 사용하는 패턴으로, 메서드 체이닝 방식으로 객체를 생성하여 코드의 가독성 및 유지보수성을 높이기 위해 흔히 사용된다.

이번 프로젝트에서 빌더 패턴을 자주 사용하면서, 그 내용을 간단히 정리해 보자 한다.

### 객체를 생성하는 방법

### 1. Constructor

생성자 함수를 이용하여 객체를 생성한다.

- 필수 인자를 받는 필수 생성자와, 선택적 인자를 받는 생성자를 점층적으로 추가한다
- 모든 선택적 인자를 받는 생성자를 추가한다

생성자 함수를 이용하면, 선택적 인자를 받아 객체를 생성하여야 하는 케이스가 많을 경우, 각기 다른 생성자를 호출하여야 한다. 인자 수가 많아질수록 그 의미를 파악하기 어려워지며, 코드의 가독성이 떨어진다. 또한, 인자가 추가될 경우 코드를 수정하기 어려워진다.

```
Member member = new Member("kim");
Member member = new Member("kim", 30);
Member member = new Member("kim", 30, "서울시");
Member member = new Member("kim", 30, "서울시", "Software engineer");
```

### 2. Setter

Java Bean Pattern 은, `setter` 메서드를 이용해 객체를 생성하여 가독성을 높인다.

- 각 인자의 의미를 쉽게 파악할 수 있다 (가독성 높임)
- 여러 개의 생성자를 만들 필요 없음

하지만, 세터 메서드를 이용하면 객체의 일관성을 유지할 수 없다. 1회 호출만으로 객체를 생성할 수 없으며 (계속하여 setter 메서드를 이용하여 값을 set 해주어야 함), immutable 한 객체를 생성 할 수 없다.

```
Member member = new Member();
member.setName("kim");
member.setAge(30);
member.setCity("서울시");
member.setJob("Software engineer");
```

### 3. Builder

세 번째 방법은, Effective Java 에서 소개하는 빌더 패턴이다.

- 각 인자의 의미를 쉽게 파악할 수 있다 (가독성 높임)
- immutable 한 객체를 만들 수 있다
- 한 번에 객체를 생성하므로 객체의 일관성이 깨지지 않는다

lombok 의 `@Builder` 어노테이션을 사용하여 이펙티브 자바 스타일과 비슷한 빌더 패턴 코드를 사용할 수 있다.

```
Member member = Member.builder()
                .name("kim")
                .age(30)
                .city("서울시")
                .job("Software engineer")
                .build();
```

## 빌더 패턴 사용 시 주의점

객체를 생성하는 3가지 방법과, 그들의 장/단점을 비교하여 빌더 패턴이 가지는 강점에 대하여 알아보았다. 정리하자면, 빌더 패턴은 아래와 같은 이유로 사용한다.

- 객체를 깔끔하고 유연하게 생성하기 위해 사용하는 기법
- 생성자 인자가 많을 때/매개변수가 많을 때 가독성을 높이기 위해 사용
- 객체의 일관성을 보장하고 immutable 한 객체를 만들기 위해 사용

빌더 패턴을 사용할 때, 주의할 점이 몇 가지 있다.

### 1. 클래스 선언부에 `@Builder` 를 사용하지 말 것

`@Builder` 를 클래스에 적용하는 것은, 클래스에 `@AllArgsConstructor` 를 추가하고 그 생성자에 @Builder 를 적용하는 것과 같다. 이렇게 되면 객체 생성 시 받지 않아야 할 매개변수들도 빌더 패턴에 의해 노출 되게 된다. 때문에 필요한 매개변수만으로 생성자를 만들고, 그 생성자에 @Builder 어노테이션을 넣을 것을 권장한다. (롬복 공식 문서에 따르면, 명시적 생성자가 있는 경우, 클래스 선언부가 아닌 명시적 생성자에 @Builder 어노테이션을 넣을 것을 가이드하고 있다.)

빌더 패턴을 통해 객체를 만들 때, 특정 필드값을 초기화하고 싶다면 `@Builder.Default` 를 사용하여 초기값을 설정할 수 있다.

```
public class Member {
    private String name;
    private int age;
    @Builder.Default
    private String city = "서울시";
    private String job;

    @Builder
    public Member(String name, int age){
        this.name = name;
        this.age = age;
    }
}
```

`@Builder`를 클래스에 적용하는 것은 마치 클래스에 `@AllArgsConstructor(access = AccessLevel.PACKAGE)`를 추가하고 이 all-args-constructor에 @Builder 어노테이션을 적용하는 것과 같다. 이 방법은 명시적 생성자를 직접 작성하지 않은 경우에만 작동합니다. **명시적 생성자가 있는 경우 클래스가 아닌 생성자에 @Builder 어노테이션을 넣으세요.** 클래스에 `@Value`와 `@Builder`를 모두 넣으면 `@Builder`가 생성하려는 패키지 비공개 생성자가 '승리'하고 `@Value`가 만들려는 생성자가 억제된다는 점에 유의하세요.

_Finally, applying @Builder to a class is as if you added @AllArgsConstructor(access = AccessLevel.PACKAGE) to the class and applied the @Builder annotation to this all-args-constructor. This only works if you haven't written any explicit constructors yourself. If you do have an explicit constructor, put the @Builder annotation on the constructor instead of on the class. Note that if you put both `@Value` and `@Builder` on a class, the package-private constructor that `@Builder` wants to generate 'wins' and suppresses the constructor that `@Value` wants to make._

### 2. 상속 받는 자식 객체에 빌더 패턴을 적용할 때, `@SuperBuilder` 를 사용할 것

`@Buidler` 어노테이션으로는 상속 받은 필드를 빌더에서 사용하지 못하는 제약 사항이 있다. 이를 해결하고 상속받은 필드를 빌더 패턴에 적용하기 위하여 `@SuperBuilder` 어노테이션을 사용한다.

`@SuperBuilder` 는 빌더 인스턴스를 매개변수로 받는 클래스에서 protected constructor 를 생성한다. 부모와 자식 클래스 양 쪽에 모두 어노테이션을 추가해주어야 정상 동작하며, `@SuperBuilder` 는 `@Builder`와 호환되지 않는다.

```
@SuperBuilder
public class Parent {
    private String parentField;
}
...

@SuperBuilder
public class Child extends Parent {
    private String childField;
}
```

_@SuperBuilder can generate so-called 'singular' methods for collection parameters/fields. For details, see the @Singular documentation in @Builder._

_@SuperBuilder generates a protected constructor on the class that takes a builder instance as a parameter. This constructor sets the fields of the new instance to the values from the builder._

_@SuperBuilder is not compatible with @Builder._

## References.

- https://johngrib.github.io/wiki/pattern/builder/
- https://projectlombok.org/features/Builder
- https://projectlombok.org/features/experimental/SuperBuilder
