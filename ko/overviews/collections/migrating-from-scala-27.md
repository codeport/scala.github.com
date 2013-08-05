---
layout: overview-large
title: 스칼라 2.7로부터 포팅하기

disqus: true

partof: collections
num: 18
outof: 18
language: ko
---

기본 스칼라 앱을 새 컬렉션에서 사용하기 위해 포팅하는 작업은 거의 자동으로 이루어져야만 한다. 주의해야 할 것은 두세가지 뿐이다.

일반적으로, 스칼라 2.7 컬렉션의 옛 기능은 그대로 남아있다. 몇몇 특징은 사용하지 않을 것이 권장되며, 향후 배포판에서는 제거될 수 있다. 스칼라 2.8에서 이러한 기능을 사용한다면 _사용금지 경고_를 보게될 것이다. 2.8에서도 그대로 사용되지만, 성능이나 의미는 변경되어 사용 금지를 표시하는 것이 바람직하지 않은 경우도 있다. 이런 경우에는 스칼라 2.8에서 _이전 경고(migration warning)_ 가 발생한다. 코드를 어떻게 바꿔야 할지 참고하기 위해 사용금지와 이전 경고를 모두 다 보고 싶다면 `-deprecation`와 `-Xmigration` 플래그를 `scalac`에 전달하라(`-Xmigration`는 확장 옵션이기 때문에 `X`로 시작한다는 점에 유의하라). `scala` REPL에 같은 플래그를 전달하면 대화식 세션에서도 다음과 같이 경고 메시지를 볼 수 있다.

    >scala -deprecation -Xmigration
    Welcome to Scala version 2.8.0.final
    Type in expressions to have them evaluated.
    Type :help for more information.
    scala> val xs = List((1, 2), (3, 4))
    xs: List[(Int, Int)] = List((1,2), (3,4))
    scala> List.unzip(xs)
    <console>:7: warning: method unzip in object List is deprecated: use xs.unzip instead of List.unzip(xs)
           List.unzip(xs)
                ^
    res0: (List[Int], List[Int]) = (List(1, 3),List(2, 4))
    scala> xs.unzip
    res1: (List[Int], List[Int]) = (List(1, 3),List(2, 4))
    scala> val m = xs.toMap
    m: scala.collection.immutable.Map[Int,Int] = Map((1,2), (3,4))
    scala> m.keys
    <console>:8: warning: method keys in trait MapLike has changed semantics:
    As of 2.8, keys returns Iterable[A] rather than Iterator[A].
           m.keys
             ^
    res2: Iterable[Int] = Set(1, 3)

예전 라이브러리에 있던 기능 전체가 다 변경되어서 금지 경고를 표시할 수 없는 경우로는 다음 두 가지가 있다.

1. 예전의 `scala.collection.jcl` 패키지는 삭제되었다. 이 패키지는 자바 컬렉션 라이브러리 설계를 스칼라에서 일부 흉내내기 위한 시도였다. 하지만, 그로 인해 많은 대칭성이 깨져 버렸다. 대부분의 경우 자바 컬렉션을 원하는 프로그래머들은 `jcl`을 거치지 않고 바로 `java.util`을 사용했다. 스칼라 2.8에서는 `jcl` 패키지를 대치하는 [자바변환(JavaConversions)]({{ site.baseurl }}/overviews/collections/conversions-between-java-and-scala-collections.md) 객체를 사용한 스칼라와 자바 컬렉션간의 자동 변환이 제공된다.
2. 프로젝션(Projection)은 더 일반화되고 다듬어져서 이제 뷰로 제공된다. 프로젝션을 사용하는 경우가 그리 많지 았았던 것 같으므로, 이 변경에 영향받는 경우도 적을 것이다.

따라서 여러분이 작성한 코드가 `jcl`이나 프로젝션을 사용한다면 코드를 조금은 다시 작성할 필요가 있다.

