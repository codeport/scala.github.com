---
layout: overview-large
title: Def 매크로

disqus: true

partof: macros
num: 1
outof: 7
language: ko
---
<span class="label warning" style="float: right;">EXPERIMENTAL</span>

**유진 부르마코(Eugene Burmako)**

## 동기 

컴파일 시점 메타프로그래밍은 다음과 같은 프로그래밍 테크닉을 가능케 해주는 귀중한 도구이다.

* 언어 가상화(원래의 프로그래밍 언어의 의미를 재정의/중복정의할 수 있어서 DSL을 한층 더 깊게 내장할수 있다)
* 프로그램의 대상화(reification: 프로그램이 자기 코드를 스스로 검사할 수 있는 방법을 제공한다)
* 자기 최적화(프로그램 대상화를 기반으로 도메인에 특화된 코드를 스스로 적용할 수 있다)
* 알고리즘에 의한 프로그램 구성(원래의 프로그래밍 언어만 가지고는 작성하는 작업이 쉽지 않던 부분을 자동화할 수 있다)

이 소갯글에서는 스칼라를 위한 매크로 시스템을 알려줄 것이다. 매크로 기능을 사용하면 프로그래머가 매크로 정의를 작성할 수 있다. 이런 매크로 정의는 컴파일 도중에 컴파일러가 투명하게(즉, 영향받지 않고) 로드할 수 있고 실행할 수 있다. 이를 통해 스칼라에서 컴파일 시점 메타프로그래밍이 가능해진다.

## 간단한 소개

다음은 프로토타임과 마찬가지인 매크로 정의이다.

    def m(x: T): R = macro implRef

처음 보면 매크로 정의는 일반적인 함수 정의와 같아 보인다. 차이는 몸체가 조건부 키워드 `macro`로 시작하고, 그 뒤에 매크로를 구현하는 정적 메소드에 대한 전체 경로를 포함한 이름이 들어간다는 점 뿐이다.

타입체크를 하는 동안 컴파일러가 매크로 적용 `m(args)`를 발견하면, 이를 그 구현 메소드를 호출해서 확장한다. 이때 구현 메소드에 매크로의 인자로 들어간 식의 추상 구문 트리(abstract syntax tree, 메모리상에서 식을 파싱한 결과를 트리구조로 표현한 것)를 넘겨준다. 매크로 구현을 호출하면 돌려받는 결과는 또 다른 추상 구문 트리이다. 컴파일러는 이 트리를 매크로 호출 부분 대신 끼워 넣고, 타입 체크를 계속 진행하게 된다.

다음 코드는 `Asserts.assertImpl`를 구현으로 사용하는 매크로 정의를 보여준다(`Asserts.assertImpl`의 구현은 더 아래에 있다).

    def assert(cond: Boolean, msg: Any) = macro Asserts.assertImpl

`assert(x < 10, "limit exceeded")`와 같은 매크로 호출은 컴파일시 다음과 같은 매크로 구현체 호출을 불러 일으킨다.

    assertImpl(c)(<[ x < 10 ]>, <[ “limit exceeded” ]>)

여기서 `c`는 매크로 호출 지점에서 컴파일러가 수집해 가지고 있는 정보를 포함하고 있는 컨택스트이다. 또 다른 두 인자는 각각 두 식 `x < 10`, `limit exceeded`을 표현하는 추상 구현 트리이다.

이 문서에서는 `<[ expr ]>`라는 표기를 사용해 `expr`이라는 식을 표현하는 추상 구문 트리를 표시할 것이다. 이 표기방식 자체는 우리가 제안하는 스칼라 언어 확장(역주: 이 문서는 스칼라에 매크로를 도입하자는 제안서였던 것을 수정한 것이다)에는 대응되는 부분이 없다. 실제로 구문 트리는 `scala.reflect.api.Trees` 트레잇에 있는 타입을 사용해 구현되어야 할 것이고, 위의 두 식은 아마도 다음과 비슷하게 만들어질 수 있을 것이다.

    Literal(Constant("limit exceeded"))

    Apply(
      Select(Ident(newTermName("x")), newTermName("$less"),
      List(Literal(Constant(10)))))

다음은 `assert` 매크로를 실제로 구현한 예이다.

    import scala.reflect.macros.Context
    import scala.language.experimental.macros

    object Asserts {
      def raise(msg: Any) = throw new AssertionError(msg)
      def assertImpl(c: Context)
        (cond: c.Expr[Boolean], msg: c.Expr[Any]) : c.Expr[Unit] =
       if (assertionsEnabled)
          <[ if (!cond) raise(msg) ]>
       else
          <[ () ]>
    }

