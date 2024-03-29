---
layout: post
title:  "Redis Cluster tutorial 1"
date:   2018-10-14 17:44:00 +0900
categories: docs redis
---

comes from <https://redis.io/topics/cluster-tutorial>

### 시작하기
Redis Cluster 를 이해하기 위한 문서로, 어떻게 설정하는지, 테스트하는지, 조작하는지에 대한 설명입니다. 사용자 입장에서 Redis Cluster 가 어떻게 동작하는지에 대해 알아봅니다.   
> ##### 앞으로의 튜토리얼은 Redis version 3.0 or higher 에서만 가능합니다.


### Redis Cluster 101 레디스 클러스터 소개
여러 대의 레디스 인스턴스를 이용하여 **자동 샤딩** 될 수 있도록 구성할 수 있습니다.  

파티셔닝된 레디스 인스턴스에 대해 **일정 수준의 가용성** 을 제공합니다. (일부 노드가 실패하는 경우에도 조작이 가능하지만 다수의 실패가 발생하면 불가능합니다)

즉, Redis Cluster 를 이용하면 아래와 같은 이점이 있습니다.
* 자동으로 데이터를 분산
* 일부 노드가 사용불가능한 경우에도 운영 가능


### Redis Cluster TCP ports 레디스 클러스터와 TCP 포트
Redis Cluter 노드는 두 개의 TCP connection 을 필요로 합니다. 예를 들어 6379 port 는 클라이언트용, 16379 는 데이터용으로 사용됩니다.

