---
layout: overview-large
title: 집합(Set)

disqus: true

partof: collections
num: 6
language: ko
---

`Set(집합)`은 `Iterable(반복가능)` 중에서 중복 원소가 없는 것이다. 일반적인 집합의 연산은 다음 표에 정리되어 있고, 변경 가능한 집합의 연산은 그 다음에 오는 표에 정리되어 있다. 연산들은 다음과 같은 범주에 들어간다.

* **검사** 연산으로 `contains`, `apply`, `subsetOf`가 있다. `contains` 메소드는 집합에 원소가 속해 있는지를 검사한다. 집합에 있어 `apply` 메소드는 `contains`과 동일하다. 따라서 `set(elem)`과 `set contains elem`는 같다. 따라서 집합을 원소가 포함되어있는지를 검사하는 검사 함수로 사용 가능하다. 

예를 들면 다음과 같다.


    val fruit = Set("apple", "orange", "peach", "banana")
    fruit: scala.collection.immutable.Set[java.lang.String] = 
    Set(apple, orange, peach, banana)
    scala> fruit("peach")
    res0: Boolean = true
    scala> fruit("potato")
    res1: Boolean = false


* **추가** 연산에는 `+` and `++`가 있다. 이들은 집합에 하나 이상의 원소를 추가한 새 집합을 만들어 낸다.
* **제거** 연산인 `-`, `--`는 집합에서 하나 이상의 원소를 제거한 새 집합을 만들어 낸다.
* **집합 연산**으로 합집합, 교집합, 차집합이 있다. 각 연산은 두가지 버전이 존재한다. 하나는 영어문자를 사용한 것이고, 다른 하나는 기호로 된 이름을 사용한 것이다. 영문자 버전은 `intersect`, `union`, `diff`이며, 각각 순서대로 `&`, `|`, `&~`이란 기호 이름과 대응된다. 사실은 `Traversable`에서 상속받은 `++`도 `union` 또는 `|`에 대한 별칭이라 생각할수 있다. 다만 차이가 나는 것은 `++`는 `Traversable`을 매개변수로 받을 수 있지만, `union`과 `|`에는 집합만을 허용한다는 점이다.

### 집합(Set)의 연산 ###

| 사용법   	  	    | 하는일				     |
| ------       	       	    | ------					     |
|  **검사:**               |						     |
|  `xs contains x`  	    |`x`가 `xs`의 원소인지 여부를 반환한다.        |
|  `xs(x)`        	    |`xs contains x`와 같다.                        |
|  `xs subsetOf ys`  	    |`xs`가 `ys`의 부분집합인지 여부를 반환한다.         |
|  **추가:**           |						     |
|  `xs + x`                 |`xs`의 모든 원소와 `x`를 원소로 하는 새 집합을 반환한다.|
|  `xs + (x, y, z)`         |`xs`의 모든 원소와 덧붙인 모든 값들을 원소로 하는 새 집합을 반환한다.|
|  `xs ++ ys`  	            |`xs`의 모든 원소와 `ys`의 모든 원소를 원소로 하는 새 집합을 반환한다.|
|  **제거:**               |						     |
|  `xs - x`  	            |`xs`의 모든 원소 중 `x`를 제거한 나머지를 원소로 하는 새 집합을 반환한다.|
|  `xs - (x, y, z)`         |`xs`의 모든 원소 중 열거한 원소들을 제외한 나머지를 원소로 하는 새 집합을 반환한다.|
|  `xs -- ys`  	            |`xs`의 모든 원소 중 `ys`의 원소들을 제거한 나머지를 원소로 하는 새 집합을 반환한다.|
|  `xs.empty`  	            |`xs`와 동일한 타입의 빈 집합을 반환한다.         |
|  **이항 연산:**   |						     |
|  `xs & ys`  	            |`xs`와 `ys`의 교집합 연산이다.          |
|  `xs intersect ys`        |`xs & ys`와 같다.                              |
|  <code>xs &#124; ys</code>  	            |`xs`와 `ys`의 합집합 연산이다.          |
|  `xs union ys`  	    |<code>xs &#124; ys</code>와 같다..                              |
|  `xs &~ ys`  	            |`xs`와 `ys`의 차집합 연산이다.          |
|  `xs diff ys`  	    |`xs &~ ys`와 같다..                             |