앞에서 본 바와 같이 매크로 구현은 몇가지 인자를 받는다. 첫번째로 오는 것은 `scala.reflect.macros.Context` 타입의 값이다. 그 다음에 매크로 정의 매개변수에 있는 인자들과 같은 매개변수 목록이 온다. 하지만 원래의 매크로 매개변수가 `T` 타입이었다면, 매크로 구현에서의 매개변수는 `c.Expr[T]`가 되어야 한다. `Expr[T]`는 `Context`에 정의되어 있으며, 타입 `T`의 추상 구문 트리를 감싸고 있다. `assertImpl` 매크로 구현의 결과 타입은 구문 트리를 감싼 `c.Expr[Unit]` 타입이 된다.

한가지 더 일러둘 것은 매크로가 아직 실험적인 단계이기 때문에, 명시적으로 활성화해야만 사용할 수 있다는 점이다. `import scala.language.experimental.macros`를 사용하는 파일 앞에 붙여주거나, 컴파일시 `-language:experimental.macros`를 (컴파일러에 스위치를 지정해서) 사용해야 한다.

### 제네릭 매크로

매크로 정의와 구현은 제네릭할 수도 있다. 매크로 구현에 타입 매개변수가 있다면, 실제 타입 인자를 명시적으로 매크로 정의 몸체에 추가해야만 한다. 매크로 구현에서 타입 매개변수를 `WeakTypeTag(약한 타입 태그)` 컨텍스트 바운드를 통해 제한할 수도 있다. 이렇게 하면 매크로 호출 위치에서 인스턴스화된 실제 타입 매개변수를 표현하는 타입 태그가 매크로 확장시 전달된다.

다음 코드 조각은 `QImpl.map` 매크로 구현을 참조하는 매크로 정의 `Queryable.map`을 보여준다.

    class Queryable[T] {
     def map[U](p: T => U): Queryable[U] = macro QImpl.map[T, U]
    }

    object QImpl {
     def map[T: c.WeakTypeTag, U: c.WeakTypeTag]
            (c: Context)
            (p: c.Expr[T => U]): c.Expr[Queryable[U]] = ...
    }

값 `q`가 `Queryable[String]` 타입이라면 아래와 같이 매크로를 호출하면,

    q.map[Int](s => s.length)

다음과 같은 리플렉션이 들어간 

    QImpl.map(c)(<[ s => s.length ]>)
       (implicitly[WeakTypeTag[String]], implicitly[WeakTypeTag[Int]])

## 완전한 예제

이번 절에서는 `printf` 매크로를 처음부터 끝까지 만들어 볼 것이다. 이 매크로는 포맷 문자열을 컴파일시 검증해서 적용한다. 
설명의 편의를 위해 콘솔 스칼라 컴파일러를 사용할 것이다. 하지만, 메이븐이나 SBT에서도 다음에 설명하는 방식대로 매크로를 사용 할 수 있다.

매크로 작성은 매크로 정의를 쓰는 것으로부터 시작된다. 매크로 정의는 매크로의 외관(facade) 역할을 한다. 매크로 정의 자체는 멋질것 하나 없는 평범한 함수이다. 하지만, 그 몸체는 구현에 대한 참조일 뿐 아무것도 아니다. 앞에서 이야기한 바와 같이 매크로를 정의하기 위해서는 `scala.language.experimental.macros`를 임포트하거나 컴파일러에 특별한 스위치 `-language:experimental.macros`를 지정해야 한다.

    import scala.language.experimental.macros
    def printf(format: String, params: Any*): Unit = macro printf_impl

매크로 구현은 그 구현을 사용하는 매크로 정의와 대응되어야만 한다(보통은 오직 하나의 정의만 존재하지만, 여러개가 있을 수도 있다). 
줄여 말하면, 매크로 정의에 있는 매개변수의 타입 `T`는 구현의 시그니쳐에서는 타입 `c.Expr[T]`가 되어야 한다. 전체 규칙은 꽤 복잡하지만, 문제가 되지는 않는다. 왜냐하면 오류가 있다면 컴파일러가 원하는 타입의 시그니쳐를 오류 메시지에 표시해주기 때문이다.

    import scala.reflect.macros.Context
    def printf_impl(c: Context)(format: c.Expr[String], params: c.Expr[Any]*): c.Expr[Unit] = ...

