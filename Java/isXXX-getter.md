# Lombok으로 boolean 필드의 getter/setter 생성하기

개발을 하다보면, boolean 필드를 사용하는 경우가 자주 있다.

응답 DTO 에 아래와 같이 boolean 필드값을 포함하여 리턴하려고 하는데, json으로 직렬화 하는 과정에서 is 가 빠지고 XXX 형태로 리턴하게 된다.

```
@Getter
public class Response {
    private String name;
    private int age;
    private boolean isActive;
}
```

```
{
    "name": "홍길동",
    "age": 20,
    "active": true
}
```

**_왜 그럴까?_**

lombok 의 `@Getter` 어노테이션은, primitive boolean 타입 필드에 대하여 getter 메서드를 생성할 때, 접두사로 `get`이 아닌 `is`를 사용한다. (boolean isFoo -> getIsFoo (x) isFoo (o) )

또한, `is` 를 접두사로 가지는 boolean 필드의 경우, setter/getter 을 생성할 때 따로 접두사를 붙이지 않는다.

_A default getter simply returns the field, and is named getFoo if the field is called foo (or isFoo if the field's type is boolean)._

_For boolean fields that start with is immediately followed by a title-case letter, nothing is prefixed to generate the getter name._

즉, 위의 boolean isActive 필드의 setter/getter 는 아래와 같이 만들어 질 것이다.

```
public void setActive() {}
public boolean isActive(){}
```

그나마 isXXX 필드일 때에는 아주 어색해보이지는 않은데, boolean 필드가 hasXXX, canXXX 등 다른 접두사를 가진다면 조금 문제가 생긴다.

```
@Getter
public class Response {
    private String id;
    private boolean hasNext;
}
```

위의 경우 hasNext 필드는 `isHasNext()` 라는 getter method 를 가지게 되는데, 굉장히 어색해 보인다. 어떻게 해야 boolean 필드에 대해서도 적절한 형태의 getter method 를 만들 수 있을까?

### 해결

**1. wrapper class Boolean 사용**

wrapper 타입의 Boolean 을 사용하면 is 가 생략되지 않고, getXXX 형태의 getter method 를 사용할 수 있다.

하지만 wrapper class 는 기본값이 null 이기 때문에, null 을 허용하지 않는 불리언 값이라면 primitive 타입을 사용하는 것이 적절하다.

**2. 직접 get 메서드 생성**

lombok 으로 생성되는 getter 메서드를 사용하지 않고, 직접 getter 메서드를 생성하는 방법이다.

롬복에서는 동일한 매개변수를 가진 메서드가 이미 존재하는 경우, 메서드를 생성하지 않는다. 즉, 사용자가 getter/setter 메서드를 생성해 놓았다면, 롬복은 메서드가 이미 있는 경우 메서드 생성을 건너뛴다.

```
@Getter
public class Response {
    private String name;
    private int age;
    private boolean isActive;

    public boolean getIsActive(){
        return this.isActive;
    }
}
```

위와 같이 메서드를 작성하면, `isActive` 에 대한 getter method 는 이미 존재하는 것으로 판단하여 메서드 생성을 하지 않는다.

## References.

- https://projectlombok.org/features/GetterSetter
