---
layout: overview-large
title: 반복가능(Iterable) 트레잇

disqus: true

partof: collections
num: 4
language: ko
---

컬렉션 계층 구조의 맨 위에서 두번째에 있는 것이 `Iterable`(반복가능)이다. 이 트레잇에 있는 모든 메소드는 추상 메소드 `iterator`를 기반으로 정의되어 있다. 이 추상 메소드는 컬렉션의 원소를 하나씩 내어 놓는다. 트레잇 `Traversable`의 `foreach`는 `Iterable`에서 `iterator`를 기반으로 정의되어 있다. 다음은 실제 구현이다.

    def foreach[U](f: Elem => U): Unit = {
      val it = iterator
      while (it.hasNext) f(it.next())
    } 

`Iterable`의 하위 클래스 중 상당수는 이 `foreach` 표준 구현을 재정의(override)하고 있다. 왜냐하면 더 효율적인 구현이 가능하기 때문이다. `foreach`가 `Traversable`의 모든 메소드 구현에 사용됨을 기억하라. 따라서, 성능을 진지하게 고려해야 한다.

`Iterable`에는 반복자를 반환하는 메소드가 두 개 더 있다. 이들은 각각 `grouped`와 `sliding`이다. 그러나 이 반복자들은 원래의 컬렉션의 한 원소만을 반환하는 것이 아니고, 전체 원소의 부분 열을 반환한다. 각 메소드는 이런 부분 열의 최대 크기를 인자로 받는다. `grouped` 메소드는 원소를 일정한 "덩어리(chunked)" 단위로 반환하는 반면, `sliding`은 원소에 대한 "미닫이 창(sliding window)"을 반환한다. 아래 REPL 수행 예를 보면 이 둘 사이의 차이를 명확히 알 수 있을 것이다.

    scala> val xs = List(1, 2, 3, 4, 5)
    xs: List[Int] = List(1, 2, 3, 4, 5)
    scala> val git = xs grouped 3
    git: Iterator[List[Int]] = non-empty iterator
    scala> git.next()
    res3: List[Int] = List(1, 2, 3)
    scala> git.next()
    res4: List[Int] = List(4, 5)
    scala> val sit = xs sliding 3
    sit: Iterator[List[Int]] = non-empty iterator
    scala> sit.next()
    res5: List[Int] = List(1, 2, 3)
    scala> sit.next()
    res6: List[Int] = List(2, 3, 4)
    scala> sit.next()
    res7: List[Int] = List(3, 4, 5)

`Traversable` 트레잇의 메소드 중 이터레이터가 있는 경우에만 효율적으로 구현할 수 있는 메소드 몇 가지를 `Iterable` 트레잇에서 재정의하고 있다. 다음 표에서 이를 요약하였다.

### Iterable 트레잇의 연산 ###

| 사용법  	  	    | 하는일				     |
| ------       	       	    | ------					     |
|  **추상 메소드:**     |						     |
|  `xs.iterator`	    |`iterator`는 `xs`의 모든 원소를 `foreach`가 순회하는 순서대로 하나씩 제공하는 반복자이다.|
|  **다른 반복자:**     |						     |
|  `xs grouped size`   	    |고정된 크기의 "덩어리"를 컬렉션에서 내어놓는 반복자이다.|
|  `xs sliding size`   	    |고정된 크기의 미닫이 창을 내어놓는 반복자이다.|
|  **부분 컬렉션:** 	    |						     |
|  `xs takeRight n`	    |`xs`의 마지막 `n`개의 원소로 이루어진 컬렉션을 반환한다(순서가 없는 컬렉션이라면 임의의 `n`개를 반환한다).|
|  `xs dropRight n`	    |`xs takeRight n`의 결과를 제외한 나머지 컬렉션을 반환한다.|
|  **묶기(zip):** 	    |						     |
|  `xs zip ys`	    	    |`xs`와 `ys`에서 같은 위치에 있는 원소를 각각 가져와 만든 튜플로 이루어진 컬렉션을 반환한다. 길이가 다르다면 짧은 컬렉션의 원소 갯수 만큼만 반환한다. |
|  `xs zipAll (ys, x, y)`   |`zip`과 같지만, 길이가 다른 경우 `x`나 `y`를 더 짧은쪽 리스트의 원소가 모자란 경우 대신 사용한다.|
|  `xs.zipWithIndex`	    |`xs`의 원소와 그 위치를 나타내는 첨자를 튜플로 만든 컬렉션을 반환한다.|
|  **비교:** 	    |						     |
|  `xs sameElements ys`	    |`xs`와 `ys`가 같은 순서로 같은 원소를 포함하고 있는지 비교한다.|

상속 계층에서 Iterable의 하위에는 다음 세 트레잇이 있다. [열(Seq)](http://www.scala-lang.org/docu/files/collections-api/collections_5.html), [집합(Set)](http://www.scala-lang.org/docu/files/collections-api/collections_7.html), [맵(Map)](http://www.scala-lang.org/docu/files/collections-api/collections_10.html)이 그것이다. 이 세 트레잇의 공통점은 모두 다 [부분함수(PartialFunction)](http://www.scala-lang.org/api/current/scala/PartialFunction.html) 트레잇의 `apply`와 `isDefinedAt` 메소드를 정의하고 있다는 것이다. 하지만, 각 트레잇이 [부분함수(PartialFunction)](http://www.scala-lang.org/api/current/scala/PartialFunction.html)를 구현한 방법은 각각 다르다.

열에서 `apply`는 위치에 따라 첨자를 부여한다. 첨자는 첫번째 원소 `0` 부터 시작한다. 따라서 `Seq(1, 2, 3)(1)`은 `2`를 반환한다. 집합의 경우 `apply`는 포함관계 검사이다. 예를 들어 `Set('a', 'b', 'c')('b')`은 `true`를, `Set()('a')`는 `false`를 반환한다. 마지막으로 맵의 경우 `apply`는 선택(검색)이다. 예를 들어 `Map('a' -> 1, 'b' -> 10, 'c' -> 100)('b')`의 결과는 `10`이다.

다음 글에서는 위 세 컬렉션을 더 자세히 살펴볼 것이다.

번역: 오현석(enshahar@gmail.com)
