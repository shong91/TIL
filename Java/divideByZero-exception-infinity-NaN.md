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
