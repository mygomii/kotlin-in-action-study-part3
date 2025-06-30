# 📌 코루틴과 플로우를 활용한 동시성 프로그래밍

<details>
<summary><strong>14.1 동시성과 병렬성</strong></summary>
  
- 동시성은 여러 작업을 동시에 실행하는 것
- 하지만 모든 작업을 물리적으로 함계 실행할 필요는 없음
- 코드의 여러 부분을 돌아가면서  실행하는 것도 동시성 시스템
- CPU 코어가 하나뿐인 시스템에서 실행되는 애플리케이션까지도 동시성을 사용할 수 있다는 듯
- 이런 경우 여러 동시성 태스크를 계속 전환해 가면서 동시성을 달성

- 병렬성은 여러 작업을 여러 CPU 코어에서 물리적으로 동시에 실행하는 것을 말함
- 병렬 계산은 현대적 멀티코어 하드웨어를 효과적으로 사용할 수 있고, 그 효율을 더 높이는 경우도 많음
</details>

<details>
<summary><strong>14.2 코틀린의 동시성 처리 방법: 일시 중단 함수와 코루틴 </strong></summary>
  
- 코루틴은 코틀린의 강력한 특징으로 비동기적으로 실행되는 넌블로킹 동시성 코드를 우아하게 작성할 수있게 해줌
- 스레드와 같은 전통적 방법과 비교하면 코루틴이 훨씬 가볍게 작동
- 구조화된 동시성을 통해 코루틴은 동시성 작업과 그 생명주기를 관리할 수 있는 기능도 제공
</details>

<details>
<summary><strong>14.3 스레드와 코루틴 비교</strong></summary>
  
### **스레드(Thread)**

- 운영체제(OS) 단위의 동시성 실행 단위
- 각 스레드는 자체 스택 메모리를 사용
- 스레드는 생성 비용이 높음 (수 ms~수십 ms)
- 수천 개 이상의 스레드 생성은 메모리, 스케줄링 비용 측면에서 한계가 있음

### **코루틴(Coroutine)**

- 언어 단위의 동시성 실행 단위 (Kotlin 언어 레벨에서 제공)
- 스레드보다 가볍고 효율적
    - 단일 스레드 위에서 수만 개의 코루틴 동시 실행 가능
    - 코루틴은 **스레드 풀** 또는 메인 스레드 위에서 동작
- 컨텍스트 스위칭 비용이 낮음 (스레드와 달리 OS 개입이 거의 없음)
- suspend/resume로 상태 저장 및 재개 → 비동기 작업에 적합

| **항목** | **스레드(Thread)** | **코루틴(Coroutine)** |
| --- | --- | --- |
| 생성 비용 | 높음 | 낮음 |
| 실행 단위 | OS 단위 | 언어 단위 (Kotlin) |
| 개수 | 수천 개 한계 | 수만 개 가능 |
| 컨텍스트 전환 비용 | 높음 | 낮음 |
| 비동기 지원 | 직접 관리 (callback, Future 등) | 언어 차원 (suspend, launch) |

</details>


<details>
<summary><strong>14.4 잠시 멈출 수 있는 함수: 일시 중단 함수</strong></summary>

- 코틀린 코루틴이 스레드, 반응형 스트림, 콜백과 같은 다른 동시성 접근 방식과 다른 핵심 속성으로 상당수의 경우 코드 형태를 크게 변경할 필요가 없다는 점

## 14.4.1 일시 중단 함수를 사용한 코드는 순차적을 보인다

- 코루틴의 `일시 중단 함수(suspend 함수)`를 사용하면, 코드가 **비동기 작업임에도 마치 동기적이고 순차적인 코드처럼 보임**
- 콜백 기반의 코드(콜백 지옥)나 반응형 스트림 코드와 달리, 코루틴의 `suspend` 함수는 **중단과 재개**가 자연스럽게 처리되므로 코드의 가독성이 높아짐

```kotlin
// 콜백 기반
api.fetchData { result ->
    process(result) {
        updateUI(it)
    }
}
```

```kotlin
// 코루틴
val data = api.fetchData()
val processed = process(data)
updateUI(processed)
```

</details>


<details>
<summary><strong>14.5 코루틴을 다른 접근 방법과 비교 </strong></summary>

- 자바나 다른 프로그래밍 언어에서 동시성 코드를 작성하느 다른 접근 방식을 사용한 경험이 있다면 이들과 코루틴이 어떻게 다른지, 또 코루틴이 어떻게 더 나은지 확인하고 싶을 거임

```kotlin
// 콜백을 써서 여러 함수를 연속적으로 호출
fun fetchData(callback: (String) -> Unit) 
fun processData(data: String, callback: (String) -> Unit) 
fun displayResult(result: String) 

fun main() {
    fetchData { data ->
        processData(data) { processed ->
            displayResult(processed)
        }
    }
}
```

- 이런 예제는 콜백 지옥이라는 별명으로 널리 알려져 있음

```kotlin
// 퓨쳐를 사용해 여러 함수를 연속적을 호출 
fun fetchData(): CompletableFuture<String> 
fun processData(data: String): CompletableFuture<String> 
fun displayResult(result: String) 

fun main() {
    fetchData()
	    .thenCompose { data -> processData(data) }
	    .thenAccept { processed -> displayResult(processed) }
}
```

```kotlin
// 반응형 스트림을 사용해 같은 로직 구현하기 
fun fetchData(): Single<String> 
fun processData(data: String): Single<String> 
fun displayResult(result: String)

fun main() {
    fetchData()
	    .flatMap { data -> processData(data) }
	    .subscribe { processed ->
		    displayResult(processed)
      }
}
```

- 두 접근 방식 모두 인지적 부가 비용이 있고, 함수를 선언하거나 사용할 때 새로운연산자를 코드에 도입해야함
- 이와 비교해보면 코틀린. 코루틴을 사용하는 접근 방식에서는 함수에 `suspend` 변경자만 추가하면 됨
- 나머지 코드는 그대로 순차적인 모양을 유지하면서도 여전히 스레드를 블록시키는 단점을 피할 수. ㅣㅆ음

## 14.5.1 일시 중단 함수 호출

```kotlin

suspend fun fetchData(): String 
suspend fun processData(data: String): String 
fun displayResult(result: String) 

fun main() = runBlocking {
    val data = fetchData()
    val processed = processData(data)
    displayResult(processed)
}
```

- 일시 중단 함수는 실행을 일시 중단할 수 있기 때문에 일반 코드 아무 곳에서나 호출 할 수 없음
- 일시 중단 함수는 일시 중단할 수 있는 코드 블록 안에서만 호출할 수 있음
</details>


<details>
<summary><strong>14.6 코루틴의 세계로 들어가기: 코루틴 빌더</strong></summary>

- 코루틴에서 일시 중간 함수를(suspend)를 호출하는 것을 알아보자
- `runBlocking` : 블로킹 코드와 일시 중단 함수의 세계를 연결할 때 쓰임
- `launch` : 값을 반환하지 않는 새로운 코루틴을 시작할때 쓰임
- `async` : 비동기적으로 값을 계산할 때 쓰임

## 14.6.1 일반 코드에서 코루틴 세계로: runBlocking 함수

- `runBlocking`은 일반 코드(메인 함수, 테스트 코드)에서 코루틴을 실행할 때 사용
- 코루틴이 완료될 때까지 **현재 스레드를 블로킹(blocking)**
- 테스트나 간단한 예제 코드에서 코루틴을 호출할 때 유용

