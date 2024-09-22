## 문서화 주석에 대한 상속 방법

### 정리

**내용 요약**

api를 작성할 때는 잘 작성된 문서화 주석이 동반되어야 한다.

자바 언어로 문서화 주석을 작성할 때는 javadoc이 기술된 설명을 문서로 변환해준다.

그러나 올바른 문서화 주석을 작성하기 위해서는 몇 가지 수칙을 지켜야하며 다음과 같다.

1. 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 대하여 문서화 주석을 달아야 한다.

    + 단, 기본 생성자에는 문서화 주석을 달 수 없으니 공개 클래스는 기본 생성자를 작성해서는 안 된다.

2. 메서드에 대한 문서화 주석을 작성할 때는 메서드와 클라이언트 사이의 규약을 명료하게 작성해야 한다.

    + 상속용으로 설계된 클래스의 메서드가 아니라면, 메서드가 무엇을 하는지에 대한 내용이 반드시 들어가야한다.

    + 클라이언트가 해당 메서드를 호출하기 위한 전제조건과, 메서드가 성공적으로 수행된 이후 만족해야 하는 사후조건을 모두 나열해야 한다.

3. 제네릭 타입이나 제네릭 메서드를 문서화할 때는 모든 타입 매개변수에 주석을 달아야 한다.

4. 패키지를 설명하는 문서화 주석은 package-info.java 파일에 작성하고, 해당 파일은 패키지 선언을 포함해야 한다.

5. 모듈 관련 문서화 주석ㄷ은 module-info.java 파일에 작성한다.

6. javadoc은 메서드 주석을 상속시킬 수 있다. 문서화 주석이 없는 api 요소가 있다면 자바독이 가장 가까운 문서화 주석을 찾아준다.

    + 이 때, 상위클래스 자체보다 해당 클래스가 구현한 인터페이스를 먼저 찾는다.

    + @inheritDoc 태그를 이용하면 상위 타입의 문서화 주식 일부를 상속할 수 있다. 이 기능을 활용해 여러 문서화 주석에 대한 유지보수를 쉽게 할 수 있다.


### 심화 탐구


**출발점**

본문을 읽으며 문서화 주석을 작성하는 문서화 주석의 상속에 대한 부분이 구체적으로 이해가 되지 않았다.

해당 문서화 주석의 상속 부분은 특히 문서화 주석을 작성할 때, 작성 효율을 높여주는 내용이라 여겨지기에 구체적으로 살펴보며 사용 예시까지 정리해보도록 하겠다.


**설명**

먼저 문서화 주석을 작성하기 위해서는 패키지 내에 package-info.java 파일을 생성해야 한다.

```java
// package-info.java 파일
package docummentation;
```

---

문서화 주석을 상속하기 위해서는 기본적으로 1) 클래스끼리의 상속관계가 이루어지거나, 2) 문서화 주석이 작성된 인터페이스를 구현해야한다.

첫 번째에 해당하는 클래스끼리의 상속관계에 대한 예시를 작성하면 다음과 같다.


```java
package docummentation;

public class Parent {

    /**
     * Print welcome message to employee.
     *
     * @param name
     *            your name
     */
    public void hello(String name){
        System.out.println("Hello " + name);
    }

}
```

부모에 해당하는 Parent 클래스에 hello 메서드에 대한 문서화 주석이 작성되어 있다.


```java
package docummentation;

public class Child extends Parent{

    /**
     * {@inheritDoc}
     */
    public void hello(String name){
        System.out.println("hi " + name);
    }
}

```

Parent 클래스를 상속하고 있는 Child 클래스가 {@inheritDoc} 태그를 통해 부모 클래스의 주석을 가져오고 있다.

주의해야 할 점은, 오버라이드된 메서드만이 부모 클래스에서 정의된 원본 메서드의 주석을 가져온다는 것이다.

---

만일 부모 클래스의 문서화 주석을 상속받고 싶지 않고 그대로 보여주고 싶다면 @inheritDoc 대신 @see 태그를 사용하여 링크를 걸어줄 수 있다.

```java
package docummentation;

public class Child extends Parent{

    /**
     * @see Parent#hello(String)
     */
    public void hello(String name){
        System.out.println("hi " + name);
    }
}
```

이와 같이 작성하면 javadoc이 해당 주석을 변환해줄 때 링크를 걸어주어 바로 Parent 클래스의 hello 메서드로 이동할 수 있게 해준다.


---


Reference:

https://self-learning-java-tutorial.blogspot.com/2018/01/javadoc-inheritdoc-inherit-documentation.html