## 1. 채널 기초
~~~
- 채널은 일종의 파이프이다. 송신측에서 채널에 send로 데이터를 전달하고 수신측에서 채널을 통해 receive받습니다.
  (trySebd와 tryReceive도 있습니다. 과거에는 null을 반환한는 offer와 poll가 있었습니다.)
- send와 receive모두 suspension point라고 볼수 있다.
- trySebd와 tryReceive는 suspension point가 없기 때문에 기다리지 않는다.
- send와 receive는 의존적이기 때문에 같은 코루틴에서 사용하는것은 위험할 수 있다.
- 채널에서 더이상 보낼자료가 없으면 close메서드를 이용해 채널을 닫을 수 있습니다. 채널은 for in을 이용해서 
  반복적으로 receive할 수 있고 close되면 for in은 자동으로 종료됩니다.
- channel.close를 하지않으경우 for문이 언제까지 돌아야할지 모르기때문에 수행시간이 너무 길어져서 에러가 나다.
- 상대방이 close를 하지않는경우 명시적으로 맞춰서 가지고 와야한다.
~~~
 
## 1-2. 채널 프로듀서
~~~
- 생산자와 소비자의 패턴이 매우 일반적이다. 생산자는 데이터를 만들어 내는것이고 소비자는 데이터를 가지고 가는것이다.
- 채널을 이용해서 한쪽에서 데이터를 만들고 다른쪽에서 받는 것을 도와주는 확장 함수들이 있다.
- produce 코루틴을 만들고 채널을 제공한다.
- consumeEach 채널에서 반복해서 데이터를 받아간다.
- produce를 사용하면 produceScope를사용하는데 CoroutineScope 인터페이스와 SendChannel 인터페이스를 함께 상속받습니다.
- ProducerScope = CoroutineScope + SendChannel (this.send 와 this.coroutineContext를 사용할 수 있다.)
- 확장함수를 사용하면 조금 더 간결하게 코드를 짤수 있다.
~~~

## 2. 채널 파이프 라인
~~~
- 파이프 라인은 일반적인 패턴이며 하나의 스트림을 프로튜서가 만들고, 다른 코루틴에서 그스트림을 읽어 새로운 스트림을 만드는 패턴
- 채널을 이용해서 또다른 채널을 만드는것 
- 처음 만드는채널은 리스브 채널로 receiv메서드 send 메서드 X
- 채널은 리스브 채널 + 센드 채널
- for(x in stringNumbers)을 통해 채널을 돌리수는 없다.
- 여러개의 채널을 순차적으로 데이터를 가공해나갈수 있다.
  (홀수 생성이면 리스브 채널에서 수를 생성하고 다른 채널에서 조건문을 사용하여 분류할 수 있다.)
- 파이프라인을 연속으로 타면서 원하는 결과를 얻을 수 있다.
- 
~~~

~~~kt
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
// 소수 판별식
// 실용적인 코드보다는 파이프 라인의 사용을 보여준다. 
fun CoroutineScope.numbersFrom(start: Int) = produce<Int> { //SC + CoroutineScope 2, 3, ...
    var x = start
    while (true) {
        send(x++)
    }
}

fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int): ReceiveChannel<Int> = produce {
    for (i in numbers) {
        if (i % prime != 0) {
            send(i)
        }
    }
}


fun main() = runBlocking<Unit> {
    var numbers = numbersFrom(2) // RC , 3 ,4 ,5 //채널이 대체. //채널 대체

    repeat(10) {
        val prime = numbers.receive() // 2    들어간건 numbers에 빠지면서 필터에는 3부터 들어간다.
        println(prime)
        numbers = filter(numbers, prime) // nubmer : 3,4,5 prime 2
    }
    println("완료")
    coroutineContext.cancelChildren()
}
~~~

## 3. 채널 팬아웃, 팬인
~~~
- 팬아웃은 여러 코루틴이 동시에 채널을 구독하는 것이다.
- 팬인은 반대로 생산자가 많은것
- 공정한 채널 두개의 코루틴에서 채널을 서로 사용할때 공정하게 기회를 준다는것을 알수 있다.
- 채널은 공평하게 동작을한다.
- select를 사용하면 먼저 끝나는 요청을 처리하는것이 중요할 수 있습니다. 
- 두개의 채널이 있다면 먼저오는 데이터를 먼저 수행한다.


- 채널에 대해 onReceive를 사용하는 것 이외에도 아래의 상황에서 사용할 수있다.
Job - onJoin // 런치를 사용하면 누구의 런치가 빨리 끝나는냐를 볼수 있다.
Deferred - onAwait // 작업이 끝났을때 .onAwait를 작동해 달라고 사용할 수 있다.
SendChannel - onSend
ReceiveChannel - onReceive, onReceiveCatching
delay - onTimeout
~~~

## 4. 채널 버퍼링
~~~
- Channel 생성자는 인자로 버퍼의 사이즈를 지정 받는다. 지정하지 않으면 버퍼를 생성하지 않는다.
- Channel<Int>(10) // 채널의 버퍼 갯수가 10
- 송수신이 있다면 송신후 수신이 되지않더라고 계속 데이터를 보낼 수 있다.
- 랑데뷰 버퍼 사이즈를 랑데뷰로 지정한다면 실제로는 사이즈가 0이다. 버퍼가 없는것이 랑데뷰이다.

- 사이즈 대신 사용할 수 있는 다른 설정 값이 있다.
  UNLIMITED - 무제한으로 설정  // 실제 무제한이 아닌 링크드리스트처럼 새로운 데이터가 들어오면 계속 할당한다.
                                 메모리가 부족하면 오류가 난다.
  CONFLATED - 오래된 값이 지워짐. // 처리하지 못한값을 그냥 버려버린다.
  BUFFERED - 64개의 버퍼. 오버플로우엔 suspend  // 기본적으로 64개의 버퍼를 하다가 오버플로우가 발생하면 suspend를 한다.
  
- 버퍼 오버플로우 정책에 따라 다른 결과가 나올 수 있다.
  SUSPEND - 잠이 들었다 깨어납니다.
  DROP_OLDEST - 예전 데이터를 지웁니다. // 두번째 인자에 사용
  DROP_LATEST - 새 데이터를 지웁니다. // 처리하지 못한 값들을 지운다.
~~~
