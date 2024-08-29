# Optional과 함수형 프로그래밍

## 정리

Java 8 이전에는 특정한 상황에서 값을 반환하지 못할 때, 예외나 null을 반환하는 방식을 이용했다. 하지만 예외는 예외적인 경우에만 던져져야 하며, 스택 트레이스를 끌고오므로 비용이 크다. null은 null을 검사하거나 다루는 과정을 필수적으로 요구한다. 

Java 8부터는 `Optional<T>`의 도입으로 이러한 문제가 해결할 수 있게 되었다. 이는 불변 컨테이너이기도 하고, `Collection<T>` 인터페이스를 구현하고 있지는 않지만 가능하기도 하다.

중요한 점 하나는 Optional을 반환 타입으로 가지는 메서드에 대해서 null 값을 리턴해서는 안된다는 것이다. 

또한, Optional은 checked exception과 그 의도가 비슷하다. 이는 API 이용자가 어떤 값도 반환되지 않을 수 있음을 마주하게 하는 까닭이다. 물론, checked exception을 던질 때는 클라이언트 측에서 보일러 플레이트가 추가적으로 필요하다. 

함수형 인터페이스 `Supplier<T>`는 Optional을 이용할 때 파라미터 타입로 자주 이용된다. `orElseGet`, `filter`, `map`, `flatMap`, `ifPresentOrElse` 등이 대표적이다. 또한, Java 9 부터는 `stream()` 메서드를 이용해 Optional을 Stream으로 변환할 수 있게 되었다. 

하지만 컨테이너 타입(collection 혹은 배열 등)의 경우에는 Optional로 래핑되지 말아야 한다. 여기에서 Optional은 거추장스럽다. 게다가 Optional은 성능이 중요한 상황에서는 그 설계 상 부적절한 경우도 있다. 

원시 타입의 래핑 객체를 지니는 옵셔널을 반환하는 것도 마찬가지다. 대신 `OptionalInt`, `OptionalDouble` 등이 존재한다. 이러한 경우에는 원시 타입의 Optional를 쓰는 편이 좋다.

그리고 Optional은 자신의 역할에만 남아야지, Optional이 콜렉션의 key / value / element로 쓰이거나 인스턴스로 저장되는 것은 권장되지 않는다. 즉, Optional은 그 이름에서 볼 수 있듯이, 반환값으로서 판단 과정에서 갖는 역할을 언제나 중시해야 한다. 


## 심화 탐구

### 출발점

