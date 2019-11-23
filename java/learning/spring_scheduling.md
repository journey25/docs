## Spring Scheduling

1. Introduction

Executor라고 이름한 이유는 구현체들이 pool 이라고 보장할 수 없기 때문; single-thread로 실행되거나 동기로 실행될 수 있다
2. The Spring TaskExecutor abstraction

java.util.concurrent.Executor interface를 구현하고 있는 이유는 java 5 등 하위버전에서 thread-pool 사용 시의 호환성 때문
3. TaskExecutor Types
  - SimpleAsyncTaskExecutor
thread를 재사용하지 않고 매번 호출될 때마다 새로운 thread를 생성
대신 concurrency limit을 지원하고 해당 limit을 넘어서 들어오는 요청은 block 함
일반적인 그 "pooling" 이 아님 
  - SynctaskExecutor
동기로 실행한다
각각의 요청을 호출한 thread에서 실행 : 멀티스레딩이 필요하지 않은 simple test case 에서 이용
  - ConcurrentTaskExecutor
java.util.concurrent.Executor로 랩핑되어 있고
대신 ThreadPoolTaskExecutor 를 쓸 수 있다 (Executor configuration을 bean 설정 시 이용할 수 있음)
  - ConcurrentTaskExecutor는 거의 사용되지 않는데 ThreadPoolTaskExecutor를 사용하기에는 불안할 경우 사용
  - SimpleThreadPoolTaskExecutor
  - ThreadPoolTaskExecutor
  - TimerTaskExecutor
  - WorkManagerTaskExecutor

Q. xml 에서 executor를 설정하지 않으면 기본 executor가 실행되는데 이 때 SimpleAsyncTaskExecutor가 실행된다 
SimpleAsyncTaskExecutor는 매번 thread를 생성하는데 executor 에 pool-size와 queue-capacity의 영향을 모르겠다

[26 Task Executoion and Scheduling](http://docs.spring.io/spring/docs/3.1.x/spring-framework-reference/html/scheduling.html#scheduling-task-executor)
 from docs.spring.io
