# 6. 제네릭 타입과 variance 한정자를 활용하라.
> Java 공부를 했던 분이라면 Generic 사용 시 공변성, 반변성, 반공변성에 대해 들어본적이 있을 것입니다. Kotlin에서도 마찬가지로 Generic을 활용 시에 variance 한정자(out, in)을 권장하고 있습니다.

## invariant(불공변성)
```kotlin
// invariant
class Cup<T>

fun main(){
    val anys: Cup<Any> = Cup<Int>()	// Type mismatch
    val nothings: Cup<Nothing> = Cup<Int>()	// Type mismatch
}
```
위 스니펫에서는 타입 파라미터 T에 variance 한정자(out, int) 모두 사용되지 않았으므로 기본적으로 invariant 합니다. invariant는 타입끼리 서로 연관성이 없다는 의미로 Cup< Int >, Cup< Number > 등등 어떠한 관련성도 갖지 않는다는 것입니다. 
<br>

## covariant(공변성)
```kotlin
// covariant
class Cup<out T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>){
    val b: Cup<Dog> = Cup<Puppy>()	// OK
    val a: Cup<Puppy> = Cup<Dog>()	// Error
}	
```
타입 파라미터와 함께 ``out`` 한정자를 사용하여 타입 파라미터를 covairant하게 만들 수 있습니다. 이는 A가 B의 서브타입이라고 했을 때, Cup< A >가 Cup< B >의 서브타입이라는 의미입니다. 
<br>

