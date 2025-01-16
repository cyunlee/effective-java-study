# 아이템 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

## ✔️ 인터페이스(Interface)
> 다른 클래스를 작성할 때 기본이 되는 틀을 제공하면서, 다른 클래스 사이의 중간 매개 역할까지 담당하는 일종의 추상 클래스

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다.
그리고, <u>클라이언트에게 **자신의 인스턴스로 무엇을 할 수 있는지** 알려주기 위한 용도로만 사용해야 한다.</u>

### 인터페이스를 잘못 사용한 예

인터페이스를 잘못 사용한 대표적인 예로 **상수 인터페이스 안티패턴**이 있다.

<hr>

## ✔️ 상수 인터페이스(Constant Interface)
> 오직 상수만을 정의한 인터페이스

```java
public interface Constants {
    String APP_NAME = "MyApplication";
    int MAX_USERS = 100;
    double PI = 3.1415926535;
}
```
인터페이스는 인스턴스를 가질 수 없기 때문에 일반 필드를 선언할 수 없다.

따라서, 책의 예제에서는 static final 키워드를 붙였지만 `public static final` 키워드를 생략해도 모든 필드는 자동으로 **static final** 상수로 취급된다.

```java
public interface ObjectStreamConstants {

    static final short STREAM_MAGIC = (short)0xaced;

    static final short STREAM_VERSION = 5;

    static final byte TC_BASE = 0x70;

    static final byte TC_NULL = (byte)0x70;
    
    .
    .
    .
    
    public static final int PROTOCOL_VERSION_1 = 1;

    public static final int PROTOCOL_VERSION_2 = 2;
}
```

위 코드에서 볼 수 있듯이 `java.io.ObjectStreamConstants`와 같은 자바 플랫폼 라이브러리에도 상수 인스턴스가 존재하지만 사용하면 안된다.

<hr>

## ✔️ 상수 인터페이스의 문제점

### 1️⃣ 내부 구현 노출
클래스 내부의 상수를 API로 공개하게 되어 캡슐화가 깨진다.

### 2️⃣ 사용자 혼란 유발
인터페이스의 본래 목적(행위 정의)과 맞지 않아 클라이언트 코드에 혼란을 준다.

### 3️⃣ 종속성과 유지보수 문제
클라이언트 코드가 상수에 종속되며, 상수를 더 이상 사용하지 않아도 바이너리 호환성을 위해 인터페이스를 유지해야 한다.

#### 바이너리 호환성(Binary Compatibility)이란?
> 이미 컴파일된 클래스 파일(바이트코드)이 변경 없이 기존에 컴파일된 다른 클래스 파일과 호환성을 유지하는 것

즉, 소스 코드를 다시 컴파일하지 않아도 변경된 클래스가 기존 클래스와 함께 정상적으로 동작하는지를 말한다.

### 4️⃣ 하위 클래스 오염
`final`이 아닌 클래스가 상수 인터페이스를 구현하면, 모든 하위 클래스도 불필요한 상수를 상속받아 오염된다.

<hr>

## ✔️ 상수 인터페이스의 대안

### 1️⃣ 클래스나 인터페이스 자체에 추가

```java
//Integer 클래스
public final class Integer {
    public static final int MIN_VALUE = 0x80000000; // -2^31
    public static final int MAX_VALUE = 0x7fffffff; // 2^31-1
}

//Double 클래스
public final class Double {
    public static final double MIN_VALUE = 4.9e-324; // 가장 작은 양의 수
    public static final double MAX_VALUE = 1.7976931348623157e+308; // 가장 큰 값
}

public class TestConstants {
    public static void main(String[] args) {
        System.out.println("Integer의 최소값: " + Integer.MIN_VALUE);
        System.out.println("Integer의 최대값: " + Integer.MAX_VALUE);

        System.out.println("Double의 최소값: " + Double.MIN_VALUE);
        System.out.println("Double의 최대값: " + Double.MAX_VALUE);
    }
}
```

클래스 내부에 상수를 정의하면, 상수와 클래스의 연관성을 명확히 표현할 수 있다.

#### 인터페이스에 상수를 사용해도 괜찮은 경우

```java
public interface HttpStatus {
    int OK = 200;
    int NOT_FOUND = 404;
    int INTERNAL_SERVER_ERROR = 500;
}
```
<u>상수를 공유하기 위해 **상수 인터페이스**를 따로 만들지 말라는 의미이다!</u>

만약 상수가 **인터페이스의 행위와 강하게 연관있는 경우**라면 사용해도 괜찮다. 

위의 코드처럼 **HTTP 상태 코드**는 HTTP 통신과 강하게 연관된 값들이기 때문에 인터페이스에 상수를 정의해도 의미가 있다.


### 2️⃣ enum

상태나 값이 제한적일 경우 `enum`을 사용하는 것이 직관적이고 타입 안전성을 제공한다.

```java
public enum HttpStatus {
    OK(200), NOT_FOUND(404), INTERNAL_SERVER_ERROR(500);

    private final int code;

    HttpStatus(int code) {
        this.code = code;
    }

    public int getCode() {
        return code;
    }
}
```

### 3️⃣ `final` 클래스로 상수 정의
```java
public final class HttpStatus { //상속 방지
    private HttpStatus() {}; //인스턴스화 방지
    
    public static final int OK = 200;
    public static final int NOT_FOUND = 404;
    public static final int INTERNAL_SERVER_ERROR = 500;
}
```

유틸리티 클래스에 정의된 상수를 사용하기 위해서는 `HttpStatus.OK`처럼 **클래스 이름까지** 함께 명시해야 한다.

**static import**하면 클래스 이름을 생략할 수 있다.

```java
import static effectivejava.chapter4.item22.constantutilityclass.HttpStatus;

public class Test {
    String getStatus(int httpStatus) {
        if(httpStatus == 200) return "OK";
        else if(httpStatus == 404) return "NOT FOUND";
        else if(httpStatus == 500) return "INTERNAL SEVER ERROR"
    }
}
```

<hr>

## ✔️ 결론

인터페이스는 **타입을 정의하는 용도**로만 사용해야 하며, 상수 공개용 수단으로 사용하지 말자! 




