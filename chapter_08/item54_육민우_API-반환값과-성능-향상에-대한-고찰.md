# API 반환값과 성능 향상에 대한 고찰

## item54 - null이 아닌, 빈 컬렉션이나 배열을 반환하라

### 🔍 내용 요약

컬렉션이나 배열 같은 컨테이너를 반환하는 메서드에서 컨테이너가 비었을 때 null을 반환하면 클라이언트에서 컨테이너가 비었는지 확인하는 방어 코드를 넣어줘야 한다.
컨테이너가 비어있는 경우가 거의 없는 상황에서는 한참 뒤에 오류가 발생하기도 하고 반환하는 메서드 자체도 null을 반환하는 상황을 특별히 취급해줘야 해서 코드가 복잡해진다.

빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 쪽이 낫다는 주장도 있지만 이는 두 가지 면에서 틀린 주장이다.

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.(빈 불변 컬렉션, 길이 0인 배열)

<br>

---

### 🧐 심화 탐구

컬렉션이나 배열 같은 컨테이너 개념의 객체를 반환하는 메서드에서 컨테이너가 비었을 때 null 대신 빈 컨테이너를 반환해서 이를 사용하는 측에서 방어 코드를 작성하는 수고와 빈 컨테이너로 인해 발생할 수 있는 예상치 못한 오류를 막고자 하는 것이 이번 아이템의 핵심으로 보였다. 빈 컨테이너를 반환하는게 확실한 성능 저하를 유발하는 것이 아니라면 일관된 타입의 객체를 반환하기도 하고 개념적으로도 null보다 빈 컨테이너를 반환하는 것이 더 자연스럽게 느껴져서 충분히 납득할 수 있는 주장이었다.

지난 프로젝트를 진행하며 컨설턴트님의 코드 리뷰에서 예외를 던지는 코드는 비용이 높아서 성능상 문제가 될 수 있기 때문에 정말 예외가 발생했는지 먼저 구분하고 예외를 던질만한 상황이 아니면 반환값을 통해 알리는 것이 좋다는 리뷰를 받았는데 이와 관련해서 간단하게 null과 exception을 던지는 메서드의 성능 테스트를 ChatGPT의 도움을 받아서 작성해보았다.

```java
package item54;

public class ExceptionVsNullTest {

    static int cnt;

    // Method that returns null on error
    public static String returnNullOnError(int value) {
        if (value < 0) {
            return null;
        }
        return "Valid value: " + value;
    }

    // Method that throws an exception on error
    public static String throwExceptionOnError(int value) throws IllegalArgumentException {
        if (value < 0) {
            throw new IllegalArgumentException("Negative value not allowed");
        }
        return "Valid value: " + value;
    }

    // Benchmark method for returnNullOnError
    public static long benchmarkNullMethod(int iterations) {
        long startTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            String result = returnNullOnError(-1);
            if (result == null) {
                cnt++;
            }
        }
        return System.nanoTime() - startTime;
    }

    // Benchmark method for throwExceptionOnError
    public static long benchmarkExceptionMethod(int iterations) {
        long startTime = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            try {
                String result = throwExceptionOnError(-1);
            } catch (IllegalArgumentException e) {
                cnt++;
            }
        }
        return System.nanoTime() - startTime;
    }

    public static void main(String[] args) {
        int iterations = 10000000;

        // Benchmark returnNullOnError
        long nullMethodTime = benchmarkNullMethod(iterations);
        System.out.println("Time taken for returnNullOnError: " + nullMethodTime / 1000000 + " ms");

        // Benchmark throwExceptionOnError
        long exceptionMethodTime = benchmarkExceptionMethod(iterations);
        System.out.println("Time taken for throwExceptionOnError: " + exceptionMethodTime / 1000000 + " ms");
    }
}

// 출력
// Time taken for returnNullOnError: 5 ms
// Time taken for throwExceptionOnError: 5111 ms
```

음수가 파라미터로 들어오면 null을 반환하는 메서드와 예외를 던지는 메서드를 작성해서 음수 입력에 대해 사용자에게 알려주는 메서드를 작성하고 이를 천만번 반복했을 때 실행시간을 보여주는 코드로 작성했다. 결과에서는 대략 1000배 정도의 실행시간 차이가 발생하는 것으로 확인됐다. 컴파일러의 최적화로 실제 프로젝트 환경과는 다른 성능 차이를 보일 수 있다.

코드 리뷰를 통해 양수가 들어올 것으로 기대한 이 메서드들의 경우 음수가 파라미터로 들어올 경우 메서드의 정상 동작 자체에 문제가 생기는 것이 아니어서 null을 반환하는 것이
맞다고 생각했었는데 이번 아이템을 통해(컨테이너 반환은 아니지만) 빈문자열을 반환하는 것이 나은지에 대한 고민도 필요한 것 같다.

<br>

---

### 🧠 고민해볼 점
