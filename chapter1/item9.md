# 9. use를 사용하여 리소스를 닫아라
> &nbsp; AutoClosable을 상속받는 Closable 인터페이스를 구현하는 리소스들은 더 이상 필요로 하지 않을 때 명시적으로 close를 해야 합니다. 
- InputStream, OutputStream
- java.sql.Connection
- java.io.Reader(FileReader, BufferedReader, CSSParser)
- java.new.Socket, java.util.Scanner

이런 리소스들 Root Space에서 리소스에 대한 레퍼런스가 없어질 때 GC가 동작을 하는데 굉장히 느리며 리소스를 유지하는 비용이 많이 들어갑니다. 따라서 명시적으로 close 메소드를 호출하는 것이 좋습니다.

```kotlin
fun countCharactersInFile(path: String): Int{
	val reader = BufferedReader(FileReader(path))
	try{
		return reader.lineSequence().sumBy { it.length }	
	}finally{
		reader.close()
	}
}
```
일반적으로 리소스를 사용하고 닫을 때는 try-catch 블록을 사용하게 됩니다. 하지만 위 코드는 복잡하고 리소스를 닫을 때 IOException이 발생할 수 있는데 해당 예외를 별도로 처리하지 않습니다. 
<br>

```kotlin
fun countCharactersInFile(path: String): Int{
	BufferedReader(FileReader(path)).use { reader ->
		return reader.lineSequence().sumBy { it.length }
	}
}
```
다행히 코틀린에서는 간결하면서도 try-catch에서 처리하지 못한 예외까지 처리해줄 수 있는 ``use`` 메소드를 제공하고 있습니다. ``use`` 메소드는 Closable을 구현하는 모든 객체에서 사용할 수 있습니다.