55장은 Optional의 목적을 이해하고 적절히 사용하는 일에 초점을 두었다. 그러나 본문에서 제시된 내용 외에도 추가적인 Optional 사용 시 주의사항이 많다. 이러한 주의사항은 대개 Optional의 메서드를 적절히 파악하지 못하거나 사용하지 못해 발생한다. [dzone](https://dzone.com/articles/using-optional-correctly-is-not-optional)에서 이를 다루는 좋은 글을 참고하면 좋다.

언제나 새로운 개념의 도입이나 등장은 이전 개념에 빚지고 있다. Java 8에서 람다 표현식, Stream API와 함께 Optional이 도입되었음을 감안한다면 Optional 역시 함수형 프로그래밍에 뿌리를 두고 있음을 추측할 수 있다. 즉, Optional의 등장은 Java 내부적으로는 NPE와 관련된 것으로, 외부적으로는 함수형 프로그래밍이라는 더 큰 범위에서 이해되어야 한다.

### 설명

#### Optional 도입과 현황

Java 8에서 Optional이 도입된 이후, Oracle은 technical resources 문서에서 다음과 같이 밝힌다.

> It is important to note that the intention of the Optional class is not to replace every single null reference. Instead, its purpose is to help design more-comprehensible APIs so that by just reading the signature of a method, you can tell whether you can expect an optional value. This forces you to actively unwrap an Optional to deal with the absence of a value.
> [Oracle: java8-optional](https://www.oracle.com/technical-resources/articles/java/java8-optional.html)

즉, Optional의 목적은 메소드의 시그니처를 읽는 것만으로도 더 이해하기 좋은 API 설계를 돕는 것이다. 그러나 Optional의 목적을 지켜도 (심지어는 위 55장에서 밝힌 내용들을 지켜도) 실제 코드에서는 나쁜 냄새(code smell)가 나기 쉽다.

그런데 왜 이렇게 Optional은 혼동을 일으키기 쉬운걸까? 게다가 "A Study on the Current Status of Functional Idioms in Java" (Hiroto Tanaka, 2019)에서는 Optional을 잘못 이용하는 사례를 발견함과 함께, Java에서의 함수형 구문(functional idiom)이 자주 사용되지 않는다고 밝힌 바 있다. 해당 논문에서 분석한 바에 따르면, github에서 java로 작성된 100개 프로젝트에서 Optional을 채택한 건 오직 2% 뿐이었다. (p. 2417)

물론, 이는 github에 공개된 프로젝트라는 점을 감안해야 하기도 하는 수치다. 특히, 호환성을 유지하기 위해 함수형 프로그래밍 관련 기능 자체를 도입하지 않은 경우도 있었다. 게다가 selenium의 커밋 기록과 같이 Optional이라는 함수형 구문을 받아들이지 않는 이유를 함수 호출의 복잡성을 피하기 위함으로 꼽았다.


#### Optional과 함수형 프로그래밍

- Monad와의 개념적 유사성

함수형 프로그램이에서는 모나드(Monad)라는 개념이 존재한다. 모나드는 간단하게 말하자면, 함수형 프로그래미에서의 디자인 패턴이다. 모나드는 값과 연산을 캡슐화하는 구조나 컨테이너다.

Java의 Optional은 모나드와 개념적으로 유사하다. 이는 데이터의 연속적인 변환이나 조작을 안전하게 처리하는 방식에서 드러난다.

- 기본 연산 관점

모나드에는 Unit과 Bind라는 기본 연산이 존재한다. Unit은 주어진 값을 감싸는 역할을 하고, Bind 연산은 모나드 안의 값을 꺼내어 연속적인 계산 혹은 변환을 수행하고 새로운 모나드 값을 반환하도록 한다.

Optional이 value를 래핑하는 방식이나, `orElseGet` 등의 메소드를 이용하는 방식을 떠올려보면 이해가 쉬울 것이다. Unit 연산을 위해서는 `Optional.of()` 혹은 `Optional.nullable()`을 이용하고, Bind 연산을 위해서는 `Optional.flatMap()` 연산을 이용하는 셈이다. 

- 모나드가 따라야 하는 규칙

모나드는 좌항 등원(left identity), 우항 등원(right identity), 결합 법칙(associativity)를 만족해야 한다. 이러한 규칙은 모나드의 안정성을 보장한다. 

좌항 등원은 모나드에 값을 넣고, 그 값에 함수를 적용하는것과 직접 함수에 그 값을 적용하는 것은 동일하다는 뜻이다. `bind(unit(x), f)`가 `f(x)`와 같은 값을 가져야 한다.

우항 등원은 모나드에 함수를 적용한 뒤, 이를 다시 모나드로 변환하는 것은 원래의 모나드와 동일해야 한다는 뜻이다. `bind(m, unit)`은 `m`과 같은 값을 가져야 한다. 

결합 법칙은 모나드 내 함수 체이닝은 적용 순서에 상관없이 동일한 결과를 가져야 함을 의미한다. 즉, `bind(bind(m, f), g)`는 `bind(m, x -> bind(f(x0, g))`와 같은 결과를 가져야 한다. 

하지만 Java의 Optional은 좌항 등원을 만족하지 않는데, 이는 다분히 의도적이다. JDK 측에서는 Optional이 [다른 언어보다 스코프가 좁으므로, 그 이상을 고려하지는 않았다](https://mail.openjdk.org/pipermail/lambda-dev/2013-February/008305.html)고 밝힌 바 있다. 재밌는 것은, 여전히 JDK 팀이 "an option monad"라고 Optional을 지칭한다는 점이다. 

### 나가며

함수형 프로그래밍 언어 Haskell에는 Maybe라는 모나드가 존재한다. Haskell에서는 다음과 같이 Maybe 모나드를 이용할 수 있다고 한다.

```haskell
data Maybe a = Just a | Nothing
```

sqrt를 정의하고 싶을 때, sqrt에 들어오는 아규먼트가 양수여야 한다면 sqrt 함수는 haskell에서 다음과 같이 이용될 수 있다.

```haskell
sqrt x | x >= 0 = Just $ ...
       | otherwise = Nothing
```

이번 글에서는 Java 8부터 도입된 Optional과 다른 언어를 함께 살펴봄으로써, 오히려 Java가 함수형 프로그래밍의 특성을 통해 문제를 해결하려고 했음을 알게 되었다.

### References

https://www.baeldung.com/java-monads-optional

https://sidburn.github.io/blog/2016/04/03/understanding-bind

https://www.oracle.com/technical-resources/articles/java/java8-optional.html

https://mail.openjdk.org/pipermail/lambda-dev/2013-February/008305.html

https://medium.com/@afcastano/monads-for-java-developers-part-1-the-optional-monad-aa6e797b8a6e

https://wiki.haskell.org/Maybe