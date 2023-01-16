# hashCode의 규약을 지켜라

- hashCode는 해시테이블을 구축할 때 사용되는 메서드로 해시함수로 볼 수 있음.

## hashCode의 규약

1. 어떤 객체를 변경하지 않았다면, hashCode는 여러 번 호출해도 결과가 항상 같음.
2. equals 메서드의 실행 결과로 두 객체가 같다고 나오면 hashCode 메서드의 호출 결과도 같음

```kotlin
class FullName(
	var name: String,
	var surname: String
){
	override fun equals(other: Any?): Boolean = 
		other is FullName && other.name == name && other.surname == surname
}

val s = mutableSetOf<FullName>()
s.add(FullName("Marcin","Moskala"))
val p = FullName("Marcin","Moskala")
print(p in s) // false -> 같은 해시코드가 아니라서 
print(p == s.first()) // true
```

- 따라서 equals 구현을 오버라이드 할 때 hashCode도 함께 오버라이드 추천
- 또한, hashCode는 최대한 요소를 넓게 퍼뜨려야 함. 같은 해시코드를 반환한다면 사실상 링크드 리스트와 차이가 없음.

```kotlin
class Proper(val name: String){
	override fun equals(other: Any?): Boolean{
		equalsCounter++
		return Other is Proper && name == other.name
	}

	override fun hashCode(): Int{
		return name.hashCode()
	}

	companion object{
		var equalsCounter = 0
	}
}

class Terrible(val name: String){
	override fun equals(other: Any?): Boolean{
		equalsCounter++
		return other is Terrible && name == other.name
	}
		
	override fun hasCode() = 0
	
	companion object{
		var equalsCounter = 0
	}
}

val properSet = List(10000) { Proper("$it")}.toSet()
println(Proper.equalsCounter) // 0
val terribleSet = List(10000) { Terrible("$it") }.toSet()
println(Terrible.equalsCounter) // 50116683

Proper.equalsCounter = 0
println(Proper("A") in properSet) // true
println(Proper.equalsCounter) // 1

Proper.equalsCounter = 0
println(Proper("A") in properSet) // false
println(Proper.equalsCounter) // 0

Terrible.equalsCounter = 0
println(Terrible("9999") in terribleSet) // true
println(Terrible.equalsCounter) // 4324

Terrible.equalsCounter = 0
println(Terrible("A") in terribleSet) // false
println(Terrible.equalsCounter) // 10001
```

## hashCode 구현하기

- 직접 정의할 일은 거의 없지만 equals를 별도로 정의했으면 hashCode도 같이 정의
- hashCode는 기본적으로 equals에서 비교에 사용되는 프로퍼티를 기반으로 해시코드 생성

```kotlin
class DateTime(
	private var millis: Long = 0L,
	private var timeZone: TimeZone? = null
){
	private var asStringCache = ""
	private var changed = false

	override fun equals(other: Any?): Boolean = 
		other is DateTime && other.millis == millis && other.timeZone == timeZone

	override fun hashCode(): Int{
		var result = millis.hashCode()
		result = result * 31 + timeZone.hashCode()
		return result
	}
}
```

- 이 때 유용한 함수로 코틀린/JVM의 Objects.hashCode가 있음. 해시 계산
- 코틀린 stblib에는 함수가 따로 없으므로 아래와 같은 함수 구현해서 사용

```kotlin
override fun hashCode(): Int = 
	hashCodeOf(timeZone, millis)

inline fun hashCodeOf(vararg values: Any?) =
	values.fold(0){ acc, value ->
		(acc*31) + value.hashCode()
	}
```