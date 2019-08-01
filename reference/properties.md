---
type: doc
layout: reference
category: "Syntax"
title: "프로퍼티와 필드 : 게터, 세터, const, lateinit"
---

# 프로퍼티와 필드

## 프로퍼티 선언

코틀린 클래스의 프로퍼티는 값 변경이 가능하도록 *var*{: .keyword } 키워드를 이용하거나 읽기 전용으로 *val*{: .keyword } 키워드를 이용해 선언할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String? = ...
    var zip: String = ...
}
```
</div>

프로퍼티를 사용하려면 자바 필드를 사용하는 것 처럼 이름을 참조하면 됩니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // 코틀린에선 'new' 키워드를 사용하지 않음
    result.name = address.name // 접근자가 호출
    result.street = address.street
    // ...
    return result
}
```
</div>

## 게터와 세터

프로퍼티를 선언하는 전체 문법은 다음과 같습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
var <이름>[: <타입>] [= <초기자(Initializer)>]
    [<게터(Getter)>]
    [<세터(Setter)>]
```
</div>

초기자, 게터, 세터는 생략 가능합니다. 프로퍼티 타입은 초기자에서 추측할 수 있으면 생략 가능합니다.
(또는 다음처럼 게터, 세터에서 추측 가능해도 생략 됩니다.)

예시 :

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
var allByDefault: Int? // 에러 : 명시적인 초기자가 필요. 암묵적인 기본 게터와 세터. 
var initialized = 1 // Int 타입에 기본 게터 세터.
```
</div>

변경할 수 없는 프로퍼티의 온전한 문법은 변경할 수 있는 프로퍼티를 선언하는 문법과 두가지 차이점이 있습니다. 
`var` 대신 `val` 키워드로 시작하고, 세터를 지정할 수 없습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
val simple: Int? // Int 타입, 기본 게터에 생성자에서 반드시 초기화 되어
val inferredType = 1 // Int 타입, 기본 게터
```
</div>

프로퍼티에 커스텀 접근자를 설정할 수 있습니다. 커스텀 게터를 정의하면 프로퍼티에 접근할 때마다 매번 호출될 것입니다.
(그리고 이걸 통해 계산된 프로퍼티를 구현할 수 있습니다.)

커스텀 게터의 예를 살펴봅시다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```
</div>

커스텀 세터를 정의해두면, 프로퍼티에 값을 할당할 때마다 호출될 것입니다.
커스텀 세터는 다음과 같이 생겼습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 문자열을 파싱하고 다른 프로퍼티에 값을 할당
    }
```
</div>

컨벤션에 따르면 세터의 인자 이름은 `value` 입니다만, 다른 이름으로 지정해서 써도 됩니다.

코틀린 1.1 이후부터는 프로퍼티 타입을 게터에서 추측할 수 있으면 생략할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val isEmpty get() = this.size == 0  // Boolean 타입을 가짐
```
</div>

만약 프로퍼티의 기본적인 구현은 유지하되 가시성을 변경하거나 어노테이션을 붙이고 싶으면, 본문은 쓸 필요 없이 접근자만 써면 됩니다. 

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
var setterVisibility: String = "abc"
    private set // 세터는 프라이빗이고 기본 구현

var setterWithAnnotation: Any? = null
    @Inject set // Inject를 어노테이션으로 붙임
```
</div>

### 백킹 필드

코틀린 클래스에서는 필드를 바로 선언할 수 없습니다. 하지만 프로퍼티에 백킹 필드가 필요한 경우에는 코틀린에서 자동으로 만듭니다. 이 백킹 필드는 `field` 식별자를 쓰는 접근자로 참조할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
var counter = 0 // 참고 : 초기자에서 백킹 필드를 직접 만들어줌
    set(value) {
        if (value >= 0) field = value
    }
```
</div>

`field` 식별자는 오직 프로퍼티의 접근자에서만 쓸 수 있습니다.

백킹 필드는 프로퍼티가 접근자 중 적어도 하나의 기본 구현을 쓰는 경우, 또는 `field` 식별자를 통해 커스텀 접근자를 참조하는 경우에 만들어집니다.

