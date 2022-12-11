# 2. 일반적인 알고리즘을 반복해서 구현하지 말라.
> &nbsp; 동일한 알고리즘을 여러 번 반복해서 구현하지 말고 표준 라이브러리 사용하거나 특정 알고리즘을 반복해서 사용해야 하는 경우에 직접 정의하여 사용하는 것을 권장하고 있습니다.

<br>

## 표준 라이브러리 사용

표준 라이브러리를 사용할 경우에 단순히 코드가 짧아지는 것 이외에도 다양한 장점이 있습니다.
- 코드 작성 속도 증가
- 함수의 이름으로 무엇을 하는지 확실하게 알 수 있음
- 직접 구현 시 실수 가능성 존재

```java
	override fun saveCallResult(item: SourceResponse){
    	var sourceList = ArrayList<SourceEntity>()
        item.sources.forEach{
        	var sourceEntity = SourceEntity()
            sourceEntity.id = it.id
            sourceEntity.category = it.category
            sourceEntity.country = it.country
            sourceEntity.description = it.description
            sourceList.add(sourceEntity)
        }
        db.insertSources(sourceList)
    }
```
위 스니펫은 다른 자료형으로 매핑하는 처리 후에 DB에 추가해주고 있습니다.  
<br>

```kotlin
	override fun saveCallResult(item: SourceResponse){
    	val sourceEntries = item.sources.map(::sourceToEntry)
        db.insertSOurces(sourceEntries)
    }
    
    private fun sourceToEntry(source: Source) = SourceEntity()
    	.apply{
     		id = source.id
            category = source.category
            country = source.country
            description = source.description
    }
```
이를 표준라이브러리에 있는 map 함수를 사용하고 팩토리 메소드를 활용하거나 기본 생성자를 활용하면 더 편리하게 사용할 수 있습니다. 

<br>

## 나만의 유틸리티 작성
상황에 따라 표준 라이브러리에 없는 알고리즘을 직접 구현해야할 필요가 있습니다. 
```kotlin
	fun Iterable<Int>.product() = 
    		fold(1) { acc, i -> acc*i }
```
여러번 사용하지 않더라도 함수로 만드는 것을 권장하고 있으며 함수 네이밍으로도 어떤 동작을 할 것인지 대략적으로 유추가 가능합니다. 또한 위 코드처럼 확장함수를 사용하여 유틸리티를 작성하는데 top-level 함수로 정의하는 것과 다른 장점이 존재합니다.

- 함수는 상태를 유지하지 않으므로 행위를 나타내기 좋음
- 구체적인 타빙이 있는 객체에만 사용을 제한
- 수정할 객체를 매개변수로 받는 것보다는 확장 리시버로 사용하는 것이 가독성 좋음
- 함수의 시그니처를 찾기 편함