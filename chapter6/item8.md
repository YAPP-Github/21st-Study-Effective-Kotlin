# 아이템 8 : API의 필수적이지 않는 부분을 확장 함수로 추출하라

### 요약

“두 방식 중에 어떤 것이 우월하다고 말할 수 없다”

API의 필수적인 부분 → 멤버

필수가 아님 → 확장함수

### 확장 함수

- **직접 멤버를 추가할 수 없는, 데이터와 행위를 분리하도록 설계된 프로젝트에서 사용**
- **같은 타입, 같은 이름으로 여러 개 만들 수 있음**
    - 위험 가능성이 존재하기에, 멤버 함수로 사용하지 않는 편이 나음
- **파생 클래스에서 override 할 수 없음 - `virtual이 아님`**
    - 컴파일 시점에 정적으로 선택되기 때문
- **클래스 레퍼런스에서 멤버로 표시되지 않음 - `멤버가 높은 우선순위를 가짐`**
    - 어노테이션 프로세서가 처리하지 않음
    

### 번외 - 확장함수는 어떻게 생성되는가?(면접 질문)

**Kotlin**

```kotlin
fun Int.toHourString() = if (this < 12) {
    this.toString()
} else {
    (this - 12).toString()
}

fun main(){
    val a = 12.toHourString()
}
```

**Java Decompile**

```java
public static final String toHourString(int $this$toHourString) {
   return $this$toHourString < 12 ? String.valueOf($this$toHourString) : String.valueOf($this$toHourString - 12);
}

public static final void main() {
   String a =toHourString(12);
}

```

정적 메서드로 생성되고, 인수 부분에 this의 값을 전달하여 호출한다.
