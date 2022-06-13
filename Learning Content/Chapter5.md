# 코루틴의 내부구조

## 1. Monad
~~~
- 모나드는 파라미터의 이름이다.
- monad: (T) ->Box<U>    (하나의 람다를 받는다.)
- 값을 받아서 박스를 반화나는 함수이다.
- Null의 추상(Optional, Option, Maybe), 에러의 추상 (Either), 비동기 실행(IOMonad)에 사용된다.
- 1930년 경에는 Monad가 인자 하나를 받는 함수를 칭한느 말이었기느도 함(요즘은 쓰지않음)
- 모나드:(값) -> 박스(값)
- flatMap(bind, chain, >>=)함수의 인자는 모나드이다.(요즘에는 flatMap가 쓰인다.)
~~~

## 2. Monad의 사촌 Functor
~~~
- Functor는 값을 받아 값을 리턴하는 파라미터
- Monad는 값을 받아서 박스를 리턴하고 Functor은 값을 받아서 값을 리턴한다.
- 연속적인 Monad체인(비동기 연산을 마치고 박스에 담긴 자료로 넣어 연속적으로 호출 가능)
- Either(Left,Right) -> 성공과 실패에 따라 Left,Right로 연결
- 에러인 경우에는 아무것도 하지않고 성공하면 다른 작업을 할수 있다.

~~~

## 3. Continuation Passing Style
~~~
- Continuation-> 넓은 의미로 다음에 수행해야할 내용을 의미함
- 좁은 의미로는 다음에 수행해야 할 함수
- Continuation Passing Style -> 계속 Continuation을 전달하는 방식
- Last call이 중요한다. 이유는 Last call에 대한 최적화를 할수 있다.
- 되돌아가야 하면 스택에 쌓아야하지만 Last call 할 필요가 없다.

- Tail recursion optimization 위한 tailrec키워드만 있다.
- tailrec를 사용하는 재귀함수만 최적화를 해준다.
- Tail caa optimization과는 상관이 없는 최적화이다.
~~~

## 4. CPS의 명백한 단점
~~~
- 끝없는 들여쓰기, 이해하기 어려움
- CALL/CC (Call with Concurrent Continuation)가 있었으면 조금더 짧아진다.
~~~

## 5. CPS의 명백한 장점
~~~
- return이 없으면 만들수 있다.
- 예외가 없다면 만들수 있다.
~~~

