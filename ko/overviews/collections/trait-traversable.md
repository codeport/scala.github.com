---
layout: overview-large
title: 순회가능(Traversable) 트레잇

disqus: true

partof: collections
num: 3
language: ko
---

컬렉션 계층의 최상위에는 트레잇 `Traversable(순회가능)`이 있다. 이 트레잇에 
있는 유일한 추상적인 연산이 바로 `foreach`이다.

    def foreach[U](f: Elem => U) 

`Traversable`을 구현하는 컬렉션 클래스는 단지 이 메소드만 정의하면 된다. 
다른 메소드는 자동으로 `Traversable`에서 상속된다.

`foreach` 메소드는 컬렉션의 모든 원소를 차례로 방문하면서 주어진 연산 f를 각 원소에 
적용한다. 이 연산 f의 타입은 `Elem => U`로 `Elem`은 컬렉션의 원소의 
타입이며, `U`는 임의의 결과 타입이다. `f`는 부작용을 위해 호출된다. 따라서 
f가 내놓는 결과값은 `foreach`가 무시한다.

`Traversable`에는 여러 구체적 메소드가 정의되어 있다. 이들은 아래 표에 나열되어 있다. 
각 메소드들은 다음과 같은 분류에 속한다.

* **병합** 연산 `++`는 두 방문가능한 객체를 함께 이어붙이거나, 어떤 방문가능 객체에 다른 반복자의 모든 원소를 추가한다.
* **맵** 연산인 `map`, `flatMap`, `collect`는 인자로 넘겨진 함수를 컬렉션 원소에 적용한 결과로 이루어진 새 컬렉션을 만들어낸다.
* **변환** 연산인 `toArray`, `toList`, `toIterable`, `toSeq`, `toIndexedSeq`, `toStream`, `toSet`, `toMap`은 `Traversable` 
컬렉션을 더 구체적인 데이터 구조로 변환한다. 런타임에 수신자가 이미 변환 결과 컬렉션 타입이었다면, 각 변환 메소드는 수신자를 그대로 반환한다. 예를 들어 리스트에 `toList`를 적용하면 그 리스트 자신이 반환된다.
* **복사** 연산으로 `copyToBuffer`와 `copyToArray`가 있다. 이름이 말하는데로 각각 컬렉션 원소를 버퍼나 배열에 복사한다.
* **크기 정보** 연산 `isEmpty`, `nonEmpty`, `size`, `hasDefiniteSize`: 순회가능한 컬렉션은 유한할 수도 있고, 무한할 수도 있다. 
무한한 순회가능한 컬렉션의 예를 들자면 자연수의 스트림 `Stream.from(0)`이 있다. 메소드 `hasDefiniteSize`는 컬렉션이 무한 컬렉션일 가능성이 있는지를 알려준다. `hasDefiniteSize`가 참을 반환하면 컬렉션이 유한하다는 것이 확실하다. 하지만, 컬렉션이 내부 원소를 아직 완전히 계산해 채우지 않은 경우에는 거짓을 반환하기 때문에, 거짓을 반환한다 해도 유한할 수도 있고 무한할 수도 있다.
* **원소 가져오기** 연산 `head`, `last`, `headOption`, `lastOption`, `find`등은 컬렉션의 첫 원소나 마지막 원소를 선택하거나, 조건을 만족하는 첫 원소를 선택한다. 하지만 모든 컬렉션에서 "첫번째"나 "마지막"의 의미가 잘 정의되어 있는 것은 아니라는 점에 유의하라. 예를 들어 해시 집합은 해시값에 따라 원소를 저장하는데, 이 해시값은 매 실행시마다 변할 수 있다. 이런 경우 해시 집합의 
"첫번째" 원소는 프로그램이 실행될 때마다 바뀔 수 있다. 어떤 컬렉션이 항상 같은 순서로 원소를 표시한다면 이를 _순서있다_ 고 한다. 대부분의 컬렉션은 순서가 있으나, 일부(_e.g._ 해시 집합)는 그렇지 않다 -- 이들은 순서를 포기하는 대신 효율을 택한 것이다. 동일한 테스트를 반복하거나, 디버깅을 할 때 때로 순서가 꼭 필요하다. 이 때문에 모든 스칼라 컬렉션 타입에는 순서가 있는 대체물이 반드시 존재한다. 예를 들어 `HashSet`에 순서가 부여된 것은 `LinkedHashSet`이다.
* **부분 컬렉션**을 가져오는 연산에는 `tail`, `init`, `slice`, `take`, `drop`, `takeWhile`, `dropWhile`, `filter`, `filterNot`, `withFilter` 등이 있다. 이들은 모두 어떤 첨자 범위나 술어 함수(predicate, 참-거짓을 반환하는 함수)에 의해 식별되는 부분 컬렉션을 반환한다. 
* **분할** 연산인 `splitAt`, `span`, `partition`, `groupBy` 등은 대상 컬렉션의 원소를 구분해서 여러 부분 컬렉션으로 나눈다.
* **원소 검사** 연산 `exists`, `forall`, `count`는 주어진 술어 함수를 가지고 컬렉션 원소들을 검사한다.
* **폴드** 연산 `foldLeft`, `foldRight`, `/:`, `:\`, `reduceLeft`, `reduceRight`는 인접한 두 원소에 대해 이항 연산자(binary operator)를 반복적용한다.
* **특정 폴드** `sum`, `product`, `min`, `max`는 특정 타입(비교가능하거나 수)의 컬렉션에만 작용한다.
* **문자열** 연산 `mkString`, `addString`, `stringPrefix`는 컬렉션을 문자열로 바꾸는 여러가지 방법을 제공한다.
* **뷰** 연산은 `view` 메소드를 오버로딩한 두 메소드이다. 뷰는 지연 계산될 수 있는 컬렉션이다. 뷰에 대해서는 [나중에](#Views) 	다룰 것이다.

### Traversable 클래스의 연산 ###

| 사용법 	  	    | 하는일				     |
| ------       	       	    | ------					     |
|  **추상 메소드:**     |						     |
|  `xs foreach f`	    |함수 `f`를 `xs`의 모든 원소에 적용한다.|
|  **병합:**     	    |						     |
|  `xs ++ ys`	    	    |`xs`와 `ys`의 모든 원소들로 이루어진 컬렉션. `ys`는  [1회 순회가능(TraversableOnce)](http://www.scala-lang.org/api/current/scala/collection/TraversableOnce.html) 컬렉션이다.  즉, [순회가능(Traversable)](http://www.scala-lang.org/api/current/scala/collection/Traversable.html)이거나 [반복자(Iterator)](http://www.scala-lang.org/api/current/scala/collection/Iterator.html)여야 한다.|
|  **맵:**     	    |						     |
|  `xs map f`		    |함수 `f`를 `xs`의 모든 원소에 적용해 반환된 결과로 이루어진 컬렉션을 반환한다.|
|  `xs flatMap f`	    |결과값이 컬렉션인 함수 `f`를 `xs`의 모든 원소에 적용해 반환된 결과를 차례로 이어붙여서 이루어진 컬렉션을 반환한다.|
|  `xs collect f`	    |부분함수(partial function) `f`를 모든 `xs`에 호출해서 결과값이 정의되어 있는 경우에 대해서만 그 값들을 모아서 이루어진 컬렉션을 반환한다.|
|  **변환:**         |						     |
|  `xs.toArray`	    	    |컬렉션을 배열(Array)로 변환한다.|
|  `xs.toList`	    	    |컬렉션을 리스트(List)로 변환한다.|
|  `xs.toIterable`    	    |컬렉션을 반복가능객체(Iterable)로 변환한다.|
|  `xs.toSeq`	    	    |컬렉션을 열(Seq)로 변환한다.|
|  `xs.toIndexedSeq`   	    |컬렉션을 첨자가 있는 열(IndexedSeq)로 변환한다.|
|  `xs.toStream`    	    |컬렉션을 스트림(Stream)으로 변환한다.|
|  `xs.toSet`	    	    |컬렉션을 집합(Set)으로 변환한다.|
|  `xs.toMap`	    	    |컬렉션을 키/값 쌍의 맵으로 변환한다. 컬렉션의 원소가 튜플이 아니라면 이 메소드 호출은 컴파일시 타입 오류가 난다.|
|  **Copying:**             |						     |
|  `xs copyToBuffer buf`    |컬렉션의 모든 원소를 버퍼 `buf`에 복사한다.|
|  `xs copyToArray(arr, s, n)`|첨자 `s`부터 시작해 최대 `n`개의 원소를 배열 `arr`에 복사한다. 마지막 두 매개변수는 생략할 수 있다.|
|  **크기 정보:**           |						     |
|  `xs.isEmpty`	    	    |컬렉션이 비어있다면 참을 반환한다.|
|  `xs.nonEmpty`    	    |컬렉션에 원소가 하나라도 있다면 참을 반환한다.|
|  `xs.size`	    	    |컬렉션의 원소의 갯수를 반환한다.|
|  `xs.hasDefiniteSize`	    |`xs`가 유한한 크기를 가졌는지 알려져 있다면 참을 반환한다.|
|  **원소 가져오기:**   |						     |
|  `xs.head`	    	    |컬렉션의 첫 원소(순서가 없는 컬렉션이라면 임의의 원소)를 반환한다.|
|  `xs.headOption`	    |`xs`가 비어있다면 None, 그렇지 않다면 첫 원소를 Option에 담아서 반환한다.|
|  `xs.last`	    	    |컬렉션의 마지막 원소(순서가 없는 컬렉션이라면 임의의 원소)를 반환한다.|
|  `xs.lastOption`	    |`xs`가 비어있다면 None, 그렇지 않다면 마지막 원소를 Option에 담아서 반환한다.|
|  `xs find p`	    	    |`xs`에서 `p`를 만족하는 첫번째 원소를 Option에 담아 반환한다. 만족하는 원소가 없다면 None을 반환한다.|
|  **부분 컬렉션:**      |						     |
|  `xs.tail`	    	    |`xs.head`를 제외한 나머지 컬렉션이다.|
|  `xs.init`	    	    |`xs.last`를 제외한 나머지 컬렉션이다.|
|  `xs slice (from, to)`    |컬렉션 `xs`에서 첨자 범위에 속하는 원소들(`from`부터 시작해서 `to`까지. 단, `from`에 있는 원소는 포함하고, `to`에 있는 원소는 포함하지 않음)로 이루어진 컬렉션을 반환한다.|
|  `xs take n`	    	    |컬렉션 `xs`에서 앞에서부터 `n`개의 원소로 구성된 컬렉션(만약 순서가 없는 컬렉션이라면 임의의 `n`개의 원소가 선택된다)이다.|
|  `xs drop n`	    	    |컬렉션에서 `xs take n`하고 난 나머지 컬렉션을 반환한다.|
|  `xs takeWhile p`	    |컬렉션 `xs`의 맨 앞에서부터 술어 `p`를 만족하는 동안 원소를 수집해 만들어진 컬렉션. `p`를 만족하지 않는 첫 원소에서 수집은 끝난다. 따라서, 만약 `xs`의 첫 원소가 `p`를 만족하지 않으면 빈 컬렉션을 반환한다.|
|  `xs dropWhile p`	    |컬렉션 `xs`의 맨 앞에서부터 따져서 술어 `p`를 최초로 만족하는 원소로부터 `xs`의 마지막 원소까지로 이루어진 컬렉션이다.|
|  `xs filter p`	    |`xs`의 원소 중 술어 `p`를 만족하는 원소로 이루어진 컬렉션이다.|
|  `xs withFilter p`	    |컬렉션에 필요시 계산하는 필터를 추가한다. 이 결과 컬렉션에 `map`, `flatMap`, `foreach`, `withFilter` 등이 호출되면 `xs` 중에 술어 `p`를 만족하는 원소에 대해서만 처리가 이루어진다.|
|  `xs filterNot p`	    |`xs`의 원소 중 술어 `p`를 만족하지 않는 원소로 이루어진 컬렉션이다.|
|  **분할:**        |						     |
|  `xs splitAt n`	    |`n`위치를 기준으로 `xs`를 둘로 분할한다. `(xs take n, xs drop n)` 쌍과 동일한 컬렉션 쌍을 반환한다.|
|  `xs span p`	    	    |`xs`를 술어 `p`를 가지고 둘로 분할하되, `(xs takeWhile p, xs.dropWhile p)`과 같은 컬렉션 쌍을 반환한다.|
|  `xs partition p`	    |`xs`를 술어 `p`를 만족하는 원소들과 만족하지 않는 원소의 두 컬렉션으로 분할한 튜플을 반환한다. `(xs filter p, xs.filterNot p)`과 같은 결과를 반환한다.|
|  `xs groupBy f`	    |`xs`를 분류 함수 `f`에 따르는 컬렉션의 맵으로 분할한다.|
|  **원소 검사:**  |						     |
|  `xs forall p`	    |술어 `P`가 `xs`의 모든 원소에 대해 성립하는지 여부를 반환한다.|
|  `xs exists p`	    |술어 `P`를 만족하는 원소가 `xs`에 있는지 여부를 반환한다.|
|  `xs count p`	    	    |`xs`에서 술어 `P`를 만족하는 원소의 갯수를 반환한다.|
|  **폴드:** 		    |						     |
|  `(z /: xs)(op)`	    |이항 연산 `op`를 `xs`의 인접 원소에 대해 `z`부터 시작해 왼쪽부터 오른쪽으로 차례로 적용한다.|
|  `(xs :\ z)(op)`	    |이항 연산 `op`를 `xs`의 인접 원소에 대해 `z`부터 시작해 오른쪽부터 왼쪽으로 차례로 적용한다.|
|  `xs.foldLeft(z)(op)`	    |`(z /: xs)(op)`과 같다.|
|  `xs.foldRight(z)(op)`    |`(xs :\ z)(op)`과 같다.|
|  `xs reduceLeft op`	    |이항 연산 `op`를 비어있지 않은 `xs`의 인접 원소에 대해 왼쪽부터 오른쪽으로 차례로 적용한다.|
|  `xs reduceRight op`	    |이항 연산 `op`를 비어있지 않은 `xs`의 인접 원소에 대해 오른쪽부터 왼쪽으로 차례로 적용한다.|
|  **특정 폴드:**      |						     |
|  `xs.sum`	    	    |컬렉션 `xs`의 모든 수 원소를 곱한 값이다.|
|  `xs.product`	    	    |컬렉션 `xs`의 모든 수 원소를 곱한 값이다.|
|  `xs.min`	    	    |순서가 있는 컬렉션 `xs`에서 가장 작은 원소이다.|
|  `xs.max`	    	    |순서가 있는 컬렉션 `xs`에서 가장 큰 원소이다.|
|  **문자열:**             |						     |
|  `xs addString (b, start, sep, end)`|`StringBuilder` `b`에 `xs`의 모든 원소를 `sep`으로 구분해 `start`와 `end` 사이에 나열한 문자열을 추가한다. `start`, `sep`, `end`는 생략 가능하다.|
|  `xs mkString (start, sep, end)`|컬렉션 `xs`의 모든 원소를 `sep`로 구분해 문자열 `start`와 `end` 사이에 넣는다. `start`, `sep`, `end`는 생략 가능하다.|
|  `xs.stringPrefix`	    |`xs.toString`이 반환하는 문자열의 맨 앞에 표시될 컬렉션 이름이다.|
|  **뷰:**               |						     |
|  `xs.view`	    	    |`xs`에 대한 뷰를 만든다.|
|  `xs view (from, to)`     |`xs`에 대해 첨자 범위에 속하는 원소에 대한 뷰를 만든다.|

번역: 오현석(enshahar@gmail.com)
