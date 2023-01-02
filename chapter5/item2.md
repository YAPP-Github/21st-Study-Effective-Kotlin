# 아이템2: 기본 생성자에 이름 있는 옵션 아규먼트를 사용하라

자바에서 사용하던

- 점층적 생성자 패턴
- 빌더 패턴

이 두 패턴에서 생긴 문제를 **코틀린에서 해결하는 내용**

## 점층적 생성자 패턴

- 여러 가지 종류의 생성자를 사용하는 패턴

```kotlin
class Pizza{
	  val size: String
    val cheese: Int
	  val olives: Int
	  val bacon: Int

  	constructor(size: String, cheese: Int, olives: Int, bacon: Int){
        this.size = size
        this.cheese = cheese
        this.olives = olives
        this.bacon = bacon
    }
    constructor(size: String): this(size, 0)
}
```

- 보일러 플레이트 코드가 발생

### 개선

```kotlin
class Pizza(
	  val size: String
    val cheese: Int = 0,
	  val olives: Int = 0,
	  val bacon: Int = 0,
)
```

- **디폴트 아규먼트를 통해 위와 같이 사용할 경우의 장점**
    - 파라미터의 값들을 원하는 대로 지정할 수 있음
    - argument를 원하는 순서로 지정할 수 있음
    - **명시적으로 이름을 붙여 arugment를 지정하므로 의미가 명확**
        
        ```kotlin
        val myPizza = Pizza(
        	  size = "L",
            cheese = 1,
        )        
        ```      
        - 이와 같이 명확한 argument를 지정할 수 있음

### 정리

- **default argments를 사용하는 생성자가 점층적 생성자 패턴보다 훨씬 강력함**

## 빌더 패턴

```kotlin
class Pizza(
	  val size: String
    val cheese: Int,
	  val olives: Int,
	  val bacon: Int,
){
    class Builder(private val size: String){
        private var cheese: Int = 0
        private var olives: Int = 0
        private var bacon: Int = 0
    
        fun setCheese(value: Int): Builder = apply{
            cheese = value
        }
        fun setOlives(value: Int): Builder = apply{
            olives = value
        }
				fun setBacon(value: Int): Builder = apply{
            bacon = value
        }
				fun build() = Pizza(size, cheese, olives, bacon)
		} 
}

val myPizza = Pizza.Builder("L").setCheese(1).setOlives(2).setBacon(3).build()
```

- 자바에서는 `named arugment`와 `default argument` 를 사용할 수 없어서 위 방식을 사용했었음

### 개선

- [이 코드와 같음](https://www.notion.so/2-97a6e9d7e0e946a7aac3446fb2c47c73)
- **구현이 훨씬 쉽고, 가독성이 좋음**
    - 빌더 패턴을 이용하는 것 보다 **구현하는 시간이 짧음**
    - 파라미터의 이름을 변경하는 경우에도 바꾸기 더 편함
- **더 명확함**
    - 객체 생성 방법을 알고싶을 때, **생성자만 보면 됨**
    - 빌더 패턴은 `setCheese` 등여러 메서드를 확인해야함
- **더 사용하기 쉬움**
    - 추가적인 knowledge가 필요 없이 **코틀린의 내장 기능**으로만 사용 가능
- **동시성과 관련된 문제가 없음**
    - 코틀린의 함수 parameter는 immutable
    - **빌더 패턴의 프로퍼티는 mutable이므로 thread-safety하지 않음**

### 빌더 패턴이 더 좋은 경우

```kotlin
// before
val dialog = AlertDialog(
    context = context,
		buttonDescription = ButtonDescription(R.string.file, { d, id -> })
    ..
)

//after
	val dialog = AlertDialog.Builder(context)
			.setButton(R.string.file, {d ,id -> })
```

- **추가적인 타입을 만들지 않고 아래 장점을 얻을 수 있음**
    - 빌더 패턴을 사용하면 값의 의미를 묶어서 지정할 수 있음
    - 특정 값을 누적하는 형태로 사용할 수 있음
- **팩토리로 사용할 수 있음**
    - 하지만 이것도
    

하지만 이런 점들을 고려하고도 코틀린에서 빌더 패턴을 사용했을 때의 장점을 찾지 못함

**아래의 경우를 제외하고는 default arguments, DSL을 사용하는 것이 좋음**

- 빌더 패턴을 사용하는 다른 언어로 작성된 라이브러리를 그대로 옮길 때
- default argument와 DSL을 지원하지 않는 다른 언어에서 쉽게 사용할 수 있게 API를 설계할 때
    
    

### 더 좋은 방법 DSL

```kotlin
val response = client.patch("") {
            jsonBody {
                "nickname" withString nickname 
                "profileImageUrl" withString profileImageUrl 
                "favoriteCategories" withPojos favoriteCategories
                "favoriteTags" withPojos favoriteTags
            }
        }

override infix fun String.withString(value: String) {
        builder.put(this, value)
}
..
override infix fun String.withBoolean(value: Boolean) {
        builder.put(this, value)
}
```

- 현재 프로젝트에서 사용하고 있는 DSL
- 빌더보다 더 만들기 어렵지만, **더 유연하고 가독성이 좋은 코드**를 만들 수 있음

## 정리

점층적 생성자 패턴(Telescoping Constructor Pattern) **→ default argument**

빌더 패턴 → **기본 생성자 or DSL**
