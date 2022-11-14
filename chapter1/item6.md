# 아이템 6 : 사용자 정의 오류보다는 표준 오류를 사용하라

### 표준 오류를 사용하라

- 직접 오류를 정의하는 것 보다, 표준 라이브러리의 오류를 재사용하는 것이 좋음
- 이미 알려진 오류이고, 규약이기에 API를 더 쉽게 배우고 이해할 수 있음

### 일반적으로 사용되는 예외

- **`IllegalArgumentException`, `IllegalStateException`**
    - **적합하지 않거나(illegal) 적절하지 못한(inappropriate) 인자**를 메소드에 넘겨주었을 때 발생
- **`IndexOutOfBoundsException`**
    - **인덱스가 범위를 넘어갔을 경우**, 일반적으로 컬렉션 또는 배열과 함께 사용됨
- **`ConcurrentModificationException`**
    - **동시 수정을 금지했는데, 발생**했을 경우
- **`UnsupportedOperationException`**
    - 사용자가 사용하려고 했던 메서드가 **현재 객체에서 사용할 수 없을 경우** 발생
- **`NoSuchElementException`**
    - 사용자가 사용하려고 했던 요소가 **존재하지 않음**
