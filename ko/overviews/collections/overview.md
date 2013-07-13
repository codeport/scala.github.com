---
layout: overview-large
title: 변경할 수 있는 컬렉션과 변경할 수 없는 컬렉션

disqus: true

partof: collections
num: 2
language: ko
---

시스템적으로 스칼라의 컬렉션을 변경할 수 있는(mutable) 컬렉션과 변경할 수 없는(immutable)
컬렉션으로 나눌 수 있다. _변경할 수 있는_ 컬렉션은 바로 수정하고 확장할 수 있다. 즉, 컬렉션
요소를 부가작용으로 변경하고 추가하거나 제거할 수 있다. 반면에 _변경할 수 없는_ 컬렉션은
절대 변경되지 않는다. _변경할 수 없는_ 컬렉션에서도 추가나 삭제, 갱신같은 작업을 할 수 있지만
이는 기존의 컬렉션은 변경하지 않고 놔둔 채 새로운 컬렉션은 반환한다.

모든 컬렉션 클래스는 `scala.collection` 패키지와 `scala.collection` 패키지의 하위
패키지인 `mutable`, `immutable`, `generic`에 있다. 클라이언트 코드에서 필요한
컬렉션 클래스의 대부분은 `scala.collection`, `scala.collection.immutable`,
`scala.collection.mutable` 패키지에 있다.

`scala.collection.immutable` 패키지의 컬렉션은 변경되지 않는다는 것을 보장하고 생성된
이후에 절대로 변경되지 않을 것이다. 그래서 언제 어디서든 반복적으로 같은 컬렉션에 접근하더라도
항상 같은 요소를 가진 컬렉션에 접근한다는 것을 믿을 수 있다.

`scala.collection.mutable` 패키지의 컬렉션은 컬렉션을 즉석에서 변경하는 작업을 가지고
있다. 그러므로 변경할 수 있는 컬렉션을 다룰 때는 코드가 언제 컬렉션을 변경하는 지를
이해해야 한다.

