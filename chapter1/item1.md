# 아이템 1 : 가변성을 제한하라

### 왜 가변성을 제한해야 하는가?

- 코틀린의 요소(클래스, 객체, 함수) 중 일부는 **상태**를 가질 수 있음
    - 읽기, 쓰기가 가능한(read-write property) 프로퍼티 `var`, `mutable` 객체
- 이는 **양날의 검**임. **가변성**을 가지는 요소들을 적절하게 관리하는 것은 **어려움**
    - 디버그, 코드 실행 추론이 어려움
    - 동기화 문제가 발생할 수 있음
    - 테스트가 어려움
    - 상태 변경을 알려야 함
- **가변성**은 **시스템의 상태를 나타내기 위한 중요한 방법**임
    - 변경이 일어날 부분을 신중하고 확실하게 결정해야 함

### 코틀린에서 가변성 제한하기

- 읽기 전용 프로퍼티(`val`)
    - 불변을 의미하는 것은 아님
    - `val` 옆에 **상태가 적혀서 초기화**되므로, **코드 실행 예측에 용이함**
       ![Untitled 1](https://user-images.githubusercontent.com/70064912/201721458-e7fba2da-a113-4697-bb90-0ef5910467c8.png)

        
        **게터로 정의했을경우 스마트캐스트 불가능 예제**
        
    - `val`에 mutable 객체를 담는다면 내부적으로 변할 수 있는 것에 주의
    
- 가변 컬렉션, 읽기 전용 컬렉션 구별
    - ![Untitled 2](https://user-images.githubusercontent.com/70064912/201721597-8a9ca0cf-c61a-4b9c-960b-52e53c68a10b.png)
    
    - 코틀린의 컬렉션은 위 의존성을 가지며, **읽기 전용, 읽고 쓰기가 가능한 컬렉션**으로 나뉘어짐
    - **읽기 전용 컬렉션이 내부 값을 변경할 수 없다는 의미는 아님. 인터페이스가 이를 지원하지 않음**
        
       ![Untitled 3](https://user-images.githubusercontent.com/70064912/201721617-c93cea82-1729-40b7-87df-ae3d95a78e29.png)

       ![Untitled 4](https://user-images.githubusercontent.com/70064912/201721629-755d5ce2-3b08-4503-994d-34763f728d47.png)

     
    - `map`은 **내부적으로 `ArrayList`로 연산**되지만, **List로 캐스팅되어 반환**되기 때문에 **읽기 전용 메서드만 사용할 수 있음**
    - 내부적으로 인터페이스를 사용하므로 **플랫폼 고유의 컬렉션**을 사용할 수 있게 됨
    
- 시스템 해킹(다운캐스팅 금지)
    - **추상화를 무시하는 행위**이며, 안전하지 않고 예측하지 못한 결과를 초래
    
   ![Untitled 5](https://user-images.githubusercontent.com/70064912/201721649-a80d2149-a20f-4100-9787-e8f4a7c99aee.png)

   ![Untitled 6](https://user-images.githubusercontent.com/70064912/201721678-fa8a6d4f-c72c-4476-9f95-a91aef248721.png)
  
    - `toMutableList`를 통해 새로운 mutable 컬렉션을 생성하는 방식이 **안전함**

- 데이터 클래스의 `copy`
    - **Immutable한 객체의 장점**
        - 코드를 이해하기 쉬움
        - 병렬처리가 안전함
        - 쉽게 캐시할 수 있음
        - 방어적 복사본(새로운 객체를 생성하여 반환)을 만들 필요가 없음
        - 다른 객체를 만들 때 활용하기 좋음
    - String, Int등 기본적으로 **Immutable**하고, 변경할 수가 없음
    - 따라서 **기존의 객체의 일부를 수정하고 새로운 객체를 만드는 메서드**를 가짐(`*plus, minus, map, filter*`) 등
    - **임의의 모델 클래스를 만들 때, `val`로 변수를 선언하고, `data calss`로 만들어 `copy` 메서드를 활용하는 것이 좋음**

- 변경 가능 지점 노출하지 말기
    - **상태를 나타내는 mutable 객체를 노출하는 것**은 **돌발적인 수정이 일어날 수 있음**
    - **해결책1** : 방어적 복제
        
      ![Untitled](https://user-images.githubusercontent.com/70064912/201721707-9251ae35-9187-4176-a581-312b9b8b6d2f.png)

        
    - **해결책2** : 읽기 전용 슈퍼타입으로 캐스팅
    
    ```jsx
    private val _user = MutableLiveData<User>()
    val user :LiveData<User> = _user
    ```
    
    - **ViewModel에서 LiveData를 observe할 때, 위와 같은 방식으로 사용하는 것도 위 이유와 같음**

## 정리

- `var`보다는 `val`
- mutable보다는 immutable
- 상태를 바꾸기 위해서 `copy`를 통해 방어적 복제
- 읽기 전용 컬렉션을 사용하자
- 변이 지점을 적절하게 설계하자
- mutable 객체를 노출하지 말자
