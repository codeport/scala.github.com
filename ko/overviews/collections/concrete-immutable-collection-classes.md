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

Lists are very efficient when the algorithm processing them is careful to only process their heads. Accessing, adding, and removing the head of a list takes only constant time, whereas accessing or modifying elements later in the list takes time linear in the depth into the list.

[Vector](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Vector.html) is a collection type (introduced in Scala 2.8) that addresses the inefficiency for random access on lists. Vectors allow accessing any element of the list in "effectively" constant time. It's a larger constant than for access to the head of a list or for reading an element of an array, but it's a constant nonetheless. As a result, algorithms using vectors do not have to be careful about accessing just the head of the sequence. They can access and modify elements at arbitrary locations, and thus they can be much more convenient to write.

Vectors are built and modified just like any other sequence.

    scala> val vec = scala.collection.immutable.Vector.empty
    vec: scala.collection.immutable.Vector[Nothing] = Vector()
    scala> val vec2 = vec :+ 1 :+ 2
    vec2: scala.collection.immutable.Vector[Int] = Vector(1, 2)
    scala> val vec3 = 100 +: vec2
    vec3: scala.collection.immutable.Vector[Int] = Vector(100, 1, 2)
    scala> vec3(0)
    res1: Int = 100

Vectors are represented as trees with a high branching factor (The branching factor of a tree or a graph is the number of children at each node). Every tree node contains up to 32 elements of the vector or contains up to 32 other tree nodes. Vectors with up to 32 elements can be represented in a single node. Vectors with up to `32 * 32 = 1024` elements can be represented with a single indirection. Two hops from the root of the tree to the final element node are sufficient for vectors with up to 2<sup>15</sup> elements, three hops for vectors with 2<sup>20</sup>, four hops for vectors with 2<sup>25</sup> elements and five hops for vectors with up to 2<sup>30</sup> elements. So for all vectors of reasonable size, an element selection involves up to 5 primitive array selections. This is what we meant when we wrote that element access is "effectively constant time".

Vectors are immutable, so you cannot change an element of a vector and still retain a new vector. However, with the `updated` method you can crate a new vector that differs from a given vector only in a single element:

    scala> val vec = Vector(1, 2, 3)
    vec: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3)
    scala> vec updated (2, 4)
    res0: scala.collection.immutable.Vector[Int] = Vector(1, 2, 4)
    scala> vec
    res1: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3)

As the last line above shows, a call to `updated` has no effect on the original vector `vec`. Like selection, functional vector updates are also "effectively constant time". Updating an element in the middle of a vector can be done by copying the node that contains the element, and every node that points to it, starting from the root of the tree. This means that a functional update creates between one and five nodes that each contain up to 32 elements or subtrees. This is certainly more expensive than an in-place update in a mutable array, but still a lot cheaper than copying the whole vector.

Because vectors strike a good balance between fast random selections and fast random functional updates, they are currently the default implementation of immutable indexed sequences:


    scala> collection.immutable.IndexedSeq(1, 2, 3)
    res2: scala.collection.immutable.IndexedSeq[Int] = Vector(1, 2, 3)

## Immutable stacks

If you need a last-in-first-out sequence, you can use a [Stack](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Stack.html). You push an element onto a stack with `push`, pop an element with `pop`, and peek at the top of the stack without removing it with `top`. All of these operations are constant time.

Here are some simple operations performed on a stack:


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

Immutable stacks are used rarely in Scala programs because their functionality is subsumed by lists: A `push` on an immutable stack is the same as a `::` on a list and a `pop` on a stack is the same a `tail` on a list.

## Immutable Queues

A [Queue](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Queue.html) is just like a stack except that it is first-in-first-out rather than last-in-first-out.

Here's how you can create an empty immutable queue:

    scala> val empty = scala.collection.immutable.Queue[Int]()
    empty: scala.collection.immutable.Queue[Int] = Queue()

You can append an element to an immutable queue with `enqueue`:

    scala> val has1 = empty.enqueue(1)
    has1: scala.collection.immutable.Queue[Int] = Queue(1)

To append multiple elements to a queue, call `enqueue` with a collection as its argument:

    scala> val has123 = has1.enqueue(List(2, 3))
    has123: scala.collection.immutable.Queue[Int]
      = Queue(1, 2, 3)

To remove an element from the head of the queue, you use `dequeue`:

    scala> val (element, has23) = has123.dequeue
    element: Int = 1
    has23: scala.collection.immutable.Queue[Int] = Queue(2, 3)

Note that `dequeue` returns a pair consisting of the element removed and the rest of the queue.

