# item 3: Unit?을 리턴하지 말라

item 13

### Unit?

null을 리턴 가능하니까 다음과 같이 사용할 수 있지만, 오해를 불러일으키기 쉽고 정형화된 패턴도 아니라 가독성이 좋지 않음.  Boolean 으로 대체하는것이 좋다. 

```kotlin
fun unitFunction() : Unit?
unitFunction() ?: return // 
```

```kotlin
// good case
if (person != null && person.isAdult){
	view.showPerson(person)
}else{
	view.showError()
}
```

```kotlin
// bad case
fun Person.adultProcess(view: View): Unit?{
	print()
	if (isAdult)
		view.showPerson(this)
	else
		return null
}

// null 리턴시 adultProcess와 view.showError가 둘다 실행 됨
person?.let {it.adultProcess(view)} ?: view.showError() 

// 그나마 나은 경우

fun Person.adultProcess(view): Unit{
		if (isAdult)
			view.showPerson(this)
		else
			view.showError()
}
person?.let{it.adultProcess(view)}
```