컴파일러 API는 `scala.reflect.macros.Context`로 노출되어 있다. 가장 중요한 부분인 리플렉션(reflection) API는 `c.universe`로 억세스할 수 있다. 관례적으로 `c.universe._`를 임포트하곤 한다. 왜냐하면 일상적으로 활용되는 대부분의 함수와 타입이 그 안에 정의되어 있기 때문이다.

    import c.universe._

매크로는 우선 넘어온 형식 문자열을 분석해야 한다. 매크로 구현 함수는 컴파일시점에 실행되기 때문에 값에 대해 동작하는 대신 구문 트리에 대해 동작한다. 
이는 `printf` 매크로가 받는 형식 문자열이 `java.lang.String` 타입의 객체가 아니고, 컴파일시점의 리터럴(literal)이라는 것을 의미한다. 
또한, 이로 인해 아래의 코드는 `printf(get_format(), ...)`에 대해서는 동작할 수 없다. 왜냐하면 이런 경우에 `format`이 문자열 리터럴이 아니고, 함수 호출을 표현하는 AST(추상 구문 트리)가 되기 때문이다.

    val Literal(Constant(s_format: String)) = format.tree

전형적인 매크로는(여기서 다루고 있는 매크로도 예외가 아니다) 스칼라 코드를 표현하는 AST를 만들어내기 위해 사용된다. 스칼라 코드가 생성되는 과정에 대해 더 잘 알고 싶은 독자는 [리플렉션 개요(the overview of reflection)](http://docs.scala-lang.org/overviews/reflection/overview.html)를 참조하라. 아래 코드는 AST를 만듦과 동시에 또한 타입 정보도 조작한다. 
`Int`와 `String` 타입을 어떻게 각각 담아두는지를 살펴보라. 
위에 링크한 리플렉션에 대한 글을 보면 타입을 조작하는 법에 대해서도 자세히 다루고 있다.
코드 생성시 마지막 단계는 만들어진 코드를 한 `Block`으로 묶는 것이다. AST를 만드는 간편한 방법인 `reify` 함수를 호출한단 사실에 유의하라.

    val evals = ListBuffer[ValDef]()
    def precompute(value: Tree, tpe: Type): Ident = {
      val freshName = newTermName(c.fresh("eval$"))
      evals += ValDef(Modifiers(), freshName, TypeTree(tpe), value)
      Ident(freshName)
    }

    val paramsStack = Stack[Tree]((params map (_.tree)): _*)
    val refs = s_format.split("(?<=%[\\w%])|(?=%[\\w%])") map {
      case "%d" => precompute(paramsStack.pop, typeOf[Int])
      case "%s" => precompute(paramsStack.pop, typeOf[String])
      case "%%" => Literal(Constant("%"))
      case part => Literal(Constant(part))
    }

    val stats = evals ++ refs.map(ref => reify(print(c.Expr[Any](ref).splice)).tree)
    c.Expr[Unit](Block(stats.toList, Literal(Constant(()))))

아래 코드는 `printf` 매크로의 전체 구현이다. 
예제를 따라하고 싶다면 빈 디렉터리를 만들고 `Macros.scala`라는 파일에 아래 코드를 복사해 넣도록 하라.

    import scala.reflect.macros.Context
    import scala.collection.mutable.{ListBuffer, Stack}

    object Macros {
      def printf(format: String, params: Any*): Unit = macro printf_impl

      def printf_impl(c: Context)(format: c.Expr[String], params: c.Expr[Any]*): c.Expr[Unit] = {
        import c.universe._
        val Literal(Constant(s_format: String)) = format.tree

        val evals = ListBuffer[ValDef]()
        def precompute(value: Tree, tpe: Type): Ident = {
          val freshName = newTermName(c.fresh("eval$"))
          evals += ValDef(Modifiers(), freshName, TypeTree(tpe), value)
          Ident(freshName)
        }

        val paramsStack = Stack[Tree]((params map (_.tree)): _*)
        val refs = s_format.split("(?<=%[\\w%])|(?=%[\\w%])") map {
          case "%d" => precompute(paramsStack.pop, typeOf[Int])
          case "%s" => precompute(paramsStack.pop, typeOf[String])
          case "%%" => Literal(Constant("%"))
          case part => Literal(Constant(part))
        }

        val stats = evals ++ refs.map(ref => reify(print(c.Expr[Any](ref).splice)).tree)
        c.Expr[Unit](Block(stats.toList, Literal(Constant(()))))
      }
    }

`printf` 매크로를 사용하기 위해 같은 디렉터리에 `Test.scala`라는 파일을 만들고 다음 코드를 입력하라.
매크로를 사용하는 것이 함수 호출과 마찬가지로 간단하다는 사실을 확인하라. `scala.language.experimental.macros`를 임포트할 필요는 없다.

    object Test extends App {
      import Macros._
      printf("hello %s!", "world")
    }

매크로 사용에 있어 중요한 것 중 하나는 컴파일되는 시점의 분리이다. 매크로를 확장하기 위해 컴파일러는 매크로 구현을 실행 가능한 형태로 가지고 있어야 한다. 따라서 매크로 구현은 매크로를 사용하는 쪽을 컴파일하기 전에 컴파일되어야 한다. 그렇지 않으면 다음과 같은 오류가 발생한다.

    ~/Projects/Kepler/sandbox$ scalac -language:experimental.macros Macros.scala Test.scala
    Test.scala:3: error: macro implementation not found: printf (the most common reason for that is that
    you cannot use macro implementations in the same compilation run that defines them)
    pointing to the output of the first phase
      printf("hello %s!", "world")
            ^
    one error found

    ~/Projects/Kepler/sandbox$ scalac Macros.scala && scalac Test.scala && scala Test
    hello world!

## 팁과 트릭

### 명령행 스칼라 컴파일러에서 매크로 활용하기

이 시나리오는 앞에서 이미 다룬 것이다. 요약하면, 매크로를 먼저 `scalac`로 컴파일하고, 그 후 매크로를 적용하는 부분을 컴파일하라. 그렇게 하면 문제가 없어야 한다. REPL은 더 사용하기 편하다. 왜냐하면 REPL은 입력되는 줄을 각각 컴파일해 실행하기 때문이다. 따라서 매크로를 정의하고 바로 이를 적용해볼 수 있다.

### 메이븐이나 SBT에서 매크로 사용하기

이 가이드에서 제공한 방식은 가장 단순한 명령행 컴파일을 사용하지만, 메이븐이나 SBT와 같은 빌드도구에서도 매크로를 활용할 수 있다. [https://github.com/scalamacros/sbt-example](https://github.com/scalamacros/sbt-example)이나 [https://github.com/scalamacros/maven-example](https://github.com/scalamacros/maven-example)에서 완전한 예제를 볼 수 있다. 
간략하게 말하자면 다음 두가지를 알아두어야 한다.
* 매크로를 사용하기 위해서는 라이브러리 의존성에 scala-reflect.jar가 필요하다.
* 별도 컴파일 제약을 지키려면 매크로를 별도의 프로젝트에 위치시켜서 먼저 컴파일해야 한다.

### 스칼라 IDE나 인텔리J IDEA에서 사용하기

스칼라 IDE나 인텔리J IDEA 모두에서 매크로가 잘 작동한다고 알려져 있다. 다만, 매크로를 별도의 프로젝트로 분리해야 한다.

### 매크로 디버깅하기

매크로를 디버깅하는 것(즉, 매크로 확장 로직을 디버깅하는 것)은 아주 간단하다. 매크로가 컴파일러 안에서 확장되기 때문에, 디버깅을 하려면 컴파일러를 디버거 안에서 실행시키면 된다. 이를 위해 1) 스칼라 홈 디렉터리 아래 lib의 모든(!) 라이브러리를 디버깅시 클래스패스에 추가하고, 2) `scala.tools.nsc.Main`를 시작점으로 설정하고, 3) 컴파일러의 명령행 인자를 `-cp <매크로 클래스들에 대한 경로> Test.scala`로 하라(`Test.scala`는 디버깅하기 위해 만든 매크로 호출이 들어있는 프로그램이다). 이렇게 하고 나서 매크로 구현 부분에 중단점을 설정하고 컴파일러를 디버거에서 실행시키면 된다.

