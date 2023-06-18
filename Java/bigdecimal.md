# BigDecimal 정복기

금융 도메인이나 정산 시스템을 개발할 때에는, 무엇보다 가장 중요한 것이 숫자의 정합성이다. 돈에 가장 민감한 도메인이기에, 1원 단위의 오차로 인한 개발 건 수정이 잦게 일어났던 경험이 있다.

비용 관련 프로그램을 개발하면서 처음 마주한 것이 BigDecimal 이라는 클래스였다.

자바는 IEEE 754 부동 소수점 방식을 사용하여, 정확한 실수를 저장하지 않고 근사치 값을 저장한다. 때문에 `float`, `double` 은 소수점의 정밀도가 정확하지 않아 오차가 발생하게 된다.

| Type   | 범위                              |
| ------ | --------------------------------- |
| float  | 1.4E-45 ~ 3.4028235E38            |
| double | 4.9E~324 ~ 1.7976931348623157E308 |

이과 달리, BigDecimal 은 부동 소수점 방식이 아닌, 정수를 이용해 실수를 표현한다. 이를 통해 정확한 소수점 연산을 가능하게 한다.

## BigDecimal 클래스

본격적으로 BigDecimal 클래스를 파악해보자.

BigDecimal 에서 유효 숫자의 역할을 하는 unscaledValue와 32-bit의 scale로 이루어져 있고, 실제로 이 값들이 나타내는 BigDecimal 값은 unscaledValue \* (10 ^ -scale) 이다. 즉, scale이 0 이상이면 scale은 소숫점 오른쪽 자리의 숫자 개수가 되고, 음수이면 unscaledValue에 10이 (-scale)번 곱해지게 된다.

```
private final BigInteger intVal;
private final int scale;
private transient int precision;
private transient String stringCache;
private final transient long intCompact;
```

- intVal: 위에서 설명한 unscaledValue를 나타낸다. (BigInteger)
- scale: 위에서 설명한 scale을 나타낸다.
- precision: 유효숫자 개수를 나타내고 처음 계산되기 전까지는 0이다. 0이 아니면(즉, 계산 된 값이라면) precision에 있는 값은 그 만큼의 정확성을 보장한다.
- stringCache: Canonical string representation을 저장하기 위함이다. 즉, 우리가 일반적으로 숫자를 쓸 때 그 표현법(ex. 17.2234)을 string으로 저장하고 있다. 아마도 toString() 같은 메소드를 만날 때 매번 계산하는게 비효율적이기 때문에 클래스 내부에서 저장하고 있는것으로 추측할 수 있겠다.
- intCompact: 위에서 설명한 unscaledValue를 나타낸다. (Long) 표현하고자 하는 수의 유효숫자 개수가 적어 그 절댓값을 Long 타입으로 나타낼 수 있을때는 intCompact에 저장된 값을 사용하고, 그렇지 않으면 intVal에 저장후 사용한다.

**1. 숫자 표현하기**

앞서 말했다시피, 부동 소수점 방식을 사용하는 double 로는 완벽하게 딱 떨어지는 소수를 표현할 수 없다. 그러므로, 정확한 1.1 을 생성하고자 한다면 String "1.1" 이용하여 생성할 수 있다.

```
BigDecimal value = new BigDecimal(1.1);     // 1.100000088817841970015...
BigDecimal value = new BigDecimal("1.1");   // 1.1
```

**2. 숫자 비교하기**

```
public class BigDecimal extends Number implements Comparable<BigDecimal> {}
```

BigDecimal 은 Comparable 을 구현하고 있기 때문에, equals() 를 통해 비교할 수 있다. 여기서 주의할 점은, `0` 과 `0.000` 은 다르다는 점이다.

이는 BigDecimal 객체에는 val과 scale 속성이 있기 때문이다. 또한 euqals() 메서드는 val 과 scale이 모두 같은 경우에만 두 개의 BigDecimal 객체를 같다고 간주합니다. 즉, BigDecimal 42는 euqals()으로 비교하면 42.0과 같지 않습니다.

_This is because a BigDecimal object has value and scale attributes. Moreover, the equals method considers two BigDecimal objects equal only if they are equal in both value and scale. That is to say, BigDecimal 42 is not equal to 42.0 if we compare them with equals._

```
BigDecimal BD1 = new BigDecimal("0");
assertThat(BigDecimal.ZERO.equals(BD1)).isTrue(); // pass

BigDecimal BD2 = new BigDecimal("0.0000");
assertThat(BigDecimal.ZERO.equals(BD2)).isTrue(); // fail
```

**_1. compareTo_**

앞서 말했다시피 BigDecimal 은 Comparable 을 구현하고 있기 때문에, compareTo 이용해 값을 비교하는 것 역시 가능하다.
우리는 **BigDecimal.ZERO.compareTo(givenBdNumber) == 0** 을 이용하여 두 숫자가 같은지 확인할 수 있다.

```
assertThat(BigDecimal.ZERO.compareTo(BD1)).isSameAs(0); // pass
assertThat(BigDecimal.ZERO.compareTo(BD2)).isSameAs(0); // pass
```

**_2. signum_**

또한, signum() 을 사용할 수도 있다. signum() 을 이용하여 주어진 객체의 값이 음수(-1), 0, 양수(1) 인지 확인할 수 있다. signum() 은 scale 속성은 무시한다.

```
assertThat(BD1.signum()).isSameAs(0);   // pass
assertThat(BD2.signum()).isSameAs(0);   // pass
```

**3. 숫자 연산하기**

```
BigDecimal add(BigDecimal val)
BigDecimal subtract(BigDecimal val)
BigDecimal multiply(BigDecimal val)
BigDecimal divide(BigDecimal val)
BigDecimal remainder(BigDecimal val)
```

기본 사칙연산을 아래 메서드로 수행할 수 있다. 주의할 점은 나눗셈 연산 시 반올림 처리 방법을 지정해주어야 한다는 점이다.

아무리 BigDecimal 클래스일지라도, 나누어 떨어지지 않는 수는 정확하게 표현할 수 없다. 따라서 divide() 를 사용할 때에는 소수점 몇 번째 자리까지, 어떻게 처리할 것인지 명시해 주어야 한다.

```
public BigDecimal divide(BigDecimal divisor, MathContext mc){}
```

## References.

- https://huzz.tistory.com/24
- https://www.baeldung.com/java-bigdecimal-zero
