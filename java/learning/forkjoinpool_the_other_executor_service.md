## ForkJoinPool: the Other ExecutorService

[ForJoinPool: the Other ExecutorService](https://blog.jessitron.com/2014/02/forkjoinpool-other-executorservice.html) from blog.jessitron.com


#### About ForkJoinPool 
  - 작업(task) 사이의 의존도를 정확히 알고 있는 ExecutorService
  - 병렬처리를 위해 고안되었고 병렬에 병렬도 가능하며 병렬로 처리되었던 각 작업은 완료 후에는 하나로 통합함 
    - split off then regroup

#### Ordinary ExecutorServices 의 문제
  - 작업을 나눠서 각각 다른 thread 에서 실행되도록 하는데, 작업이 작으면 문제가 됨
    - 작업을 실행하는 시간보다 thread에 할당하는 데에 오버헤드가 발생하므로-
  - 가장 큰 문제는 thread 가 작업을 쪼개고 처리 결과를 기다리게 된다는 것, 대기 상태로 남아 있는 스레드가 많아지고 사용(일)할 수 있는 thread가 없어진다 - deadlock의 가능성 

#### ForkJoinPool 의 작동방식
  - ForkJoinPool은 작업을 thread에 보내지 않고 직접 관리한다. 잘게 쪼개고 다시 합치는 과정이 한 곳에서 이루어 지고 잘게 쪼갠 작업들을 도울 thread 가 있다면 해당 작업을 돕는 방식 
  - *장점* unpredictable parallel computation을 지원하고, pool-induced deadlock을 방지하며, thread간의 switching 작업을 최소화한다
  - *단점* regular old ExecutorServices 보다 사용하기 어렵다 
    세부작업들이 생성되고, 실행하고 조인하는 과정을 처리해야 함 - 이에 대한 계획이 필요하고 코딩 전에 ForkJoinPool이 해답일지 고민해야 함

#### ForkJoinPool의 이용
  - Scala의 Futures : ForkJoinPool과 ordinary ExecutorService의 차이점을 숨긴 채 작동
  - Akka 의 actor messaging 
  - Clojure 의 collection processing with Reducers
