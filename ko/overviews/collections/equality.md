---
layout: overview-large
title: 동일성

disqus: true

partof: collections
num: 13
language: ko
---

스칼라 컬렉션 라이브러리는 동일성 판단과 해싱에 있어 일괄적인 방법을 사용한다. 먼저, 컬렉션을 집합(set), 맵(map)과 열(sequence)로 나누어 종류가 다를 경우 언제나 다른 것으로 본다. 예를 들어 `Set(1, 2, 3)`과 `List(1, 2, 3)`은 같은 요소를 갖지만 서로 다르다. 종류가 같은 컬렉션끼리는 요소가 일치하는 경우에만 같은 것으로 본다. (열의 경우 순서 또한 동일해야 한다). 예를 들어 `List(1, 2, 3) == Vector(1, 2, 3)`과 `HashSet(1, 2) == TreeSet(2, 1)`은 참이다.

동일성 판단에 있어 해당 컬렉션의 변경 가능 여부는 중요하지 않다. 변경 가능한 컬렉션에서는 단순히 동일성 판단 시점의 요소만을 고려한다. 이는 한 컬렉션이 요소의 추가 또는 제거에 따라 서로 다른 시각에 각기 다른 컬렉션과 동일할 수도 있음을 의미한다. 변경 가능한 컬렉션을 해시맵의 키로 사용할 경우 다음과 같은 문제가 생길 수 있다:

    scala> import collection.mutable.{HashMap, ArrayBuffer}
    import collection.mutable.{HashMap, ArrayBuffer}
    scala> val buf = ArrayBuffer(1, 2, 3)
    buf: scala.collection.mutable.ArrayBuffer[Int] = 
    ArrayBuffer(1, 2, 3)
    scala> val map = HashMap(buf -> 3)
    map: scala.collection.mutable.HashMap[scala.collection.
    mutable.ArrayBuffer[Int],Int] = Map((ArrayBuffer(1, 2, 3),3))
    scala> map(buf)
    res13: Int = 3
    scala> buf(0) += 1
    scala> map(buf)
    java.util.NoSuchElementException: key not found: 
    ArrayBuffer(2, 2, 3) 

이 예제에서 마지막 줄의 선택은 이전 줄에서 배열 `xs`의 해시 코드가 달라짐에 따라 십중팔구 실패하게 된다. 즉, 해시 코드 기반 탐색시 `xs`가 저장된 시점과는 다른 위치를 참조하게 되는 것이다.
