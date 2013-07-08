---
layout: overview-large
title: 배열

disqus: true

partof: collections
num: 10
language: ko
---

[배열(Array)](http://www.scala-lang.org/api/{{ site.scala-version }}/scala/Array.html)은 스칼라에 있는 특별한 종류의 컬렉션이다. 한편, 스칼라 배열은 자바의 배열과 일대일 대응한다. 즉, 스칼라 배열 `Array[Int]`는 자바의 `int[]`로 표현되며, `Array[Double]`은 자바의 `double[]`로, `Array[String]`은 `Java String[]`로 표현될 수 있다. 그러나 이와 동시에, 스칼라 배열은 자바 배열 유사체보다 훨씬 많은 것을 제공한다. 첫 번째로, 스칼라 배열은 **제너릭(Generic)** 할 수 있다. 이는 `Array[T]`를 사용할 수 있다는 의미이며, `T`는 타입 매개 변수나 추상 타입을 의미한다. 두 번째로, 스칼라 배열은 스칼라 시퀀스와 호환된다. `Seq[T]`가 필요할 때, `Array[T]`를 전달할 수 있다는 것을 의미한다. 마지막으로, 스칼라 배열은 모든 시퀀스 연산을 지원한다. 직접 예시를 통해서 알아보자:

    scala> val a1 = Array(1, 2, 3)
    a1: Array[Int] = Array(1, 2, 3)
    scala> val a2 = a1 map (_ * 3)
    a2: Array[Int] = Array(3, 6, 9)
    scala> val a3 = a2 filter (_ % 2 != 0)
    a3: Array[Int] = Array(3, 9)
    scala> a3.reverse
    res1: Array[Int] = Array(9, 3)

주어진 스칼라 배열은 자바의 배열과 똑같이 표현된다. 스칼라는 이러한 추가적인 기능을 어떻게 지원할까? 사실, 위의 질문의 답은 스칼라 2.8 버전과 그 이전 버전에 따라 다르다. 예전에는, 스칼라 컴파일러가 "마법같이" 박싱(boxing)과 언박싱(unboxing) 과정에서, 필요할 때 배열을 `Seq` 객체로 변환하고 되돌렸다. 자세한 내용은 꽤 복잡하며, 특히 제너릭 타입 `Array[T]`에서 새로운 배열을 생성할 때는 더욱 어렵다. 헷갈리는 복합 경계 조건(corner case)들이 있고, 배열 연산의 성능을 예측하기 어려웠다.

스칼라 2.8의 설계는 훨씬 간단하다. 컴파일러가 하는 모든 마법은 거의 사라졌다. 대신에 스칼라 2.8의 배열 구현은 암시적 변환의 체계적인 사용을 가능케 한다. 스칼라 2.8에서는, 배열은 시퀀스**처럼** 행동하지 않는다. 실제로 기본 배열의 데이터 타입이 `Seq`의 자식 타입(subtype)이 아니기 때문에 그럴 수 없다. 대신에, 배열과 `Seq`의 자식 클래스인 `scala.collection.mutable.WrappedArray`의 인스턴스 사이에는 암시적으로 "감싸주는(wrapping)" 변환이 존재한다. 직접 알아보자:

    scala> val seq: Seq[Int] = a1
    seq: Seq[Int] = WrappedArray(1, 2, 3)
    scala> val a4: Array[Int] = s.toArray
    a4: Array[Int] = Array(1, 2, 3)
    scala> a1 eq a4
    res2: Boolean = true

위의 상호작용은 배열에서 `WrappedArray`로의 암시적인 변환이 있기 때문에, 배열이 시퀀스와 호환되는 것을 보여준다.  `WrappedArray`에서 `배열`로 변환하기 위해서는, `Traversable`에 정의된 `toArray` 메소드를 사용할 수 있다. 마지막 REPL 줄에서 `toArray`를 통한 감싸고 풀어주는 변환이, 시작했던 배열과 동일한 것을 만들어주는 것을 볼 수 있다. 

배열에 적용되는 또 다른 암시적 변환이 존재한다. 이 변환은 모든 시퀀스 메소드를 배열에 "더하지만" 배열 자체를 시퀀스로 바꾸지 않는다. "더한다"는 것은 배열이 모든 시퀀스 메소드를 지원하는 `ArrayOps` 타입의 객체로 감싸진다는 것을 의미한다. 일반적으로, 이 `ArrayOps` 객체는 오래 가지 않는다; 보통 시퀀스 메소드가 불려진 후에는 접근할 수 없고, 메모리 공간은 재활용된다. 현대의 가상 머신은 종종 이러한 객체의 생성을 생략하기도 한다.

이러한 배열에서의 두 가지 암시적 변환의 차이점은 다음 REPL 다이얼로그에서 확인할 수 있다:

    scala> val seq: Seq[Int] = a1
    seq: Seq[Int] = WrappedArray(1, 2, 3)
    scala> seq.reverse
    res2: Seq[Int] = WrappedArray(3, 2, 1)
    scala> val ops: collection.mutable.ArrayOps[Int] = a1
    ops: scala.collection.mutable.ArrayOps[Int] = [I(1, 2, 3)
    scala> ops.reverse
    res3: Array[Int] = Array(3, 2, 1)

`WrappedArray`인 `seq`를 반전시키는 것이 다시 `WrappedArray`를 돌려줌을 볼 수 있다. 감싸진 배열은 `Seqs`이고, 어떤 `Seq`를 반전시키는 것은 다시 `Seq`를 얻게 될 것이기 때문에 논리적인 일이다. 한편, `ArrayOps` 클래스의 값 ops를 반전시키면 `Seq`가 아니라 `배열`을 돌려줄 것이다.

위의 `ArrayOps` 예제는 `WrappedArray`와의 차이를 보여주기 위해 상당히 인위적이다. 일반적으로, `ArrayOps`에 값을 정의하는 일은 일어나지 않는다. 단순히 배열에서 `Seq` 메소드를 호출하면 된다:

    scala> a1.reverse
    res4: Array[Int] = Array(3, 2, 1)

`ArrayOps` 객체는 암시적 변환에 의해 자동으로 삽입된다. 따라서 윗줄은 아래와 동일하다. 

    scala> intArrayOps(a1).reverse
    res5: Array[Int] = Array(3, 2, 1)

`intArrayOps`가 앞서 삽입되었던 암시적 변환이다. 이것은 어떻게 컴파일러가 `WrappedArray`로 변환해주는 위에 있는 다른 암시적 변환을 제치고, `intArrayOps`를 선택하는지에 대한 문제를 던져준다. 결국 두 변환 모두 입력이 정해진 reverse 메소드를 지원하는 타입으로 배열을 변환하는 것이다. 질문에 대한 답은 두 암시적 변환에 우선 순위가 있다는 것이다. `ArrayOps` 변환은 `WrappedArray` 변환에 우선한다. `ArrayOps`는 `Predef` 객체에 선언되어있는 반면에,  `WrappedArray`는 `Predef`에서 상속받은 `scala.LowPriorityImplicits` 클래스에 선언되어있다. 자식 클래스와 자식 객체(subobject)의 암시적 변환은 부모 클래스의 암시적 변환에 우선한다. 따라서, 두 가지 변환이 모두 가능할 경우에는, `Predef`에 있는 것이 선택된다. 문자열에도 유사한 방법이 사용된다.

이제 어떻게 배열이 시퀀스와 호환되며, 어떻게 모든 시퀀스 연산이 지원되는지 알게 되었을 것이다. 제너릭은 어떻게 이루어질까? 자바에서는 타입 매개변수 `T`에 대해 `T[]`를 사용할 수 없다. 그러면 어떻게 스칼라의 `Array[T]`가 구현될까?  사실 `Array[T]` 같은 제너릭 배열은 런타임시에 `byte[]`, `short[]`, `char[]`, `int[]`, `long[]`, `float[]`, `double[]`, `boolean[]` 같은 자바의 8개 원시 타입 중 하나이거나, 객체의 배열일 수 있다. 이러한 모든 타입들의 런타임시 형태는 `AnyRef` (혹은, 해당하는 `java.lang.Object`)이기 때문에, 스칼라 컴파일러는 `Array[T]`를 `AnyRef`로 맵핑한다. 런타임시에 `Array[T]` 타입의 원소가 접근되거나 갱신되면 실제 배열 타입을 검사하고, 자바 배열에 대해 올바른 배열 연산을 수행한다. 이러한 타입 검사는 배열 연산을 조금 느리게 만든다. 제너릭 타입 배열에 접근하는 것은 원시 배열이나 객체(object) 배열에 접근하는 것보다 3 ~ 4배 정도 느리다고 예상할 수 있다. 만약 최대 성능을 원한다면, 제너릭 배열보다는 구체적인 타입 배열을 쓰는 것이 좋다. 제너릭 배열을 구현하는 것만으로는 충분치 않으며, 생성하는 방법 또한 필요하다. 이것은 더욱 어려운 문제이고, 당신의 도움이 조금 필요하다. 문제를 설명하기 위해, 아래의 배열을 생성하기 위한 제너릭 메소드를 기술하는 접근을 살펴보자.

    // this is wrong!
    def evenElems[T](xs: Vector[T]): Array[T] = {
      val arr = new Array[T]((xs.length + 1) / 2)
      for (i <- 0 until xs.length by 2)
        arr(i / 2) = xs(i)
      arr
    }

`evenElems` 메소드는 인수로 넘어온 벡터 `xs`의 짝수 위치에 있는 모든 원소를 포함한 새로운 배열을 반환한다. `evenElems` 본문의 첫째 줄은 인수와 동일한 타입을 가지는 배열을 생성한다. 그러므로 실제 타입 매개 변수 `T`에 따라 `Array[Int]`, `Array[Boolean]`, 혹은 자바의 다른 원시 타입 배열이나 참조 타입의 배열일 수 있다. 하지만 이 타입들은 런타임시에 모두 다르게 구현이 되는데, 스칼라는 어떻게 런타임에서 올바른 타입을 찾아낼 수 있을까? 사실, 이것은 주어진 정보로는 해결할 수 없다. 이는 타입 매개 변수 `T`에 대응하는 실제 타입이 런타임시에 지워지기 때문이다. 이것이 위의 코드를 컴파일 하려고 하면 아래와 같은 에러 메시지가 출력되는 이유이다.

    error: cannot find class manifest for element type T
      val arr = new Array[T]((arr.length + 1) / 2)
            ^

여기에 필요한 것은, 실제 `evenElems`의 타입이 무엇인지에 대한 런타임 힌트를 컴파일러에게 제공해서 도와주는 것이다. 이 런타임 힌트는 `scala.reflect.ClassManifest` 클래스 매니페스트의 형태를 가진다. 클래스 매니페스트란 타입의 최상위 클래스가 무엇인지를 설명하는 타입 설명 객체이다. 클래스 매니페스트대신에, 타입의 모든 것을 설명하는 `scala.reflect.Manifest`라는 전체 매니페스트도 있다. 그러나 배열 생성시에는, 클래스 매니페스트만이 필요하다.

스칼라 컴파일러에게 클래스 매니페스트를 만들도록 지시를 내린다면 알아서 클래스 매니페스트를 생성한다. "지시한다"는 의미는 클래스 매니페스트를 아래와 같이 암시적 매개 변수로 요청한다는 뜻이다:

    def evenElems[T](xs: Vector[T])(implicit m: ClassManifest[T]): Array[T] = ...

문맥 범위(context bound)를 이용하면 더 짧게 클래스 매니페스트를 요청할 수 있다. 이것은 타입 뒤에 쌍점(:)과 함께 따라오는 클래스의 이름 `ClassManifest`를 기술하는 것을 의미한다.:

    // this works
    def evenElems[T: ClassManifest](xs: Vector[T]): Array[T] = {
      val arr = new Array[T]((xs.length + 1) / 2)
      for (i <- 0 until xs.length by 2)
        arr(i / 2) = xs(i)
      arr
    }

변경된 두 가지 버전의 `evenElems`는 정확히 동일하다. 두 경우 모두, `Array[T]`가 생성될 때, 컴파일러는 타입 매개변수 T에 대해서 클래스 매니페스트를 검색하고, `ClassManifest[T]`의 암시적 값을 찾는다. 해당 값이 발견되면, 매니페스트는 올바른 종류의 배열을 생성한다. 그러지 않으면, 위와 같은 에러 메시지를 보게 될 것이다.

아래의 `evenElems` 메소드를 사용하는 REPL 상호작용을 살펴보자.

    scala> evenElems(Vector(1, 2, 3, 4, 5))
    res6: Array[Int] = Array(1, 3, 5)
    scala> evenElems(Vector("this", "is", "a", "test", "run"))
    res7: Array[java.lang.String] = Array(this, a, run)

양 쪽 모두, 스칼라 컴파일러가 자동으로 원소의 타입(`Int`와 `String`)에 대한 클래스 매니페스트를 생성하였고, `evenElems` 메소드의 암시적 매개 변수로 전달하였다. 컴파일러는 모든 구체적인 타입에 대해서 매니페스트를 생성해주지만, 인수 자체가 클래스 매니페스트 없는 타입 매개 변수일 경우에는 불가능하다. 예를 들어, 아래의 실패 사례를 살펴보자:

    scala> def wrap[U](xs: Array[U]) = evenElems(xs)
    <console>:6: error: could not find implicit value for 
     evidence parameter of type ClassManifest[U]
         def wrap[U](xs: Array[U]) = evenElems(xs)
                                          ^
여기에서 일어난 일은 `evenElems`는 타입 매개 변수 `U`에 대해서 클래스 매니페스트를 요청했지만, 아무 것도 발견되지 않았다. 물론 이러한 경우에 대한 해결책은 `U`에 대한 또 다른 암시적 클래스 매니페스트를 요청하는 것이다. 따라서 아래와 같이 동작한다:

    scala> def wrap[U: ClassManifest](xs: Array[U]) = evenElems(xs)
    wrap: [U](xs: Array[U])(implicit evidence$1: ClassManifest[U])Array[U]

이 예시를 통해 `U`의 정의에 대한 문맥 범위가 `ClassManifest[U]` 타입의 `evidence$1`라는 이름의 암시적 매개 변수로 약칭한 것을 볼 수 있다.

요약하면, 제너릭 배열의 생성은 클래스 매니페스트를 요구한다. 그러므로 타입 매개 변수 `T`의 배열을 생성할 때, `T`에 대한 암시적 클래스 매니페스트를 제공해주어야 한다. 이를 위한 가장 쉬운 방법은, `[T: ClassManifest]` 처럼 문맥 범위 안에 타입 매개 변수와 함께 `ClassManifest`를 선언해주는 것이다.

