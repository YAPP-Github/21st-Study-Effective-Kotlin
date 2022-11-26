# 1. 가독성을 목표로 설계하라.
> &nbsp; 가독성은 사람에 따라 다르게 느끼겠지만 일반적으로 코드를 읽고 얼마나 빠르게 이해할 수 있는지를 의미합니다. 아래에 코드 가독성을 높이기 위한 방법을 소개하고 있습니다.

## 인식 부하 감소
```kotlin
// 구현 A
if(person != null && person.isAdult){
    view.showPerson(person)
}else{
    view.showError()
}
    
// 구현 B
person?.takeIf { it.isAdult }
    ?.let(view::showPerson)
    ?: view.showError()
```
구현 B처럼 단순히 코드가 짧다고 가독성이 좋다고 말하기는 어렵습니다. 구현 B를 이해하기 위해서는 안전호출(?.), takeIf, let, Elvis 연산자에 대한 이해가 선행되어야 합니다. 그에 비해 구현 A는 if/else, 메소드 호출을 사용하기 때문에 빠르게 이해가 되어 가독성이 더 좋습니다. 
<br>

```kotlin
// 구현 A
if(person != null && person.isAdult){
    view.showPerson(person)
    view.hideProgressWithSuccess()
}else{
    view.showError()
    view.hideProgress()
}
    
// 구현 B
person?.takeIf {it.isAdult}
    ?.let{
        view.showPerson(it)
        view.hideProgressWithSuccess()
    } ?: run{
        view.showError()
        view.hideProgress()
    }
```
가독성 뿐만 아니라 수정이 필요할 경우에도 구현 A가 더 간편합니다. 작업을 더 추가할 경우에 구현 B에서 ``view::showPerson``과 같은 제한된 함수 레퍼런스를 사용할 수 없고 run 연산자에 대한 추가 이해가 필요합니다. 결론적으로 ``인지 부하``를 줄이는 방향으로 코드를 작성해야 모두에게 가독성이 좋고 추후에 수정도 편리할 수 있습니다. 

<br>

## 극단적이 되지 않기
```kotlin
class Person(val name: String){
    var person: Person? = null
    
    fun printName(){
        person?.let{
            print(it.name)
        }
    }
}
```
위에서 언급했던 let, run 에 대한 이해가 필요하다고 무조건 좋지 않다는 것은 아닙니다. 다음 예시처럼 person 변수처럼 가변 프로퍼티가 있고 null이 아닐 때만 특정 작업을 수행할 때 let을 사용하면 편리하게 작성할 수 있습니다. 
<br>

```kotlin
    students.filter {it.result >= 50}
        .joinToString(separator = "\n"){
            "${it.name} ${it.surname}, ${it.result}"
        }
        .let(::print)
```
그 외에도 일련의 연산 이후 연산 결과를 특정 메소드의 매개변수로 이동할 때도 let을 사용하면 편리하게 작성할 수 있습니다.
<br>

```kotlin
    var obj = FileInputStream("/file.gz")
        .let(::BufferedInputStream)
        .let(::ZipInputStream)
        .let(::ObjectInputStream)
        .readObject() as SomeObject
```
데코레이터 패턴 방식으로 사용하여 객체를 래핑할 때도 유용합니다. 위 코드들은 모두 디버깅이 어렵고 코틀린 초보자에게는 인지 부하가 늘어나지만, 충분히 지불할 만한 가치가 있으므로 사용해도 좋습니다. 결론적으로 "단순히 인지 부하가 늘어난다고 사용을 하면 안된다" 라고 극단적으로 생각하지 않고 정당한 이유가 있다면 사용해도 좋을 것 같습니다.