```kotlin
fun main() = runBlocking {
    // 코루틴 안에서 suspend 함수 호출 가능
    fetchData()
}
```

## 14.6.2 발사 후 망각 코루틴 생성: launch 함수

- `launch` 는 코루틴을 실행하고 **값을 반환하지 않음** (Job 객체 반환)
- 비동기 작업을 **“발사 후 잊어버리기”** 스타일로 처리
- 예: 화면 갱신, 로깅, 이벤트 처리 등에 적합

```kotlin
launch {
   fetchData()
}
```

## 14.6.3 대기 가능한 연산: async 빌더

- `async`는 코루틴을 실행하고 **`Deferred`** 객체를 반환
- **비동기 작업의 결과를 나중에 받아서 사용할 수 있음**
- `.await()`를 호출해 결과를 가져옴
- 
</details>

<details>
<summary><strong>14.7 어디서 코드를 실행할지 정하기: 디스패처 </strong></summary>
	
- 코루틴의 디스패처는 코루틴을 실행할 스레드를 결정함
- 본질적으로 코루틴은 특정 스레드에 고정되지 않음
- 코루틴은 한 스레드에서 실행을 일시중단하고 디스패처가 지시하는 대로 다른 스레드에서 실행을 재가할 수 있음

---

## 스레드 풀(`Thread pool`)이란?

- 스레드 집합을 관리하고, 집합에 속한 스레드를 웨에서 작업(우리의 경우 코루틴) 실행을 허용
- 작업이 실행될 때마다 새 스레드를 할당하는 대신, 스레드 풀은 일정한 수의 스레드를 유지하면서 내부 논리와 구현에 따라 들어오는 작업을 분배
- 스레드를 새로 생성해 할당하고 시작하는 작업은 비용이 많이 들기 때문

## 14.7.1 디스패처 선택

- 코루틴은 기본적으로 부모 코루틴에서 디스패처를 상속 받으므로 모든 코루틴에 대해 명시적으로 디스패처를 지정할 필요 없음
- 선택할 수 있는 디스패처들이 있음
    - 코루틴을 기본 환경에서 실행할때 (`Dispatchers.Default`)
    - UI 프레임워크와 함께 작업할 때 (`Dispatchers.Main`)
    - 스레드를 블로킹하는 API를 사용할때 (`Dispatchers.IO`)

- 다중 스레드를 사용하는 범용 디스패처:  `Dispatchers.Default`
    - CPU 연산 집중적인 작업 (예: 계산, 정렬, 데이터 처리 등)에 적합
    - 기본적으로 CPU 코어 수에 맞춰 스레드 풀 생성
    - 예시: 데이터 파싱, 복잡한 알고리즘 실행
    
    ```kotlin
    fun main() = runBlocking {
        launch(Dispatchers.Default) {
            println("Default Dispatcher: ${Thread.currentThread().name}")
            val sum = (1..1_000_000).sum()
            println("Sum: $sum")
        }
    }
    ```
    
- UI 스레드에서 실행: `Dispatchers.Main`
    - Android나 JavaFX/Swing에서 UI 업데이트나 사용자 인터랙션 처리
    - 화면 그리기, 뷰 변경, 사용자 입력 처리 등
    - 예시: 버튼 클릭 리스너에서 API 호출 후 UI 반영
    
    ```kotlin
    class MainActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
    
            CoroutineScope(Dispatchers.Main).launch {
                // UI 스레드에서 실행됨
                println("Main Dispatcher: ${Thread.currentThread().name}")
                // 예: 버튼 클릭 후 UI 갱신
            }
        }
    }
    ```
    
- 블로킹되는 IO 작업 처리: `Dispatchers.IO`
    - 파일, 네트워크, 디스크, 데이터베이스 등 입출력 작업
    - 블로킹 호출이 많은 작업에 적합
    - 많은 수의 스레드를 동적으로 생성하여 효율적으로 처리
    
    ```kotlin
    fun main() = runBlocking {
        launch(Dispatchers.IO) {
            println("IO Dispatcher: ${Thread.currentThread().name}")
            val content = readFile("example.txt")
            println("파일 내용:\n$content")
        }
    }
    
    suspend fun readFile(path: String): String {
        // 파일 읽기 (Blocking)
        return File(path).readText()
    }
    ```
    

| **Dispatcher** | **특징** | **예시** |
| --- | --- | --- |
| Dispatchers.Default | CPU 연산 집중 작업 | 리스트 합계, 데이터 분석 |
| Dispatchers.Main | UI 스레드 (Android) | 버튼 클릭, TextView 갱신 |
| Dispatchers.IO | 블로킹 I/O 작업 | 파일 읽기, 네트워크 |

## 14.7.2 코루틴 빌더에 디스패처에 전달

- 코루틴 빌더(`launch`, `async`, `runBlocking`)에 **디스패처를 직접 전달**하여 해당 코루틴이 어떤 스레드에서 실행될지 명확히 지정할 수 있음
- 이렇게 하면 특정 작업이 CPU 연산인지, UI 작업인지, I/O 작업인지에 따라 적절한 디스패처를 선택해 효율적으로 실행할 수 있음

## 14.7.3 withContext를 사용해 코루틴 안에서 디스패처 바꾸기

- 코루틴 안에서 다른 디스패처로 작업을 실행해야 할 때는 `withContext()`를 사용해야함
- `withContext()`는 **중단점(`suspend point`)을 제공**하며, 지정한 디스패처에서 실행한 후 결과를 반환함
- ex)
    - UI에서 네트워크 호출이 필요할 때, 메인(UI) 디스패처에서 코루틴이 실행 중이라면,
    - 네트워크 호출은 Dispatchers.IO에서 실행하도록 스위칭하고,
    - 결과를 받아서 UI 업데이트는 다시 Dispatchers.Main으로 돌아가면 됨

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch(Dispatchers.Main) {
        val data = withContext(Dispatchers.IO) {
            fetchData()
        }
        updateUI(data)
    }
}

suspend fun fetchData(): String {
    delay(1000) // 네트워크 호출 시뮬레이션
    return "data from server"
}

fun updateUI(data: String) {
    println("UI 업데이트: $data")
}
```

- `fetchData()`는 I/O 디스패처에서 실행됨.
- `updateUI()`는 다시 메인(UI) 디스패처로 돌아옴.

## 14.7.4 코루틴과 디스패처는 스레드 안전성 문제에 대한 마법 같은 해결책이 아니다

- **코루틴과 디스패처**는 여러 스레드 간의 작업 분배를 쉽게 해주지만, **스레드 안전성 자체를 보장하지는 않음**
- 공유된 가변 상태(예: 변수, 컬렉션)에 접근할 때는 여전히 **적절한 동기화**가 필요함
- 예를 들어 `Dispatchers.Default`로 실행되는 여러 코루틴이 동시에 같은 변수에 접근하면 `Race Condition(경쟁 상태)`이 발생할 수 있다.
</details>


<details>
<summary><strong>14.8 코루틴은 코루틴 콘텍스트에 추가적인 정보를 담고 있다.</strong></summary>

- `코루틴 컨텍스트(CoroutineContext)`는 코루틴의 실행 환경을 정의하는 메타정보를 담고 있음
- 이 컨텍스트에는 디스패처뿐만 아니라 **`Job`, `CoroutineName`, 예외 처리자 등 추가 정보**도 들어감
    - `Dispatchers.IO` → 디스패처 지정
    - `Job` → 코루틴의 취소와 완료 상태 관리
    - `CoroutineName` → 디버깅용 이름 태깅
    - `CoroutineExceptionHandler` → 예외 처리기
</details>


<hr>
<details>
<summary><strong>15.1 코루틴 스코프가 코루틴 간의 구조를 확립한다</strong></summary>
	
- 구조화된 동시성을 통해 각 코루틴은 코루틴 스코프에 속하게 됨
- 코루틴 스코프는 코루틴 간의 부모-자식 관계를 확립하는데 도움을 줌
- `launch`, `asyn` 코루틴 빌더 함수들은 사실 `CoroutineScope` 인터페이스의 확장 함수
- 즉 다른 쿠로틴 빌더의 본문에서 `launch`, `asyn` 를 사용해 새로운 코루틴을 만들면 이 새로운 코루틴은 자동으로 해당 코루틴의 자식이 됨

```kotlin
fun main(): Unit = runBlocking {
    launch {
        delay(200)
        println("Task from runBlocking")
    }

    coroutineScope { // 새로운 코루틴 스코프 생성
        launch {
            delay(500)
            println("Task from nested launch")
        }

        delay(100)
        println("Task from coroutineScope")
    }

    println("Coroutine scope is over")
}

