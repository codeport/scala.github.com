---
layout: overview-large
title: 문자열

disqus: true

partof: collections
num: 11
language: ko
---

배열처럼, 문자열이 직접적으로 시퀀스인 것은 아니지만 시퀀스로 변환될 수 있으며 모든 시퀀스 연산을 지원한다. 다음은 문자열에서 호출할 수 있는 연산의 예를 보여준다. 

    scala> val str = "hello"
    str: java.lang.String = hello
    scala> str.reverse
    res6: String = olleh
    scala> str.map(_.toUpper)
    res7: String = HELLO
    scala> str drop 3 
    res8: String = lo
    scala> str slice (1, 4)
    res9: String = ell
    scala> val s: Seq[Char] = str
    s: Seq[Char] = WrappedString(h, e, l, l, o)

이러한 연산들은 두 가지의 암시적 변환에 의해 지원된다. 먼저 낮은 우선 순위의 변환은 `String`을 `immutable.IndexedSeq`의 자식 클래스인 `WrappedString`로 바꾼다. 이 변환은 위에 있는 마지막 줄처럼, 문자열이 Seq로 변환되는 곳에서 도입된다. 높은 우선 순위의 변환은 문자열을 모든 불변 시퀀스의 메소드를 문자열에 추가한 `StringOps` 객체로 변환시킨다. 이 변환은 예제에 나와있듯이 `reverse`, `map`, `drop`, 그리고 `slice` 메소드가 불릴 때 암시적으로 삽입된다.