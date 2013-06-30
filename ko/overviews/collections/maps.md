---
layout: overview-large
title: 맵(Map)

disqus: true

partof: collections
num: 7
language: ko
---

[Map(맵)](http://www.scala-lang.org/api/current/scala/collection/Map.html)은 키와 값의 쌍으로 구성된 [Iterable(반복가능)](http://www.scala-lang.org/api/current/scala/collection/Iterable.html)한 컬렉션이다(_맵핑_ 또는 _연관_ 이라고도 한다). 스칼라의 [Predef](http://www.scala-lang.org/api/current/scala/Predef$.html) 클래스에는 `key -> value`를 `(key, value)` 대신 쓸수 있도록 하는 묵시적 변환이 정의되어 있다. 예를 들어 `Map("x" -> 24, "y" -> 25, "z" -> 26)`와 `Map(("x", 24), ("y", 25), ("z", 26))`는 의미가 완전히 동일하다. 다만 전자가 더 보기 좋을 뿐이다.

맵에서 제공하는 기본 연산들은 집합과 비슷하다. 이를 분류하면 아래와 같다. 전체는 그 이후 표에 요약해 두었다.

* **찾기** 연산 `apply`, `get`, `getOrElse`, `contains`, `isDefinedAt` 들은 맵을 키를 받아 값을 반환하는 부분함수로 바꾼다. 맵에서 바탕이 되는 찾기 메소드는 `def get(key): Option[Value]`이다. 연산 "`m get key`"는 맵에 주어진 `key`와 대응하는 연관관계쌍이 들어 있는지를 본다. 만약 그런 쌍이 있다면 그 연관쌍을 `Some`으로 포장해서 내어 놓는다. 맵에 키가 정의되어 있지 않다면 `get`은 `None`을 반환한다. 맵에는 또한 키에 대해 값을 `Option`으로 감싸지 않고 바로 반환하는 `apply` 메소드도 정의되어 있다. 만약 키가 없다면 예외가 발생한다.
* **추가와 변경** 연산 `+`, `++`, `updated`를 사용하면 새로운 연관관계를 맵에 추가하거나, 기존의 관계를 변경할 수 있다.
* **제거** 연산 `-`, `--`는 맵에서 연관 관계를 제거하기 위해 사용한다.
* **부분 컬렉션** 생성 메소드 `keys`, `keySet`, `keysIterator`, `values`, `valuesIterator`는 맵의 모든 키나 모든 값을 별도로 여러 형태로 반환한다.
* **변환** 연산 `filterKeys`와 `mapValues`는 맵을 필터링해 새 맵을 만들거나 기존 맵의 연관관계를 변환할 때 사용한다.

### 맵 클래스의 연산 ###

| 사용법   	  	    | 하는일				     |
| ------       	       	    | ------					     |
|  **찾기:**             |						     |
|  `ms get k`  	            |맵 `ms`에 있는 키 `k`에 대응하는 값을 옵션으로 반환한다. 찾을 수 없다면 `None`이 반환된다.|
|  `ms(k)`  	            |(`ms apply k`라고 명시적으로 쓸 수 있음) 맵 `ms`에서 키 `k`에 대응하는 값을 반환한다. 값이 없다면 예외를 던진다.|
|  `ms getOrElse (k, d)`    |맵 `ms`에서 키 `k`에 대응하는 값을 반환한다. 값이 없다면 디폴트 값으로 지정된 `d`를 반환한다.|
|  `ms contains k`  	    |맴 `ms`에 키 `k`에 대한 맵핑이 정의되어 있는지 여부를 반환한다.|
|  `ms isDefinedAt k`  	    |`contains`와 같다.                             |
| **추가와 변경:**|						     |
|  `ms + (k -> v)`          |`ms`의 모든 연관관계와 더불어 맵핑 `k -> v`, 즉 키 `k`에서 `v`로 가는 연관관계가 추가된 맵을 반환한다.|
|  `ms + (k -> v, l -> w)`  |`ms`의 모든 연관관계와 더불어 주어진 모든 키-값 연관관계가 추가된 맵을 반환한다.|
|  `ms ++ kvs`              |`ms`의 모든 연관관계와 더불어 `kvs`에 있는 연관관계가 추가된 맵을 반환한다. (단, `kvs`는 튜플이 원소이거나 맵 타입이어야 정상적으로 동작한다.)|
|  `ms updated (k, v)`      |`ms + (k -> v)`과 같다.|
| **제거:**             |						     |
|  `ms - k`  	            |`ms`의 모든 연관관계 중에서 키 `k`에 대한 연관관계만이 포함되지 않은 맵을 반환한다.|
|  `ms - (k, 1, m)`  	    |`ms`의 모든 연관관계 중에서 주어진 여러 키들에 대한 연관관계들이 포함되지 않은 맵을 반환한다.|
|  `ms -- ks`  	            |`ms`의 모든 연관관계 중에서 `ks`에 있는 키들에 대한 연관관계들이 포함되지 않은 맵을 반환한다.|
| **부분 컬렉션 :**     |						     |
|  `ms.keys`  	            |`ms`의 모든 키를 포함하는 반복가능 객체를 반환한다.        |
|  `ms.keySet`              |`ms`의 모든 키를 포함하는 집합 객체를 반환한다.              |
|  `ms.keyIterator`         |`ms`의 키를 내어놓는 반복자를 반환한다.          |
|  `ms.values`      	    |`ms`에서 키에 연관된 모든 값을 포함하는 반복가능 객체를 반환한다.        |
|  `ms.valuesIterator`      |`ms`에서 키에 연관된 모든 값을 포함하는 반복자를 반환한다.        |
| **변환:**     |						     |
|  `ms filterKeys p`        |`ms`에서 키가 술어 `p`를 만족하는 연관관계만을 포함하는 새 맵 뷰를 반환한다.|
|  `ms mapValues f`         |`ms`에서 키에 연관된 모든 값에 대해 함수 `f`를 적용해 얻을 수 있는 새 맵 뷰를 반환한다.|

변경 가능 맵은 아래 표에 정리된 연산을 추가로 지원한다.


### mutable.Map 클래스에 정의된 연산 ###

| 사용법   	  	    | 하는일				     |
| ------       	       	    | ------					     |
|  **추가와 변경:**|						     |
|  `ms(k) = v`              |(`ms.update(x, v)`라고 명시적으로 쓸 수도 있음). 키 `k`에서 값 `v`로 가는 맵핑을 맵 `ms`에 부작용을 사용해 추가한다. 이미 키 `k`에 대한 값이 정의되어 있었다면 이를 덮어쓴다.|
|  `ms += (k -> v)`         |키 `k`에서 값 `v`로 가는 맵핑을 맵 `ms`에 부작용을 사용해 추가하고 `ms` 자신을 반환한다.|
|  `ms += (k -> v, l -> w)` |지정된 맵핑들을 맵 `ms`에 부작용을 사용해 추가하고 `ms` 자신을 반환한다.|
|  `ms ++= kvs`             |`kvs`에 있는 모든 연관관계들을 맵 `ms`에 부작용을 사용해 추가하고 `ms` 자신을 반환한다.|
|  `ms put (k, v)`          |키 `k`에서 값 `v`로 가는 맵핑을 맵 `ms`에 부작용을 사용해 추가하고, 이전에 `k`와 연관된 값이 있었다면 이를 옵션 객체로 반환한다.|
|  `ms getOrElseUpdate (k, d)`|키 `k`가 맵 `ms`에 정의되어 있다면 그 값을 반환하고, 그렇지 않다면 `ms`에 새 연관관계 `k -> d`를 추가한 다음 `d`를 반환한다.|
|  **제거:**|						     |
|  `ms -= k`                |키 `k`에 해당하는 맵핑을 맵 `ms`에서 부작용을 사용해 제거한 후, `ms`자신을 반환한다.|
|  `ms -= (k, l, m)`        |지정된 키들에 해당하는 맵핑을 맵 `ms`에서 부작용을 사용해 제거한 후, `ms`자신을 반환한다.|
|  `ms --= ks`              |`ks`에 있는 키들에 해당하는 맵핑을 맵 `ms`에서 부작용을 사용해 제거한 후, `ms`자신을 반환한다.|
|  `ms remove k`            |키 `k`에 대한 맵핑을 맵 `ms`에서 부작용을 사용해 제거하고, 이전에 `k`와 연관된 값이 있었다면 이를 옵션 객체로 반환한다.|
|  `ms retain p`            |`ms`에서 (키,값) 튜플에 대한 술어 `p`를 만족하는 연관 관계들만 남기고 나머지는 다 제거한 다음, `ms` 자신을 반환한다|
|  `ms.clear()`             |`ms`에서 모든 매핑을 제거한 다음,                  |
|  **변환:**      |						     |
|  `ms transform f`         |`ms`의 모든 연관 쌍을 `f`를 사용해 변환한다. `ms`자신을 반환한다.|
|  **복사:**             |						     |
|  `ms.clone`               |`ms`과 같은 맵핑들을 포함하는 새로운 변경가능한 맵을 반환한다.|

맵의 추가와 제거 연산은 집합의 그것과 비슷하다. 집합에서와 마찬가지로 변경 가능한 맵도 부작용을 사용하지 않는 추가 연산 `+`, `-`, `updated`를 지원한다. 하지만 이런 연산들은 변경 가능 맵을 복사하기 때문에 자주 사용되지는 않는다. 대신 변경 가능한 맵 `m`은 보통 "그 자리에서" `m(key) = value`이나 `m += (key -> value)` 연산을 사용해 변경된다. 업데이트 연산에는 이전에 `key`와 연관되어 있던 값을 `Option`으로 돌려주는 `m put (key, value)`도 있다. `put`은 만약 `key`가 맵에 존재하지 않았다면 `None`을 반환한다.

`getOrElseUpdate`는 캐시처럼 동작하는 맵을 억세스할 때 유용하다. `f`를 호출하면 비용이 많이 드는 계산을 수행해야 하는 경우를 생각해 보자.

    scala> def f(x: String) = { 
           println("taking my time."); sleep(100)
           x.reverse }
    f: (x: String)String

더 나아가 `f`에 부작용이 없다고 한다면, 동일한 인자로 이 함수를 호출할 때마다 항상 같은 결과를 받을 것이다. 이런 경우 예전에 계산했던 값을 인자와 `f`의 결과값을 연관시켜 맴에 저장해 두고, 새로운 인자 값이 들어와 맵에서 예전에 계산해 둔 결과를 찾을 수 없는 경우에만 `f`의 결과를 계산하게 한다면 비용이 줄어든다. 이 경우 맵을 함수 `f`의 계산 결과에 대한 _캐시(cache)_ 라 부를 수도 있다.

    val cache = collection.mutable.Map[String, String]()
    cache: scala.collection.mutable.Map[String,String] = Map()

이제 더 효율이 좋은 `f` 함수의 캐시된 버전을 만들 수 있다.

    scala> def cachedF(s: String) = cache.getOrElseUpdate(s, f(s))
    cachedF: (s: String)String
    scala> cachedF("abc")
    taking my time.
    res3: String = cba
    scala> cachedF("abc")
    res4: String = cba

`getOrElseUpdate`의 두번째 인자는 "이름에 의한 호출(by-name)"이므로, 위의 `f("abc")` 계산은 오직 `getOrElseUpdate`가 두번째 인자 값을 필요로 하는 경우에만 수행된다. 이는 정확하게 말하자면 첫 번째 인자를 `cache` 맵에서 못 찾은 경우이다. 맵의 기본 연산을 활용해 `cachedF`를 직접 구현할 수도 있었겠지만, 그렇게 하려면 조금 길게 코드를 작성해야 한다.

    def cachedF(arg: String) = cache get arg match {
      case Some(result) => result
      case None => 
        val result = f(x)
        cache(arg) = result
        result
    }

### Synchronized Sets and Maps ###

To get a thread-safe mutable map, you can mix the `SynchronizedMap` trait trait into whatever particular map implementation you desire. For example, you can mix `SynchronizedMap` into `HashMap`, as shown in the code below. This example begins with an import of two traits, `Map` and `SynchronizedMap`, and one class, `HashMap`, from package `scala.collection.mutable`. The rest of the example is the definition of singleton object `MapMaker`, which declares one method, `makeMap`. The `makeMap` method declares its result type to be a mutable map of string keys to string values.

      import scala.collection.mutable.{Map,
          SynchronizedMap, HashMap}
      object MapMaker {
        def makeMap: Map[String, String] = {
            new HashMap[String, String] with
                SynchronizedMap[String, String] {
              override def default(key: String) =
                "Why do you want to know?"
            }
        }
      }

<center>Mixing in the `SynchronizedMap` trait.</center>

The first statement inside the body of `makeMap` constructs a new mutable `HashMap` that mixes in the `SynchronizedMap` trait:

    new HashMap[String, String] with
      SynchronizedMap[String, String]

Given this code, the Scala compiler will generate a synthetic subclass of `HashMap` that mixes in `SynchronizedMap`, and create (and return) an instance of it. This synthetic class will also override a method named `default`, because of this code:

    override def default(key: String) =
      "Why do you want to know?"

If you ask a map to give you the value for a particular key, but it doesn't have a mapping for that key, you'll by default get a `NoSuchElementException`. If you define a new map class and override the `default` method, however, your new map will return the value returned by `default` when queried with a non-existent key. Thus, the synthetic `HashMap` subclass generated by the compiler from the code in the synchronized map code will return the somewhat curt response string, `"Why do you want to know?"`, when queried with a non-existent key.

Because the mutable map returned by the `makeMap` method mixes in the `SynchronizedMap` trait, it can be used by multiple threads at once. Each access to the map will be synchronized. Here's an example of the map being used, by one thread, in the interpreter:

    scala> val capital = MapMaker.makeMap  
    capital: scala.collection.mutable.Map[String,String] = Map()
    scala> capital ++ List("US" -> "Washington",
            "Paris" -> "France", "Japan" -> "Tokyo")
    res0: scala.collection.mutable.Map[String,String] =
      Map(Paris -> France, US -> Washington, Japan -> Tokyo)
    scala> capital("Japan")
    res1: String = Tokyo
    scala> capital("New Zealand")
    res2: String = Why do you want to know?
    scala> capital += ("New Zealand" -> "Wellington")
    scala> capital("New Zealand")                    
    res3: String = Wellington

You can create synchronized sets similarly to the way you create synchronized maps. For example, you could create a synchronized `HashSet` by mixing in the `SynchronizedSet` trait, like this:

    import scala.collection.mutable
    val synchroSet =
      new mutable.HashSet[Int] with
          mutable.SynchronizedSet[Int]

Finally, if you are thinking of using synchronized collections, you may also wish to consider the concurrent collections of `java.util.concurrent` instead.