/* 예상출력
Task from coroutineScope  
Task from runBlocking  
Task from nested launch  
Coroutine scope is over
*/
```

- `runBlocking` 이 부모 스코프 역할
- `coroutineScope` 안에서 또 다른 코루틴 스코프 생성됨
- `coroutineScope` 내에서 `launch` 로 자식 코루틴 시작
- 모든 자식이 끝날 때까지 `coroutineScope`는 종료되지 않음

## 15.1.1 코루틴 스코프 생성: `coroutineScope`  함수

- `coroutineScope { ... }` 함수는 **새로운 코루틴 스코프**를 생성
- 이 스코프 내에서 시작된 코루틴은 **모두 자식 코루틴**이 됨
- 부모 코루틴은 **자식들이 전부 완료될 때까지 기다림**
- **`구조화된 동시성(structured concurrency)`** 을 구현하는 핵심 함수 중 하나

| 함수 이름 | coroutineScope |
| --- | --- |
| 반환 시점 | **모든 자식 코루틴이 끝난 후** |
| 차이점 | launch는 Job 반환, coroutineScope는 결과값 반환 |
| 예외 처리 | 자식 중 하나라도 예외가 나면 스코프 전체가 종료됨 |

```kotlin
fun main() = runBlocking {
    coroutineScope {
        launch {
            delay(1000)
            println("Child coroutine 1")
        }

        launch {
            delay(500)
            println("Child coroutine 2")
        }

        println("All children launched")
    }

    println("coroutineScope 끝남")
}

/* 결과 
All children launched  
Child coroutine 2  
Child coroutine 1  
coroutineScope 끝남
*/
```

- `coroutineScope`는 **모든 자식 *코루틴이 완료될 때까지 기다리는 구조화*된 코루틴 블록**

## 15.1.2 코루틴 스코프를 컴포넌트와 연관시키기: `CoroutineScope`

- `CoroutineScope`는 코루틴을 실행할 컨텍스트(Context) 를 담고 있는 인터페이스.
- 일반적으로 컴포넌트(예: ViewModel, Activity 등)에 코루틴을 묶어서, 컴포넌트가 사라질 때 코루틴도 같이 종료되도록 함

```kotlin
class MyComponent : CoroutineScope {
    private val job = Job()

    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main + job

