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
