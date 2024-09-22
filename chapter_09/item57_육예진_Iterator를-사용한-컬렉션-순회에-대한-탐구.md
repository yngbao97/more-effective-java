## Iterator를 사용한 컬렉션 순회에 대한 탐구

### 정리

**내용 요약**

지역변수는 사용처를 명확히 하고, 엉뚱한 곳에서의 사용을 방지하기 위해 범위를 최소화 하는 것이 좋다.  
지역변수의 범위를 줄이는 방법은 다음과 같다.  
1. 가장 처음 쓰이기 직전 선언하기  
    - 미리 선언하면 코드가 어수선하고 초기값을 찾기 어렵다.
2. 대부분의 경우 선언과 동시에 초기화하기  
    - 초기화에 필요한 정보가 충분하지 않다면 선언을 미룬다.
3. 반복문의 변수는 반복문이 종료된 후에도 사용해야 하는 상황이 아니라면 while문보다 for문을 사용하기
4. 메서드를 작게 유지하고 한가지 기능에 집중하기  
    - 한 메서드에 많은 기능이 있다면 기능과 상관없는 변수에 접근할 위험이 있다.


### 심화 탐구

**출발점**

지역변수를 사용할 범위 안에서만 유효하도록 접근 가능 범위를 줄이라는 내용으로, 받아들이기 어려운 내용은 아니었다.  
하지만, 반복문의 변수를 설명하는 부분의 코드 예시에서 Iterator를 사용했는데 지금까지 사용해오던 반복문의 형태와 조금 달라서   
Iterator의 순회 방법이 잘 이해되지 않아 해당 코드를 테스트해보고 각각의 차이점을 정리해보고자 했다.


**설명**

<hr>

1. 전통적인 for문과 Iterator를 사용하는 변형된 for문

    지금까지 일반적으로 사용해왔던 for문의 형태는 다음과 같았다.
    ```java
    for (int i = 0; i < N; i++) {
        doSomething(about(i));
    }
    ```
    이것은 `initialization; condition; update`의 구조를 가진다.  
    **initialization**: 반복문이 시작되기 전에 한번, 반복문 변수를 초기화 하는 부분이다.  
    **condition**: 매 반복마다 평가되는 조건으로, 해당 조건이 true일때 루프 바디가 실행된다.  
    **update**: 수식에 따라 매 반복마다 루프 바디가 실행된 후 반복문 변수를 갱신한다.

    그러나 Iterator를 사용하는 for문의 형태는 다음과 같다.
    ```java
    for (Iterator<String> i = c.iterator(); i.hasNext(); ) {
        doSomething(about(i.next()));
    }
    ```
    이처럼 **객체를 초기화 하는 부분**과 **반복 여부를 판단하는 조건부**만 존재한다.  
    next()메서드를 사용해 변수가 가리키는 원소를 갱신하고 반환하며, 이 부분은 루프 바디에서 실행된다는 점이 전통적인 for문과의 차이점이다.

    Iterator는 특수한 문법을 활용해 컬렉션을 순회하기 때문에 전통적인 for문처럼 update 부분을 소괄호 안에 선언하는 것이 구조적으로 맞지 않다.


2. Iteraor의 `hasNext()` 메서드와 `next()` 메서드

    책에 나와있는 예시코드를 옮겨 실제 실행 가능한 코드로 수정했다.

    ```java
    import java.util.ArrayList;
    import java.util.Iterator;
    import java.util.List;

    public class Test {
        public static void main(String[] args) {
            
            List<String> c = new ArrayList<>(List.of("1", "2", "3", "4", "5"));
            List<String> c2 = new ArrayList<>(List.of("1", "2", "3", "4", "5"));
                
            for (Iterator<String> i = c.iterator(); i.hasNext(); ) {
                System.out.print(i.next() + " ");
            }
            // 1 2 3 4 5

            System.out.println();
            
            Iterator<String> i = c.iterator();
            while(i.hasNext()) {
                System.out.print(i.next() + " ");
            }
            // 1 2 3 4 5

            System.out.println();
            
            Iterator<String> i2 = c2.iterator();
            while (i.hasNext()) {
                System.out.println(i2.next() + " ");
            }
            // 출력없음
        }
    }
    ```
    공식문서에 따르면 `hasNext()` 메서드는 현재 원소에서 다음 원소가 있는지 확인하며 `next()` 메서드는 순회에서 다음 원소를 반환한다.
    예시 코드를 실행시켜보니 Iterator는 원소를 순회할때 next() 메서드에 따라 가리키는 원소가 갱신되고 해당 상태를 유지하며 순회가 진행되는 것으로 보였다.   
    
    실제로 for문에서 사용된 `Iterator<String> i = c.iterator();` 변수는 유효범위가 for문의 루프바디이기 때문에 루프가 끝나면 변수 자체가 유효하지 않았지만,  
    2개의 while문에 사용된 `Iterator<String> i = c.iterator();` 변수는 while문이 끝나도 유효하기 때문에  
    첫번째 while문에서 마지막 원소까지 순회가 끝난 상태를 유지한 채 두번째 while문이 실행된다.

    이때, 두번째 while문의 조건이 순회가 끝난 상태인 순회 변수 i를 기준으로 하고 있기 때문에 while문 루프 바디로 진입하지 못한다. 따라서 출력값이 아무것도 없게 되는 것이다.

3. Iterator 변수 사용 범위에 따른 반복문 for, while

    for문에서 사용되는 Iterator 변수는 루프 바디 안에서만 유효함으로 상태 업데이트 및 원소 접근이 루프 바디 안에서만 가능하지만, while문은 Iterator 변수가 별도로 선언되고 바디루프와 별개로 상태 업데이트가 가능하기 때문에 보다 자유로운 활용이 가능하다.

    저자는 지역변수의 접근 범위를 줄여야 한다는 이유에서 for문 사용을 권장하지만,  
    무작정 for문을 고집하기보단 변수의 활용 목적과 범위를 이해하고 그에 맞게 명시적으로 사용하려는 노력이 무엇보다 중요할 것 같다.

<hr>

### 참고자료
- [Interface Iterator - Oracle](https://docs.oracle.com/javase/8/docs/api/java/util/Iterator.html)