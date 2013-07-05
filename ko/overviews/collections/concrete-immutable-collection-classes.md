---
layout: overview-large
title: 변경 불가능한 컬렉션에 속하는 구체적인 클래스들

disqus: true

partof: collections
num: 8
language: ko
---

스칼라에는 필요에 따라 선택 가능한 구체적인 선택 불가능 컬렉션 클래스가 많이 있다. 각각의 차이는 어떤 트레잇을 구현하고 있는가(맵, 집합, 열), 무한한가 유한한가, 각각의 연산의 수행 시간(복잡도) 등에 있다. 여기서는 스칼라에서 가장 많이 사용되는 변경 불가능한 컬렉션 타입들을 설명할 것이다.

## 리스트(List)

[List(리스트)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/List.html)는 유한한 변경 불가능한 열이다. 리스트의 첫 원소를 가져오거나, 첫 원소를 제외한 나머지를 가져오는 연산에는 상수 시간이 걸린다. 또한 새 원소를 리스트 맨 앞에 붙이는(보통 콘스cons라 부름) 연산도 상수시간이 소요된다. 다른 연산들은 보통 선형 시간(즉, 리스트의 길이 N에 비례)이 걸린다.

리스트는 계속해서 스칼라 프로그래밍시 사용하는 주요 데이터 구조였다. 따라서 여기서 다시 자세히 설명할 필요는 없을 것이다. 2.8에서 가장 큰 변화는 `List` 클래스, 그리고 그 하위 클래스인 `::`와 하위 객체 `Nil`이 이제는 논리적인 위치에 맞게 `scala.collection.immutable` 패키지에 들어가 있다는 점이다. 여전히 `scala` 패키지에도 타입 별칭 `List`, `Nil`, `::`가 존재한다. 따라서 사용자 관점에서 볼때는 예전에 사용하던 방식대로 계속 사용 가능하다.

또 다른 변화는 이제 리스트도 컬렉션 프레임워크에 더 밀접하게 통합되어 있따는 점이다. 따라서 이전보다 덜 특별하게 취급된다. 예를 들어 원래는 `List` 짝 객체에 존재하던 여러 연산이 이제는 사용하지 않도록 표시되었다. 이러한 연산들은 모든 컬렉션이 상속하는 [일정한 생성 메소드들]({{ site.baseurl }}/overviews/collections/creating-collections-from-scratch.html)로 대치되었다.

## 스트림

[Stream(스트림)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Stream.html)은 리스트와 같지만 각 원소가 지연계산된다는 점에서 차이가 있다. 따라서 무한한 스트림도 가능하다. 요청을 받을 때만 원소를 계산한다. 그 점을 제외하면 나머지 동작 특성은 리스트와 같다.
리스트가 `::` 연산으로 구성되는 반면, 스트림은 비슷한 `#::`로 구성된다. 다음은 정수 1, 2, 3을 포함하는 스트림을 만드는 과정을 보여준다. 

    scala> val str = 1 #:: 2 #:: 3 #:: Stream.empty
    str: scala.collection.immutable.Stream[Int] = Stream(1, ?)

이 스트림의 머리는 1이고, 꼬리는 2와 3이다. 꼬리는 출력이 되지 않는다. 왜냐하면 아직 그 값을 계산하지 않았기 때문이다! 스트림은 지연계산을 위한 것이기 때문에, 스트림의 `toString` 메소드는 추가 계산을 수행하지 않게 조심하도록 구현되어 있다.

다음은 좀 더 복잡한 예제이다. 이 예에서는 주어진 두 정수로 시작하는 피보나치 수열의 스트림을 계산한다. 피보나치 수열은 각 원소가 바로 이전 두 원소의 합인 수열이다. 

    scala> def fibFrom(a: Int, b: Int): Stream[Int] = a #:: fibFrom(b, a + b)
    fibFrom: (a: Int,b: Int)Stream[Int]

이 함수는 믿을 수 없이 간단하다. 열의 첫번째 원소는 분명 `a`이고, 나머지 열은 `b` 다음에 `a + b`가 오는 피보나치 수열이다. 교묘한 부분은 이 열을 계산할 때 무한한 재귀호출을 하지 않는다는 점이다. `::`를 `#::` 대신 사용했다면 함수 호출이 자기 자신을 호출하게 되어서 무한 재귀 호출이 발생했을 것이다. 하지만 `#::`를 사용하기 때문에 우항은 요청되기 전까지 계산되지 않는다.

다음은 1을 두개 사용해 생성되는 피보나치 수열의 앞 부분 항을 몇 개 보여준다.

    scala> val fibs = fibFrom(1, 1).take(7)
    fibs: scala.collection.immutable.Stream[Int] = Stream(1, ?)
    scala> fibs.toList
    res9: List[Int] = List(1, 1, 2, 3, 5, 8, 13)

