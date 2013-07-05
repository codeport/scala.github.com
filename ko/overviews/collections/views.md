---
layout: overview-large
title: 뷰

disqus: true

partof: collections
num: 14
language: ko
---

컬렉션에는 새로운 컬렉션을 만들기 위한 메소드가 꽤 들어 있다. 예를 들면 `map`, `filter`, `++` 등이 그렇다. 이러한 메소드를 변환메소드(transformer)라 부르는데, 이유는 최소 하나 이상의 컬렉션을 수신객체로 받아서 결과값으로 다른 컬렉션을 만들어내기 때문이다.

변환메소드를 구현하는 방법은 크게 두가지가 있다. 하나는 _바로 계산하기(strict)_ 방식이다. 이는 변환 메소드에서 새로 만들어지는 컬렉션에 모든 원소를 만들어서 집어넣어서 반환하는 방식이다. 또 다른 방식은 바로 계산하지 않기(non-strict) 또는 _지연 계산하기(lazy)_ 방식이다. 이는 결과 컬렉션 전체를 구성해 반환하는 대신 그에 대한 대행 객체(proxy)를 반환해서, 실제 결과 컬렉션의 원소를 요청받았을 때에만 이를 만들어내도록 하는 방법이다.

바로 계산하지 않는 변환메소드의 예로 아래 지연계산 맵 연산을 살펴보자.

    def lazyMap[T, U](coll: Iterable[T], f: T => U) = new Iterable[T] {
      def iterator = coll.iterator map f
    }

`lazyMap`이 인자로 받은 `coll`의 모든 원소를 하나하나 읽지 않고 새 `Iterable`을 반환한다는 사실에 유의하라. 주어진 함수 `f`는 요청에 따라 새 컬렉션의 `iterator`에 적용된다.

`Stream`을 제외한 스칼라의 컬렉션에 있는 변환메소드는 기본적으로 바로 계산하는 방식을 택한다. 스트림은 모든 변환 메소드가 지연계산으로 되어 있다. 하지만, 모든 컬렉션을 지연 계산 방식의 컬렉션으로 바꾸거나 그 _역으로_ 바꾸는 구조적인 방법이 있다. 그 방법은 바로 컬렉션 뷰를 활용하는 것이다. _뷰(view)_ 는 어떤 기반 컬렉션을 대표하되 모든 변환메소드를 지연계산으로 구현하는 특별한 종류의 컬렉션이다.

컬렉션에서 그 뷰로 변환하려면 해당 컬렉션의 `view` 메소드를 사용하면 된다. 
To go from a collection to its view, you can use the view method on the collection. If `xs` is some collection, then `xs.view` is the same collection, but with all transformers implemented lazily. To get back from a view to a strict collection, you can use the `force` method.

