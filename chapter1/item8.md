# 8. 적절하게 null을 처리하라.

> &nbsp; ``String.toIntOrNull()``, ``Iterable<T>.firstOrNull(() -> Boolean)`` 메소드와 같이 null을 반환하더라도 명확한 의미를 갖으면 이를 처리하는 개발자는 편리하게 처리할 수 있습니다. Kotlin에서 nullable 타입을 처리하는 방법은 크게 3가지 방법이 있습니다.
  - ?., 스마트 캐스팅,  Elvis 연산자 등을 활용
  - Exception throw
  - 함수, 프로퍼티 리팩토링하여 nullable 타입이 나오지 않게 처리

## null 안전하게 처리

### 안전 호출,  Elvis 연산자 활용  
```kotlin
  printer?.print()	// 안전호출
  if (printer != null) printer.print()	// 스마트 캐스팅
```
printer가 null이 아닐 경우에만 print 메소드를 호출하는 코드로 애플리케이션 사용자 관점에서 안전하고 개발자에게도 편리하여 자주 활용됩니다.

```kotlin
  val printerName1 = printer?.name ?: "Unnamed"
  val printerName2 = printer?.name ?: return
  val printerName3 = printer?.name ?:
  		throw Error("Printer must be named")
```
안전 호출과 비슷하게 자주 사용되는 방법은 Elvis 연산자를 활용하여 return, throw 로  처리하는 것입니다.
  
<br>

### 스마트 캐스팅
많은 클래스에서 nullable 관련 처리를 제공합니다. 예를 들어 컬렉션 처리에 Element가 없다는 것을  null보다는 비어 있는 컬렉션을 제공해주는 메소드``Collection<T>?.orEmpty()``가 존재합니다.
```kotlin
  println("What is your name?")
  val name = readLine()
  if(!name.isNullOrBlank(){
  println("Hello ${name.toUpperCase()}")
  
  val new: List<News>? = getNews()
  if(!news.isNullOrEmpty()){
  	news.forEach { notifyUser(it) }
  }
```
위 코드에서  처리 한번 null check한 이후에 스마트 캐스팅으로 인해 null을 안전하게 처리할 수 있습니다.
  
<br>

## 오류 throw
```kotlin
  fun process(user: User){
  	requireNotNull(user.name)
  	val context = checkNotNull(context)
  	val networkService = getNetworkService(context) ?: throw NoInternetConnection()
  	networkService.getData {data, userData -> show(data!!, userData!!)
  }
```
안전 호출에서 null로 인해 실행이 안되는 경우에 대해 고려하지 못했을 경우에는 예상되는 동작이 수행이 되지 않아 이상하게 생각이 될 수 있습니다. 원인도 찾기 어려울 수 있는데 이 경우에는 개발자에게 오류를 강제로 발생시켜 주는 편이 좋습니다. ``throw``,   ``!!``, ``requireNotNull``, ``checkNotNull``을 활용하시면 됩니다.  
  
<br>

## not-null assertion(!!) 주의사항
> null 이 절대 아닐 것이라고 확증할 때 사용되는 ``!!``은 미래에 코드가 어떻게  변할지 모르기 때문에 절대 권장되지 않습니다. 외부 라이브러리를 사용할 때 정도에만 고려해볼 수는  있습니다.
  
<br>

## 의미 없는 nullability 피하기
> nullability는 null 처리에 대한 오버헤드가 필요하기에 최대한 nullability를 피하는게 좋을 수 있습니다. 이를 위한 대표적인 방법 몇 가지만 소개하도록 하겠습니다.
  - 여러 클래스의 nullability 처리를 지원해주는 메소드 적극 활용
  ex) ``List<T>.get()``, ``List<T>,getOrNull()``
  - lateinit, notNull Delegate 사용하여 지연 초기화
  - 빈 컬렉션 대신 null 리턴 금지
  - enum 사용 시 null보다는 내부에 None 정의하여 활용
  
<br>

## lateinit 프로퍼티와 notNull 델리게이트
```kotlin
  class UserControllerTest{
  	private var dao: UserDao? = null
  	private var controller: UserController? = null
  
  	@BeforeEach
  	fun init(){
  	    dao = mockk()
  	    controller = UserController(dao!!)
  	}
  
  	@Test
  	fun test(){
  	    controller!!.doSomething()
  	}
  }
```
위 코드처럼 프로퍼티를 사용할 때마다 ``!!``을 사용하는 것은 바람직하지 않습니다. JUnit에서 @BeforeEach 처럼 다른 함수들보다 먼저 호출되는 함수에서 프로퍼티를 초기화할 경우에는 null로 초기화하기 보다는 lateinit  을 사용하는 것이 더 좋습니다.
<br>

```kotlin
  class UserControllerTest{
  	private lateinit var dao: UserDao
  	private lateinitvar controller: UserController
  
  	@BeforeEach
  	fun init(){
  	    dao = mockk()
  	    controller = UserController(dao)
  	}
  
  	@Test
  	fun test(){
  	    controller.doSomething()
  	}
  }
```
lateinit은 프로퍼티를 처음 사용하기 전에 무조건 초기화 될 것이라고  예상되는  상황에서 주로 사용됩니다. Android, iOS에서 라이프 사이클(lifecycle) 갖는 클래스처럼 메소드 호출에 명확한 순서가 있는 경우가 대표적입니다. 
  
```kotlin
  class DoctorActivity:Activity() {
  	private var doctorId: Int by Delegates.notNull()
  	private var formNotification: Boolean by Delegates.notNull()
  	
  	overrid fun onCreate(savedInstanceState: Bundle?){
  		super.onCreate(savedInstanceState)
  		doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
  		fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
  	}
  }
```
lateinit은 primitive type을 사용할 수 없습니다. 이 경우에는 lateinit보다 조금은오버헤드가 있지만 Delegates.notNull을 사용하시면 됩니다. lateinit이 Delegates.notNull보다 성능상 이점이 있는 이유는 아래 글로 대체합니다.
  
- 참고 : https://codechacha.com/ko/diff-between-deligate-and-lateinit-in-kotlin/

