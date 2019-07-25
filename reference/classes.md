---
type: doc
layout: reference
category: "Syntax"
title: "Classes and Inheritance"
related:
    - functions.md
    - nested-classes.md
    - interfaces.md
---

# 클래스와 상속

## 클래스

코틀린 클래스는 *class*{: .keyword }: 키워드로 선언합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Invoice { ... }
```

</div>

클래서 선언은 클래스 이름, 클래스 헤더(파라미터를 서술하고 주요 생성자 등)와 중괄호({})로 감싸진 본문으로 이뤄져 있습니다. 헤더와 본문은 필수가 아니기 때문에 본문이 없으면 {} 부분은 생략이 가능합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Empty
```

</div>

### 생성자

코틀린 클래스는 **주생성자(primary constructor)**  **부생성자(secondary constructors)** 를 가질 수 있습니다.
주생성자는 클래스 헤더의 한 부분이며, 클래스 이름(과 생략 가능한 인자 타입들) 다음에 나옵니다. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Person constructor(firstName: String) { ... }
```

</div>


만약 주 생성자에 어노테이션이나 접근 제한자가 없으면 *constructor*{: .keyword } 키워드는 생략 가능합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Person(firstName: String) { ... }
```

</div>

주 생성자에는 코드를 기술할 수 없습니다. 초기화 코드는 *init*{: .keyword } 이라는 키워드로 시작하는 **초기화 블록** 에 기술할 수 있습니다.

인스턴스 초기화 때, 초기화 블록은 클래스 본문에 서술된 순서되로 실행되며, 클래스 속성 초기화와 연동이 됩니다.
During an instance initialization, the initializer blocks are executed in the same order as they appear 
in the class body, interleaved with the property initializers:

<div class="sample" markdown="1" theme="idea">

```kotlin
//샘플시작
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)
    
    init {
        println("First initializer block that prints ${name}")
    }
    
    val secondProperty = "Second property: ${name.length}".also(::println)
    
    init {
        println("Second initializer block that prints ${name.length}")
    }
}
//샘플 끝

fun main() {
    InitOrderDemo("hello")
}
```

</div>

참고로 주생성자의 파라미터는 초기화 블록에서 사용됩니다. 또한 주생성자는 클래스 본문에 선언된 속성들을 초기화하는데 사용될 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

</div>

실제로 주생성자를 속성으로 선언하고 초기화 하는데 사용하려면 더 간단한 문법으로 표현할 수 있습니다. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) { ... }
```

</div>

보통 속성을 사용할 때 처럼, 주 생성자로 선언된 속성은 변경이 가능한 (*var*{: .keyword }) 또는 읽기 전용인 (*val*{: .keyword })가 될 수 있습니다.
만약 생성자에 어노테이션 또는 접근 제한자가 있으면 키워드 the *constructor*{: .keyword } 를 꼭 붙여줘야 하는데, 필요한 제한자 뒤에 써주면 됩니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Customer public @Inject constructor(name: String) { ... }
```

</div>

