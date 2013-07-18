---
layout: overview-large
title: 타입 매크로

disqus: true

partof: macros
num: 3
outof: 7
language: ko
---
<a href="/overviews/macros/paradise.html"><span class="label important" style="float: right;">매크로 낙원(MACRO PARADISE)</span></a>

**유진 부르마코(Eugene Burmako)**

타입 매크로는 출시 전(pre-relase) 기능으로서 일명 매크로 낙원이라 불리는 공식 스칼라 저장소의 실험적 브랜치에 포함되어 있다. [매크로 낙원](/overviews/macros/paradise.html) 페이지의 안내에 따라 야간 빌드(nightly builds)를 내려받고 사용해 보자.

## 간단한 소개

Def 매크로가 컴파일러로 하여금 특정 메소드 호출 시 사용자 정의 함수를 실행하도록 하는 것처럼, 타입 매크로를 이용해 특정 타입을 사용할 때 컴파일러를 후킹할 수 있다. 아래 예제는 `H2Db` 매크로의 정의와 사용법을 보여주는데, 이 매크로는 데이터베이스 테이블을 표현하면서 간단한 CRUD기능을 갖는 케이스 클래스를 생성한다.

    type H2Db(url: String) = macro impl

    object Db extends H2Db("coffees")

    val brazilian = Db.Coffees.insert("Brazilian", 99, 0)
    Db.Coffees.update(brazilian.copy(price = 10))
    println(Db.Coffees.all)

