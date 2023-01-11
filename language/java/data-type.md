# 자료형

## 기본 자료형

|자료형|크기|표현 범위|
|-|-|-|
|byte|1 byte|-128 ~ 127|
|char|2 byte|0 ~ 65,535|
|short|2 byte|-32,768 ~ 32,767|
|int|4 byte|-2,147,483,648 ~ 2,147,483,647|
|long|8 byte|-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807|
|float|4 byte|IEEE 754-1985 표준|
|double|8 byte|IEEE 754-1985 표준|

## Type Casting

- Implicit type casting (암묵적 형변환)  

  - 표현 가능한 값의 범위가 더 큰 데이터 타입으로의 변환은 암묵적으로 발생한다. 반대로 narrowing conversion일 경우에는 명시적으로 변환해 주어야 한다.  
    ``` java
    // ok: byte -> short로의 암묵적 형변환이 발생한다.
    byte one_byte = ...;
    short two_bytes = one_byte;
    int four_bytes = two_bytes;
    long eight_bytes = four_bytes;
    float half_precision = eight_bytes;  // it's also ok

    // bad: narrowing conversion은 암묵적 형변환이 발생하지 않는다.
    long eight_bytes = ...;
    int four_bytes = eight_bytes;  
    ...
    float half_precision = ...;
    long eight_bytes = half_precision;

    // ok
    long eight_bytes = ...;
    int four_bytes = (int)eight_bytes;
    ```

  - 서로 다른 자료형을 가진 변수 사이의 산술 연산에서는 피연산자 중 표현 가능한 값의 범위가 더 큰 타입으로 암묵적 형변환이 발생한다. 이때, 정수형 간의 산술 연산에서는 반드시 `int` 이상의 타입으로 형변환이 발생한다.  

    ``` java
    // bad: 표현식의 평가 결과는 int형이다.
    short ex1 = (byte)1 + (short)2;

    // ok: 두 피연산자 중에서 값의 표현 범위가 더 큰 long형이 연산 결과이다.
    long ex2 = (byte)1 + (long)2;

    // bad: 두 피연산자 중에서 값의 표현 범위가 더 큰 float형이 연산 결과이다.
    long ex3 = (long)1 + (float)2;
    ```

- Explicit type casting (명시적 형변환)  
