# 7. 결과 부족이 발생할 경우 null과 Failure를 사용하라.
> &nbsp; 함수 호출 시에 항상 기대하는 결과만 만들어내지는 않습니다. 파일 읽기/쓰기 실패, 네트워크 통신, 파싱 에러 등 예기치 않은 상황에서 처리하는 메커니즘을 소개합니다.

이 상황을 처리하는 대표적인 방법은 크게 2가지가 있습니다. 

- 충분히 예측할 수 있는 범위의 오류는 ``null`` 또는 ``Failure(sealed class)`` 를 리턴한다.
- 예측하기 어려운 예외적인 범위의 오류는 예외를 throw 해서 처리하는 것이 좋다

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
	// ...
    if(incorrectSign){
	return null    
    }
    
    // ...
    return result
}

inline fun <reified T> String.readObject(): Result<T>{
	// ...
    if(incorrectSign){
    	return Failure(JsonParsingException())
    }
    //..
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```
위 코드와 같이 ``null``과 ``Failure``로 예상되는 오류를 표현할 때는 명시적으로 처리할 수 있고 놓치기 어렵습니다. ``null``로 반환할 경우에는 Elvis 연산자를 활용하여 null-safety 기능도 활용할 수 있다는 장점도 있습니다. 

- ``null`` : 추가적인 정보를 전달할 일이 없을 경우는 단순하게 null을 반환
- ``Failure`` : Throwable 과 같은 추가적인 정보 전달이 필요할 경우에 사용

