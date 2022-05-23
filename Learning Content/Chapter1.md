# Chapter1

## 1. 처음 만나는 코루틴
~~~
- 합의를 해서 나누어 쓰는것(비선점형 멀티 태스킹) -> A가 자기의 리소스를 쓰고 B에게 리소스를 주는것
- 선점형 멀티태스킹(중재자를 두는것 == 운영체제)
- SMP(Symmetric Multiprocessor)와 가시성 -> 락이나 메모리 베리어가 필요할 수 있음
- 데이터를 갱신해도 문제 캐시라인을 통채로 변경해야한다.
- 코루틴은 비동기와 병렬성을 순차적으로 짤수 있다.
- 플로우는 비동기와 병렬성을 스트림의 형태로 쓸수 있다.
~~~

## 2-1. 스코프 빌더
~~~
- Coroutine Scope는 코루틴의 scope를 정의하는 인터페이스입니다.
- Coroutine Context는 코루틴이 앞으로 수행될 방식을 결정
- launch는 새로운 코루틴을 만들어서 다른 코드를 함께 수행하게 하는 코드
- delay는 잠시 쉬는함수 launch와 함께 사용하면 먼저 작동이 가능하다.
- 코루틴의 장점은 양보를 하면서 최선의 결과를 얻을 수 있다.
~~~

## 2-2. 스코프 빌더
~~~
- sleep은 thread가 쉬는 동안에 양보를 안하고 delay는 쉬는 동안에 양보를 한다.
- runBlocking 계층적,구조적이다.
- 부모를 캔슬하면 자식도 캔슬된다.
- suspend 중단 가능한 함수
~~~