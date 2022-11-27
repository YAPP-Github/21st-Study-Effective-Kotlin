# item 4: 변수 타입이 명확하지 않은 경우 확실하게 지정하라

타입 추론 시스템은 유용하지만, 명확할 때만 사용해야 함

불명확한 경우 definition을 찾아봐야 하는데, 그것 자체로 가독성을 해치게 된다.

- 명확한 경우

```kotlin
val num = 10
val name = "sejin"
val ids = listOf(1,2,345)
```

- 불명확한 경우

```kotlin
val data = getSomeData() 

val data: UserData = getSomeData() //이렇게 고치세요
```