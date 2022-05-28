## 1. 플로우 기초
~~~
- Flow는 코틀린에서 쓸 수 있는 비동기 스트림
- .collect와 함께써야한다 
- flow는 콜드 스트림이다.(콜트스트림은 요청이 있을때만 값이 전달된다.)
- withTimeoutOrNull방식으로 flow를 취소할 수 있다.
- flow이외에도 flowOf, asFlow등의 플로우 빌더가 있다.
- flowOf는 여러 값을 인자로 전달해 플로우를 만듭니다.
- asFlow는 컬렉션이나 시퀀스를 전달해 플로우를 만들 수 있다.
~~~