## 벡터

리스트는 그것을 다루는 알고리즘이 목록의 첫 원소만을 처리하도록 주의깊게 고안되어 있는 경우에는 아주 효율적이다. 리스트의 첫 원소를 억세스하고, 추가하고, 삭제하는 작업은 상수 시간만 걸리지만, 그외 다른 곳에 위치한 원소를 억세스하거나 변경하는 작업은 리스트의 길이에 정비례하는 시간이 걸린다.

[Vector(벡터)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Vector.html)는 임의 접근시 성능이 떨어지는 리스트의 문제점을 해결한 컬렉션 유형이다(스칼라 2.8에 도입됨). 벡터에 속한 어떤 원소이건 "실질적으로" 상수 시간에 억세스할 수 있다. 상수 자체는 리스트의 첫 원소를 가져오는 것이나 배열의 원소를 읽는 시간보다는 더 큰 편이다. 하지만, 여전히 상수시간은 상수시간이다. 따라서 벡터를 사용하는 알고리즘에서는 열의 첫 원소만을 처리하도록 조심할 필요가 없다. 원한다면 어떤 위치에 있는 원소이던지 억세스하거나 변경할 수 있다. 따라서 훨씬 사용하기에 편리하다. 

벡터도 다른 열 객체들과 마찬가지로 생성하고 변경할 수 있다.

    scala> val vec = scala.collection.immutable.Vector.empty
    vec: scala.collection.immutable.Vector[Nothing] = Vector()
    scala> val vec2 = vec :+ 1 :+ 2
    vec2: scala.collection.immutable.Vector[Int] = Vector(1, 2)
    scala> val vec3 = 100 +: vec2
    vec3: scala.collection.immutable.Vector[Int] = Vector(100, 1, 2)
    scala> vec3(0)
    res1: Int = 100

벡터는 분기계수(branching factor)가 큰 트리로 표현된다(트리나 그래프에서 분기계수란 각 노드가 가질 수 있는 자식의 갯수를 의미한다). 모든 트리 노드는 최대 32개의 벡터 원소를 가지거나, 32개의 다른 트리 노드를 포함할 수 있다. 32개 이하의 원소를 가지는 벡터는 노드 하나로 표현할 수 있다. `32 * 32 = 1024`개 까지의 원소는 단 한번만 간접참조를 거치면 된다. 트리 루트에서 두 단계를 거치면 최대 2<sup>15</sup> 원소를 지원할 수 있고, 3 단계는 2<sup>20</sup>, 네 단계는 2<sup>25</sup>개의 원소, 다섯 단계를 거치는 경우에는 2<sup>30</sup>개 까지 가능하다. 따라서 적당한 범위내의 크기를 가지는 벡터라면 최대 5번의 원시 배열 선택을 거치면 모두 처리할 수 있다. 앞에서 "실질적"이라는 말로 원소 억세스에 대해 설명했던 이유가 바로 이것이다.

벡터는 변경 불가능하다. 따라서 벡터의 원소를 변경할 수 없다. 하지만 `updated` 메소드를 사용하면 기존 벡터와 요소 하나만 차이가 나는 새 벡터를 만들 수 있다.

    scala> val vec = Vector(1, 2, 3)
    vec: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3)
    scala> vec updated (2, 4)
    res0: scala.collection.immutable.Vector[Int] = Vector(1, 2, 4)
    scala> vec
    res1: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3)

마지막 줄에서 보듯, `updated`를 호출해도 원래의 벡터 `vec`은 변화가 없다. 원소 억세스시와 마찬가지로 함수형 벡터 변경도 또한 "실질적으로는 상수 시간"에 수행된다. 벡터의 중간에 있는 원소를 변경 하려면 그 원소를 포함하고 있는 노드와 루트에서 해당 노드에 도달하는 경로에 있는 모든 선조 노드들도 복사하는 방식으로 이루어진다. 따라서 함수형으로 변경하는 경우에 최대 5개의 노드를 복사/생성해야 함을 의미한다. 이때 각 노드는 32개의 원소를 포함하거나, 최대 32개까지 다른 하위 트리에 대한 참조를 저장하게 된다. 물론 변경 가능한 배열에서 직접 변경하는 것보다는 확실히 더 비용이 비싸다. 하지만 전체 벡터를 복사하는 것과 비교하면 훨씬 적은 비용이 든다.

벡터는 빠른 임의 선택과 빠른 임의의 함수적 변경 사이의 균형을 잘 잡고 있다. 따라서 첨자가 있는 열(immutable.IndexedSeq) 트레잇의 기본 구현은 벡터로 되어 있다.

    scala> collection.immutable.IndexedSeq(1, 2, 3)
    res2: scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 3)

