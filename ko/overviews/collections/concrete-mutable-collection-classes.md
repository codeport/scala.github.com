---
layout: overview-large
title: 변경 가능한 컬렉션에 속하는 구체적인 클래스들

disqus: true

partof: collections
num: 9
language: ko
---

스칼라 표준 라이브러리가 제공하는 가장 일반적으로 사용되는 변경 불가능한 컬렉션 클래스에 대해 살펴보았다. 이제 변경가능한 컬렉션 클래스를 살펴보자.

## 배열 버퍼

[ArrayBuffer(배열버퍼)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/ArrayBuffer.html)는 배열과 크기를 저장한다. 배열 버퍼에 대한 대부분의 연산은 배열에 대한 연산과 같은 복잡도이다. 왜냐하면 이러한 연산들은 단지 하부의 배열을 억세스하거나 변경하기 때문이다. 추가로 배열 버퍼에서는 데이터를 효율적으로 맨 마지막에 추가할 수 있다. 배열 버퍼에 데이터를 추가하는 작업의 상환 시간 복잡도(amortized time complexity)도 상수 시간이다. 따라서 배열 버퍼는 새 데이터가 맨 마지막에만 추가되는 큰 컬렉션을 효율적으로 구축할 때 유용하다.

    scala> val buf = scala.collection.mutable.ArrayBuffer.empty[Int]
    buf: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer()
    scala> buf += 1
    res32: buf.type = ArrayBuffer(1)
    scala> buf += 10
    res33: buf.type = ArrayBuffer(1, 10)
    scala> buf.toArray
    res34: Array[Int] = Array(1, 10)

## 리스트 버퍼

[ListBuffer(리스트버퍼)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/ListBuffer.html)는 배열 대신 내부적으로 연결된 리스트를 사용한다는 점만 다르고 배열 버퍼와 비슷하다. 만약 컬렉션을 구축한 다음 리스트로 변환할 생각이라면, 배열 버퍼 대신 리스트 버퍼를 사용하도록 하라.

    scala> val buf = scala.collection.mutable.ListBuffer.empty[Int]
    buf: scala.collection.mutable.ListBuffer[Int] = ListBuffer()
    scala> buf += 1
    res35: buf.type = ListBuffer(1)
    scala> buf += 10
    res36: buf.type = ListBuffer(1, 10)
    scala> buf.toList
    res37: List[Int] = List(1, 10)

## 스트링빌더

배열 버퍼가 배열을 만들 때 유용하고, 리스트 버퍼는 리스트를 만들 때 유용한 것과 같이, [StringBuilder(스트링빌더)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/StringBuilder.html)는 스트링을 만들 때 유용하다. 스트링빌더는 자주 사용되기 때문에 기본 네임스페이스에 임포트되어 있다. 단지 아래와 같이 `new StringBuilder`라고 하면 만들 수 있다.

    scala> val buf = new StringBuilder
    buf: StringBuilder = 
    scala> buf += 'a'
    res38: buf.type = a
    scala> buf ++= "bcdef"
    res39: buf.type = abcdef
    scala> buf.toString
    res41: String = abcdef

## 연결 리스트

연결 리스트는 변경 가능한 열로 각 노드는 다음번 노드를 가리키는 포인터로 서로 연결되어 있다. 이들은 [LinkedList(연결리스트)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/LinkedList.html) 클래스로 지원된다. 대부분의 언어에서 `null`이 빈 연결 리스트로 사용된다. 스칼라 컬렉션에서는 그렇게 할 수 없는데 왜냐하면 빈 열도 열이 제공하는 모든 메소드를 지원해야 하기 때문이다. 특히 `LinkedList.empty.isEmpty`가 `true`를 반환해야지 `NullPointerException`을 발생시키면 안된다. 따라서 빈 연결 리스트는 특별한 방식으로 처리된다. 빈 연결리스트의 `next` 필드는 자기 자신을 가리킨다. 변경 불가능한 리스트와 마찬가지로 연결리스트도 순차적으로 순회할 때 가장 성능이 좋다. 추가로 연결리스트를 사용하면 원소나 연결 리스트를 다른 연결 리스트에 추가하는 것이 쉽다.

## 이중 연결 리스트

이중연결리스트는 (단일)연결리스트와 비슷하지만, `next` 필드 외에도 이전 노드를 가리키는 변경 기능한 필드 `prev`가 추가되었다는 점이 다르다. 이렇게 링크를 추가하면 원소 삭제가 상수시간으로 매우 빨라진다. 스칼라에서는 [DoubleLinkedList(이중연결리스트)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/DoubleLinkedList.html) 클래스로 이를 지원한다.

## 변경 가능한 리스트