## Ranges

A [Range](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/Range.html) is an ordered sequence of integers that are equally spaced apart. For example, "1, 2, 3," is a range, as is "5, 8, 11, 14." To create a range in Scala, use the predefined methods `to` and `by`.

    scala> 1 to 3
    res2: scala.collection.immutable.Range.Inclusive
      with scala.collection.immutable.Range.ByOne = Range(1, 2, 3)
    scala> 5 to 14 by 3
    res3: scala.collection.immutable.Range = Range(5, 8, 11, 14)

If you want to create a range that is exclusive of its upper limit, then use the convenience method `until` instead of `to`:

    scala> 1 until 3
    res2: scala.collection.immutable.Range.Inclusive
      with scala.collection.immutable.Range.ByOne = Range(1, 2)

Ranges are represented in constant space, because they can be defined by just three numbers: their start, their end, and the stepping value. Because of this representation, most operations on ranges are extremely fast.

## Hash Tries

Hash tries are a standard way to implement immutable sets and maps efficiently. They are supported by class [immutable.HashMap](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/HashMap.html). Their representation is similar to vectors in that they are also trees where every node has 32 elements or 32 subtrees. But the selection of these keys is now done based on hash code. For instance, to find a given key in a map, one first takes the hash code of the key. Then, the lowest 5 bits of the hash code are used to select the first subtree, followed by the next 5 bits and so on. The selection stops once all elements stored in a node have hash codes that differ from each other in the bits that are selected up to this level.

Hash tries strike a nice balance between reasonably fast lookups and reasonably efficient functional insertions (`+`) and deletions (`-`). That's why they underly Scala's default implementations of immutable maps and sets. In fact, Scala has a further optimization for immutable sets and maps that contain less than five elements. Sets and maps with one to four elements are stored as single objects that just contain the elements (or key/value pairs in the case of a map) as fields. The empty immutable set and the empty immutable map is in each case a single object - there's no need to duplicate storage for those because and empty immutable set or map will always stay empty.

## Red-Black Trees

Red-black trees are a form of balanced binary trees where some nodes are designated "red" and others designated "black." Like any balanced binary tree, operations on them reliably complete in time logarithmic to the size of the tree.

Scala provides implementations of immutable sets and maps that use a red-black tree internally. Access them under the names [TreeSet](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/TreeSet.html) and [TreeMap](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/TreeMap.html).


    scala> scala.collection.immutable.TreeSet.empty[Int]
    res11: scala.collection.immutable.TreeSet[Int] = TreeSet()
    scala> res11 + 1 + 3 + 3
    res12: scala.collection.immutable.TreeSet[Int] = TreeSet(1, 3)

Red black trees are the standard implementation of `SortedSet` in Scala, because they provide an efficient iterator that returns all elements in sorted order.

## Immutable BitSets

A [BitSet](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/BitSet.html) represents a collection of small integers as the bits of a larger integer. For example, the bit set containing 3, 2, and 0 would be represented as the integer 1101 in binary, which is 13 in decimal.

Internally, bit sets use an array of 64-bit `Long`s. The first `Long` in the array is for integers 0 through 63, the second is for 64 through 127, and so on. Thus, bit sets are very compact so long as the largest integer in the set is less than a few hundred or so.

Operations on bit sets are very fast. Testing for inclusion takes constant time. Adding an item to the set takes time proportional to the number of `Long`s in the bit set's array, which is typically a small number. Here are some simple examples of the use of a bit set:

    scala> val bits = scala.collection.immutable.BitSet.empty
    bits: scala.collection.immutable.BitSet = BitSet()
    scala> val moreBits = bits + 3 + 4 + 4
    moreBits: scala.collection.immutable.BitSet = BitSet(3, 4)
    scala> moreBits(3)
    res26: Boolean = true
    scala> moreBits(0)
    res27: Boolean = false

## List Maps

A [ListMap](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/collection/immutable/ListMap.html) represents a map as a linked list of key-value pairs. In general, operations on a list map might have to iterate through the entire list. Thus, operations on a list map take time linear in the size of the map. In fact there is little usage for list maps in Scala because standard immutable maps are almost always faster. The only possible difference is if the map is for some reason constructed in such a way that the first elements in the list are selected much more often than the other elements.

    scala> val map = scala.collection.immutable.ListMap(1->"one", 2->"two")
    map: scala.collection.immutable.ListMap[Int,java.lang.String] = 
       Map(1 -> one, 2 -> two)
    scala> map(2)
    res30: String = "two"






