Let's see an example. Say you have a vector of Ints over which you want to map two functions in succession:

    scala> val v = Vector(1 to 10: _*)
    v: scala.collection.immutable.Vector[Int] =
       Vector(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    scala> v map (_ + 1) map (_ * 2)
    res5: scala.collection.immutable.Vector[Int] = 
       Vector(4, 6, 8, 10, 12, 14, 16, 18, 20, 22)

In the last statement, the expression `v map (_ + 1)` constructs a new vector which is then transformed into a third vector by the second call to `map (_ * 2)`. In many situations, constructing the intermediate result from the first call to map is a bit wasteful. In the example above, it would be faster to do a single map with the composition of the two functions `(_ + 1)` and `(_ * 2)`. If you have the two functions available in the same place you can do this by hand. But quite often, successive transformations of a data structure are done in different program modules. Fusing those transformations would then undermine modularity. A more general way to avoid the intermediate results is by turning the vector first into a view, then applying all transformations to the view, and finally forcing the view to a vector:

    scala> (v.view map (_ + 1) map (_ * 2)).force
    res12: Seq[Int] = Vector(4, 6, 8, 10, 12, 14, 16, 18, 20, 22)  

Let's do this sequence of operations again, one by one:

    scala> val vv = v.view
    vv: scala.collection.SeqView[Int,Vector[Int]] = 
       SeqView(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

The application `v.view` gives you a `SeqView`, i.e. a lazily evaluated `Seq`. The type `SeqView` has two type parameters. The first, `Int`, shows the type of the view's elements. The second, `Vector[Int]` shows you the type constructor you get back when forcing the `view`.

Applying the first `map` to the view gives:

    scala> vv map (_ + 1)
    res13: scala.collection.SeqView[Int,Seq[_]] = SeqViewM(...)

The result of the `map` is a value that prints `SeqViewM(...)`. This is in essence a wrapper that records the fact that a `map` with function `(_ + 1)` needs to be applied on the vector `v`. It does not apply that map until the view is `force`d, however. The "M" after `SeqView` is an indication that the view encapsulates a map operation. Other letters indicate other delayed operations. For instance "S" indicates a delayed `slice` operations, and "R" indicates a `reverse`. Let's now apply the second `map` to the last result.

    scala> res13 map (_ * 2)
    res14: scala.collection.SeqView[Int,Seq[_]] = SeqViewMM(...)

You now get a `SeqView` that contains two map operations, so it prints with a double "M": `SeqViewMM(...)`. Finally, forcing the last result gives:

scala> res14.force
res15: Seq[Int] = Vector(4, 6, 8, 10, 12, 14, 16, 18, 20, 22)

Both stored functions get applied as part of the execution of the `force` operation and a new vector is constructed. That way, no intermediate data structure is needed.

One detail to note is that the static type of the final result is a Seq, not a Vector. Tracing the types back we see that as soon as the first delayed map was applied, the result had static type `SeqViewM[Int, Seq[_]]`. That is, the "knowledge" that the view was applied to the specific sequence type `Vector` got lost. The implementation of a view for some class requires quite a lot of code, so the Scala collection libraries provide views mostly only for general collection types, but not for specific implementations (An exception to this are arrays: Applying delayed operations on arrays will again give results with static type `Array`).

There are two reasons why you might want to consider using views. The first is performance. You have seen that by switching a collection to a view the construction of intermediate results can be avoided. These savings can be quite important. As another example, consider the problem of finding the first palindrome in a list of words. A palindrome is a word which reads backwards the same as forwards. Here are the necessary definitions:

    def isPalindrome(x: String) = x == x.reverse
    def findPalidrome(s: Seq[String]) = s find isPalindrome

Now, assume you have a very long sequence words and you want to find a palindrome in the first million words of that sequence. Can you re-use the definition of `findPalidrome`? If course, you could write:

    findPalindrome(words take 1000000)

This nicely separates the two aspects of taking the first million words of a sequence and finding a palindrome in it. But the downside is that it always constructs an intermediary sequence consisting of one million words, even if the first word of that sequence is already a palindrome. So potentially, 999'999 words are copied into the intermediary result without being inspected at all afterwards. Many programmers would give up here and write their own specialized version of finding palindromes in some given prefix of an argument sequence. But with views, you don't have to. Simply write:

    findPalindrome(words.view take 1000000)

This has the same nice separation of concerns, but instead of a sequence of a million elements it will only construct a single lightweight view object. This way, you do not need to choose between performance and modularity.

The second use case applies to views over mutable sequences. Many transformer functions on such views provide a window into the original sequence that can then be used to update selectively some elements of that sequence. To see this in an example, let's suppose you have an array `arr`:

    scala> val arr = (0 to 9).toArray
    arr: Array[Int] = Array(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

You can create a subwindow into that array by creating a slice of a view of `arr`:

    scala> val subarr = arr.view.slice(3, 6)
    subarr: scala.collection.mutable.IndexedSeqView[
    Int,Array[Int]] = IndexedSeqViewS(...)

This gives a view `subarr` which refers to the elements at positions 3 through 5 of the array `arr`. The view does not copy these elements, it just provides a reference to them. Now, assume you have a method that modifies some elements of a sequence. For instance, the following `negate` method would negate all elements of the sequence of integers it's given:

    scala> def negate(xs: collection.mutable.Seq[Int]) =
             for (i <- 0 until xs.length) xs(i) = -xs(i)
    negate: (xs: scala.collection.mutable.Seq[Int])Unit

Assume now you want to negate elements at positions 3 through five of the array `arr`. Can you use `negate` for this? Using a view, this is simple:

    scala> negate(subarr)
    scala> arr
    res4: Array[Int] = Array(0, 1, 2, -3, -4, -5, 6, 7, 8, 9)

What happened here is that negate changed all elements of `subarr`, which were a slice of the elements of `arr`. Again, you see that views help in keeping things modular. The code above nicely separated the question of what index range to apply a method to from the question what method to apply.

After having seen all these nifty uses of views you might wonder why have strict collections at all? One reason is that performance comparisons do not always favor lazy over strict collections. For smaller collection sizes the added overhead of forming and applying closures in views is often greater than the gain from avoiding the intermediary data structures. A probably more important reason is that evaluation in views can be very confusing if the delayed operations have side effects.

Here's an example which bit a few users of versions of Scala before 2.8. In these versions the Range type was lazy, so it behaved in effect like a view. People were trying to create a number of actors like this:


    val actors = for (i <- 1 to 10) yield actor { ... }

They were surprised that none of the actors was executing afterwards, even though the actor method should create and start an actor from the code that's enclosed in the braces following it. To explain why nothing happened, remember that the for expression above is equivalent to an application of map:

    val actors = (1 to 10) map (i => actor { ... })

Since previously the range produced by `(1 to 10)` behaved like a view, the result of the map was again a view. That is, no element was computed, and, consequently, no actor was created! Actors would have been created by forcing the range of the whole expression, but it's far from obvious that this is what was required to make the actors do their work.

To avoid surprises like this, the Scala 2.8 collections library has more regular rules. All collections except streams and views are strict. The only way to go from a strict to a lazy collection is via the `view` method. The only way to go back is via `force`. So the `actors` definition above would behave as expected in Scala 2.8 in that it would create and start 10 actors. To get back the surprising previous behavior, you'd have to add an explicit `view` method call:

    val actors = for (i <- (1 to 10).view) yield actor { ... }

In summary, views are a powerful tool to reconcile concerns of efficiency with concerns of modularity. But in order not to be entangled in aspects of delayed evaluation, you should restrict views to two scenarios. Either you apply views in purely functional code where collection transformations do not have side effects. Or you apply them over mutable collections where all modifications are done explicitly. What's best avoided is a mixture of views and operations that create new collections while also having side effects.