## contravariant(반변성)
```kotlin
// contravariant
class Cup<in T>
open class Dog
class Puppy(): Dog()

fun main(args: Array<String>){
    val b: Cup<Dog> = Cup<Puppy>()	// Error
    val a: Cup<Puppy> = Cup<Dog>()	// OK
    
    val anys: Cup<Any> = Cup<Int>()	// Error
    val nothings: Cup<Nothing> = Cup<Int>()	// OK
}
```
``in`` 한정자는 반대로 타입 파라미터를 contravariant하게 만들고 A가 B의 서브타입일 때 Cup< B >가 Cup< A > 의 슈퍼타입이라는 것을 의미합니다.
![](https://velog.velcdn.com/images/ows3090/post/f8c352f2-be0a-4f8f-b3d9-5abe20bf30a6/image.png)

## 함수타입
```kotlin
fun printProcessedNumber(transition: (Int) -> Any){
	print(transition(42))
}
```
함수 파라미터로 Int를 받고 Any를 리턴하는 함수를 받고 있습니다. 여기서 (Int) -> Any는 ``(Int) -> Number``, ``(Number) -> Any``, ``(Number) -> Number``, ``(Number) -> Int`` 를 대입하더라도 문제가 없습니다.
<br>

```kotlin
val intToDouble: (Int) -> Number = { it.toDouble() }
val numberAsText: (Number) -> Any = { it.toShort() }
val identity: (Number) -> Number = { it }
val numberToInt: (Number) -> Int = { it.toInt() }
val numberHash: (Any) -> Number = { it.hashCode() }
printProcessedNumber(intToDouble)
printProcessedNumber(numberAsText)
printProcessedNumber(identity)
printProcessedNumber(numberToInt)
printProcessedNumber(numberHash)
```
위 예시에서 알 수 있듯이 코틀린 함수 타입의 모든 파라미터 타입은 contravariant 입니다. 즉, 더 상위 계층의 타입을 대입할 수 있습니다. 또한 리턴 타입은 covariant로 원래의 계층보다 낮은 계층의 타입으로 대입 가능합니다.

<br>

## variance 한정자의 안정성
> variance 한정자가 많이 사용되는 것 중 하나가 List 입니다. 여기서 MutableList는 variance 한정자가 붙지 않는데 어떠한 이유로 List를 더 많이 사용하게 되고 다른 지를 variance 한정자의 안정성 측면에서 확인하면 이해하기 수월해질 수 있습니다.

### Java Array 문제점
```Java
// java
Integer[] numbers = { 1,4,2,1 };
Object[] objects = numbers;
object[2] = "B";	// RuntimeException : ArrayStoreException
```
자바의 배열은 기본적으로 제네릭으로 정의되어 있으며 covariant(공변성) 입니다. 그래서 objects에 numbers를 할당할 수 있으나 내부구조에서 사용되는 있는 실질적인 타입이 바뀌는 것은 아닙니다. 따라서 위 스니펫은 Integer 배열에 String 타입의 값을 할당하려고 하니 오류가 발생하게 되는 것입니다. Kotlin에서는 이러한 결함을 해결하기 위해 InArray, CharArray들을 invariant하게 만들었습니다. 
<br>

### covariant 한정자 주의사항
> Kotlin에서 public한 ``in`` , ``out`` 한정자 위치가 존재합니다.
``in`` : 함수의 매개변수
``out`` : 함수의 리턴 타입


```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

class Box<out T>{
    private var value: T? = null
    
    // 실제로는 Compile Error
    fun set(value: T){
    	this.value = value
    }
    
    fun get(): T = value ?: error("Value not set")
}

val puppyBox = Box<Puppy>()
val dogBox: Box<Dog> = puppyBox
dugBox.set(Hound())		// Error

val dogHouse = Box<Dog>()
val box: Box<Any> = dogHouse
box.set("Some string")		// Error
box.set(42)		// Error
```
여전히 Kotlin에서 직접 제네릭 클래스를 생성 시에 여러가지 안전하지 않는 상황이 있습니다. 위 스니펫에서는 Java Array와 동일하게 캐스팅 이후에는 실질적인 객체가 그대로 유지되기 때문에 다른 서브타입을 대입하려고 하면 오류가 발생합니다. Kotlin에서는 이러한 상황을 막기 위해 public ``in`` 한정자 위치에 covariant 타입 파라미터가 오는 것을 금지합니다.
<br>

```kotlin
class Box<out T>{
    private var value: T? = null
    // var value: T? = null		// 컴파일 오류
    
    private set(value: T){
    	this.value = value
    }
    
    // fun set(Value:T){	// 컴파일 오류
    // 		this.value = value
	// }    
    
    fun get(): T = value ?: error("Value not set")
}
```
컴파일이 정상적으로 되기 위해서는 private로 가시성을 제한하면 오류가 발생하지 않습니다. 객체 내부에서는 업캐스트 객체에 out 한정자가 적용되지 않기 때문입니다. 추가로 get 메소드의 리턴 타입은 public ``out`` 한정자 위치로 안전하기 때문에 따로 제한은 없습니다. 
<br>

### contravariant 한정자 주의사항
> covariant와 public ``in`` 위치로 인한 문제와 동일하게 contravariant과 public ``out`` 위치에서도 문제라 발생합니다.

```kotlin
open class Car
interface Boat
class Amphibious: Car(), Boat

class Box<in T>(
    // compile error
    val value: T
)

val garage: Box<Car> = Box(Car())
val amphibiousSpot: Box<Amphibious> = garage
val boat: Boat = amphibiousSpot.value    // Car

val noSpot: Box<Nothing> = Box<Car>(Car())
val not: Nothing = noSpot.value
```
위 스니펫도 사실상 컴파일 오류가 나타나지만 왜 Kotlin에서 금지해놨는지 이해할 필요가 있습니다. amphibiousSpot에는 contravarient 로 인해 Car 타입의 Box 객체가 저장되고 있습니다. 이후에 value를 통해서 값을 가져오게 되는데 사실상 Car 타입은 Boat와 연관성이 없기 때문에 문제가 됩니다.
따라서 코틀린에서는 contravarient 를 public ``out`` 한정자 위치에서 사용하는 것을 금지하고 있습니다.
<br>

```kotlin
class Box<in T>{
    // var value: T? = null
    private var value: T? = null
    
    fun set(value: T){
    	this.value = value
    }
    
    // fun get(): T = value ?: error("Value not set")
    private fun get(): T = value ?: error("Value not set")
}
```
동일하게 private 가시성을 추가하면 문제가 없도록 만들 수도 있습니다.

<br>

### variance 한정자의 위치
> variance 한정자는 크게 2가지 위치에서 사용할 수 있습니다.

1. 선언 부분
```kotlin
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<Any> = boxStr
```
선언 부분에 한정자를 적용하면 클래스와 인터페이스 사용되는 모든 곳에 영향을 줍니다.
<br>

2. 클래스와 인터페이스 활용하는 위치
```kotlin
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str")
val boxAny: Box<out Any> = boxStr
```
모든 인스턴스에 variance 한정자를 적용하지 않고 특정 인스턴스에만 적용해야 할 때 사용합니다.

MutableList에 ``in`` 한정자를 포함하면 값을 리턴할 수 없지만 단일 파라미터 타입에는 ``in`` 한정자를 붙여서 contravariant를 가지게 하는 것은 가능합니다.
```kotlin
interface Dog
interface Cutie
data class Puppy(val name: String): Dog, Cutie
data class Hound(val name: String): Dog
data class Cat(val name: String): Cutie

fun fillWithPuppies(list: MutableList<in Puppy>{
    list.add(Puppy("Jim"))
    list.add(Puppy("Beam"))
}

val dogs = mutableListOf<Dog>(Hound("Pluto"))
fillWithPuppies(dogs)
println(dogs)
// [Hound[name=Pluto), Puppy(name=Jim), Puppy(name=Beam)]

val animals = mutableListOf<Cutie>(Cat("Felix"))
fillWIthPuppies(animals)
println(animals)
// [Cat(name=Felix), Puppy(name=Jim), Puppy(name=Beam)]
```
## 정리
> Kotlin에서 variance를 사용하여 제약을 걸 수 있는 기능을 제공합니다. 
- invariant : 기본 타입 파라미터
- covariant : ``out`` 한정자 사용
- contravariant : ``in`` 한정자 사용