[MutableList(변경 가능한 리스트)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/MutableList.html)는 리스트의 맨 마지막을 표시하는 노드(비어있는 노드임)를 가리키는 필드와 단일연결리스트로 구성된다. 따라서 리스트 뒤에 다른 리스트를 붙이는 작업이 상수 시간에 수행될 수 있다. 왜냐하면 리스트의 끝을 찾기 위해 전체 리스트를 순회할 필요가 없기 때문이다. [MutableList(변경가능리스트)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/MutableList.html)는 현재 스칼라에서 [mutable.LinearSeq(선형 열)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/LinearSeq.html)의 표준 구현이다. 

## 큐

스칼라는 변경 불가능한 큐와 더불어 변경 가능한 큐도 제공한다. `mQueue` 사용법은 변경 불가능한 큐와 비슷하다. 다만 `enqueue` 대신 `+=`나 `++=`를 원소/목록 추가를 위해 사용한다. 또한 변경 가능한 큐에서 `dequeue` 메소드는 단지 머리 원소를 큐에서 제거하고 그 원소를 반환한다. 다음은 실행 예이다.

    scala> val queue = new scala.collection.mutable.Queue[String]
    queue: scala.collection.mutable.Queue[String] = Queue()
    scala> queue += "a"
    res10: queue.type = Queue(a)
    scala> queue ++= List("b", "c")
    res11: queue.type = Queue(a, b, c)
    scala> queue
    res12: scala.collection.mutable.Queue[String] = Queue(a, b, c)
    scala> queue.dequeue
    res13: String = a
    scala> queue
    res14: scala.collection.mutable.Queue[String] = Queue(b, c)

## 배열 열

배열 열은 크기가 고정된 변경 가능한 열이다. 내부적으로 원소를 `Array[Object]`에 저장한다. 이는 스칼라에서 [ArraySeq(배열열)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/ArraySeq.html)클래스로 구현되어 있다.

`ArraySeq` 사용하는 전형적인 상황은 배열과 같은 성능 특성이 필요하지만 각 원소의 타입을 모르거나, 런타임에 제공할 `ClassManifest(클래스 메니페스트)`가 없어서 일반적인 열 인스턴스를 만들어야만 할 때를 들 수 있다.  이에 대해서는 [배열]({{ site.baseurl }}/overviews/collections/arrays.html)에 대해 설명할 때 더 자세히 다룰 것이다.

## 스택

변경 불가능한 스택에 대해 보았다. 마찬가지로 변경 가능한 버전도 있다. 지원하는 클래스는 [mutable.Stack](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/Stack.html)이다. 변경 가능 스택은 변경 불가능한 스택과 똑같이 동작하지만, 기존 스택의 내부 상태를 변경한다는 점에서만 차이가 있다.

    scala> val stack = new scala.collection.mutable.Stack[Int]           
    stack: scala.collection.mutable.Stack[Int] = Stack()
    scala> stack.push(1)
    res0: stack.type = Stack(1)
    scala> stack
    res1: scala.collection.mutable.Stack[Int] = Stack(1)
    scala> stack.push(2)
    res0: stack.type = Stack(1, 2)
    scala> stack
    res3: scala.collection.mutable.Stack[Int] = Stack(1, 2)
    scala> stack.top
    res8: Int = 2
    scala> stack
    res9: scala.collection.mutable.Stack[Int] = Stack(1, 2)
    scala> stack.pop    
    res10: Int = 2
    scala> stack    
    res11: scala.collection.mutable.Stack[Int] = Stack(1)

## 배열 스택

[ArrayStack(배열스택)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/ArrayStack.html)은 변경 가능한 스택의 또 다른 구현이다. 내부적으로 배열을 사용하고, 필요할 때마다 배열 크기를 재설정한다. 배열스택은 빠른 첨자 검색을 제공하며, 일반적인 변경 가능 스택에 비해 대부분의 연산에서 조금 더 효율적이다.

## 해시 테이블

해시테이블은 내부 배열에 원소를 저장한다. 각 원소의 배열상의 위치는 원소의 해시 코드에 의해 결정된다. 해시테이블에 원소를 추가하는 것은 이미 그 위치에 저장된 원소가 없다면(즉, 해시 코드가 같은 원소가 없다면) 상수 시간만 필요하다. 따라서 원소들의 해시 코드가 고르게 분포되어 있는 한 해시 테이블 연산은 매우 빠르다. 따라서 스칼라에서 변경 가능한 맵의 기본 구현은 해시 테이블을 기반으로 한다. 필요하면 직접 [mutable.HashSet(해시집합)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/HashSet.html)과 [mutable.HashMap(해시맵)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/HashMap.html)이라는 이름을 사용해 해시 테이블을 사용할 수 있다.

