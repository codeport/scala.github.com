---
layout: overview-large
title: 열 트레잇 Seq, IndexedSeq, LinearSeq

disqus: true

partof: collections
num: 5
language: ko
---

[Seq(열)](http://www.scala-lang.org/api/current/scala/collection/Seq.html) 트레잇은 순서가 있는 열을 표현한다. 
열이란 `length`가 있고 원소들이 고정된 첨자로 표현되는 위치를 가지고 있는 반복가능한 객체를 말한다. 첨자는 `0`부터 시작한다.

표에 열이 제공하는 연산이 정리되어 있다. 각 연산은 다음과 같이 분류할 수 있다.

* **참조와 길이** 연산으로 `apply`, `isDefinedAt`, `length`, `indices`, `lengthCompare`가 있다. `Seq`에 있어, `apply` 연산은 첨자에 의해 원소를 참조하는 것이다. 따라서 `Seq[T]` 타입의 열은 `Int` 인자(첨자)를 받아서 열의 원소인 타입 `T` 객체를 반환하는 부분함수로 볼 수 있다. 즉, `Seq[T]`는 `PartialFunction[Int, T]`를 구현한다. 열의 원소에는 0부터 열의 `length`-1 까지 첨자가 부여된다. `length` 메소드는 일반적인 컬렉션의 `size`에 대한 별명이다. `lengthCompare` 메소드는 두 열(무한열인 경우도 포함됨)의 길이를 비교해준다. 
* **첨자 검색 연산** `indexOf`, `lastIndexOf`, `indexofSlice`, `lastIndexOfSlice`, `indexWhere`, `lastIndexWhere`, `segmentLength`, `prefixLength`는 주어진 값과 같거나 주어진 술어와 일치하는 원소의 첨자를 반환한다.
* **덧붙임** 연산 `+:`, `:+`, `padTo`는 열의 맨 앞이나 뒤에 원소를 추가해 만들어지는 새 열을 반환한다.
* **갱신** 연산 `updated`, `patch`는 원래의 열의 일부 원소를 다른 값으로 대치한 새 열을 반환한다.
* **정렬** 연산 `sorted`, `sortWith`, `sortBy`는 여러 기준에 의해 열을 정렬한다.
* **반전** 연산 `reverse`, `reverseIterator`, `reverseMap`은 열의 원소를 역순으로 처리하거나 내어 놓는다.
* **비교** `startsWith`, `endsWith`, `contains`, `containsSlice`, `corresponds`는 두 열을 연관짓거나 열 안에서 원소를 찾는다.
* **중복집합(Multiset)** 연산인 `intersect`, `diff`, `union`, `distinct`는 두 열의 원소에 대해 집합과 비슷한 연산을 수행하거나, 중복을 제거한다.

열이 변경 가능하다면 추가로 부작용을 사용하는 `update` 메소드를 제공한다. 이를 사용해 열의 원소를 변경할 수 있다. 스칼라에서 항상 그렇듯이 `seq(idx) = elem`는 단지 `seq.update(idx, elem)`를 짧게 쓴 것 뿐이다. 따라서 `update`를 정의하면 공짜로 대입 문법을 사용할 수 있게 된다. `update`와 `updated`가 다름에 유의하라. `update`는 어떤 열의 원소를 그 자리(새 열을 만들지 않고 열 자체를 갱신)에서 변경하며 변경 가능한 열에서만 사용할 수 있다. `updated`는 모든 열에서 사용 가능하며 항상 원래의 열은 그대로 두고 새로운 열을 반환한다.

### Seq 클래스의 연산 ###

| 사용법  	  	    | 하는일				     |
| ------       	       	    | ------					     |
|  **참조와 길이:** |						     |
|  `xs(i)`    	  	    |(명시적으로 `xs apply i`라고 쓸수도 있음)`xs`에서 첨자 `i` 번째에 있는 원소를 반환한다.|
|  `xs isDefinedAt i`	    |`i`가 `xs.indices` 안에 있는지 여부를 반환한다.|
|  `xs.length`	    	    |열의 길이를 반환한다(`size`와 동일).|
|  `xs.lengthCompare ys`    |`xs`가 `ys`보다 짧으면 `-1`, 길면 `+1`, 같은 길이이면 `0`을 반환한다. 길이가 무한한 경우라도 동작한다(현재는 인자로 정수를 받아 열의 길이와 비교하는 것으로 바뀜)|
|  `xs.indices`	     	    |`xs`의 첨자 범위를 반환한다. `0`부터 `xs.length - 1`까지 범위를 반환한다.|
|  **첨자 검색:**        |						     |
|  `xs indexOf x`   	    |`xs`에서 `x`와 같은 첫번째 원소의 첨자를 반환한다(몇 가지 변형이 존재한다).|
|  `xs lastIndexOf x`       |`xs`에서 `x`와 같은 마지막 원소의 첨자를 반환한다(몇 가지 변형이 존재한다).|
|  `xs indexOfSlice ys`     |`xs`에서 해당 위치에서부터 시작하는 슬라이스(slice,부분열)이 `ys`와 같은 첫번째 위치를 반환한다.|
|  `xs lastIndexOfSlice ys` |`xs`에서 해당 위치에서부터 시작하는 슬라이스(slice,부분열)이 `ys`와 같은 마지막 위치를 반환한다.|
|  `xs indexWhere p`   	    |`xs`에서 `p`를 만족하는 첫번째 원소의 첨자를 반환한다(몇 가지 변형이 존재한다).|
|  `xs segmentLength (p, i)`|`xs(i)`부터 시작해서 연속적으로 `p`를 만족하는 가장 긴 열의 길이를 구한다.|
|  `xs prefixLength p` 	    |`xs`의 맨 앞에서부터 시작해서 연속적으로 `p`를 만족하는 가장 긴 열의 길이를 구한다.|
|  **덧붙임:** 	    |						     |
|  `x +: xs` 	    	    |`xs`의 앞에 `x`를 붙여서 만들어진 새 열을 반환한다.|
|  `xs :+ x` 	    	    |`xs`의 뒤에 `x`를 붙여서 만들어진 새 열을 반환한다.|
|  `xs padTo (len, x)` 	    |`xs`의 뒤에 `x`를 전체 길이가 `len`이 될 때까지 붙여서 만들어진 새 열을 반환한다.|
|  **갱신:** 	    |						     |
|  `xs patch (i, ys, r)`    |`xs`에서 첨자 `i` 번째부터 `r`개 만큼의 원소를 `ys`의 원소로 바꿔치기 해서 얻은 새 열을 반환한다.|
|  `xs updated (i, x)`      |`xs`에서 첨자 `i`에 있는 원소를 `x`로 바꾼 새 열을 반환한다.|
|  `xs(i) = x`	    	    |(또는 명시적으로 `xs.update(i, x)`라고 쓴다. `mutable.Seq`에서만 사용 가능하다). `xs`의 `i` 번째 원소를 `x`로 변경한다.|
|  **정렬:** 	    |						     |
|  `xs.sorted`	            |`xs`의 원소를 원소 타입의 표준적인 순서를 사용해 정렬한 결과 열을 반환한다.|
|  `xs sortWith lt`	    |`xs`의 원소를 `lt`를 비교 연산으로 사용해 정렬한 결과 열을 반환한다.|
|  `xs sortBy f`	    |`xs`의 원소를 정렬한 결과 열을 반환한다. 비교시 두 원소에 각각 `f`를 적용한 다음 그 결과값을 서로 비교한다.|
|  **반전:** 	    |						     |
|  `xs.reverse`	            |`xs`의 원소를 역순으로 나열한 열을 반환한다.|
|  `xs.reverseIterator`	    |`xs`의 원소를 역순으로 내어 놓는 이터레이터이다.|
|  `xs reverseMap f`	    |`xs`의 원소를 역순으로 순회하면서 `f`를 적용해 나온 결과 값으로 이루어진 열을 반환한다.|
|  **비교:** 	    |						     |
|  `xs startsWith ys`	    |`xs`가 열 `ys`로 시작하는지 여부를 반환한다(몇 가지 변형이 존재한다).|
|  `xs endsWith ys`	    |`xs`가 열 `ys`로 끝나는지 여부를 반환한다(몇 가지 변형이 존재한다).|
|  `xs contains x`	    |`xs`에 `x`원소가 존재하는지 여부를 반환한다.|
|  `xs containsSlice ys`    |`xs`에 `ys`과 같은 부분열이 존재하는지 여부를 반환한다.|
|  `(xs corresponds ys)(p)` |`xs`와 `ys`에서 서로 대응하는 원소가 술어 `p`를 만족하는지 여부를 반환한다.|
|  **중복집합 연산:** |						     |
|  `xs intersect ys`	    |`xs`와 `ys`의 중복 교집합 연산 결과인 열을 반환한다. `xs`에서의 원소 순서를 유지한다.|
|  `xs diff ys`	    	    |`xs`와 `ys`의 중복 차집합 연산 결과인 열을 반한한다. `xs`에서의 원소 순서를 유지한다.|
|  `xs union ys`	    |중복 합집합 연산 결과인 열을 반환한다. `xs ++ ys`와 같다.|
|  `xs.distinct`	    |`xs`에서 중복을 제거한(중복된 원소는 하나만 남기고, 하나만 있는 원소는 그대로 둔) 부분열을 반환한다.|

트레잇 [열(Seq)](http://www.scala-lang.org/api/current/scala/collection/Seq.html)에는 [선형열(LinearSeq)](http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html)과 [첨자열(IndexedSeq)](http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html) 두 하위 트레잇이 있다. 이 두 트레잇에 새로 추가된 메소드는 없지만, 성능상  차이가 나는 특성을 각각 제공한다. 선형 열은 효율적인 `head`와 `tail` 연산이 제공되는 반면, 첨자열은 효율적인 `apply`, `length`, 그리고 (변경 가능한 경우) `update` 연산을 제공한다. 자주 사용되는 선형 열로는 `scala.collection.immutable.List`, `scala.collection.immutable.Stream`이 있다. 자주 사용되는 첨자열로는 `scala.Array`와 `scala.collection.mutable.ArrayBuffer`를 들 수 있다. `Vector(벡터)` 클래스는 첨자열과 선형열 사이에서 흥미로운 절충안을 제공한다. 벡터는 상수 시간에 첨자에 의한 참조가 가능하며 선형 억세스도 상수시간에 가능하다. 이로 인해 첨자 참조와 선형 억세스가 동시에 사용되는 억세스 패턴인 경우 벡터가 좋은 선택일 수 있다. 벡터에 대해서는 [나중에](#vectors) 다룰 것이다.

### 버퍼 ###

변경 가능한 열의 하위 범주 중 중요한 것은 `Buffer(버퍼)`이다. 버퍼는 기존 원소를 변경할 수 있을 뿐 아니라, 원소를 추가, 삭제 하거나, 버퍼의 마지막에 효율적으로 원소를 추가할 수 있다. 버퍼가 추가로 지원하는 주요 메소드로는 원소를 끝에 추가하기 위한 `+=`, `++=`, 원소를 앞에 추가하기 위한 `+=:` and `++=:`, 원소를 삽입하기 위한 `insert`와 `insertAll`, 그리고 원소를 제거하기 위한 `remove`, `-=`가 있다. 이 연산을 아래 표에 정리해 두었다.

자주 사용되는 버퍼 구현을 두 가지 들자면 `ListBuffer`와 `ArrayBuffer`가 있다. 이름이 암시하든 `ListBuffer`는 `List`에 의해 뒷받침되며 원소를 효율적으로 `List`로 변환할 수 있다. 반면 `ArrayBuffer`는 배열에 의해 뒷받침되며 배열로 쉽게 변환 가능하다.

#### 버퍼 클래스의 연산 ####

| 사용법              | 하는 일 |
| ------                   | ------                                                           |
|  **추가:**          |                                                                  |
|  `buf += x`              |원소 `x`를 버퍼의 끝에 추가하고 `buf` 자신을 반환한다.|
|  `buf += (x, y, z)`      |여러 원소를 버퍼의 끝에 추가한다.|
|  `buf ++= xs`            |`xs`의 모든 원소를 버퍼의 끝에 추가한다.|
|  `x +=: buf`             |`x`를 버퍼의 앞에 추가한다.|
|  `xs ++=: buf`           |`xs`의 모든 원소를 버퍼의 앞에 추가한다.|
|  `buf insert (i, x)`     |원소 `x`를 버퍼의 첨자 `i` 번째 원소 앞에 삽입한다.|
|  `buf insertAll (i, xs)` |`xs`의 모든 원소를 버퍼의 첨자 `i`번째 원소 앞에 삽입한다.|
|  **제거:**           |                                                                  |
|  `buf -= x`              |원소 `x`를 버퍼에서 제거한다. (중복이 있는 경우 맨 첫 `x`만을 제거한다.)|
|  `buf remove i`          |첨자 `i` 번째에 이는 원소를 버퍼에서 제거한다.|
|  `buf remove (i, n)`     |첨자 `i`번째에 있는 원소 부터 `n`개의 원소를 버퍼에서 제거한다.|
|  `buf trimStart n`       |처음 `n` 개의 원소를 버퍼에서 제거한다.|
|  `buf trimEnd n`         |마지막 `n` 개의 원소를 버퍼에서 제거한다.|
|  `buf.clear()`           |모든 원소를 버퍼에서 제거한다.|
|  **복사:**            |                                                                  |
|  `buf.clone`             |`buf` 같은 원소를 포함하고 있는 새로운 버퍼를 만든다.|