    fun destroy() {
        job.cancel() // 컴포넌트 종료 시 모든 코루틴 취소
    }
}
```

- `job.cancel()` 호출 시, 이 `scope` 안에서 시작된 모든 코루틴이 취소됨.
- 이 구조를 쓰면 메모리 누수나 유령 코루틴`(leaking coroutine)` 을 막을 수 있음

---

- `Android ViewModel`에는 이미 `viewModelScope`가 있음.
- 이 `scope`에 코루틴을 연결하면 `ViewModel`이 사라질 때 자동 취소됨.

| 목적 | 컴포넌트와 코루틴의 수명 일치 |
| --- | --- |
| 구현 방식 | 클래스에 CoroutineScope 구현 + Job 보관 |
| 장점 | 자원 누수 방지, 안전한 구조화된 동시성 |
| Android 예시 | viewModelScope, lifecycleScope 사용 |

### **CoroutineScope vs coroutineScope 차이**

| **항목** | CoroutineScope **(인터페이스)** | coroutineScope **(함수)** |
| --- | --- | --- |
| 정체 | **인터페이스** | **suspend 함수** |
| 목적 | 클래스에 코루틴 실행 환경을 부여 | 코루틴 안에서 **자식 코루틴을 안전하게 실행** |
| 주 용도 | ViewModel, Activity 등에서 **코루틴 생명주기 관리** | 특정 suspend 블록 안에서 **코루틴을 구조화** |
| 컨텍스트 | coroutineContext 프로퍼티로 제공 | 부모 컨텍스트를 자동 상속 |
| 종료 처리 | Job.cancel() 로 **전체 코루틴 종료** | 블록 내 자식이 끝날 때까지 **자동으로 대기** |
| 예 | viewModelScope, lifecycleScope | coroutineScope { launch { ... } } |
- **`*CoroutineScope`**는 **스코프를 담는 그릇***
- **`*coroutineScope`**는 **일시적으로 안전한 구조를 만들어주는** `suspend 블록`*

## 15.1.3 GlobalScope의 위험성

- GlobalScope는 전역 스코프
    - 앱이 종료되거나 프로세스가 죽지 않는 이상, 코루틴이 계속 실행됨.
    - 그래서 일반적인 코루틴과 달리, 부모 스코프와 관계없이 독립적으로 동작함
- 생명 주기와 무관
    - `Activity`, `ViewModel`, `Fragment` 등이 사라져도 코루틴은 계속 실행됨.
    - 메모리 누수(leak) 와 예상치 못한 동작 발생 가능
- 예외 전파 안 됨
    - `GlobalScope`에서 발생한 예외는 부모 코루틴으로 전파되지 않음.
    - `구조화된 동시성(structured concurrency)`의 장점이 사라짐
- 취소 불가
    - `GlobalScope.launch { ... }` 로 만든 코루틴은 명시적으로 잡지 않으면 취소할 방법이 없음.

```kotlin
// 예제 (문제 있는 코드)
fun startSomething() {
    GlobalScope.launch {
        delay(1000)
        println("Global coroutine finished")
    }
}
```

- !!!구조화된 스코프 사용해야함!!!
    - `viewModelScope`, `lifecycleScope`, `coroutineScope`, `supervisorScope` 같은 스코프를 명확히 지정해서 사용해야 안전

## 15.1.4 코루틴 콘텍스트와 구조화된 동시성

- 코루틴은 항상 `CoroutineContext`를 가지고 실행됨
    - 예: `Dispatchers.Main`, `Job`, `CoroutineName`, `CoroutineExceptionHandler` 등
- 이 코루틴 콘텍스트는 부모 → 자식으로 상속됨
- 즉, `launch`나 `async`로 새 코루틴을 만들면 부모 코루틴의 콘텍스트를 자동으로 이어받음
- 콘텍스트에는 중요한 요소인 Job이 포함돼 있어서, 자식 코루틴이 부모와 연결되고, 부모가 취소되면 자식도 같이 취소됨
- 이것이 구조화된 동시성(Structured Concurrency) 의 핵심!

| **요소** | **역할** |
| --- | --- |
| CoroutineContext | 코루틴 실행 환경 |
| Job | 코루틴의 생명 주기 및 계층 관리 |
| Dispatcher | 어떤 스레드에서 실행할지 결정 |
| Name, ExceptionHandler | 디버깅, 예외 처리에 사용 |

</details>


<details>
<summary><strong>15.2 취소</strong></summary>

- 취소는 코드가 완료되기 전에 실행을 중단하는 것을 의미
- 현대 애플리케이션은 계산 작업을 취소할 수 있어야 견고하고 효율적임
- 취소는 불필요한 작업을 막아줌
- 취소는 메모리나 리소스 누수를 방지하는 데도 도움을 줌
- 취소는 오류 처리에서도 중요한 역할을 함

## 15.2.1 취소 촉발

- 여러 코루틴 빌더 함수의 반환값을 취소를 톡발하는 핸들로 사용할 수 있음
- `launch` 코루틴 빌더는 `Job` 을  반환하고 `async` 코루틴 빌더는 `Deferred` 를 반환함
- 둘다 `cancel` 을 호출해 해당 코루틴의 취소를 촉발할 수 있음

## 15.2.2 시간제한이 초과된 후 자동으로 취소 호출

- 코틀린 코루틴 라이브러리는 코루틴의 취소를 자동으로 촉발할 수 있는 몇 가지 편리한 함수도 제공해줌
- `withTimeout`, `withTimeoutOrNull` 함수는 계산에 쓸 최대 시간을 제한하면서 값을 계산할 수 있게 해줌
- `withTimeout` 함수는 타임아웃이 되면 예외(`TimeoutCancellationException`)을 발생시킴, 타임아웃을 처리하면 `withTimeout` 호출을 `try` 블록으로 감싸고 발생한 `TimeoutCancellationException` 을 잡아내야함
- `withTimeoutOrNull` 함수는 타임아웃이 발생하면 `null` 을 반환함

*⇒ `withTimeout` 이 발생시키는 `TimeoutCancellationException` 을 잊지 말고 잡아야함, 잡지않으면 호출한 코루틴이 의도와 다르게 취소될 수 있음. 이 문제를 완전히 피하려면 `withTimeoutOrNull` 함수를 사용하는 편이 좋음*

## 15.2.3 취소는 모든 지식 코루틴에게 전파된다

- 코루틴을 취소하면 해당 코루틴의 모든 자식 코루틴도 자동으로 취소됨  이는 구조화된 동시성의 강력한 기능
- 각 코루틴은 자신이 시작한 다른 코루틴을 알고 있기 때문에 취소할 때 스스로 자식들을 정리할 수 있으며, 불필요한 작업을 계속하거나 불필요하게 데이터를 메모리에 더 오래 유지하는 제멋대로인 코루틴이 남지 않음

## 15.2.4 취소된 코루틴은 특별하 지점에서 CancellationException을 던진다

- 취소 매커니즘은 `CancellationException` 이라는 특수한 예외를 특별한 지점에서 던지는 방식으로 작동함
- 취소된 코루틴은 이시 중단 지점에서 `CancellationException` 을 던짐, 일시 중단 지점은 코루틴의 실행을 일시 중단할 수 있는 지점
- 일반적으로 코루틴 라이브러리 안의 모든 일시 중단 함수는 `CancellationException` 이 던져질 수 있는 지점을 도입

```kotlin
coroutineScope {
	log("A")
	delay(500.milliseconds) // <- 이 지점에서 함수가 취소될 수 있음 
	log("B")
	log("C")
}
```

- 위 코드에서는 영역이 취소됐는지 여부에 따라 `A` 나 `ABC` 가 출력되며, `AB` 는 절대 출력되지 않음. 이는 `B` 와 `C` 사이에 취소 지점이 없기 때문

## 15.2.5 취소는 협력적이다

- 코틀린 코루틴에 기본적으로 포함된 모든 함수는 이 취소 가능함
- `ktor` 같은 라이브러리에서 제공하는 일시 중단 API를 사용할 때도 해당 라이브러리의 일시 중단 함수는 내부적으로 취소 가능하다고 가정할 수 있음
- 하지만 직접 작성한 코드에서는 직접 코루틴을 쉬소 가능하게 만들어야함

```kotlin
fun main() = runBlocking {
    val job = launch {
        var i = 0
        // CPU 집중 루프이기 때문에 delay 등이 없어 취소되지 않음
        while (i < 1000) {
            // 취소 가능하게 만들기 위한 체크
            if (!isActive) {
                println("취소 요청 감지됨. 종료함.")
                break
            }
            println("일하는 중... $i")
            i++
        }
    }

    delay(100) // 잠시 기다림
    println("main: 취소 요청")
    job.cancel() // 취소 요청
    job.join()   // 취소 완료 기다림
    println("main: 완료")
}
```

## 15.2.6 코루틴이 취소됐는지 확인

- 코루틴이 취소됐는지 확인할 때는 `CoroutineScope` 의 `isActive` 속성을 확인함. 이 갑이 `false` 라면 코루틴은 더 이상 활성 상태가 아님
- 이 경우 현재 작업을 완료하고, 획득한 리소스를 닫은 후 반환할 수 있음
- `isActive` 를 확인해서 `false` 일 때 명시적을 반환하는 대신 코틀린 코루틴은 편의 함수로 `ensureActive` 를 제공. 이 함수는 코루틴이 더이상 활성 상태가 아닐 경우 `CancellationException` 을 던짐

| **항목** | isActive | ensureActive() |
| --- | --- | --- |
| 반환값 | Boolean (true or false) | 취소되면 **예외 발생** |
| 용도 | 수동으로 분기 처리할 때 | 취소되면 **즉시 중단하고 예외 처리**하고 싶을 때 |
| 위치 | 반복문, 계산 루프 등 | 반복문, 무한 루프, 처리 순서 중단 지점 |

## 15.2.7 다른 코루틴에게 기회를 주기:  yield 함수

- `yield` 함수는 코드 안에서 취소 가능 지점을 제공할 뿐만 아니라 현재 점유된 디스패처에서 다른 코루틴이 작업할 수 있게 해줌

```kotlin
fun main() = runBlocking {
    val job1 = launch {
        repeat(5) { i ->
            println("Job 1 - Step $i")
            yield() // 다른 코루틴에게 기회 주기
        }
    }

    val job2 = launch {
        repeat(5) { i ->
            println("Job 2 - Step $i")
            yield()
        }
    }

    joinAll(job1, job2)
    println("모든 작업 완료")
}

/* 결과
Job 1 - Step 0  
Job 2 - Step 0  
Job 1 - Step 1  
Job 2 - Step 1  
...  
모든 작업 완료
*/ 
```

## 15.2.8 리소스를 얻을 때 취소를 염두에 두기

- **코루틴이 취소되었는지 확인하지 않고 리소스를 얻거나 보유하면 위험**함.
- 예: 파일 열기, 데이터베이스 커넥션, 락(`lock`) 획득 등의 작업 중 코루틴이 취소되면
    
    → **리소스를 제대로 정리하지 못해 누수 발생** 가능.
    
- 따라서, **리소스를 얻기 전에 취소 여부를 확인하거나**, **`try-finally` 또는 `use` 블록을 활용해 정리 작업을 보장**해야 함.

```kotlin
// 예제: 취소된 상태에서 리소스를 획득하지 않도록 방지
val job = launch {
    if (!isActive) return@launch // 취소되었으면 리소스 획득하지 않음

    val file = File("data.txt")
    file.bufferedReader().use { reader -> // use 블록으로 안전하게 정리
        reader.lineSequence().forEach {
            println(it)
            delay(100) // 처리 중에도 취소될 수 있음
        }
    }
}
```

```kotlin
// 예제 2: 취소된 상태에서 락 획득하지 않기
val lock = ReentrantLock()