## 변경 불가능한 스택

후입선출(LIFO, 나중에 넣은게 처음 나오는) 열이 필요하다면 [Stack(스택)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Stack.html)이 있다. 스택 위에 원소를 넣을 때는 `push`를 쓰고, 스택의 맨 위 원소를 꺼내올 때는 `pop`을 쓴다. 또한 스택의 맨 위에 있는 원소를 꺼내지 않고 확인만 하고 싶다면 `top`을 사용하면 된다. 이 세 연산은 모두 상수 시간이 소요된다.

다음 예는 스택에 대해 간단한 연산을 수행하는 것을 보여준다.


    scala> val stack = scala.collection.immutable.Stack.empty
    stack: scala.collection.immutable.Stack[Nothing] = Stack()
    scala> val hasOne = stack.push(1)
    hasOne: scala.collection.immutable.Stack[Int] = Stack(1)
    scala> stack
    stack: scala.collection.immutable.Stack[Nothing] = Stack()
    scala> hasOne.top
    res20: Int = 1
    scala> hasOne.pop
    res19: scala.collection.immutable.Stack[Int] = Stack()

변경 불가능한 스택은 스칼라에서 거의 사용되지 않는다. 왜냐하면 리스트가 스택의 모든 역할을 다 할 수 있기 때문이다. 변경 불가능한 스택에 `push`하는 것은 리스트에 `::` 하는 것과 같고, `pop`은 리스트에 `tail`을 하는 것과 같다.

## 변경 불가능한 큐

[Queue(큐)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Queue.html)는 스택과 비슷하지만 후입선출이 아니고 선입선출(FIFO,처음 넣은 것이 처음 나오는)이라는 점이 다르다.

다음은 변경 불가능한 빈 큐를 생성하는 것을 보여준다.

    scala> val empty = scala.collection.immutable.Queue[Int]()
    empty: scala.collection.immutable.Queue[Int] = Queue()

`enqueue`를 사용해 변경 불가능한 큐에 원소를 추가할 수 있다.

    scala> val has1 = empty.enqueue(1)
    has1: scala.collection.immutable.Queue[Int] = Queue(1)

여러 원소를 큐에 추가하려면 `enqueue` 호출시 컬렉션을 인자로 넘기면 된다.

    scala> val has123 = has1.enqueue(List(2, 3))
    has123: scala.collection.immutable.Queue[Int]
      = Queue(1, 2, 3)

큐의 맨 앞에서 원소를 제거하기 위해서는 `dequeue`를 사용하라.

    scala> val (element, has23) = has123.dequeue
    element: Int = 1
    has23: scala.collection.immutable.Queue[Int] = Queue(2, 3)

`dequeue`가 반환하는 것이 제거된 맨 앞의 원소와 그 원소를 제외한 나머지 큐의 튜플이라는 사실에 주의하라.

## 범위(Range)

[Range(범위)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Range.html)는 동일한 간격으로 떨어진 정수들의 순서가 있는 열이다. 예를 들어 "1, 2, 3"이나 "5, 8, 11, 14" 등이 범위이다. 스칼라에서 범위를 만들 때에는 미리 정의된 `to`와 `by` 메소드를 사용한다.

    scala> 1 to 3
    res2: scala.collection.immutable.Range.Inclusive
      with scala.collection.immutable.Range.ByOne = Range(1, 2, 3)
    scala> 5 to 14 by 3
    res3: scala.collection.immutable.Range = Range(5, 8, 11, 14)

상한값을 제외한 나머지만 포함한 범위를 만들고 싶다면 `to`대신에 `until` 메소드를 사용하면 된다.

    scala> 1 until 3
    res2: scala.collection.immutable.Range.Inclusive
      with scala.collection.immutable.Range.ByOne = Range(1, 2)

범위는 하한값, 상한값, 그리고 증가값의 세 값으로 표현 가능하기 때문에 상수 공간 복잡도를 가진다. 이런 표현방식을 사용하기 때문에 범위에 대한 연산은 매우 빠르다.

## 해시 트라이(hash trie)