이 16379 포트는 _Cluster bus_ 로 이용되는데, [바이너리 프로토콜](#binary-protocol)을 이용한 노드끼리의 커뮤니케이션에 이용됩니다. 대략 아래의 작업을 위해 이용되고 있습니다.
1. 노드의 실패 감지
2. 설정 업데이트
3. 복구 권한

#### _주의점_
>
##### 1. 클라이언트는 절대로 데이터 포트를 이용해서는 안됩니다. (ex. 16379)
##### 2. 클라이언트를 위한 포트와 클러스터 버스를 위한 포트의 갭은 고정값으로 항상 10000 입니다.
##### 3. 클라이언트 포트는 모든 클라이언트에게 개방되어야 하고 추가로 다른 클러스터 노드들에게도 개방되어야 합니다. (키 마이그레이션에 이용됩니다)
##### 4. 클러스터 버스 포트는 모든 다른 노드들에게 개방되어야 합니다.


### Redis Cluster data sharding 레디스 클러스터의 샤딩
Redis Cluster 는 consistent hashing 을 이용하지 않고 다른 형태의 샤딩을 이용는데, 모든 키는 **hash slot** 을 일부라고 여깁니다.

Redis Cluster 에는 16384 개의 hash slot 이 있고 hash slot 을 얻기 위해 _CRC16_ 을 이용해 16384의 [나머지값](#modulo)을 구합니다

각 Redis node 는 hash slot 의 일부분을 책입집니다. 예를 들면,
* Node A 는 hash slot 0 ~ 5500
* Node B 는 5501 ~ 11000
* Node C 는 11001 ~ 16383

노드가 추가/삭제 될 때 hash slot 만 이동하면 하면 되므로 쉽게 할 수 있습니다. hash slot 의 이동은 어떠한 정지 명령도 요구하지 않으므로 노드의 추가/삭제, hash slot 의 퍼센트 변경에 따른 다운타임이 없습니다.

Redis Cluster 는 멀티키 지원을 하는데 하나의 명령으로 실행되는, 동일한 hash slot을 가지고 있는 키들끼리 가능합니다. 사용자는 **hash tage** 를 이용해 다수의 키가 동일한 hash slot 을 가지도록 강제할 수 있습니다.

```
키의 일부를 {} 로 감싸면 {} 안에 있는 문자만이 hash 됩니다.
즉 {foo}key 와 key{foo} 가 같은 hash slot 을 가지고 있다는 게 보장됩니다.
```


### Redis Cluster master-slave model 레디스 클러스터의 마스터-슬레이브 모델
일부 노드의 상태가 실패로 있거나 주노드와 커뮤니케이션이 불가능한 노드가 발생한 경우에도 운용이 가능하도록 하기 위해 Redis Cluster 는 마스터-슬레이브 모델을 이용합니다.

위의 예제 중 Node B 가 더이상 이용 가능지 않은 상태일 때 5501 ~ 11000 의 hash slot 에 대한 정보 제공이 되지 않으므로 클러스터 이용이 불가능해 집니다.
그래서 클러스터가 생성될 때 마스터 노드에 대한 슬레이브 노드도 생성됩니다. 최종적으로 마스터 노드 _A, B, C_ 와 슬레이브 노드 _A1, B1, C1_ 이 존재합니다. B1 은 B 의 복제본으로 Node B 가 실패하는 경우 B1 이 마스터 노드가 되어 서비스를 제공합니다. 이 때 B, B1 이 동시에 실패하는 경우에는 Cluster 이용이 불가합니다.


### Redis Cluster consistency guarantees 레디스 클러스트의 일관성 보장
**강력한 일관성** 은 보장하지 않습니다. 특정한 상황에서 쓰기 조작이 유실될 수 있습니다.

첫번째 이유 : Redis Cluster 는 비동기 복제를 이용하기 때문입니다. 쓰기 발생 시 아래와 같이 동작합니다.
* Node B 에 쓰기
* 클라이언트로 Node B 의 OK 응답
* Node B 의 슬레이브 노드 B1, B2, B3 로의 쓰기 전파

Node B가 클라이언트에 응답하기 전 슬레이브 노드 B1, B2, B3 를 기다리지 않기 때문에 언제나 발생 가능한 상황입니다. Node B 에 쓰기 연산이 도착하고 Node B 는 클라이언트에게 응답합니다. 그리고 슬레이브로의 전파 전에 Node B 가 다운되면 해당 쓰기 조작이 일어나지 않은 슬레이브가 마스터로 승격됩니다. Node B 에 일어난 쓰기는 영원히 유실되는 것입니다.

대부분의 데이터베이스를 사용하는 환경에서 데이터를 일정 시간(초) 단위로 disk 에 저장하도록 설정한 경우와 비슷한 상황이 발생됩니다. 해서 꼭 분산 처리 환경에서만 발생하는 상황은 아닙니다. 일관성을 높이기 위해 데이터베이스에서도 클라이언트에 응답하기 전 disk 저장을 보장하도록 설정할 수 있지만 이런 경우 성능 저하를 야기할 수 있습니다. Redis Cluster 에서 동기 복제를 이용하는 것과 동일한 설정입니다.  

성능 보장과 일관성 보장의 경우 한쪽은 어느 정도 포기해야 하는 부분이 있습니다. Redis Cluster 에서는 동기적으로 동작하도록 하기 위해서는  [WAIT](https://redis.io/commands/wait) 명령어로 구현될 수 있습니다. 쓰기 유실을 방지할 수 있는 방법이지만 Redis Cluster 는 동기 복제를 이용할 경우에도 특정 실패 상황에서는(슬레이브가 결국 복제 명령을 받지 못한 상태로 마스터로 승격) 데이터 유실이 발생할 수 있습니다.

추가로 쓰기가 유실될 수 있는 상황은, 네트워크 분리상황입니다. 클라이언트가 더 적은 인스턴스를 가지고 있는 네트워크 단과만 (최소 하나 이상의 마스터 필요) 연결되어 있는 경우 쓰기 연산이 유실될 수 있습니다.

예를 들어 A, B, C, A1, B1, C1 의 마스터 3대 슬레이브 3대가 있고 Z1 을 클라이언트라 가정했을 때 한 쪽에는 A, C, A1, B1, C1 이 있고 다른 한 쪽에는 B 와 Z1 가 있다고 가정해 봅니다.

Z1 은 B 에 쓰기를 보낼 수 있으니 B 가 다른 네트워크 상에 있는 마스터들과의 파티션을 빠르게 처리하면 아무 문제가 없지만, 파티션 동작이 오래걸리는 경우, 즉 B1 이 마스터로 승격할 수 있을 정도로 오래 걸리는 경우 쓰기가 유실될 수 있습니다.

Z1 이 B 에 보내는 쓰기 명령에 대해 **maximum window** 가 존재하고, 더 많은 인스턴스를 가지고 있는 네트워크 단에서 슬레이브를 마스터로 승격할 수 있는 _충분한 시간_ 이 소요되면 더 적은 인스턴스를 가지고 있는 네트워크단의 모든 마스터 노드는 쓰기를 멈춥니다. 따라서 이 _충분한 시간_ 은 Redis Cluster 설정에서 매우 중요한 요소 입니다. 이를 **node timeout** 이라고 합니다. *node timeout* 이 발생하면 슬레이브 노드를 승격시키거나 승격할 노드가 없는 경우 error 상태로 변경되고 더 이상 쓰기가 불가능 해집니다.




##### _일부 클러스터 관련 이해를 위한 부분만 옮겼습니다._
---


##### binary protocol
###### Cluster bus는 노드끼리의 데이터 교환을 위한 별도의 바이너리 프로토콜을 이용하는데 정보 교환시 더 적은 대역폭, 더 적은 시간이 소요되도록 맞춰진 프로토콜입니다.
##### modulo
###### (CRC16 으로 얻어진 값) % 16384