val job = launch {
    ensureActive() // 취소됐으면 예외 발생해서 중단
    lock.lock()
    try {
        // 작업 수행
    } finally {
        lock.unlock()
    }
}
```

## 15.2.9 프레임워크가 여러분 대신 취소를 할 수 있다.

- 많은 코루틴 기반 프레임워크 (예: **`Jetpack Compose**, **Ktor**, **Spring WebFlux**` 등)는 내부적으로 **상황에 따라 자동으로 코루틴을 취소**해줌
- **Android의 `ViewModel` + `viewModelScope`**
    - `ViewModel`이 소멸되면 `viewModelScope`에 포함된 모든 코루틴이 자동으로 `cancel()`
    
    ```kotlin
    viewModelScope.launch {
        // ViewModel이 없어지면 자동으로 취소됨
    }
    ```
    
- **Jetpack Compose의 `LaunchedEffect`**
    - `Composable`이 `recomposition`으로 바뀌거나 없어지면
        
        → 내부의 코루틴도 자동으로 취소됨
        
    
    ```kotlin
    LaunchedEffect(key) {
        // 이전 key에 해당하는 블록은 자동으로 cancel됨
    }
    ```
    
- **Ktor**
    - 클라이언트 연결이 끊기면 요청을 처리하던 코루틴도 자동으로 취소됨

```kotlin
// Compose의 LaunchedEffect 
@Composable
fun SampleScreen(query: String) {
    LaunchedEffect(query) {
        // query가 변경되면 이전 코루틴은 취소되고, 새로운 코루틴이 실행됨
        search(query)
    }
}
```

</details>

<hr>
<details>
<summary><strong>16.1 플로우는 연속적인 값의 스트림을 모델링 한다</strong></summary>
	
## 16.1.1 플로우를 사용하면 배출되자마자 원소를 처리할 수 있다.

- `Flow`는 원소를 하나씩 순차적으로 처리하는 콜드 스트림
- `emit()`으로 원소가 방출되면, `collect {}` 블록에서 즉시 처리됨
- 즉, 배출과 수집이 동시에 순서대로 진행됨

## 16.1.2 코틀린 플로우의 여러 유형

- 코틀린의 모든 풀로우는 시간이 지남에 따라 등장하는 값과 작업할 수 있는 일관된 API를 제공하지만  콜트 플로우와 핫 플로우라는 2가지 카테고리로 나뉨
- `Cold Flow`는 비동기 데이터 스트림으로 값이 실제로 소비되기 시작할 때만 값을 배출
- `Hot Flow`는 값이 실제로 소비되고 있는지와 상관업싱 값을 독립적으로 배출하며, 브로드캐스트 방식으로 동작
</details>

<details>
<summary><strong>16.2 Cold Flow</strong></summary>
	
## 16.2.1 `flow`빌더 함수를 사용해 콜드 플로우 생성

- 새로운 콜드 플로우를 생성하는 것은 간단함
- 컬렉션과 마찬가지로 새로운 플로우를 생성할 수 있는 빌더 함수가 있음
    
    → 이 함수는 `flow`라 불림
    
- 빌더 함수의 블록 안에서는 `emit` 함수를 호출해 플로우의 수집자에게 값을 제공하고, 수집자가 해당 값을 처리할 때까지 빌더 함수의 실행을 중단함
- `flow` 가 받는 블록은 `suspend` 변경자가 붙어 있으므로 빌더 내부에서 `delay` 와 같은  다른 일시 중단함수를 호출할 수 있음

```kotlin
fun main() = flow {
    println("Flow 시작됨")
    emit("A") // <- emit 함수를 호출해 플로우의 수집자에게 값을 제공
    delay(100)
    emit("B") 
}ㅇ
```

- 이 코드를 실행하면 실제로 아무런 출력도 나타나지 않는다는 점을 지적할만함
- 이는 빌더 함수가 연속적인 값의 스트림을 표현하는 `Flow<T>` 타입의 객체를 반환하기 때문
- 이 `flow`는 처음에 비활성 상태이며, 최종 연산자가 호출돼야만 빌더에서 정의된 계산이 시작됨
- *이로부터 flow가 cold라고 불리는 이유를 알 수 있음. 기본적으로 수집되기 시작할 때까지 비활성 상태이기 때문*

## 16.2.2 Cold flow는 수집되기 전까지 작업을 수행하지 않는다

- **Cold Flow**는 **collect 호출 전까지 아무 작업도 하지 않음**

```kotlin
fun createValues(): Flow<String> = flow {
    println("Flow 시작됨")
    emit("A")
    delay(100)
    emit("B")
}

fun main(): Unit = runBlocking {
    println("collect 호출 전")
    createValues().collect { println("받음: $it") }
}
```

- `collect` 는 일시 중단 함수(`suspend function`)
- `collect`는 `Flow` 안에서 하나씩 방출되는 값을 비동기적으로 기다리며 수집해야 함
- 방출이 `emit()`으로 일어나는데, 이것도 `suspend` 함수이므로
    
    → 그걸 받는 `collect`도 자연스럽게 일시 중단 함수여야 함
    

## 16.2.3 flow 수집 취소

- `Flow`의 수집(`collect`)은 일시 중단(`suspend`) 함수이므로 취소가 가능
- 코루틴이 취소되면 `collect`도 중단되며, `Flow` 빌더 블록(`flow {}`) 내부의 실행도 함께 멈춤

```kotlin
val flow = flow {
    emit(1)
    delay(100)
    emit(2)
    delay(100)
    emit(3)
}