특별한 도구가 필요할 때는 매크로 확장의 결과(즉, 매크로가 만들어낸 코드)를 디버깅할 필요가 있을 때이다. 이 코드는 직접 작성된 것이 아니기 때문에 그 안에 중단점을 걸 수가 없다. 또한 소스상 한 단계씩 실행(single stepping)도 불가능하다. 언젠가는 스칼라 IDE나 인텔리J IDEA 팀에 의해 디버거에 이런 부분에 대한 지원이 추가되겠지만, 현재 사용 가능한 유일한 대안은 진단 메시지를 추가하는 것 뿐이다. `-Ymacro-debug-lite` (아래 설명함)를 추가하면 매크로가 내어 놓는 코드를 출력해 준다. 생성된 코드의 실행을 추적하기 위해서는 `println`을 만들어지는 코드에 추가하라.

### 만들어진 코드 살펴보기

`-Ymacro-debug-lite` 옵션을 사용해 매크로 확장에 의해 생긴 코드의 의사 스칼라 코드나 원 AST 표현을 볼 수 있다. 두 방식 모두 서로 다른 장점이 있다. 전자는 표면적인 검사에 유용하고, 후자는 미세한 디버깅시 필수적이다.

    ~/Projects/Kepler/sandbox$ scalac -Ymacro-debug-lite Test.scala
    typechecking macro expansion Macros.printf("hello %s!", "world") at
    source-C:/Projects/Kepler/sandbox\Test.scala,line-3,offset=52
    {
      val eval$1: String = "world";
      scala.this.Predef.print("hello ");
      scala.this.Predef.print(eval$1);
      scala.this.Predef.print("!");
      ()
    }
    Block(List(
    ValDef(Modifiers(), newTermName("eval$1"), TypeTree().setType(String), Literal(Constant("world"))),
    Apply(
      Select(Select(This(newTypeName("scala")), newTermName("Predef")), newTermName("print")),
      List(Literal(Constant("hello")))),
    Apply(
      Select(Select(This(newTypeName("scala")), newTermName("Predef")), newTermName("print")),
      List(Ident(newTermName("eval$1")))),
    Apply(
      Select(Select(This(newTypeName("scala")), newTermName("Predef")), newTermName("print")),
      List(Literal(Constant("!"))))),
    Literal(Constant(())))