`scala.collection` 패키지의 컬렉션은 변경할 수 있을 수도 없을 수도 있다.
예를 들어 [collection.IndexedSeq\[T\]](http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html)는
[collection.immutable.IndexedSeq\[T\]](http://www.scala-lang.org/api/current/scala/collection/immutable/IndexedSeq.html)와
[collection.mutable.IndexedSeq\[T\]](http://www.scala-lang.org/api/current/scala/collection/mutable/IndexedSeq.html) 모두의 수퍼클래스이다. 일반적으로 `scala.collection` 패키지의 루트 컬렉션은
변경할 수 없는 컬렉션과 같은 인터페이스를 정의하고 `scala.collection.mutable`
패키지의 변경할 수 있는 컬렉션은 이 변경할 수 없는 인터페이스에 부가작용을 하는
수정작업이 추가되어 있다.

루트 컬렉션과 변경할 수 없는 컬렉션의 차이점은 루트 컬렉션은 루트 컬렉션의 클라이언트만
컬렉션을 변경할 수 없지만 변경할 수 없는 컬렉션은 아무도 컬렉션을 변경할 수 없다는 점이다.
루트 컬렉션의 정적 타임이 컬렉션을 변경하는 작업을 제공하지 않더라도 런타임에서의 타입은
다른 클라이언트가 변경할 수 있는 컬렉션일 수 있다.

스칼라는 기본적으로 항상 변경할 수 없는 컬렉션을 사용한다. 예를 들어 접두사를 사용하지 않고
다른 곳에서 `Set`을 임포트하지 않고 `Set`을 사용하면 변경할 수 없는 Set이 되고
`Iterable`를 작성하면 변경할 수 없는 iterable 컬렉션이 된다. 이는 `scala` 패키지로
임포트한 기본 바인딩때문이다. 변경가능한 컬렉션을 사용하려면 명시적으로
`collection.mutable.Set`나 `collection.mutable.Iterable`를 사용해야 한다.

변경할 수 있는 컬렉션과 변경할 수 없는 컬렉션을 둘 다 사용할 때 유용한 관례는
`collection.mutable` 패키지를 임포트하는 것이다.

    import scala.collection.mutable

이제 접두사 없이 사용한 `Set`이 여전히 불변 컬렉션을 참조하는 한편, `mutable.Set`은 가변
컬렉션을 참조한다.

컬렉션 계층에서 남은 패키지는 `collection.generic`이다. 이 패키지에는 컬렉션을 구현하는
코드가 있다. 일반적으로 컬렉션 클래스는 클래스의 일부 작업의 구현을 `generic`의 클래스로
위임한다. 그럼에도 컬렉션 프레임워크의 사용자들은 예외적인 상황에서만 `generic`의 클래스를
참조해야 한다.

편의성과 하위 호환성때문에 일부 중요한 타입은 `scala` 패키지에 별칭을 가지고 있으므로
임포트하지 않고도 간단한 이름으로 이러한 타입을 사용할 수 있다. 이 대표적인 예가 `List`
타입이고 다음과 같은 방법으로 접근할 수 있다.

    scala.collection.immutable.List   // List가 정의되거 있는 곳
    scala.List                        // via the alias in the  scala package
    scala.List                        // scala 패키지의 별칭으로 접근
    List                              // 항상 scala._가 자동 임포트되므로

별칭이 있는 다른 타입으로는
[Traversable](http://www.scala-lang.org/api/current/scala/collection/Traversable.html), [Iterable](http://www.scala-lang.org/api/current/scala/collection/Iterable.html), [Seq](http://www.scala-lang.org/api/current/scala/collection/Seq.html), [IndexedSeq](http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html), [Iterator](http://www.scala-lang.org/api/current/scala/collection/Iterator.html), [Stream](http://www.scala-lang.org/api/current/scala/collection/immutable/Stream.html), [Vector](http://www.scala-lang.org/api/current/scala/collection/immutable/Vector.html), [StringBuilder](http://www.scala-lang.org/api/current/scala/collection/mutable/StringBuilder.html), [Range](http://www.scala-lang.org/api/current/scala/collection/immutable/Range.html)가 있다.

다음 그림에 `scala.collection` 패키지의 모든 컬렉션이 나와 있다. 일반적으로 변경할 수 없는
구현체 뿐 아니라 변경할 수 있는 구현체도 가지는 고수준의 추상 클래스나 트레이트가 있다.

[<img src="{{ site.baseurl }}/resources/images/collections.png" width="550">]({{ site.baseurl }}/resources/images/collections.png)

다음 그림에는 `scala.collection.immutable` 패키지의 모든 컬렉션이 나와 있다.

[<img src="{{ site.baseurl }}/resources/images/collections.immutable.png" width="550">]({{ site.baseurl }}/resources/images/collections.immutable.png)

다음 그림에는 `scala.collection.mutable` 패키지의 모든 컬렉션이 나와 있다.

[<img src="{{ site.baseurl }}/resources/images/collections.mutable.png" width="550">]({{ site.baseurl }}/resources/images/collections.mutable.png)
(이 세 그림은 decodified.com의 Mathias가 만들었다.)

## 컬렉션 API 살펴보기 ##

위의 그림에 가장 중요한 컬렉션 클래스가 나와 있고 모든 클래스에 공통된 아주 약간의 특성이 있는데
예를 들면 모든 컬렉션을 컬렉션 클래스명뒤에 요소를 작성하는 형태의 일관된 문법으로 생성할 수 있다.

    Traversable(1, 2, 3)
    Iterable("x", "y", "z")
    Map("x" -> 24, "y" -> 25, "z" -> 26)
    Set(Color.red, Color.green, Color.blue)
    SortedSet("hello", "world")
    Buffer(x, y, z)
    IndexedSeq(1.0, 2.0)
    LinearSeq(a, b, c)

같은 원리가 특정 컬렉션 구현체에도 다음과 같이 적용된다.

    List(1, 2, 3)
    HashMap("x" -> 24, "y" -> 25, "z" -> 26)

위에 작성한 것과 같은 방법으로 이러한 모든 컬렉션을 `toString`으로 표현한다.

모든 컬렉션은 `Traversable`이 제공하는 API를 지원하지만 가능하다면 타입을 전문화한다. 예를 들어 `Traversable`클래스의 `map`메서드를 실행하면 새로운 `Traversable`을 반환하지만 이 결과 타입은 하위 클래스에서 오버라이드된다. 즉, `List`에서 `map`을 호출하면 다시 `List`를 반환하고 `Set`에서 `map`을 호출하면 다시 `Set`을 반환하는 등이다.

    scala> List(1, 2, 3) map (_ + 1)
    res0: List[Int] = List(2, 3, 4)
    scala> Set(1, 2, 3) map (_ * 2)
    res0: Set[Int] = Set(2, 4, 6)

컬렉션 라이브러리 어디서나 구현된 이 동작을 _통일된 반환 타입 원리(uniform return type principle)_라고 부른다.

컬렉션 계층의 클래스 대부분은 3가지 종류가 있는데 루트 컬렉션과 변경할 수 있는 컬렉션과 변경할 수 없는 컬렉션이다. Buffer 트레이트만이 변경할 수 있는 컬렉션만 있는 유일한 예외이다.

이어서 이러한 클래스를 하나씩 살펴 볼 것이다.