val job = launch {
    flow.collect { println(it) }
}
delay(150)
job.cancel() // 수집 도중 취소
```

## 16.2.4 Cold Flow의 내부 구현

- `flow {}`는 코틀린 표준 라이브러리에 정의된 빌더 함수
- 이 함수는 실제로는 `Flow<T>` 인터페이스의 구현체를 반환함
- 내부적으로는 람다와 클래스 구현을 조합한 익명 객체를 만들어냄

```kotlin
interface Flow<out T> {
    suspend fun collect(collector: FlowCollector<T>)
}
```

- `flow {}`는 **Flow 인터페이스를 구현한 익명 클래스**를 생성함
- 내부적으로 `emit(value)` 호출은 `collector.emit(value)` 로 구현되어 있음.
- 즉, 우리가 작성한 `flow` 블록은 실제로는 다음과 같은 구조로 동작

```kotlin
return object : Flow<T> {
    override suspend fun collect(collector: FlowCollector<T>) {
        // flow 블록 안의 코드가 이 안에 들어감
        collector.emit(...)
    }
}
```

- **왜 이렇게 설계됐을까? 🤔**
    - **지연 실행(lazy)** 을 지원하고
    - 코루틴을 활용해 **비동기적이고 중단 가능한 흐름**을 만들기 위함
    - 구조적 동시성과 **suspend 함수와의 자연스러운 연동**을 위함

## 16.2.5 채널 플로우(**channelFlow**)를 사용한 동시성 플로우

- `flow {}`는 기본적으로 순차적으로 실행됨
    
    → `emit()`을 호출하면 수집자(`collect`) 가 값을 처리할 때까지 다음 코드로 진행하지 않음.
    
- 반면, `channelFlow {}` 는 생산자(`emit`) 와 소비자(`collect`) 가 동시성(`concurrency`) 을 갖고 실행될 수 있음.
    
    → 내부적으로 코루틴과 채널을 활용해 병렬 흐름을 구현함.
    
- **왜 필요할까? 🤔**
    - `flow {}`는 순차적인 플로우에는 좋지만, 여러 코루틴에서 값을 병렬로 생성하거나 다른 스레드에서 `emit` 해야 할 경우에는 `channelFlow`가 필요함

```kotlin
fun concurrentFlow(): Flow<Int> = channelFlow {
    launch {
        send(1)
    }
    launch {
        send(2)
    }
}
```

- 위 코드에서는 두 개의 코루틴이 병렬로 동작하면서 값을 보냄.

| **비교 항목** | `flow {}` | `channelFlow {}` |
| --- | --- | --- |
| 실행 방식 | 순차적 | 병렬 / 동시성 가능 |
| emit 방식 | suspend 함수 | send() 사용 (channel 기반) |
| 내부 구조 | 단일 코루틴 | 내부적으로 채널 + 다중 코루틴 |
| 사용 시점 | 단순 스트림 처리 | 동시성 필요할 때 (e.g. 여러 emit 병렬 처리) |
</details>


<details>
<summary><strong>16.3 Hot Flow</strong></summary>

- 배출과 수집이라는 같은 전체적 구조를 따르기는 하지만 `hot flow` 는 `cold flow` 와 다른 여러 속성을 가지고 있음
- `hot flow` 에서는 각 수집자가 플로우 로직 실행을 독립적으로 촉발하는 대신, 여러 구독자라고 불리는 수집자들이 배출된 항목을 공유함
- 시스템에서 이벤트나 상태변경이 발생해서 수집자가 존재하는 여부에 상관없이 값을 배출해야하는 경우에 적합
- 2가지 `hot flow` 구현이 기본적으로 제공
    - `shared flow` : 값을 브로드캐스트하기 위해 사용됨
    - `state flow` : 상태를 전달하는 특별한 경우에 사용됨
- 실제로는 `state flow` 를 `shared flow` 보다 더 자주 사용하게 될 것

## 16.3.1 공유 플로우(`SharedFlow`)는 값을 구독자에게 브로드캐스트한다

- `SharedFlow` 는 구독자가 존재하는지 여부에 상관없이 배출이 발생하는 브로드캐스트 방식으로 동작
- 이런 브로드캐스트 동작을 보여주기 위해 실제 라디오 방송국을 모델링할 수 있음
- 이런 수상한 라디오 방송국들은 실제로 존재하며, 전세계 스파이에게 개방 주파수를 통해 암호화된 메시지를 전송함, 스파이는 이를 청취하고 메시지를 해독하고 시도할 수 있음

```kotlin
class RadioStation {
		// 새 가변 공유 플로우를 비공개 프로퍼티로 정의
    private val _messageFlow = MutableSharedFlow<Int>() 
    // 공유 플로우에 대한 읽기 전역 뷰를 제공함 
    val messageFlow: SharedFlow<Int> = _messageFlow

		fun beginBroadcasting(scope: CoroutineScope) {
        scope.launch {
	        while(true) {
		        delay(500.milliseconds)
		        val number = Random.nextInt(0..10)
		        log("Emitting $number")
		        _messageFlow.emit(number) // 코루틴에서 가변 공유플로우에 값을 배출 
	        }
        
        }
    }
}
```

- 코드로 부터 `SharedFlow` 같은 `hot flow` 를 만드는 방식이  `cold flow` 와 다르다는것을 알 수 있음
- 플로우 빌더를 사용하는 대신 가변적인 플로우에 대한 참조를 얻음
- 배출이 구독자 유무와 관계없이 발생하므로 여러분이 실제 배출을 수행하는 코루틴을 시작할 책임이 있음
- 이는 별다른 어려움 없이 여러분이 여러 코루틴에서 가변 공유 플로우에 값을 배출할 수 있다는 뜻

```kotlin
fun main() = runBlocking {
	 RadioStation().beginBroadcasting(this)
}