해시 집합과 맵은 다른 집합이나 맵과 동일하다. 다음은 몇가지 예이다.

    scala> val map = scala.collection.mutable.HashMap.empty[Int,String]
    map: scala.collection.mutable.HashMap[Int,String] = Map()
    scala> map += (1 -> "make a web site")
    res42: map.type = Map(1 -> make a web site)
    scala> map += (3 -> "profit!")
    res43: map.type = Map(1 -> make a web site, 3 -> profit!)
    scala> map(1)
    res44: String = make a web site
    scala> map contains 2
    res46: Boolean = false

해시 테이블에 대한 이터레이션은 항상 어떤 정해진 순서를 따라 실행된다는 보장이 없다. 이터레이션시 단지 내부 배열에 저장된 원소들을 저장된 차례대로 처리할 뿐이다. 정해진 이터레이션 순서가 필요하다면 일반적인 해시맵이나 집합 대신 _연결(linked)_ 해시맵이나 해시집합을 사용하라. 연결 해시맵이나 집합은 일반 해시맵이나 집합과 동일하지만 추가된 순서에 따른 원소들의 연결 리스트를 추가로 가지고 있다. 이런 컬렉션에 대해 이터레이션을 하면 항상 원소가 추가된 순서에 따라 동일한 순서로 원소를 처리할 수 있다. 

## 약한 해시 맵

약한 해시 맵은 특별한 해시 맵으로 이 맵에 저장된 키에 대한 참조는 쓰레기 수집기가 살아있는 객체를 체크할 때 검토되지 않는다. 따라서 이 맵에 저장된 키와 값에 대한 약한 해시맵 바깥의 참조가 모두 사라지게 되면 해당 키와 값은 사라지게 된다. 약한 해시 맵은 같은 값에 대해 재호출시 반복 계산을 하면 비용이 너무 비싸지는 함수값 등에 대한 캐시등을 할 때 유용하다. 키와 함수 결과값이 일반 맵에 저장되어 있다면 맵의 크기는 제약 없이 커지고, 키 객체들은 결코 재활용되지 못할 것이다. 약한 해시 맵을 사용하면 이런 문제를 피할 수 있다. 어떤 키 객체가 도착불가능하게 되면 그 엔트리는 약한 해시 맵에서 제외된다. 스칼라에서 약한 해시맵은 [WeakHashMap(약해시맵)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/WeakHashMap.html)으로 제공되며, 이 클래스는 자바 클래스인 `java.util.WeakHashMap`을 둘러싼 클래스이다.

## 병렬 맵

병렬맵은 여러 쓰레드가 동시에 억세스할 수 있다. 일반적인 [Map(맵)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/Map.html) 연산과 더불어 다음과 같은 원자적 연산을 제공한다.

### ConcurrentMap(병렬맵)의 연산들

| 사용법  	  	            | 하는일				     |
| ------       	       	        | ------					     |
|  `m putIfAbsent(k, v)`  	    |키 `k`가 `m`에 정의되어 있지 않은 경우 키/값 연관관계 `k -> m`를 추가한다. |
|  `m remove (k, v)`  	        |키 `k`가 값 `v`로 연관되어 있는 경우 이를 제거한다. |
|  `m replace (k, old, new)`  	|키 `k`에 연관되어 있는 값을 `new`로 변경한다. 다만 기존값이 `old`일 경우에만 변경을 수행한다. |
|  `m replace (k, v)`  	        |키 `k`에 연관되어 있는 값을 `new`로 변경한다. 다만 맵에 키가 이미 있는 경우에만 변경을 수행한다.|

`ConcurrentMap`은 스칼라의 컬렉션 라이브러리의 트레잇이다. 하지만 현재 이를 지원하는 유일한 구현은 자바의 `java.util.concurrent.ConcurrentMap`뿐이다. 이 경우 자바 객체에서 스칼라 객체로의 변환은 [표준  자바/스칼라 컬렉션 변환]({{ site.baseurl }}/overviews/collections/conversions-between-java-and-scala-collections.html)에 의해 자동으로 수행된다.

## 변경가능한 비트집합

타입 [mutable.BitSet(비트집합)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/mutable/BitSet.html)은 객체 상태 변경이 가능하다는 점을 제외하고는 변경 불가능한 비트집합과 거의 비슷하다. 변경 가능한 비트 집합을 변경하는것은 변경 불가능한 비트집합의 원소를 변경하는 것보다 약간 더 효율적이다. 이유는 바뀔 필요가 없는 `Long` 값들을 복사하지 않기 때문이다.

    scala> val bits = scala.collection.mutable.BitSet.empty
    bits: scala.collection.mutable.BitSet = BitSet()
    scala> bits += 1
    res49: bits.type = BitSet(1)
    scala> bits += 3
    res50: bits.type = BitSet(1, 3)
    scala> bits
    res51: scala.collection.mutable.BitSet = BitSet(1, 3)


