해시 트라이는 변경 불가능한 집합과 맵을 효율적으로 구현하기 위해 쓰이는 표준적인 방법이다. 이들은 [immutable.HashMap](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/HashMap.html) 클래스가 지원한다. 해시 트라이의 구조는 벡터와 비슷하다. 해시 트라이는 각 노드가 32개의 원소나 하위 트리를 가지는 트리로 표현된다. 하지만 키의 선택이 해시 값에 기반해 이루어진다는 점이 다르다. 예를 들어 맵에서 어떤 키를 찾기 위해서는 해당 키의 해시값을 먼저 계산해야 한다. 그리고 나서 해시코드의 최하위 5비트를 사용해 첫번째 하위 트리를 찾고, 나머지 5비트를 사용해 그 다음 하위 트리를 찾는 식으로 탐색을 수행한다. 탐색 과정은 노드에 저장된 모든 해시 코드가 서로 다른 시점에 종료된다. 
해시 트라이는 충분히 빠른 검색과 충분히 효율적인 함수적 추가(`+`) 및 삭제(`-`) 사이에 잘 균형잡힌 구현을 제공한다. 그런 이유로 스칼라에서는 변경 불가능한 맵과 집합에 해시 트라이를 기본 구현으로 사용한다. 실제로는 원소가 5개 미만인 변경 불가능한 맵과 집합에 대해서는 조금 더 최적화를 한다. 원소가 1개에서 4개 사이인 집합과 맵은 해당 원소들(맵의 경우 키/값 쌍들)만을 필드로 포함하는 단일 객체로 표현된다. 빈 변경 불가능한 집합이나 맵은 각각 유일한 객체로 표현된다. 변경 불가능한 빈 집합이나 맵은 항상 비어있는 채이기 때문에 객체를 중복해 만들 필요가 없다.


## 적-흑 트리(Red-Black Tree)

적흑트리는 균형트리의 일종으로, 일부 노드는 "빨간색", 나머지는 "검은색"으로 지정될 수 있다. 다른 모든 균형 트리와 마찬가지로 트리에 대한 연산은 트리 크기의 로그(log, 대수)에 비례한 시간 복잡도를 가진다.
스칼라는 내부적으로 적흑트리를 사용하는 변경 불가능한 집합과 맵을 제공한다. [TreeSet](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/TreeSet.html)과 [TreeMap](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/TreeMap.html)이라는 이름으로 이를 사용할 수 있다.

    scala> scala.collection.immutable.TreeSet.empty[Int]
    res11: scala.collection.immutable.TreeSet[Int] = TreeSet()
    scala> res11 + 1 + 3 + 3
    res12: scala.collection.immutable.TreeSet[Int] = TreeSet(1, 3)

적흑트리는 스칼라의 `SortedSet` 집합의 기본 구현이다. 이유는 적흑트리를 사용하면 모든 원소를 정렬된 순서로 반환하는 반복자를 효율적으로 제공할 수 있기 때문이다.

## 변경 불가능한 비트 집합

[BitSet(비트집합)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/BitSet.html)은 작은 정수의 컬렉션을 큰 정수의 비트들로 표현한 것이다. 예를 들어 3, 2, 0을 포함하는 비트집합은 2진수로 1101, 즉 10진수 13으로 표현할 수 있다.
내부적으로 비트집합은 64비트 `Long` 배열을 사용한다. 배열의 최초의 `Long`은 0부터 63, 두번째는 64부터 127, 이런 식으로 사용한다. 따라서 집합에서 가장 큰 수가 몇백 정도인 경우라면 비트 집합은 메모리를 아주 작게 차지한다. 
비트 집합에 대한 연산은 아주 빠르다. 원소가 속해있는지 판단하는 것은 상수시간이 걸린다. 집합에 원소를 추가하는 것은 내부에 있는 배열의 `Long` 갯수에 비례하지만, 보통 그 갯수가 그리 크지는 않다. 다음은 비트 집합을 사용하는 예를 간단히 보여준다.

    scala> val bits = scala.collection.immutable.BitSet.empty
    bits: scala.collection.immutable.BitSet = BitSet()
    scala> val moreBits = bits + 3 + 4 + 4
    moreBits: scala.collection.immutable.BitSet = BitSet(3, 4)
    scala> moreBits(3)
    res26: Boolean = true
    scala> moreBits(0)
    res27: Boolean = false

## 리스트 맵

[ListMap(리스트맵)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/ListMap.html)은 맵을 키-값 쌍의 연결된 리스트로 표현한다. 일반적으로 리스트맵에 대한 연산을 하려면 전체 리스트를 순회해야 한다. 따라서 리스트 맵에 대한 연산은 맵 크기에 대해 선형 시간이 필요하다. 스칼라에서 리스트맵을 사용하는 경우는 흔치 않다. 왜냐하면 표준적인 변경 불가능한 맵이 거의 대부분 더 빠르기 때문이다. 성능상 리스트맵이 더 유리할 수 있는 경우가 있다면, 어떤 이유로든 맵의 첫 원소가 다른 원소들보다 훨씬 더 자주 선택되어야 하는 경우일 것이다.

    scala> val map = scala.collection.immutable.ListMap(1->"one", 2->"two")
    map: scala.collection.immutable.ListMap[Int,java.lang.String] = 
       Map(1 -> one, 2 -> two)
    scala> map(2)
    res30: String = "two"






















