# equals의 규약을 지켜라

## 동등성

- 구조적 동등성(structural equality) : equals 메소드나 해당 메소드 기반으로 만들어진 == 연산자로 확인하는 동등성
- 레퍼런스적 동등성 : === 연산자로 확인하는 동등성, 두 피연산자가 같은 객체면 true, 다르면 false 반환

코틀린의 Any 클래스에서는 equalse가 구현되어 있어서 모든 객체에 사용 가능, But 다른 타입의 객체 비교 시는 안됨(서브클래스 제외)

```kotlin
open class Animal
class Book
Animal() == Book()     // 사용 불가
Animatl() === Book()     // 사용 불가

class Cat: Animal()
Animal() == Cat()    // 사용 가능
Animal() == Cat()    // 사용 가능
```

## equals 필요한 이유

- Any 클래스에서는 기본적으로 === 연산자와 동일하게 확인하여 완전히 같은 객체인지 비교
- 이 동등성이 아닌 다른 형태로 표현하려면 직접 정의필요, 예를 들어  data class를 사용 시 프로퍼티 값만 같으면 같은 객체로 판단

```kotlin
data class FullName(val name: String, val surname: String)
val name1 = FullName("Marcin","Moskala")
val name2 = FullName("Marcin","Moskala"
val name3 = FullName("Maja","Moskala")

name1 == name1 // true
name2 == name2 // true 데이터가 같기 때문
name1 == name3 // false

name1 === name1 // true
name1 === name2 // false
name1 === name3 // false
```

- data class는 내부에 어떤 값을 갖고 있는지가 중요하기에 이와 같이 동작
- 이처럼 data class로 동등성을 조작하기에 직접 equals를 정의할 일이 많이 없지만, 상황에 따라 직접 구현해야하는 경우가 있는데 대표적으로 일부 프로퍼티로만 비교를 할 경우

```kotlin
class User(
	val id: Int,
	val name: String,
	val surname: String
){
	override fun equals(other: Any?): Boolean =
		other is User && other.id == id

	override fun hashCode(): Int = id
}
```

### 정리 (equals를 직접 구현이 필요한 경우)

1. 기본적으로 제공되는 동작과 다른 동작 필요 시
2. 일부 프로퍼티만으로 비교 필요 시
3. data class를 사용하는 것을 원치 않거나, 비교해야 하는 프로퍼티가 기본 생성자에 없는 경우

## equals의 규약

1. 반사적 동작 : if x ≠ null, then x.equals(x) = true
2. 대칭적 동작 : if x,y ≠ null, then x.equals(y) = y.equals(x)
3. 연속적 동작 : if x,y,z ≠ null , x.equals(y) = y.equals(z) = true, then x.equals(z) = true
4. 일관적 동작 : if x,y ≠ null → x.equals(y)는 항상 동일
5. 널과 관련된 동작 : if x ≠ null → x.equals(null) = false

### 반사적동작

- if x ≠ null, then x.equals(x) = true

```kotlin
class Time(
	val millisArg: Long = -1,
	val isNow: Boolean = false
){
	val millis: Long 
		get() = if(isNow) System.currentTimeMillis()
						else millisArg

	override fun equals(other: Any?): Boolean =
		other is Time && millis == other.millis
}

val now = Time(isNow = true)
now == now  // true or false
List(100000) { now }.all { it == now } // 대부분 false
```

- 예시와 같이 현재 시간을 나타내고 밀리초로 시간을 비교하는 Time 객체에서는 실행할 때마다 결과가 달라 일관적 동작을 위반
- 이처럼 equals 규약이 잘못되면 컬렉션 내부에 해당 객체 포함되어 있어도 contains 메서드로 알 수 없음.

```kotlin
sealed class Time
data class TimePoint(val millis: Long): Time()
object Now: Time()
```

- 이를 해결하는 방법으로 sealed 클래스를 사용하여 현재 시간을 나타내는지, 그게 아니면 같은 타임 스탬프로 갖고 있는가로 동등성을 확인한 방법이 있음.

### 대칭적 동작

- if x,y ≠ null, then x.equals(y) = y.equals(x)