변경 가능한 집합은 원소를 추가, 삭제, 변경하는 연산을 추가로 제공한다. 아래 표에 정리되어 있다.

### 변경 가능 집합(mutable.Set)의 연산 ###

| 사용법               | 하는일                    |
| ------       	       	    | ------					     |
|  **추가:**           |						     |
|  `xs += x`  	            |`xs`에 원소 `x`를 부작용을 사용해 추가하고, `xs` 자신을 반환한다.|
|  `xs += (x, y, z)`        |`xs`에 지정된 원소들을 부작용을 사용해 추가하고, `xs` 자신을 반환한다.|
|  `xs ++= ys`  	    |`xs`에 `ys`의 원소들을 부작용을 사용해 추가하고, `xs` 자신을 반환한다.|
|  `xs add x`  	            |`xs`에 원소 `x`를 부작용을 사용해 추가하되, `x`가 집합에 이미 포함되어 있었다면 거짓을, 그렇지 않았다면 참을 반환한다.|
|  **제거:**            |						     |
|  `xs -= x`  	            |`xs`에서 원소 `x`를 부작용을 사용해 제거하고, `xs` 자신을 반환한다.|
|  `xs -= (x, y, z)`  	    |`xs`에서 지정된 원소들을 부작용을 사용해 제거하고, `xs` 자신을 반환한다.|
|  `xs --= ys`  	    |`xs`에서 `ys`의 원소들을 부작용을 사용해 제거하고, `xs` 자신을 반환한다.|
|  `xs remove x`  	    |`xs`에서 원소 `x`를 부작용을 사용해 제거하되, `x`가 집합에 이미 포함되어 있었다면 참을, 그렇지 않았다면 거짓을 반환한다.|
|  `xs retain p`  	    |`xs`에서 술어 `p`를 만족하는 원소를 남기고 나머지를 제거한다.|
|  `xs.clear()`  	    |`xs`의 모든 원소를 제거한다.|
|  **변경:**              |						     |
|  `xs(x) = b`  	    |(명시적으로 `xs.update(x, b)`라고 쓸 수 있음) `b`가 `true`면 `x`를 `xs`에 추가하고, 그렇지 않으면 `x`를 `xs`에서 제거한다.|
|  **복제:**             |						     |
|  `xs.clone`  	            |`xs`와 같은 원소를 포함하는 새 변경 가능한 집합을 만든다.|

변경 불가능한 집합과 마찬가지로 변경가능한 집합도 원소 추가를 위한 `+`, `++`와 원소 제거를 위한 `-`, `--` 연산을 제공한다. 하지만 이 연산들은 집합을 복사하기 때문에 변경 가능한 집합에서는 잘 사용되지 않는다. 더 효율적인 방식으로 `+=`, `-=`가 있다. `s += elem`는 `elem`을 집합 `s`에 부작용을 통해 추가하며, 결과로 집합 자신을 반환한다. 마찬가지로, `s -= elem`은 원소 `elem`을 집합에서 제거하고, 집합 자신을 결과로 반환한다. `+=`와 `-=`와 별개로 반복자나 순회가능 클래스의 원소를 한꺼번에 추가, 삭제하는 `++=`와 `--=` 연산도 있다.

메소드 이름 `+=`와 `-=`는 변경 가능하거나 불가능한 집합 모두에 아주 비슷한 코드를 사용할 수 있음을 의미한다. 먼저 변경 불가능한 집합 `s`를 사용하는 REPL 실행예를 보자.

    scala> var s = Set(1, 2, 3)
    s: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
    scala> s += 4
    scala> s -= 2
    scala> s
    res2: scala.collection.immutable.Set[Int] = Set(1, 3, 4)

