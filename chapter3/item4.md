# 4. 일반적인 알고리즘을 구현할 때 제네릭을 사용하라.
> &nbsp; 일반적인 알고리즘을 직접 구현 시에 타입 파라미터를 사용한 Generic function을 구현하는 것이 좋습니다. 

```kotlin
	inline fun <T> Iterable<T>.filter(
    	predicate: (T) -> Boolean
    ): List<T>{
    	val destination = ArrayList<T>()
        for(element in this){
        	if(predicate(element)){
            	destination.add(element)
            }
        }
    }
```
대표적인 Generic function으로 stdlib 라이브러리에 있는 filter 함수가 있습니다. 타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 타입을 조금이라도 더 정확하게 추측할 수 있게 합니다.

<br>

## 제네릭 제한
타입 파라미터의 중요한 기능 중 하나는 구체적인 타입의 서브타입 또는 슈퍼타입을 사용하도록 타입을 제한할 수 있습니다. 
```kotlin
	fun <T: Comparable<T>> Iterable<T>.sorted(): List<T>{
    
    }
    
    fun <T, C: MutableCollection<in T>>
    Iterable<T>.toCollection(destination: C): C{
    
    }
    
    class ListAdapter<T: ItemAdapter>(){}
    
    inline fun <T, R: Any> Iterable<T>.mapNotNull(
    	transform: (T) -> R?
    ): List<R>{
    	return mapNotNullTo(ArrayList<R>(), transform)
    }
```
타입에 제한으로 인해 해당 타입이 제공하는 메소드를 사용할 수 있습니다. 위 예시에서 T를 Iterable< Int >로 제한을 할 경우 반복구문을 사용할 수 있으며 Comparable< T >로 제한 시에는 타입끼리 비교도 가능합니다. 많이 사용하는 패턴으로 Any로 제한을 하여 타입을 notnull하도록 만들 수 있습니다.