### 처리되지 않는 예외를 던지는 매크로

매크로가 처리되지 않는 예외를 던지면 어떤 일이 벌어질까? 예를 들어 `printf` 매크로에 잘못된 입력을 주는 경우를 생각해 보자. 
다음 출력에서 볼 수 있듯 대단한 일이 벌어지는 것은 아니다. 컴파일러는 잘못된 매크로 사용을 막고, 적절한 스택 추적 메시지를 출력하고, 오류를 보고한다.

    ~/Projects/Kepler/sandbox$ scala
    Welcome to Scala version 2.10.0-20120428-232041-e6d5d22d28 (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_25).
    Type in expressions to have them evaluated.
    Type :help for more information.

    scala> import Macros._
    import Macros._

    scala> printf("hello %s!")
    <console>:11: error: exception during macro expansion:
    java.util.NoSuchElementException: head of empty list
            at scala.collection.immutable.Nil$.head(List.scala:318)
            at scala.collection.immutable.Nil$.head(List.scala:315)
            at scala.collection.mutable.Stack.pop(Stack.scala:140)
            at Macros$$anonfun$1.apply(Macros.scala:49)
            at Macros$$anonfun$1.apply(Macros.scala:47)
            at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:237)
            at scala.collection.TraversableLike$$anonfun$map$1.apply(TraversableLike.scala:237)
            at scala.collection.IndexedSeqOptimized$class.foreach(IndexedSeqOptimized.scala:34)
            at scala.collection.mutable.ArrayOps.foreach(ArrayOps.scala:39)
            at scala.collection.TraversableLike$class.map(TraversableLike.scala:237)
            at scala.collection.mutable.ArrayOps.map(ArrayOps.scala:39)
            at Macros$.printf_impl(Macros.scala:47)

                  printf("hello %s!")
                        ^

### 경고와 오류 보고하기

사용자와 상호작용하는 표준적인 방법은 `scala.reflect.macros.FrontEnds`를 사용하는 것이다. 
`c.error`는 컴파일 오류를 보고하고 `c.info`는 경고메시지를 출력하며 `c.abort`는 오류를 보고한 후 매크로 실행을 종료한다.

    scala> def impl(c: Context) =
      c.abort(c.enclosingPosition, "macro has reported an error")
    impl: (c: scala.reflect.macros.Context)Nothing

    scala> def test = macro impl
    defined term macro test: Any

    scala> test
    <console>:32: error: macro has reported an error
                  test
                  ^

