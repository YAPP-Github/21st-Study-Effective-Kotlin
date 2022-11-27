# item 5: 리서비를 명시적으로 참조하라

함수나 프로퍼티를 참조할 때, 로컬 변수나 톱 레벨 변수가 아닌 다른 리시버로부터 가져오는 경우를 나타낼 경우가 있음. 대표적으로 this가 그러하다.

```kotlin
class User(val property: Prop){
		
		fun function(){
			doSomething(a)
			val b = ...
			doAnotherthing(b)
			doTheOtherThing(this.property)
		}

	companion object: {
		const val a = 1234
	}
}

```

### 여러 개의 리시버

특히 스코프 내에서 여러 개의 리시버 사용시 명시적으로 나타내면 좋다.

run, with, apply 등은 리시버가 암시적이기 때문에 특히 그렇다. 

```kotlin
private class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .apply {
// this.name이 우선순위이지만, 타입이 Node? 라서 사용할 수 없기에 부모의 프로퍼티가 넘어옴
                println("Created $name") 
                println("Created ${this?.name}")
//                println("Created ${this.name}") compile Error
                println("Created ${this?.name} in"
									+ " ${this@Node.name}"	

							)
            }

    fun create(name: String): Node? = Node(name)
}

fun main() {
    Node("Parent").makeChild("child")
}
```

- 첫 번째 $name의 값
    - apply scope에서의 this.name이 우선순위일 테지만, this.name은 사용할 수 없음(nullable 이라서)
    - 불가피하게 바깥 스코프인 부모의 name 사용
- 두번째 ${this?.name}의 값
    - 명시적으로 this 값을 unpack → 정상 값을 얻을 수 있음
- 세번째 ${this.name} 값
    - 컴파일 에러에 의해 사용할 수 없음. this는 nullable이기 때문
- 네번째 ${this@Node.name}
    - this 라벨을 이용해서 외부 스코프에 접근

⇒ 마커 등을 이용해 리시버를 활용하면 의미가 명확해짐.

++ 아래와 같은 경우도 리시버를 명시적으로 표시하면 편함

```kotlin
list1.forEach{ outerVal -> 
	list2.forEach{innerVal ->
//		some
	}
}
```

### DSL 마커

DSL 사용시 리시버가 중첩되더라도, 리시버를 명시적으로 붙이지 않는다.

<aside>
💡 DSL : 특정 목적으로 작성하는 랭귀지. 
Kotlin DSL: 코틀린 문법의 고차함수, 람다, 확장함수를 이용해 DSL을 작성할 수 있다.

</aside>

[https://myungpyo.medium.com/kotlin-dsl-간단히-알아보기-5f95fddf00f9](https://myungpyo.medium.com/kotlin-dsl-%EA%B0%84%EB%8B%A8%ED%9E%88-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-5f95fddf00f9)