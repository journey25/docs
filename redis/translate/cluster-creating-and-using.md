---
layout: post
title:  "Redis Cluster tutorial 3 - creating and using"
date:   2018-10-25 17:35:00 +0900
categories: docs redis
---

comes from <https://redis.io/topics/cluster-tutorial>



### Resharding the cluster 리샤딩
Redis Cluster 의 *리샤딩* 은 기본적으로 _hash slot_ 을 한쪽 노드에서 다른 노드로 이동하는 것입니다. 그리고 클러스터 생성할 때와 마찬가지로 `redis-cli` 프로그램으로 실행할 수 있습니다.

*리샤딩* 을 위해 첫번째 할 일은 아래 명령어를 실행시키는 일입니다.
```
$ redis-cli --cluster reshard 127.0.0.1:7000
```
하나의 노드만 명시하면 `redis-cli` 에서 다른 노드를 자동으로 탐색합니다.
`redis-cli` 는 현재 *리샤딩* 를 위한 관리자의 개입을 필요로 합니다. 이 말은 즉슨, 어느 정도의 규모를 옮기길 원하는지, 정확한 정보를 요구할 것입니다.

```
$ How many slots do you want to move (from 1 to 16384)?
```
> ##### 몇개의 slot 을 옮길까요?

그런 다음 `redis-cli` 는 *리샤딩* 되는 타겟노드(destination)를 알아야 합니다. (hash slot 이 옮겨질 노드) 이 때 마스터 노드, 127.0.0.1:7000, 를 이용한다고 가정했을 때 Node ID 정보가 필요합니다.

아래 명령어로 Node ID 를 항상 확인할 수 있습니다.
```
$ redis-cli -p 7000 cluster nodes | grep myself
97a3a64667477371c4479320d683e4c8db5858b1 :0 myself,master - 0 0 0 connected 0-5460
```
타겟노드가 _97a3a64667477371c4479320d683e4c8db5858b1_ 가 되겠네요.  
그런 다음 _hash slot_ 을 가져올 노드(source)를 물어볼 것입니다. `all` 이라고 칠 경우 여러 개의 마스터 노드에서 조금씩 _hash slot_ 을 가져올 것입니다.  
마지막 확인 작업이 완료되면 현재 *리샤딩* 되고 있는 slot 에 관한 메세지를 볼 수 있으며 이동된 각 실제 키들이 출력될 것입니다.
또한 예제 프로그램을 계속 구동중이었다면 *리샤딩* 되는 동안 프로그램에 영향이 없을 볼 수 있을 것입니다. 또한 *리샤딩* 이 진행되는 중에 중지하거나 재시작할 수 있습니다.  
완료 후에는 아래와 같이 실행하여 클러스터의 health 체크를 진행합니다.

```
redis-cli --cluster check 127.0.0.1:7000
```
이 때 이동시킨 마스터 노드의 slot 갯수가 줄어든 것을 확인할 수 있을 것입니다.



### Scripting a resharding operation 리샤딩 스크립트
얼마나 옮길지, 어디에서 어디로 옮길지 매번 입력하는 방식 대신, 아래 명령어를 이용할 경우 한번에 실행되게 할 수 있습니다.
```
redis-cli reshard <host>:<port> --cluster-from <node-id> --cluster-to <node-id> --cluster-slots <number of slots> --cluster-yes
```
스스로 클러스터를 확인하여 자동으로 데이터를 분산하는 방식은 `redis-cli` 에서 현재 지원하지 않기 때문에 리샤딩을 자주 실행한다면 위의 명령어를 이용할 수 있습니다. (추후에는 자동 리샤딩 기능이 추가될 수 있습니다.)
