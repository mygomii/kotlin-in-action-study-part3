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