여기서 타입이 `immutable.Set`인 `var`에 `+=`와 `-=` 연산을 적용했다. `s += 4` 식은 `s = s + 4`을 줄인 것이다. 따라서 집합 `s`에 대해 추가 메소드 `+`가 호출 된 다음 이 결과가 다시 변수 `s`에 대입된다. 이제 이와 비슷한 변경 가능한 집합의 동작을 살펴보자.


    scala> val s = collection.mutable.Set(1, 2, 3)
    s: scala.collection.mutable.Set[Int] = Set(1, 2, 3)
    scala> s += 4
    res3: s.type = Set(1, 4, 2, 3)
    scala> s -= 2
    res4: s.type = Set(1, 4, 3)

최종 결과는 앞의 예와 아주 비슷하다. `Set(1, 2, 3)`에서 시작해서 `Set(1, 3, 4)`으로 끝난다. 비록 사용된 명령은 앞에서와 같지만 실제 하는 일은 많이 다른 것이다. `s += 4`는 이제 변경 가능한 집합 `s`의 `+=` 메소드를 호출해 집합의 내부 상태를 변경한다. 마찬가지로 `s -= 2` 또한 같은 집합의 `-=` 메소드를 호출한다.

이 둘 간의 차이는 중요한 원칙을 보여준다. 바로 때로 `val`에 저장된 변경 가능한 컬렉션을 `var`에 저장된 변경 불가능한 것으로 바꾸거나, 그 _역으로_ 바꾸는 것이 가능하다는 사실이다. 외부에서 새 컬렉션이 만들어졌거나 내용이 부작용을 통해 변경되었는지를 관찰할 수 있는 동일 객체에 대한 다른 이름의 참조(alias)가 존재하지 않는 한 이 방식은 잘 동작한다.

변경 가능한 집합은 또한 `+=`와 `-=`의 변형으로 `add`와 `remove`도 제공한다. 차이는 `add`와 `remove`는 연산에 집합에 작용했는지 여부를 알려주는 불린 값을 돌려준다는 점에 있다.