/*
[main @coroutine#2] Emitting 2!
[main @coroutine#2] Emitting 10!
[main @coroutine#2] Emitting 4!

*/
```

- `RadioStation` 클래스의 인스턴스를 생성하고 `beginBroadcasting` 함수를 호출하면 구독자가 없어도 브로드 캐스트가 즉시 시작됨

```kotlin
fun main() = runBlocking {
    val station = RadioStation()
    station.beginBroadcasting(this)

		delay(600.milliseconds)
		station.messageFlow.collect {
			log("A collecting $it")
		}
}
```

- 구독자를 추가하는 방법은 콭드플로우를 수집하는 것과 동일, 그냥 `collect` 를 호출하면 됨

### *✍️  핫 플로우 이름 붙일 때 밑줄 쓰기*

- 공유플로우에서 비공개 변수 이름에 밑줄을 사용하고 공개 변셔에 밑줄을 쓰지 않는 패턴을 따르는 이유는 무엇일까?
    - 코틀린에서는 `private` 과 `public` 프로퍼티에 대해 서로 다른 타입을 부여하는 기능을 지원하지 않음
    - 플로우의 가변 버전을 `private` 으로 정의하고, 읽기 전용 타입인 `SharedFlow<T>` 속성을 `public` 으로 노출하면 공유 플로우의 가변 부분을 플로우를 소비하는 클래스에게 노출하지 않을 수 있음
    - 이는 캡슐화와 정보 은닉이라는 관심사에 따른 것
    - 무엇보다 클래스의 소비자는 보통 플로우를 구독하기만 할 뿐, 원소를 배출하지 않아야함
    - 어떤 프로퍼티에 클래스 안에서 접근할 때와 클래스 밖에서 접근할때 다른 타입을 지정하는 기능은 코틀린 2.x 추가될 예정이라고 함

### 🤔 구독자를 위한 값 재생

- 공유 플로우 구독자는 구독을 시작한 이후에 배출된 값만 수신
- 구독자가 구독 이전에 배출된 원소도 수신하기를 원한다면 `MutableSharedFlow` 를 생성할때 `replay` 파라미터를 사용해 새 구독자를 위해 제공할 값의 캐시를 설정할 수 있음

### `ShareIn` 으로 콜드 플로우를 공유 플로우로 전환

- `shareIn()`은 `콜드 플로우(cold flow)` 를 S`haredFlow(hot flow)` 로 변환해주는 연산자
- 변환된 SharedFlow는 하나의 플로우를 여러 구독자가 동시에 공유할 수 있게 해줌
- 기본적으로 `Flow`는 `collect()`가 호출될 때마다 새로 실행되지만, `shareIn()`을 사용하면 값 생성은 한 번만 발생하고, 여러 구독자가 동일한 값을 공유하게 됨

## 16.3.2 시스템 상태 추적: 상태 플로우

- 동시 시스템에서 자주 발생하는 특별한 사례는 시간이 지남에 따라 변할 수 있는 값, 즉 상태를 추적하는 것
- `StateFlow`에서는 `emit()` 대신 `update()` 함수를 사용

### 🤔 UPDATE 함수로 안전하게 상태 플로우에 쓰기

- `MutableStateFlow`에는 상태 값을 안전하게 갱신할 수 있는 `update { }` 확장 함수가 제공됨
- 이 함수는 현재 값에 기반해 새 값을 계산하고 설정함
- 내부적으로 CAS(compare-and-set) 방식으로 동작하므로, 멀티스레드 환경에서도 안전하게 동시 수정 가능

### 🤔 `stateIn` 으로 콜드 플로우를 상태 플로우로 변환하기

- `stateIn()`은 콜드 플로우(Flow) 를 핫 플로우(StateFlow) 로 전환하는 연산자
- 이로써 항상 최신 상태 값을 보존하고, 구독 시 즉시 현재 값을 받을 수 있는 흐름으로 바뀜
- 일반 `Flow`는 `collect`마다 다시 실행되지만, `stateIn`을 사용하면 동일한 상태를 유지하면서 여러 구독자가 공유 가능
- 또한 구독자는 항상 가장 최신 값을 받을 수 있음

## 16.3.3 상태 플로우와 공유 플로우의 비교

| **항목** | **StateFlow** | **SharedFlow** |
| --- | --- | --- |
| **핵심 목적** | 시스템 **상태(state)** 추적 | **이벤트(event)** 브로드캐스트 |
| **초기값 필요** | ✅ 필요 (initialValue) | ❌ 필요 없음 |
| **최신값 보관** | ✅ 항상 최신 상태 유지 (value 속성 제공) | ❌ 값을 저장하지 않음 (옵션: replay) |
| **구독 시 즉시 값 수신** | ✅ 항상 현재 값 즉시 수신 | ❌ 기본값은 이후 emit부터 수신 |
| **값 설정 방법** | value = ..., update {} | emit() 또는 tryEmit() |
| **사용 예** | UI 상태, 설정 값, 로딩 상태 등 | 버튼 클릭, 메시지 알림, 이벤트 스트림 등 |
| **replay 기능** | ❌ 없음 (항상 1개의 현재값만 유지) | ✅ 설정 가능 (replay = N) |
- 실무에서는 보통 StateFlow를 더 자주 사용하게 됨
    
    → 상태 관리가 대부분의 앱에서 핵심이기 때문.
    
- SharedFlow는 일회성 이벤트 처리에 탁월
    
    → 예: 로그인 성공 알림, 네비게이션 트리거 등
    

## 16.3.4 핫 플로우, 콜드 플로우, 공유 플로우, 상태 플로우: 언제 어떤 플로우를 사용할까?

| **항목** | **🧊 Cold Flow** | **🔥 Hot Flow** |
| --- | --- | --- |
| **실행 시점** | collect() 호출 시마다 새로 시작됨 | 외부에서 이미 emit되고 있음 |
| **수집자(구독자)** | 각 수집자가 **독립적으로 실행** | 여러 수집자가 **공유**해서 같은 흐름을 수신 |
| **emit 시점** | collect가 있어야만 emit 발생 | collect 여부와 상관없이 emit 가능 |
| **대표 클래스** | flow {}, flowOf() 등 | SharedFlow, StateFlow, channelFlow 등 |
| **예시** | 리스트 필터링, 계산, 단일 네트워크 요청 | UI 상태, 알림, 이벤트 스트림 |
| **데이터 반복** | 항상 처음부터 다시 수행 | 최신값 유지 또는 이어서 흐름 제공 |
| **비유** | 넷플릭스: 각자 원하는 때에 시청 | 라디오: 방송은 계속되고 있고, 지금부터 들을 수 있음 |

| **시나리오** | **적합한 플로우** |
| --- | --- |
| 이벤트 브로드캐스트 (ex. 버튼 클릭) | SharedFlow |
| 앱/화면 상태 관리 (ex. 로딩 상태, 로그인 상태) | StateFlow |
| 계산이나 반복 가능한 작업 (ex. 리스트 처리) | Cold Flow |
| 외부 시스템에서 계속 발생하는 데이터 수신 | SharedFlow 또는 channelFlow |
| 비동기 흐름이지만 현재 값을 항상 기억해야 함 | StateFlow |
</details>

<hr>
<details>
<summary><strong>17.1 플로우 연산자로 플로우 조작 </strong></summary>
	
- 플로우가 코틀린의 동시성 매커니즘을 활용해 시간에 따라 나타나는 여러 연속적인 값을 처리할 수 있는 고수준의 추상화라는 점을 알게됨
- 컬렉션을 조작하기 위해 사용할 수 있는 다양한 연산자가 있다는 사실을 살펴봄
- 폴로우를 변활 때도 비슷한 연산자를 쓸 수 있음
- 시퀀스와 마찬가지로 플로우도 중간 연산자와 최종 연산자를 구분함
- 중간 연산자는 코드를 실행하지 않고 변경된 플로우를 반환하며, 최종 연산자는 컬렉션, 개별 원소, 계산된 값을 반환하거나 아무 값도 반환하지 않으면서 플로우를 수집하고 실제 코드를 실행함
</details>

<details>
<summary><strong>17.2 중간 연산자는 업스트림 플로우에 적용되고 다운스트림 플로우를 반환한다 </strong></summary>

- 중간 연산자는 플로우에 적용돼 새로운 플로우를 반환함
- 플로우를 업스트림과 다운스트림 플로우로 구분해 설명할 수 있음
- *연산자가 적용되는 플로우를 업스트림 플로우라고 함*
- *중간 연산자가 반환하는 플로우를 다운스트림 플로우라고 함*
- 다운스트임 플로우는 또 다른 연산자의 업스트림 플로우로 적용할 수 있음
- 시퀀스와 마찬가지로 중간 연산자가 호출되더라도 플로우 코드가 실제로 실행되지 않음
- 반환돈 플로우는 콜드상태
- 시퀀스에서 사용할 수 있는 인기 있는 `map`, `filter` , `onEach` 등의 함수는 플로에서도 제공함
- 이들의 행동방식도 예상하는 방식과 같음. 다만, 시퀀스나 컬렉션의 원소가 아니라 플로우의 원소에 대해 작용한다는 점만 다를 뿐

## 17.2.1 업스트림 원소별로 임의의 값을 배출: `transform` 함수

- 이미 알고 있을 `map` 함수는 업스트림 플로우를 받아 우너소를 변환한 후 다운스트림 플로우에 그 원소를 배출할 수 있음

```kotlin
// flow에서 map 수행
fun main() {
	val names = flow {
		emit("JO")
		emit("MAY")
		emit("SUE")
	}
	
	val upperecaseNames = names.map {
		it.uppercase()
	}
	
	runBlocking {
		upperecaseNames.collect {
			print("$it ")
		}
	}
	
	// JO MAY SUE
}
```

- 경우에 따라 하나 이상의 원소를 배출하고 싶을때가 있음
- 코틀린 플로우에서는 `transform` 함수로 이런 일을 할 수 있음
- 이 함수는 업스트림 플로우의 각 원소에 대해 원하는 만큼의 원소를 다운스트림 플로우에 배출할 수 있게 해줌

```kotlin
fun main() {
	val names = flow {
		emit("JO")
		emit("MAY")
		emit("SUE")
	}
	
	val upperAndLowercaseNames = names.transfrom {
		emit(it.uppercase())
		emit(it.lowercase())
	}
	
	runBlocking {
		upperAndLowercaseNames.collect {
			print("$it ")
		}
	}
	
	// JO jo MAY may SUE use
}
```

- 이 예제처럼 플로우에서 단순히 값 목록을 배출하고 나중에 플로우 연산자로 변환하려는 경우
    
    `val names = flowOf("JO", "MAY", "SUE")` 와 같이 `flowOf` 라는 줄임 표현을 사용해 플로우를 만들 수 있음 
    

## 17.2.2 `take` 나 관련 연산자는 플로우를 취소할 수 있음

- 시퀀스에서 배운 `takewhile` 같은 함수들을 플로우에서도 똑같이 쓸 수 있음
- 이런 연산자를 사용하면 연산자가 지정한 조건이 더 이상 유효하지 않을 때 업스트림 플로우가 취소되며, 더 이상 원소가 배출하지 않음

## 17.2.3 플로우의 각 단계 후킹: `onStart`, `onEach` , `onCompletion` , `onEmpty`

- `onStart { }`
    - 플로우가 수집되기 직전에 호출됨
    - 초기화, 로깅, 로딩 표시 등 시작 전 작업에 적합
- `onEach { }`
    - 각 emit 직후, collect 전에 호출
    - 로깅, 디버깅, 중간 처리 등에 활용
- `onCompletion { cause -> }`
    - 플로우가 정상 완료되거나 예외로 종료될 때 호출
    - 정리 작업, 종료 로깅 등에 유용
- `onEmpty { }`
    - flow가 아무 값도 emit하지 않고 끝날 때 호출
    - 기본값 emit, fallback 처리 등에서 유용
    - kotlinx.coroutines 1.7+ 이상에서 제공됨

```kotlin
flowOf(1, 2, 3)
    .onStart { println("🚀 시작") }
    .onEach { println("🔹 값: $it") }
    .onCompletion { println("✅ 완료") }
    .collect()
    
