---
type: doc
layout: reference
category: "Syntax"
title: "인터페이스"
---

# 인터페이스

코틀린의 인터페이스는 Java 8과 매우 비슷합니다. 추상 메소드를 정의해도 되고 실제로 구현을 해도 됩니다.
추상 클래스와 차이점이 있다면, 인터페이스는 상태를 가질 수 없습니다. 
프로퍼티도 가질 수 있는데, 추상 프로퍼티가 되거나 접근자를 구현해야 합니다.

인터페이스는 *interface*{: .keyword } 키워드 를 사용해 정의합니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // 본문은 필수가 아님
    }
}
```
</div>

## 인터페이스 구현하기

클래스 또는 오브젝트는 1개 이상의 인터페이스를 구현할 수 있습니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class Child : MyInterface {
    override fun bar() {
        // 본문
    }
}
```
</div>

## 인터페이스의 프로퍼티

인터페이스에는 프로퍼티를 선언할 수 있습니다. 
인터페이스 속성으로 선언된 프로퍼티는 추상 프로퍼티거나, 접근자를 구현해서 선언될 수 있습니다.
인터페이스에 선언된 프로터티는 백킹 필드를 가질 수 없으며, 인터페이스의 접근자에서도 참조할 수 없습니다.


<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
interface MyInterface {
    val prop: Int // 추상 프로퍼티

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```
</div>

## 인터페이스의 상속

인터페이스는 다른 인터페이스에서 파생될 수 있는데, 각각의 멤버에 해당하는 내용을 구현을 하고 새로운 함수와 프로퍼티를 정의할 수도 있습니다. 이런 인터페이스를 구현한 클래스는 구현이 안된 부분만 정의하면 됩니다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String
    
    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // 'name'은 구현 안해도 됨
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```
</div>

## 오버라이딩이 겹칠 때 해결하기

슈퍼타입에 많은 타입들을 선언하다 보면 같은 메소드에 대해 상속한 내용이 겹칠 수 있습니다.
다음의 예를 살펴봅시다.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```
</div>

인터페이스 *A*와 *B*는 둘 다 *foo()*와 *bar()* 메소드로 선언했습니다.

둘 다 *foo()* 뿐만이 아니라 *bar()*도 구현하고 있습니다. (인터페이스에서 함수 본문이 없으면 기본은 추상이므로, *bar()*는 *A*에서 추상이라고 표시해두지 않았습니다.) 이제 *A*클래스에서 *C*클래스를 구상클래스로 만들면 당연히 *bar()* 메소드를 오버라이드해서 구현을 해야 합니다.

하지만 만약 *A*와 *B*로부터 파생된 *D* 를 만들려면 여러개의 인터페이스로부터 상속받은 모든 메소드를 구현해야합니다. 그리고 이 때, *D*에게 어떻게 구현할 것인지 아주 정확하게 명시해줘야 합니다. 이 규칙은 1개만 구현된 것(*bar()*)과 여러개가 구현된 것(*foo()*) 둘 다 적용됩니다. 