`H2Db` 타입 매크로의 전체 소스 코드는 [깃허브(Github)](https://github.com/xeno-by/typemacros-h2db)에서 볼 수 있으며, 이 안내서는 중요한 부분만을 다룬다. 먼저 매크로는 컴파일 시점에 데이터베이스에 접속하여 정적 타입 데이터베이스 래퍼를 만들어낸 다음, (트리 생성은 [리플렉션 개요(the reflection overview)](http://docs.scala-lang.org/overviews/reflection/overview.html)에서 설명한다) <span class="label success">NEW</span> `c.introduceTopLevel` API ([Scaladoc](https://scala-webapps.epfl.ch/jenkins/view/misc/job/macro-paradise-nightly-main/ws/dists/latest/doc/scala-devel-docs/api/index.html#scala.reflect.macros.Synthetics))를 사용하여 생성된 래퍼를 컴파일러의 최상단 정의 목록에 추가한 뒤 마지막으로 만들어진 클래스의 상위 생성자를 호출하는 `Apply`노드를 반환한다. `c.Expr[T]` 형태로 확장되는 def 매크로와 달리 타입 매크로는 `c.Tree`로 확장되는데, 이는 `Expr`이 식(term)을 표현하는 것과 달리 타입 매크로는 타입으로 확장되기 때문이다.

    type H2Db(url: String) = macro impl

    def impl(c: Context)(url: c.Expr[String]): c.Tree = {
      val name = c.freshName(c.enclosingImpl.name).toTypeName
      val clazz = ClassDef(..., Template(..., generateCode()))
      c.introduceTopLevel(c.enclosingPackage.pid.toString, clazz)
      val classRef = Select(c.enclosingPackage.pid, name)
      Apply(classRef, List(Literal(Constant(c.eval(url)))))
    }

    object Db extends H2Db("coffees")
    // 다음과 같음: object Db extends Db$1("coffees")

합성 클래스(synthetic class)를 생성하여 이를 참조하는 대신, 타입 매크로는 `Template` 트리를 반환하여 원본을 변경할 수 있다. Scalac 내부에서는 클래스와 오브젝트 선언 모두 `Template` 트리에 대한 얇은 래퍼로 표현되므로, 템플릿으로의 확장을 통해 클래스나 오브젝트의 전체 내용을 다시 쓸 수도 있는 것이다. [깃허브](https://github.com/xeno-by/typemacros-lifter)에서 이 기법을 구현한 완전한 예제를 볼 수 있다.

    type H2Db(url: String) = macro impl

    def impl(c: Context)(url: c.Expr[String]): c.Tree = {
      val Template(_, _, existingCode) = c.enclosingTemplate
      Template(..., existingCode ++ generateCode())
    }

    object Db extends H2Db("coffees")
    // 다음과 같음: object Db {
    //   <existing code>
    //   <generated code>
    // }

## 상세

타입 매크로는 def 매크로와 타입 멤버의 복합적 성격을 띤다. 한편으로는 메소드처럼 정의되면서 (값 매개변수, 타입 매개변수를 컨텍스트 지정과 함께 사용할 수 있는 등등), 다른 한편으로는 타입의 네임스페이스에 속하여 타입과 같은 방법으로만 쓸 수 있으며 ([깃허브](https://github.com/scalamacros/kepler/blob/paradise/macros/test/files/run/macro-typemacros-used-in-funny-places-a/Test_2.scala)에 완전한 예제가 있다), 타입과 다른 타입 매크로만을 재정의할 수 있다.

| 기능                                   | Def 매크로 | 타입 매크로 | 타입 멤버 |
|----------------------------------------|------------|-------------|-----------|
| 정의와 구현으로 구분됨                 | 예         | 예          | 아니오    |
| 값 매개변수를 가질 수 있음             | 예         | 예          | 아니오    |
| 타입 매개변수를 가질 수 있음           | 예         | 예          | 예        |
| 매개변수에 가변성 애노테이션 사용 가능 | 아니오     | 아니오      | 예        |
| 매개변수에 컨텍스트 지정 가능          | 예         | 예          | 아니오    |
| 오버로드 가능                          | 예         | 예          | 아니오    |
| 상속 가능                              | 예         | 예          | 예        |
| 재정의하거나 재정의될 수 있음          | 예         | 예          | 예        |

스칼라 프로그램에서 타입 매크로는 타입(type), 응용 타입(applied type), 부모 타입(parent type), 인스턴스 생성(new)과 애노테이션(annotation)의 다섯 가지 역할(role) 중 하나를 수행한다. 매크로의 역할에 따라 가능한 확장 형태가 달라지며, 이는 <span class="label success">NEW</span> `c.macroRole` API ([Scaladoc](https://scala-webapps.epfl.ch/jenkins/view/misc/job/macro-paradise-nightly-main/ws/dists/latest/doc/scala-devel-docs/api/index.html#scala.reflect.macros.Enclosures))를 통해 확인할 수 있다.

| 역할          | 예제                                            | 클래스 | 비 클래스 | 값 적용 | 템플릿 |
|---------------|-------------------------------------------------|--------|-----------|---------|--------|
| 타입          | `def x: TM(2)(3) = ???`                         | 가능   | 가능      | 불가능  | 불가능 |
| 응용 타입     | `class C[T: TM(2)(3)]`                          | 가능   | 가능      | 불가능  | 불가능 |
| 부모 타입     | `class C extends TM(2)(3)`<br/>`new TM(2)(3){}` | 가능   | 불가능    | 가능    | 가능   |
| 인스턴스 생성 | `new TM(2)(3)`                                  | 가능   | 불가능    | 가능    | 불가능 |
| 애노테이션    | `@TM(2)(3) class C`                             | 가능   | 불가능    | 가능    | 불가능 |

간단히 말해, 타입 매크로를 확장하면 매크로가 사용된 부분을 해당 매크로가 반환하는 트리로 교체한다. 매크로 확장의 유효성을 판단하려면 머리 속에서 매크로가 사용된 부분을 확장된 형태로 치환한 후 결과가 올바른지 확인해보면 된다.

예를 들어, `class C extends TM(2)(3)`에 사용된 타입 매크로 `TM(2)(3)`는 `Apply(Ident(newTypeName("B")), List(Literal(Constant(2))))`로 확장될 수 있는데, 이는 `class C extends B(2)`가 유효하기 때문이다. 그러나 `def x: TM(2)(3) = ???`처럼 타입으로 사용된 경우에는 `def x: B(2) = ???`가 되므로 불가능하다. (`B`는 타입 매크로가 아닌 경우에 한한다. 만약 타입 매크로라면 `B`는 재귀적으로 확장될 것이고 그 결과로 프로그램의 유효성을 판단할 것이다.)

## 팁과 트릭

### 클래스와 오브젝트 생성

타입 매크로를 작성하다 보면 [스택오버플로우(StackOverflow)](http://stackoverflow.com/questions/13795490/how-to-use-type-calculated-in-scala-macro-in-a-reify-clause)의 예처럼 `reify`의 사용이 불가능한 경우를 만나게 될 것이다. 이럴 때는 직접 트리를 작성하는 대신 매크로 낙원의 또다른 실험적 기능인 [준인용구(quasiquotes)](/overviews/macros/quasiquotes.html) 사용을 고려해 보라.
