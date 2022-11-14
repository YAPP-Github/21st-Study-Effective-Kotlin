## 아이템 2 : 변수의 스코프를 최소화하라

### 프로퍼티보다 지역 변수를 사용하자

### 최대한 좁은 스코프를 갖게 변수를 사용하자

```jsx
val a = 1
fun function1() { 
   val b = 2 
   print(a + b)
}
```

- 내부 스코프에서는 `a`를 참조할 수 있으나, 외부 스코프에서는 `b`를 참조할 수 없음
- `a`라는 프로퍼티가 `function1`에서만 사용된다면, 내부 스코프에만 제한을 두는것이 좋음
- **프로그램 추적 및 관리가 용이하기 위해서는 어떤 시점에 요소가 있는지 알아야 함**
- 변수의 스코프가 넓으면 **굉장히 위험**

```jsx
var user: User
for(i in users.indices){ //안 좋은 예
    user = users[i]
		print("$i $user")
}
------------------------------------------
for((i, user) in users.withIndex()){ //좋은 예
		print("$i $user")
}
```

- `user`의 스코프를 내부에 둠으로서, **프로그램을 추적하고 관리하기 쉬워짐**
- `user`가 외부에 있다면 다른 개발자에 의해 **변수가 잘못 사용될 수 있음**
- **수신 객체 지정 람다, Elvis 표현식 등을 통해 최대한 변수를 정의할 때 초기화하는 것이 좋음**

### 캡처링

```kotlin
fun main() {
    val primes: Sequence<Int> = sequence {
        var numbers = generateSequence(2) { it + 1 }
        var prime: Int
        while (true) {
            prime = numbers.first()
            yield(prime)
            numbers = numbers.drop(1)
                    .filter { it % prime != 0 }
        }
    }
    print(primes.take(10).toList())
}
```

- while문 내부에서, 외부에서 생성한 prime을 캡처링 하게되어 생기는 문제
- **가변성을 피하고, 스코프를 좁게 만들어 해결할 수 있음**
