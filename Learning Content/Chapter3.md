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
- 
~~~
