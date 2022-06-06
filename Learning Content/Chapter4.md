## 1. 코루틴 핸즈 온(코드랩)
~~~
- Retrofit 인터페이스 부분이 있는데, Retrofit은 Rest호출하기 쉽게 만들어주는 라이브러리
- 대괄호를 사용하여 주소를 치환이 가능하다.
  getOrgReposCall("abc") 이걸로 인해 
  orgs/{org}/repos...  ->  orgs/abc/repos... 변화하여 호출이된다.
- .execute()  //요청을 실행하고 현재 스레드를 차단합니다.
-
~~~

