## Choosing an Executor - ThreadPoolExecutor

### 여러 개의 Task를 실행할 때

- simple parallel computation 인 경우에는 여러 thread만으로도 충분
  - a fixed pool with as many threads as CPUs works
- simple parallel computation + other task 필요가 있는 경우에는 ForkJoinPool 이용
- I/O block을 막기 위해서는 I/O를 기다리는 thread 갯수에 limit을 두는 대신 Executors.newCachedThreadPool 가 좋은 선택 

### Pool 이 사용되는 방식

- core pool size

  task가 요청되면 Executor는 thread를 새로 만든다 until core pool size가 다 찰 때까지.
    - 이 때 idle thread 가 있어도 새로운 thread를 생성한다 (core pool size가 다 찰 때까지)
    - threadPoolExecutor.allowCoreThreadTimeout(true) 를 설정하면 core 에 있는 thread가 idle인 경우 thread 를 종료 (이 때 생성자에 RejectedExecutionHandler 를 제공해 주는 것이 바람직)
- queue capacity

  core pool size가 꽉 차면 thread가 새로 생성되는 대신 queue에 쌓임
- max pool size

  queue도 꽉차면 max pool size에 맞춰 다시 thread를 생성함
새로운 thread가 생기면 queue의 가장 앞에 있는 작업을 실행 (queue 순서 보장)
- Setting Example

  new ThreadPoolExecutor(
  5, // core pool size
  8, // max pool size, for at most (8 - 5 = 3) red threads
  10, TimeUnit.SECONDS, // idle red threads live this long
  new ArrayBlockingQueue(7)); // queue with finite maximum size

  - core : 5, max : 8, queue : 7 (8 - 5 = 3 개의 thread를 추가로 생성)
    5개의 작업이 thread에 꽉차면 뒤로 7개 작업이 들어오고 그 뒤로 3개의 작업으로 인해 thread가 추가로 생성

### Pool 이 설정되는 방식
core, queue, max가 모두 설정되는 경우는 드물다! 설정에 따라 Pool을 구분할 수 있는데,
- FixedThreadPool : limited thread number + infinite queue

  I/O or CPU context switches에 limit 을 두고 싶을 때 적합
- CachedThreadPool : infinite thread number + empty queue

  pool-induced deadlock을 방지할 수 있다   
  Thread가 바쁘면 Timeout이 발생하는데, 바쁜 건 어떻게 알 수 있지?
- ForkJoinPool : For recursive calculations 


_pool-induced deadlock_ : 어떤 작업이 다른 작업을 실행하기 위해 다른 thread를 사용하려고 할 때 core pool size가 꽉 차서 해당 작업이 queue에 들어가서 언제 실행될지 모르는 상태가 발생하는 경우 pool-induced deadlock 이라고 함

[FROM HERE](https://blog.jessitron.com/2014/01/choosing-executorservice.html?m=1)
