---
layout: overview-large
title: 자바와 스칼라 컬렉션간 변환

disqus: true

partof: collections
num: 17
language: ko
---

스칼라와 마찬가지로 자바 또한 풍부한 컬렉션 라이브러리를 제공한다. 둘 사이에는 반복자(iterator), 반복가능(iterable), 집합(set), 맵(map)과 열(sequence) 등 유사한 점이 많지만 중요한 차이점 또한 존재하는데, 특히 스칼라 라이브러리의 경우 변경 불가능한 컬렉션을 훨씬 더 강조하거나 더욱 많은 컬렉션간 변환 연산을 지원하는 점이 그러하다.

때때로 한 컬렉션 프레임워크를 다른 것으로 대체해야 할 경우가 생긴다. 예를 들어, 이미 존재하는 자바 컬렉션을 마치 스칼라 컬렉션처럼 접근하거나, 자바 컬렉션을 인자로 받는 자바 메소드에 스칼라 컬렉션을 넘기고 싶을 수 있다. 이러한 작업은 모든 주요 컬렉션간의 암시적 변환을 제공하는 [JavaConversions](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/JavaConversions$.html) 객체를 통해 손쉽게 가능하다. 구체적으로 다음과 같은 양방향 변환등을 찾을 수 있다.


    Iterator               <=>     java.util.Iterator
    Iterator               <=>     java.util.Enumeration
    Iterable               <=>     java.lang.Iterable
    Iterable               <=>     java.util.Collection
    mutable.Buffer         <=>     java.util.List
    mutable.Set            <=>     java.util.Set
    mutable.Map            <=>     java.util.Map
    mutable.ConcurrentMap  <=>     java.util.concurrent.ConcurrentMap

이러한 변환을 활성화하려면 단순히 [JavaConversions](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/JavaConversions$.html) 객체에서 임포트하면 된다:

    scala> import collection.JavaConversions._
    import collection.JavaConversions._

이제 자동적으로 스칼라 컬렉션과 이에 대응하는 자바 컬렉션간의 변환이 가능하다.

    scala> import collection.mutable._
    import collection.mutable._
    scala> val jul: java.util.List[Int] = ArrayBuffer(1, 2, 3)
    jul: java.util.List[Int] = [1, 2, 3]
    scala> val buf: Seq[Int] = jul
    buf: scala.collection.mutable.Seq[Int] = ArrayBuffer(1, 2, 3)
    scala> val m: java.util.Map[String, Int] = HashMap("abc" -> 1, "hello" -> 2)
    m: java.util.Map[String,Int] = {hello=2, abc=1}

내부적으로, 이러한 변환은 원본 컬렉션 객체로 모든 연산을 전달하는 "래퍼(wrapper)" 객체를 통해 이루어진다. 따라서 자바와 스칼라간 변환에 있어 컬렉션은 절대 복사되지 않는다. 흥미로운 것은 왕복 변환이 일어날 때, 예를 들어 자바에서 스칼라로, 다시 자바로 변환되는 경우, 처음과 정확히 동일한 컬렉션 객체가 된다는 점이다.

스칼라에서 자바로 변환은 가능하지만 반대로는 불가능한 공통 스칼라 컬렉션도 있다:

    Seq           =>    java.util.List 
    mutable.Seq   =>    java.utl.List
    Set           =>    java.util.Set 
    Map           =>    java.util.Map 

자바는 변경 가능한 컬렉션과 변경 불가능한 컬렉션을 자료형으로 구분하지 않기 때문에, `scala.immutable.List`는 `java.util.List`로 변환되지만 모든 변경 연산은 "UnsupportedOperationException"을 발생시킨다. 예제를 보자:

    scala> jul = List(1, 2, 3)
    jul: java.util.List[Int] = [1, 2, 3]
    scala> jul.add(7)
    java.lang.UnsupportedOperationException
            at java.util.AbstractList.add(AbstractList.java:131)