예를 들어 다음과 같은 경우에는 백킹 필드가 생기지 않습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```
</div>

### 백킹 프로퍼티

"암묵적인 백킹 필드"의 원칙에 해당되지 않는 것을 만들고 싶다면, *backing property*를 대신 사용하면 됩니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // 타입 매개변수는 추론됨
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```
</div>


자바처럼 기본 세터와 게터를 가진 프라이빗 프로퍼티 접근은 최적화 되어 함수 호출 오버헤드가 걸리지 않도록 되어있습니다.

## 컴파일 상수 

컴파일시 값을 가지는 프로퍼티는 *const*{: .keyword } 수정자를 이용하면 컴파일 타임 상수로 표시할 수 있습니다.
이런 프로퍼티는 다음과 같은 점을 만족해야합니다.

  * 최상위 수준이거나 또는 [*object*{: .keyword } declaration](object-declarations.html#object-declarations) 또는 [a *companion object*{: .keyword }](object-declarations.html#companion-objects) 의 멤버여야 한다.
  
  * `String` 또는 원시 타입의 값으로 초기화 되어야 함
  
  * 커스텀 게터를 가지면 안됨

이런 프로퍼티는 어노테이션에 사용될 수 있습니다. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```
</div>

## 프로퍼티와 변수의 늦은 초기화

보통 프로퍼티는 null이 아닌 타입으로 선언되는 경우 무조건 생성자에서 초기화 되어야 합니다.

하지만 그럴 수 없는 경우가 꽤나 자주 일어납니다. 
예를 들어 프로퍼티는 주입에 의존해 초기화 되거나 단위 테스트의 설정 메소드를 통해 초기화 되기도 합니다.
이런 경우 생성자에 null이 아닌 초기자를 쓸 수 없는데 클래스 안에서 이 프로퍼티를 참조할 때 null 체크를 피하고 싶을 수 있습니다.

이런 경우 수정자 `lateinit`를 프로퍼티에 표시해서 해결할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // 직접 역참조 (?? dereference directly)
    }
}
```
</div>

이 수정자는 클래스 본문 안에 선언된 `var` 프로퍼티에 사용할 수 있습니다. 
(주 생성자에서는 사용할 수 없으며, 커스텀 게터 또는 세터를 가지지 않야야 합니다.)

Kotlin 1.2에서 부터는 최상위 수준의 프로퍼티와 지역 변수에서만 사용할 수 있습니다.
프로퍼티 또는 변수의 타입은 null이면 안되고, 원시 타입을 가지면 안됩니다.

초기화 전에 `lateinit` 프로퍼티에 접근하면 프로퍼티가 초기화되지 않았다고 확실하게 알려주는 특별한 예외가 발생됩니다.

## lateinit var가 초기화 되었는지 확인하는 법 (1.2 버전 이후)

`lateinit var`가 초기화 되었는지 확인하기 위해서는  [reference to that property](reflection.html#property-references)에 `.isInitialized`를 사용합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
if (foo::bar.isInitialized) {
    println(foo.bar)
}
```
</div>

단, 이 체크는 어휘적으로 접근할 수 있는 프로퍼티(같은 타입, 바깥 타입 중 하나 또는 같은 파일 내 최상위 수준)에만 할 수 있습니다. 


## 프로퍼티 오버라이딩

다음을 확인해 보세요. [Overriding Properties](classes.html#overriding-properties)


## 위임 프로퍼티

프로퍼티의 가장 흔한 종류는 단순하게 백킹 필드를 (쓰거나) 읽습니다.
하지만 커스텀 게터와 세터를 이용하면 프로퍼티에 어떤 구현이라도 적용할 수 있습니다.
그 중간쯔음에 프로퍼티가 어떻게 동작하는지에 대한 어떤 흔한 패턴이 있습니다. 
몇가지 예로는 레이지 값, 주어진 키를 가지고 맵에서 읽기, 데이터 베이스에 접근하기, 접근할 때 리스너에게 알리기 등이 있습니다.

일반적인 방식은 [_delegated properties_](delegated-properties.html)를 이용하는 라이브러리로 구현할 수 있습니다.