```kotlin
class Complex(
	val real: Double,
	val imaginary: Double
){
	override fun euqals(other: Any?): Boolean{
		if(other is Double){
			return imaginary == 0.0 && real == other
		}
		return other is Complex && real == other.real && imaginary == other.imaginary
	}
}
```

- 같은 타입을 비교할 경우에는 문제가 없지만 다른 타입과 비교할 경우 대칭성을 위반

```kotlin
Complex(1.0, 0.0).equals(1.0)  // true
1.0.equlas(Complex(1.0, 0.0)) // false
```

- Double은 Complex와 비교할 수 없기에 대칭적으로 동작하지 않음.
- 따라서 동등성을 구현할 때 항상 대칭성을 고려해야 하며,다른 클래스는 동등하지 않게 만들어 버리는게 좋음.

### 연속적 동작

- if x,y,z ≠ null , x.equals(y) = y.equals(z) = true, then x.equals(z) = true

```kotlin
open class Date(
	val year: Int,
	val month: Int,
	val day: Int
){
	override fun equals(o: Any?): Boolean = when(0){
		is DateTime -> this == o.date
		is Date -> o.day == day && o.month == month && o.year == year
		else -> false
	}
}

class DateTime(
	val date: Date,
	val hour: Int,
	val minute: Int,
	val second: Int
): Date(date.year, date.month, date.day){
	
	override fun equals(o: Any?): Boolean = when(o){
		is DateTime -> o.date == date && o.hour == hour && o.minute == minute && o.second == second
		is Date -> date == o
		else -> false
	}
}

val o1 = DateTime(Date(1992, 10, 20), 12, 30, 0)
val o2 = Date(1992, 10, 20)
val o3 = DateTime(Date(1992, 10 ,20), 14, 45, 30)

o1 == o2 // true
o2 == o3 // true
o1 == o3 // false
```

- DateTime와 Date 끼리 비교보다 DateTime끼리 비교 시에 많은 프로퍼티를 비교함. 이로 인해 연속성 문제 발생
- 또한, 만약 같은 타입끼리만 비교할 수 있다고 하면 Date와 DateTime 은 상속 관계로 리스코프 치환 원칙 위반
- 따라서 처음부터 상속이 아닌 컴포지션을 사용하고 두 객체는 서로 비교하지 못하게 만드는게 좋음

```kotlin
data class Date(
	val year: Int,
	val month: Int,
	val day: Int
)

data class DateTime(
	val date: Date,
	val hour: Int,
	val minute: Int,
	val second: Int
)

val o1 = DateTime(Date(1992,10,20), 12, 30, 0)
val o2 = Date(1992, 10, 20)
val o3 = DateTime(Date(1992, 10, 20), 14,45, 30)

o1.equals(o2) // false
o2.equals(o3) // false
o1 == o3 // false

o1.date.equals(o2) // true
o2.equals(o3.date) // true
o1.date == o3.date // true
```

### 일관적 동작

- if x,y ≠ null → x.equals(y)는 항상 동일
- 즉 equals는 두 객체에만 의존하는 순수 함수여야 함. 이를 대표적으로 위반하는 클래스가 java.net.URL.equals()

```kotlin
import java.net.URL

fun main(){
	val enWiki = URL("https://en.wikipedia.org/")
	val wiki = URL("https://wikipedia.org/")
	println(enWiki == wiki)
}
```

- 위 코드는 네트워크 상황에 따라 결과가 달라짐. 일반적으로는 두 주소가 같은 IP 주소를 나타내어 true 출력
- 하지만 인터넷 연결이 끊어진 상황에는 false 출력
- 이처럼 동작이 일관되지 않고 네트워크 상태에 의존한다는 것은 잘못된 것임.

## equals 구현하기

- 특별한 이유가 없는 이상, 직접 equals를 구현하는 것은 좋지 않음.
- 기본적으로 제공되는 것을 쓰거나, 데이터 클래스를 만들어서 사용
- 직접 구현해야 한다면 반사적, 대칭적, 연속적, 일관적 동작 확인
- 이러한 클래스는 final로 만들고 상속이 필요 시에는 equals가 작동하는 방식 변경하면 안됨