/**
🚀 시작  
🔹 값: 1  
🔹 값: 2  
🔹 값: 3  
✅ 완료
*/
```

## 17.2.4 다운스트림 연산자와 수집자를 위한 원소 버퍼링: `buffer` 연산자

- `buffer`는 업스트림과 다운스트림 간의 처리 속도가 다를 때 원소를 중간에 임시 저장(버퍼링) 하여 성능을 향상시키는 연산자
- 업스트림의 `emit()`과 다운스트림의 `collect()`를 병렬로 수행할 수 있게 해줌
- 일반적으로 `Flow`는 순차적으로 작동한다.
    - 즉, `emit → collect → emit → collect` 식으로 하나씩 처리됨.
- 그런데 만약 `수집(collect)`이 느리면 전체 흐름이 지연됨.
    - 이럴 때 `buffer()`를 사용하면 `emit`은 계속되고, `collec`t는 뒤처진 만큼 버퍼에서 꺼내 처리할 수 있어 병렬성과 효율이 개선됨

| emit() | 데이터를 빠르게 보낼 수 있음 |
| --- | --- |
| buffer() | 중간에 데이터를 저장 |
| collect() | 느리게 소비하더라도 전체 흐름은 지연되지 않음 |

## 17.2.5 중간값을 버리는 연산자: `conflate` 연산자

- `conflate()`는 업스트림이 빠르고 다운스트림이 느릴 때, 중간에 밀린 값을 버리고 최신 값만 전달하는 연산자
- `buffer()`와 유사하지만, 버퍼에 쌓는 대신 중간값을 덮어씀

## 17.2.6 일정 시간 동안 값을 필터링하는 연산자: `debounce` 연산자

- `debounce`는 짧은 시간 안에 연속해서 발생한 값 중에서 마지막 값만 전달하는 연산자
- 연속된 이벤트 중 “지금은 잠시 멈췄다” 고 판단될 때 마지막 값 하나만 `emit`

## 17.2.7. 플로우가 실행되는 코루틴 콘텍스트를 바꾸기 : `flowOn` 연산자

- `flowOn`은 플로우의 업스트림 연산이 실행될 코루틴 디스패처(스레드) 를 지정하는 연산자
- 플로우는 기본적으로 **collect()**가 호출된 곳의 디스패처에서 실행되는데, 이걸 명시적으로 다른 디스패처에서 실행하도록 바꿀 수 있음
- `flowOn(dispatcher)`은 플로우의 업스트림 연산을 지정한 코루틴 컨텍스트에서 실행 (백그라운드 처리, 스레드 전환, UI 블로킹 방지)
- 
</details>

<details>
<summary><strong>17.3 커스텀 중간 연산자 만들기 </strong></summary>

- 코틀린 코루틴 라이브러리는 플로우를 조작할 수 있는 다양한 연산자를 제공함
- 하지만 이런 연산자들이 내부적으로 어떻게 동작하며, 어떻게 우리가 직접 커스텀 중간 연산자를 만들수 있을까?
- 코틀린 코루틴의 Flow는 `map`, `filter`, `debounce` 등 다양한 중간 연산자를 제공함
- 이 연산자들은 내부적으로 `flow` 빌더와 `emit`를 활용해 동작함
- 사용자가 직접 자신만의 중간 연산자를 만들 수 있음
- `Flow<T>`를 확장 함수 형태로 만들고, 내부에서는 새로운 `flow {}`를 반환하며 `collect`로 `upstream` 값을 수집해서 가공하고 `emit` 함

```kotlin
fun Flow<Int>.filterEven(): Flow<Int> = flow {
    collect { value ->
        if (value % 2 == 0) emit(value)
    }
}
```
</details>

<details>
<summary><strong>17.4 최종 연산자는 업스트림 플로우를 실행하고 값을 계산한다</strong></summary>
	
- 중간 연산자는 주어진 플로우를 다른 플로우로 변환하지만 실제로 코드를 실행하지 않음
- 실행은 최종 연산자가 담당
- 최종 연산자는 단일 값이나 값의 컬렉션을 계산하거나, 플로우의 실행을 촉발시켜 지정된 연산과 부수 효과를 수행함
- 가장 일반적인 최종 연산자는 `collect`
- `collect` 는 플로우의 각 원소에 대해 실행할 람다를 지정할 수 있는 유용한 지름길을 제공
- 이제는 중간 연산자를 배웠기 때문에 이 지름길 코
- 드가 `onEach` 를 호출한 다음에 파라미터 없는 `collect` 를 호출하는 코드와 같다는 사실을 짐작할 수 있음

 

```kotlin
fun main() = runBlocking {
    getTemperatures()
    .onEach {
	log(it)
    }
    .collect() 
}
```

- 최종 연산자는 업스트림 플로우의 실행을 담당하기 때문에 항상 일시 중단 함수
- `collect` 를  호출하면 플로우 전체가 수집될 때까지 일시 중단함
- `first` 나 `firstOrNull` 같은 다른 최종 연산자는 원소를 받은 다음에 업스트림 플로우를 취소할 수 있음

## 17.4.1 프레임워크는 커스텀 연산자를 제공한다.

- `collectAsState()` (Jetpack Compose):
    - `StateFlow`나 `Flow`를 `Compose`의 상태로 변환하는 최종 연산자
    - 실제로는 collect를 내부에서 수행하며 recomposition을 트리거함
- `launchIn(scope)`:
    - 별도의 `collect` 없이 지정된 `scope`에서 플로우를 시작시킴
    - `collect {}` 없이 side-effect만 수행할 때 유용
</details>