이에 대해 좀 더 알아보고 싶다면 [접근제한자](visibility-modifiers.html#constructors)를 참고해주세요.


#### 부생성자

클래스는 *constructor*{: .keyword }: 로 시작하는 부생성자를 가질 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

</div>

만약 클래스에 주 생성자가 있다면, 각각의 부생성자는 다른 부생성자를 통해 직접 또는 간접적으로 주생성자에게 위임을 해야합니다. 같은 클래스 안에 있는 다른 생성자에게 위임하는 것은 *this*{: .keyword } 키워드를 이용해서 진행합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

</div>

참고로 초기화 블록에 있는 코드가 주생성자의 일부가 됩니다. 주생성자에게 위임하는 것은 부생성자의 문장에서 이뤄집니다. 그래서 초기화 블록에 있는 코드는 부생성자의 본문 이전에 실행이 됩니다. 만약 클래스에 주생성자가 없다고 암묵적으로 위임이 되며 초기화 블록도 실행이 됩니다.

<div class="sample" markdown="1" theme="idea">

```kotlin
//샘플시작
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
//샘플 끝

fun main() {
    Constructors(1)
}
```

</div>

만약 추상 클래스가 아닌 클래스가 주생성자든, 부생성자든 생성자가 없을때 인자가 없는 주생성자를 만들어냅니다. 이 때 생성자는 퍼블릭인데, 만약 이걸 원하지 않는다면 가시성을 기본값으로 가지지 않은 빈 생성자를 선언해줘야합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class DontCreateMe private constructor () { ... }
```

</div>

> **참고**: JVM에서 만약 모든 주생성자의 매개변수가 기본값을 가지고 있으면 컴파일러가 매개변수 없는 주생성자를 만들어 기본값으로 사용합니다. 이렇게 해두면 코틀린에서 Jackson이나 JPA와 같이 매개변수가 없는 생성자를 통해 인스턴스를 만들어내는 라이브러리를 더 쉽게 사용하는데 도움이 됩니다.

><div class="sample" markdown="1" theme="idea" data-highlight-only>
>
>```kotlin
>class Customer(val customerName: String = "")
>```
>
></div>

{:.info}

### 클래스의 인스턴스 생성하기

클래스의 인스턴스를 생성하려면 보통 함수처럼 생성자를 호출합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

</div>

코틀린은 인스턴스를 만드는데 *new*{: .keyword } 키워드를 사용하지 않습니다.

중첩 클래스나 익명 클래스의 인스턴스를 만드는건 [중첩 클래스](nested-classes.html) 를 참조해주세요.

### Class Members

클래스는 다음과 같은 것들을 가질 수 있습니다.

* [생성자와 생성자 블록](classes.html#constructors)
* [함수](functions.html)
* [속성](properties.html)
* [중첩 클래스와 내부 클래스](nested-classes.html)
* [객체 선언](object-declarations.html)


## 상속

코틀린의 모든 클래스는 `Any` 라는 공통의 슈퍼 클래스를 가집니다. 이것은 슈퍼 타입을 선언하지 않아도 가지게 되는 슈퍼 클래스입니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Example // 암목적으로 Any 클래스를 상속 받습니다
```

</div>

> 참고 : `Any` 클래스는 자바의 `java.lang.Object`와는 다릅니다. 특히 이건 `equals()`, `hashCode()`, `toString()`만 가지고 있습니다.

더 자세한 내용은 [자바 호환성](java-interop.html#object-methods) 을 참고해주세요.

슈퍼 타입을 명시적으로 선언하려면 클래스 헤더 뒤에 콜론(:)과 타입을 적습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

</div>

만약 파생 클래스가 주생성자를 가지고 있으면, 주생성자의 매개변수를 이용해 기초 클래스를 거기서 초기화 할 수 있습니다(그리고 되어야 합니다)

만약 클래스가 주생성자를 가지고 있지 않다면, 각각의 부생성자는 *super*{: .keyword } 키워드를 이용해 기초 클래스를 초기화하거나 다른 생성자가 그것을 하도록 위임해야합니다.

참고로 아래처럼 각각 다른 부생성자가 기본 클래스의 다른 생성자를 호출할 수 있습니다. 


<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

</div>

### 메소드를 오버라이딩 하기

이전에 언급한 것 처럼, 코틀린에서는 모든 것을 명시적으로 합니다. 자바와 달리 코틀린은 오버라이딩을 하는 멤버들에는 명시적인 제한자를 *open* 키워드를 써줘야합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class Base {
    open fun v() { ... }
    fun nv() { ... }
}
class Derived() : Base() {
    override fun v() { ... }
}
```

</div>

`Derived.v()`에는 *override*{: .keyword } 제한자가 필요합니다. 만약 그걸 안쓰면 컴파일 오류가 납니다.

만약 `Base.nv()`처럼 *open*{: .keyword } 제한자가 함수에 명시되어있지 않다면, *override*{: .keyword } 를 쓰든 안쓰든 하위클래스에 같은 시그니쳐를 가진 메소드를 선언하면 에러가 납니다. 

*open*{: .keyword } 제한자는 파이널 클래스의 멤버에 같이 쓰면 아무런 효과가 없습니다. (예를 들어 *open*{: .keyword }가 없는 클래스에 사용)
*override*{: .keyword } 라고 표시가 된 멤버는 그 자체로 오픈이 되며, 서브 클래스에서 오버라이드가 될 수 있습니다. 만약 오버라이딩이 또 되는 것을 막고 싶다면 *final*{: .keyword } 키워드를 쓰세요.


<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() { ... }
}
```

</div>

### 속성을 오버라이드 하기

속성을 오버라이딩하는 것은 메소드를 오버라이드 하는 것과 비슷합니다. 슈퍼 클래스에서 선언된 속성을 하위 클래스에서 다시 선언하는 경우 *override*{: .keyword } 키워드를 앞에 붙여줘야 합니다. 또한 호환이 가능한 타입이어야 합니다. 선언된 속성은 각각 초기화 또는 게터 메소드를 통해 오버라이드 될 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class Foo {
    open val x: Int get() { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```

</div>

`val` 키워드로 선언된 속성은 `var` 키워드로 오버라이드 할 수 있는데, 반대로는 안됩니다. 이것은 기본적으로 `val` 속성은 게터 메소드로 선언을 하고,  파생 클래스에서 추가적으로 세터 메소드를 오버라이드 해서 `var` 속성을 구현할 수 있기 때문입니다.

참고로 *override*{: .keyword } 키워드를 주생성자에 선언되는 속성에도 이용할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

</div>

### 파생 클래스의 초기화 순서 

파생 클래스의 인스턴스가 생성될 때, 기초 클래스의 초기화가 첫번째 단계로 진행됩니다. (클래스 생성자의 인자에 대한 평가만 선행됩니다.) 그리고 파생 클래스의 초기화 로직이 실행됩니다.

<div class="sample" markdown="1" theme="idea">

```kotlin
//샘플 시작
open class Base(val name: String) {

    init { println("Initializing Base") }

    open val size: Int = 
        name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {

    init { println("Initializing Derived") }

    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }
}
//샘플 끝

fun main() {
    println("Constructing Derived(\"hello\", \"world\")")
    val d = Derived("hello", "world")
}
```

</div>

즉 이말은 기초 클래스의 생성자가 실행될 쯔음에 파생 클래스에 선언되거나 오버라이드 된 속성들이 아직 초기화되지 않았다는 것을 의미합니다. 만약 이 속성들이 기초 클래스 초기화 로직(직접적으로든 간접적이로든 오버라이드 된 *open*{: .keyword } 멤버 구현을 통해)에 사용된다면, 런타임 에러가 날 것입니다. 따라서 기초 클래스를 설계할 때는 *open*{: .keyword } 멤버들을 생성자, 속성 초기화 또는 *init*{: .keyword } 블록에 사용하지 않길 바랍니다.

### 슈퍼 클래스 구현을 호출하기 ? Calling the superclass implementation

파생 클래스 코드에서는 슈퍼클래스의 함수나 속성의 접근자 구현?를 *super*{: .keyword } 키워드를 이용해 호출할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class Foo {
    open fun f() { println("Foo.f()") }
    open val x: Int get() = 1
}

class Bar : Foo() {
    override fun f() { 
        super.f()
        println("Bar.f()") 
    }
    
    override val x: Int get() = super.x + 1
}
```

</div>

내부 클래스 안에서는 슈퍼클래스의 외부 클래스를 접근하려면 *super*{: .keyword } 키워드를 클래스 이름과 함께써서 `super@Outer` 형식으로 접근할 수 있습니다. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Bar : Foo() {
    override fun f() { /* ... */ }
    override val x: Int get() = 0
    
    inner class Baz {
        fun g() {
            super@Bar.f() // Foo 클래스에서 f 메소드 구현한 것을 호출
            println(super@Bar.x) // Foo 클래스에서 x 게터를 구현한 것을 사용
        }
    }
}
```

</div>

### 오버라이딩 규칙

코틀린에서는 상속을 구현하려면 다음과 같은 규칙을 지켜야 합니다. 만약 클래스가 같은 바로 위의 슈퍼 클래스의 동일한 멤버 구현을 상속받는 경우, 이 멤버를 오버라이드 하고 자기 자신만의 구현을 해야합니다. (아마 상속받은 것 중 하나를 쓰게 될 것 입니다.)

슈퍼 클래스에서 어떤 걸 상속받았는지 나타내기 위해서, *super*{: .keyword } 와 슈퍼 타입의 이름을 <> 안에 넣어 나타닙니다. (예 : `super<Base>`)

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // 인터페이스의 멤버는 기본적으로 'open' 입니다.
    fun b() { print("b") }
}

class C() : A(), B {
    // 컴파일러는 f 메소드가 오버라이드 되었는지 확인할 것입니다.
    override fun f() {
        super<A>.f() // A의 f메소드를 호출
        super<B>.f() // B의 f메소드를 호출
    }
}
```

</div>

`A`와 `B`를 모두 상속하는 것은 괜찮습니다. `C`클래스는 `a()`, `b()`는 오직 하나씩만 구현하기 때문에 큰 문제가 없습니다. 
하지만 `f()` 메소드는 `C` 클래스에서 두개의 구현이 있기 때문에 `C` 클래스에서는 `f()` 메소드를 오버라이드 해서 애매함을 없애야 합니다. 

## 추상 클래스

클래스 또는 멤버는 *abstract*{: .keyword } 키워드를 사용해 선언할 수 있습니다.
추상 멤버는 그 클래스에 구현을 하지 않아도 됩니다.
추상 클래스나 오픈된 메소드에 어노테이션을 붙이지 않아도 됩니다. 그냥 안써도 됩니다.

추상화가 아닌 오픈 멤버를 추상된 것으로 오버라이드 할 수 있습니다. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

</div>

## Companion Objects

## 동반 객체

자바나 C#이랑 달리 코틀린은 정적 메소드가 없습니다. 대부분의 경우에는 패키지 수준에서 함수를 사용할 것을 권고합니다.

If you need to write afunction that can be called without having a class instance but needs access to the internals
of a class (for example, a factory method), you can write it as a member of an [object declaration](object-declarations.html)
inside that class.

만약 클래스를 인스턴스화 하지 않고 함수 내부를 접근해 코드를 써야하는 경우 (예를 들어 팩토리 함수와 같은 것) 클래스 안에 [객체 선언](object-declarations.html) 의 멤버로 사용할 수 있습니다.

더 정확히 설명하자면 내부 클래스에 [동반객체](object-declarations.html#companion-objects)를 선언하면 클래스 이름을 사용해서 자바나 C#에서 정적 메소드를 사용하는 것처럼 동일한 문법으로 멤버를 사용할 수 있습니다.
