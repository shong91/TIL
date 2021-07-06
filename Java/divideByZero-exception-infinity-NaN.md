# Division by Zero in Java: Exception, Infinity, or Not a Number

##

1. 제수의 자료형이 Integer 일 때, divide by zero - ArithmeticException 을 발생시킨다.

```
// throws ArithmeticException
assertThrows(ArithmeticException.class, () -> {
    int result = 12 / 0;
});

```

2. 제수의 자료형이 float/dobule 일 때, exception 을 발생시키지 않으며 infinity/NaN 으로 리턴한다.

```
// exceptions 을 던지지 않으며, return NaN, POSITIVE_INFINITY, and NEGATIVE_INFINITY.
assertDoesNotThrow(() -> {
    float result = 12f / 0;
});

assertEquals(Double.NaN, 0d / 0);
assertEquals(Float.POSITIVE_INFINITY, 12f / 0);
assertEquals(Double.NEGATIVE_INFINITY, -12d / 0);
```

계산식 초기에 infinity/NaN 값이 나왔다면, 어떠한 산술 연산을 수행해도 그 값이 변하지 않는다.

때문에 이를 방지하기 위해

1. 0.0d, 0.0f 의 값을 정수 0 으로 치환하거나,
2. infinity/NaN 을 체크하는 방어코드를 짜는 것이 좋다.

```
if(Double.isInifinite(number) || Double.isNaN(number)) {
    // 후처리
}


```