[SI-6910](https://issues.scala-lang.org/browse/SI-6910)에 있는 바와 같이 현재로써는 오류 보고시 한 지점에서 여러 오류를 보고할 수 없다는 점에 유의하라. 따라서 각 위치에서 발생한 최초의 오류만 보고할 수 있으며 나머지는 사라진다(또한 오류는 같은 지점에서 발생한 경고보다 우선한다. 경고가 더 먼저 발생했다고 해도 그렇다).


### 더 큰 매크로 작성하기

매크로 구현 코드가 점점 커져서 구현 메소드의 몸체의 모듈화를 보장할 수준 이상이 되어버린다면, 컨택스트 매개변수를 짊어지고 다닐것을 고려해 봐야 한다. 왜냐하면 관심의 대상이 되는 대부분의 요소는 컨텍스트에 경로-의존적이기 때문이다.

접근 방식 중 하나는 `Context` 타입의 매개변수를 받는 클래스를 만들어서 매크로 구현을 그 클래스 안의 여러 메소드로 나누어 두는 것이다. 이런 방법은 자연스럽고 간단하지만, 제대로 구현하는 것은 어렵다. 다음은 전형적인 컴파일 오류이다.

    scala> class Helper(val c: Context) {
         | def generate: c.Tree = ???
         | }
    defined class Helper

    scala> def impl(c: Context): c.Expr[Unit] = {
         | val helper = new Helper(c)
         | c.Expr(helper.generate)
         | }
    <console>:32: error: type mismatch;
     found   : helper.c.Tree
        (which expands to)  helper.c.universe.Tree
     required: c.Tree
        (which expands to)  c.universe.Tree
           c.Expr(helper.generate)
                         ^

여기서는 경로-의존적인 타입 불일치가 일어났다. 스칼라 컴파일러는 `impl`의 `c`가 `Helper`의 `c`와 같음을 인지하지 못한다. 심지어 원래의 `c`를 사용해 `helper`가 만들어졌음에도 말이다.

다행스럽게도 컴파일러에게 살짝 힌트를 주어 어떤 일이 벌어진 것인지 쉽게 알아채도록 만들 수 있다. 한가지 방법은 상세타입(refinement type, 원 타입에 일부 제약을 추가한 하위 타입)을 활용하는 것이다(아래 예는 이 아이디어를 활용하는 가장 단순한 예이다. 예를 들어 명시적 인스턴스화를 없애고 호출을 간단하게 하기 위해 `Context`에서 `Helper`가는 묵시적 변환을 추가할 수도 있을 것이다).

    scala> abstract class Helper {
         | val c: Context
         | def generate: c.Tree = ???
         | }
    defined class Helper

    scala> def impl(c1: Context): c1.Expr[Unit] = {
         | val helper = new { val c: c1.type = c1 } with Helper
         | c1.Expr(helper.generate)
         | }
    impl: (c1: scala.reflect.macros.Context)c1.Expr[Unit]

또 다른 방법은 컨텍스트의 정체를 명시적인 타입 매개변수로 넘기는 것이다. 아래 예에서는 `Helper`를 만들 때 `c.type`을 지정해서 `Helper.c`가 사실은 원래의 `c`와 같음을 알린다. 스칼라의 타입 추론이 자동으로 이를 파악할 수 없으므로 약간 도와주는 것이다.

    scala> class Helper[C <: Context](val c: C) {
         | def generate: c.Tree = ???
         | }
    defined class Helper

    scala> def impl(c: Context): c.Expr[Unit] = {
         | val helper = new Helper[c.type](c)
         | c.Expr(helper.generate)
         | }
    impl: (c: scala.reflect.macros.Context)c.Expr[Unit]

## 추가 예제

스칼라 매크로는 아직 도입 초기 단계이다. 커뮤니티의 활발한 활동으로 인해 빠르게 프로토타입을 만들 수 있었고, 이제는 참고할 수 있는 코드가 존재한다. 최신 상태에 대해서는 [http://scalamacros.org/news/2012/11/05/status-update.html](http://scalamacros.org/news/2012/11/05/status-update.html)를 참조하라.

또한 아담 와스키(Adam Warski)가 작성한 짧은 자습서 [http://www.warski.org/blog/2012/12/starting-with-scala-macros-a-short-tutorial](http://www.warski.org/blog/2012/12/starting-with-scala-macros-a-short-tutorial)를 추천한다. 그 자습서 안에는 디버깅용 출력 매크로를 작성하는 방법을 한 단계씩 설명하며, SBT 프로젝트 설정과 `reify`나 `splice`의 사용법, 그리고 AST를 손으로 합치는 방법에 대해서도 설명한다.