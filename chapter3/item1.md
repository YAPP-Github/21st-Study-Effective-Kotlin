# 1. knowledge를 반복하여 사용하지 말라.
> &nbsp; 프로그래밍에서 ``knowledge``는 프로젝트 진행 시에 정의한 모든 것을 나타낼 수 있습니다. 알고리즘 작동 방식부터 UI 형태, 결과 등 모두 knowledge로 지칭할 수 있으며 그 중 가장 중요한 knowledge는 아래로 볼 수 있습니다.
- 로직(business logic) : 프로그래밍이 어떠한 방식으로 동작하는지를 나타내는 흐름
- 공통 알고리즘(common algorithm) : 원하는 동작을 하기 위한 일련의 절차

로직과 공통 알고리즘을 구별하자면 ``시간에 따른 변화``로 볼 수 있습니다. 비즈니스 로직은 시간이 지나면서 계속 변화하지만, 공통 알고리즘은 한 번 정의된 이후에 크게 변하지 않습니다. 여기서 만일 로직(knowledge) 스니펫을 다양한 곳에서 반복해서 사용한다면 어떤 문제가 발생할까요?

- 변경이 필요 시 모든 변경점을 찾아 수정하는데 시간을 많이 소요됩니다.
- 변경 이후에 정상적인 동작을 하는지 테스트를 하는 과정에서 불필요하게 많은 테스트를 진행해야 합니다.
- 모든 곳을 동일하게 수정하지 않아 예기치 못한 예외가 발생할 수도 있습니다.

이처럼 반복은 프로젝트의 확장성을 막고, 예외를 만들 수 있기에 공통된 부분은 ``추상화``를 사용하도록 해야 합니다.

<br>

## 반복된 코드 사용
무작정 추상화를 적용하는 것은 좋은 방향은 아닙니다. 얼핏보면 공통 부분을 추상화할 수 있겠지만, 더 세밀하게 확인했을 경우 공통으로 추출할 수 없는 경우도 있습니다. 추상화로 나타낼지 여부를 쉽게 알기는 어렵지만 책에서 소개해준 휴리스틱으로는 ``비즈니스 규칙이 다른곳에서 왔는지 확인``하라고 권장하고 있습니다. 비즈니스 로직이 다른 곳에서부터 왔다면 독립적으로 변경될 가능성이 높습니다. 

<br>

## 단일 책임 원칙(SRP)
> 단일 책임 원칙(Single Responsibility Principle, SRP)은 로버트 C.마틴의 클린 아키텍처 원칙인 SOLID 중 하나로 소개되고 있습니다. 단일 책임 원칙이란 ``클래스를 변경하는 이유는 단 한 가지여야 한다``라는 의미입니다. 

```kotlin
class Student{
	
    fun isPassing(): Boolean =
    	calculatePointsFromPassedCoures() > 15
        
    fun qualifiesForScholarship(): Boolean = 
    	calculatePointsFromPassedCourses() > 30
        
    private fun calculatePointsFromPassedCourses(): Int {
    	// ..
    }
}
```
Student 클래스에서는 2가지 메소드를 정의하였고 두 메소드 모두 calculatePointsFromPassedCourses 라는 메소드의 결과인 이전 학기 성적 기반으로 계산됩니다.
- isPassing : 학생이 인증을 통과했는지 여부
- qualifiesForScholarship : 학생이 장학금을 받을 수 있는지 여부

위 코드처럼 작성 시에는 문제가 발생하는데 장학금 받을 수 있는 로직을 더 어렵게 하기 위해 calculatePointsFromPassedCourses 메소드를 변경하였을 때 isPassing 메소드까지 영향을 받게 되는 것입니다. 
<br>

```kotlin
	// accreditations module
    fun Student.qualifiesForScholarship(): Boolean{
    
    }
    
    // scholarship module
    fun Student.calculatePointsFromPassedCourses(): Boolean{
    
    }
```
만일 위 스니펫과 같이 isPassing, qualifiesForScholarship의 책임을 각각 확장함수를 정의하였다면 문제가 없었을 것입니다. 정리하자면 이번 아이템에서 재사용성을 높이기 위한 방법으로 공통된 부분은 ``추상화``를 사용하고 그렇지 않은 부분에 있어서는 ``단일 책임 원칙(SRP)``을 지켜가며 균형있게 사용하는 것을 권장하고 있습니다.