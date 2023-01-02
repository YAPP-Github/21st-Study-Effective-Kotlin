# 아이템 6: 문서로 규약을 정의하라

## 서론 : 문서화의 중요성

- 아래 코드의 경우, showMessage를 외부에서 보았을 때 Toast를 출력하는 지, SnackBar를 출력하는 지 모를 수 있음

```kotlin
enum class MessageLength { SHORT, LONG }

fun Context.showMessage(
    message: String,
    length: MessageLength = MessageLength.LONG
) {
    val toastLength = when (length) {
        SHORT -> Length.SHORT
        LONG -> Length.LONG
    }
    Toast.makeText(this, message, toastLength).show()
}
```

- 이 **함수가 무엇을 하는지 명확하게 설명**하기 위해 **KDoc 주석을 붙여주는 것이 좋음**

```kotlin
/*
* 이 프로젝트에서 짧은 메시지를 사용자에게 출력할 때 사용하는 기본적인 방식입니다.
* @param message 사용자에게 보여 줄 메시지
* @param length 메시지의 길이가 어느정도 되는지 나타내는 enum 값
*/
fun Context.showMessage(
    message: String,
    length: MessageLength = MessageLength.LONG
) {
    ...
}
```

- 행위가 **문서화 되지 않고**, **요소의 이름이 명확하지 않다면** **사용하는 개발자가 내부 구현을 파헤치는 일이 생김**

## 규약(contract of an element)

- 예측되는 행위를 의미
- **만든 이의 의도를 해치지 않고, 사용하는 개발자가 원하는 부분을 적절하게 수정할 수 있도록, 이름, 주석, 타입 등에서 정한 규약을 따라야 함**

### 규약 정의

- 이름 : 일반적인 개념과 관련된 메서드는 이름만으로 동작을 예측할 수 있음 ex) sum
- 주석과 문서 : 필요한 모든 규약을 적을 수 있음
- 타입 : 객체에 대한 많은 것을 알려줌(일부 타입은 추가 설명이 필요)

### 주석을 써야할까?

- Robert C. Martin의 영향으로 주석을 지양하는 패러다임이 조성됨
- **코드만 읽어도 어느 정도 알 수 있는 코드를 작성하는 것이 일단 중요**
- **함수 또는 클래스에 더 많은 규약을 설명할 수 있으므로, 문서화에 필요함**
- **직관적으로 알 수 있는 코드의 경우 불필요한 주석을 삼가야 함**

```kotlin
fun List<Int>.product = fold(1) { acc, i -> acc * i} 
```

> 개인적으로 문서화 경험을 비추어 볼 때, 무분별한 KDoc 및 주석을 작성은 오히려 가독성을 해친다고 느꼈습니다. 직관적으로 파악할 수 있는 API는 KDoc 및 주석을 작성하지 않는 편이 좋은 것 같습니다.
> 

- 아래 예시는 `아이템 26: 함수 내부의 추상화 레벨을 통일하라` 의 내용과 연관되는데, **추상화 레벨 통일로 이해하기 쉬운 코드를 만들었음**

```kotlin
//before
fun update(){
    for (user in users) user.update()
    for(book in books) updateBook(book)
}

//after
fun update(){
    updateUsers()
    updatebooks()
}

private fun updateUsers(){
    for (user in users) user.update()
}

private fun updateBooks(){
    for(book in books) updateBook(book)
}
```

- 아래 예시에서, 코틀린 표준 라이브러리는 주석을 통해 함수의 규약을 정의해 주었음
    - JVM에서 읽기 전용이며, 직렬화가 가능한 List를 리턴한다고 정의됨
    - 이를 통해 사용자에게 이를 지키면서 자유롭게 사용할 수 있는 안정성을 부여함
    - 간단한 예시를 제공하여 사용자에게 사용법을 알려줌

```kotlin
/**
 * Returns an empty read-only list.  The returned list is serializable (JVM).
 *@samplesamples.collections.Collections.Lists.emptyReadOnlyList
*/
@kotlin.internal.InlineOnly
public inline fun <T> listOf(): List<T> = emptyList()
```

## KDoc

**주석으로 함수를 문서화 할 때 사용되는 공식적인 형식**

공식 문서 : [https://kotlinlang.org/docs/kotlin-doc.html](https://kotlinlang.org/docs/kotlin-doc.html)

- **Dokka 라는 코틀린 문서 생성 도구**를 통해 문서 파일을 제공할 수 있음

## 타입 시스템과 예측

- **리스코프 치환 원칙 -** 클래스가 어떤 동작을 할 것이라 예측되면, 서브클래스도 이를 보장해야 함
- 이를 위해 공개 함수에 대한 규약을 잘 지정해야 함
- **상위 인터페이스의 프로퍼티와 메서드의 규약을 정의하여 클래스 예측을 쉽게 할 수 있도록 함**

## 정리

- 외부 API를 정의 할 때에는 특히 규약을 잘 정의해야 함
- 이름, 문서, 주석, 타입을 통해 규약을 정할 수 있음
- 문서화를 통해 **요소를 쉽게 예측 할 수 있음**