현재 변경 가능 집합의 기본 구현은 해시 테이블을 사용해 원소를 저장한다. 변경 불가능한 집합의 기본 구현은 원소의 갯수에 따라 다른 표현방식을 사용한다. 빈 집합은 싱글턴 객체로 표현된다. 원소가 4개 이하인 집합은 모든 원소 객체를 필드로 저장하는 객체로 표현한다. 그보다 큰 변경 불가능 집합은 [해시 트라이(hash trie)](#hash-tries)로 구현되어 있다.

이런 구현의 차이로 인해 더 작은 크기(4 이하)인 경우 변경 불가능한 집합이 변경 가능한 집합보다 더 작고 효율적이다. 따라서 크기가 작은 집합을 예상한다면 변경 불가능한 집합을 사용하도록 하라.

집합에는 `SortedSet(정렬된 집합)`과 `BitSet(비트집합)`이 있다.

### 정렬된 집합 ###

[SortedSet(정렬된 집합)](http://www.scala-lang.org/api/current/scala/collection/SortedSet.html)은 주어진 순서에 따라 원소를 내어놓는(`iterator`나 `foreach` 사용) 집합이다(순서는 집합을 만들 때 자유롭게 결정할 수 있다). [SortedSet](http://www.scala-lang.org/api/current/scala/collection/SortedSet.html)의 기본 구현은 어떤 노드의 왼쪽 하위 트리에 속한 모든 원소가 오른쪽 하위 트리에 속한 모든 원소보다 작다는 불변조건(invariant)을 만족시키는 순서가 있는 이진 트리이다. 따라서 간단한 중위순회(in order traversal)를 통해 트리의 모든 원소를 증가하는 순서로 반환할 수 있다. 스칼라의 클래스 [immutable.TreeSet(트리집합)](http://www.scala-lang.org/api/current/scala/collection/immutable/TreeSet.html)은 이 불변조건을 유지하면서 동시에 _균형잡힌(balanced)_ 특성을 유지하기 위해 _적-흑(red-black)_ 트리를 구현한다. 균형이 잡힌 트리는 루트(root) 노드로부터 리프(leaf) 노드에 이르는 길이가 1 이하로 차이가 나는 경우를 말한다.

빈 [TreeSet(트리집합)](http://www.scala-lang.org/api/current/scala/collection/immutable/TreeSet.html)을 만들려면 우선 원하는 순서를 지정해야 한다.

    scala> val myOrdering = Ordering.fromLessThan[String](_ > _)
    myOrdering: scala.math.Ordering[String] = ...

그리고 나서 이 순서를 사용해 빈 트리를 다음과 같이 만든다.

    scala> TreeSet.empty(myOrdering)
    res1: scala.collection.immutable.TreeSet[String] = TreeSet()

트리의 순서를 지정하지 않는 대신 원소의 타입을 지정해 빈 트리를 만들 수도 있다. 그렇게 하면 원소의 타입에 따른 기본 순서를 사용하게 된다.

    scala> TreeSet.empty[String]
    res2: scala.collection.immutable.TreeSet[String] = TreeSet()

트리 집합으로부터 (트리를 서로 붙이거나, 걸러내는 등의 방법을 사용해) 새 집합을 만드는 경우, 원래의 집합과 같은 순서를 사용할 것이다. 예를 들면 다음과 같다.

scala> res2 + ("one", "two", "three", "four")
res3: scala.collection.immutable.TreeSet[String] = TreeSet(four, one, three, two)

정렬된 집합은 원소의 범위도 지원한다. 예를 들어 `range` 메소드는 지정된 시작 원소로부터 시작해서 지정된 끝 엘리먼트 직전까지의 모든 원소를 반환한다. `from` 메소드는 집합에서의 순서상 지정된 원소와 같거나 큰 모든 원소를 반환한다. 다음은 그 예이다.

    scala> res3 range ("one", "two")
    res4: scala.collection.immutable.TreeSet[String] = TreeSet(one, three)
    scala> res3 from "three"
    res5: scala.collection.immutable.TreeSet[String] = TreeSet(three, two)


### 비트집합(Bitset) ###

[BitSet(비트집합)](http://www.scala-lang.org/api/current/scala/collection/BitSet.html)은 음이 아닌 정수 원소들로 이루어진 집합으로, 비트를 한데 묶은(packed) 하나 또는 그 이상의 워드로 되어 있다. [BitSet(비트집합)](http://www.scala-lang.org/api/current/scala/collection/BitSet.html)의 내부 표현은 `Long`의 배열을 사용한다. 첫 `Long`은 0부터 63을 나타내고, 두번째 것은 64부터 127을 나타내는 방식을 사용한다(0부터 127 사이의 수만 표현하는 변경 불가능한 비트셋은 배열을 최적화해서 한두개의 `Long` 필드만을 사용한다). 어떤 원소가 속해 있다면 해당 원소에 대응하는 `Long` 필드내의 비트가 1로 설정되고, 그렇지 않으면 0으로 설정된다. 따라서 비트집합의 크기는 내부에 저장되는 가장 큰 정수의 값에 따라 결정된다. 만약 가장 큰 정수가 `N`이라면 집합의 크기는 `N/64`개의 `Long`워드 또는 `N/8` 바이트이며, 상태정보 저장을 위해 몇 바이트가 추가된다.

따라서 크기가 작은 정수 원소를 여러개 포함하는 경우 비트집합을 사용하면 다른 집합에 비해 작은 크기로 가능하다. 비트 집합의 또 다른 잇점은 `contains`를 사용한 포함관계 검사나 `+=`, `-=` 등을 사용한 원소 추가/제거가 모두 아주 효율적이라는 점이다.

번역: 오현석(enshahar@gmail.com)