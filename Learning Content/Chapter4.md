## 1. 코루틴 핸즈 온(코드랩)
~~~
- Retrofit 인터페이스 부분이 있는데, Retrofit은 Rest호출하기 쉽게 만들어주는 라이브러리
- 대괄호를 사용하여 주소를 치환이 가능하다.
  getOrgReposCall("abc") 이걸로 인해 
  orgs/{org}/repos...  ->  orgs/abc/repos... 변화하여 호출이된다.
- .execute()  //요청을 실행하고 현재 스레드를 차단합니다.

- blocking방식은 결과는 나오지만 사용성이 나쁘다
- callback방식의 thread를 이용하면 loadContributorsBlocking(service, req)를 현재 thread가아닌 다른 thread에서 사용할수 있다.
- callback는 다른 작업을 전부 다른 thread에서 하고 update resource만 메인 thread에서 한다.
- callback 방식의 문제점은 실행중일때 다른 동작을 할수 없다는 문제점이 존재했다.
- Retrofit을 사용하면 첫번째깨 callback전에 두번째 동작을 수행할 수 있다.
~~~

~~~kt
// blocking방식

fun List<User>.aggregate(): List<User> {
     return groupBy { it.login } // entry -> (k,v)
        .map { (login, users) ->
            User(login, users.sumOf { it.contributions })
        }
         .sortedByDescending { it.contributions }
}
~~~

~~~
- 코루틴을 이용하여 suspend functions 사용
- 기존에는 Call<List<Repo>>였지만 List<Repo>로 사용이 가능하다.
- Concurrency에는 launch, async, runblocking이 있다.
- async는 Deferred object를 리턴한다. 
- launch는 job을 리턴한다.
~~~

~~~kt
fun main() = runBlocking {
    val deferreds: List<Deferred<Int>> = (1..3).map { // 총 3개의 async가 호출이 되는것 이고
                                                      // 호출된 결과가 리스트로 들어가게 된다.
        async {
            delay(1000L * it)
            println("Loading $it")
            it
        }
    }
    val sum = deferreds.awaitAll().sum() // deferreds를 전부 기다리는것이 .awaitAll()라는 메서드이다.
                                         // 모든것을 합성을 해서 결과를 내는것이라고 할수 있다.
    println("$sum")
}
~~~

~~~kt
suspend fun loadContributorsConcurrent(service: GitHubService, req: RequestData): List<User> = coroutineScope {
    val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .body() ?: listOf()

    repos.map{ repo ->
        async {
            service
                .getRepoContributors(req.org, repo.name)
                .also { logUsers(repo, it) }
                .bodyList()
        }
    }.awaitAll().toList().flatten().aggregate()
    // 리스트로 만들어진 repos를 awaitall로 전부 기다리겠다
    // flatten()을하는 이유는 리턴값이 리스트의 리스트이기때문에 한번 풀어주는것이다.

    // 결과 자체는 List<List<User>> -> 풀어주는게 flatten() -> List<User>로 풀리게 된다.
    // scope에 소속이 되어있기 때문에 리턴이 필요하지않는다.
    // 부모에게 소속되어 있다고 본다.
}
~~~

~~~kt
suspend fun loadContributorsNotCancellable(service: GitHubService, req: RequestData): List<User> {
    val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .body() ?: listOf()

    return repos.map{ repo ->
        GlobalScope.async {
            log("starting loading for ${repo.name}")
            delay(3000)
            service
                .getRepoContributors(req.org, repo.name)
                .also { logUsers(repo, it) }
                .bodyList()
        }
    }.awaitAll().toList().flatten().aggregate()
    // scope밖이기 때문에 리턴이 필요하다.
    // 일반 함수의 반환값이 list<user>이기때문에 return을 해야한다.
    // 위 코드는 독립적인 코드인것이기 때문이다.
    // 부모가 아닌 Global이기 때문에 독자적인것이다.
    // 취소가 되지않는 이유는 구조화된 동시성안에 포함되지않는것이때문이다.
    // 누구의 자식도 아니기 때문에 취소가 되지않는것 이다.
}
~~~

~~~kt
suspend fun loadContributorsProgress(
    service: GitHubService,
    req: RequestData,
    updateResults: suspend (List<User>, completed: Boolean) -> Unit
) {
    val repos = service
        .getOrgRepos(req.org)
        .also { logRepos(req, it) }
        .body() ?: listOf()

    val allUsers = mutableListOf<User>()

    for ((index, repo) in repos.withIndex()){
        val users = service
            .getRepoContributors(req.org, repo.name)
            .also { logUsers(repo, it) }
            .bodyList()
        allUsers += users
        allUsers.aggregate()
        val isCompleted = index == repos.lastIndex
        updateResults(allUsers, isCompleted)
    }
}
~~~

~~~kt
//채널은 두개의 코루틴 사이에서 정보를 안전하게 보낼수 있다.
//send와 receive가 있다.
suspend fun loadContributorsChannels(
    service: GitHubService,
    req: RequestData,
    updateResults: suspend (List<User>, completed: Boolean) -> Unit
) {
    coroutineScope {
        val repos = service
            .getOrgRepos(req.org)
            .also { logRepos(req, it) }
            .body() ?: listOf()

        val allUsers = mutableListOf<User>()
        val channel = Channel<List<User>>()

        for(repo in repos){
            launch {
                val users = service
                    .getRepoContributors(req.org, repo.name)
                    .also { logUsers(repo, it) }
                    .bodyList()
                channel.send(users)
            }
        }

        repeat(repos.size){ index ->
            val users = channel.receive()
            allUsers += users // a 10, a20 -> a 30
            val isCompleted = index == repos.lastIndex
            updateResults(allUsers.aggregate(), isCompleted)
        }
    }
}
~~~
