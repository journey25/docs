
## 자바 9와 미래

목표 : 성능 엔지니어가 자바 9 플랫폼에 대해서 꼭 알고 있어야 하는 사항 살펴보기

자바8이 람다와 그 결과물(스트림, 디폴트메서드, 함수형 프로그래밍 요소)라면 자바9: 모듈과 그 나머지라고 할 수 있습니다. 모듈이 아키텍쳐가 잘 갖추어진 애플리케이션을 구축하는 방법론이긴 하지만 성능 측면에서는 크게 중요한 비중을 차지하지 않으므로 모듈 보다는 성능과 관련된 작은 변경사항 위주로 얘기할 예정입니다.

### 자바 9에서 소소하게 개선된 성능
#### 코드 캐시 세그먼트화
- 인터프리터 등의 논메서드 코드
- 프로파일드 코드
- 논프로파일드 코드

효과 : 스위퍼 가동 시간이 짧아지고(논메서드 영역은 스위핑할 필요가 없으므로) 풀 최적화 코드에 대한 코드 지역성이 향상
단점 : 다른 영역은 공간이 남아 도는데 특정 영역이 꽉 채워질 수는 있음

#### 콤팩트 스트링
자바에서 스트링 콘텐츠는 항상 char[] 타입으로 저장됩니다.
char 는 16바이트 타입이라서 ASCII 스트링을 저장하면 실제로 필요한 공간의 2배 정도를 더 차지, 유니코드를 단순하게 처리하려면 어쩔 수 없는 오버헤드로 취급되었습니다.
하지만!! *콤팩트 스트링* 덕분에 Latin-1 캐릭터로 표기 가능한 스트링은 byte 배열로 나타낼 수 있으므로 공간절약이 가능합니다.
이는 스트링데이터가 가득한 대용량 힙을 지닌 애플리케이션(ES, 캐시, 기타 관련 컴포넌트) 에서 상당한 성능 효과를 기대할 수 있습니다.
_참고. code in p 467_


#### 새로운 스트링 연결
자바 개발자들은 새로운 스트링객체를 만드는 걸 피하기 위해 StringBuilder 타입을 많이 써왔습니다. 하지만 이로 인해 바이트코드가 너무 많이 생성되는 코드가 됩니다.
하지만!! *invokedynamic* 을 이용해서 _StringConcatFactory.makeConcatWithConstants()_ 라는 팩토리 메서드로 스트링 연결 레시피를 제공합니다. 이 기법을 이용하면 새로운 커스텀 메서드용 바이트코드 작성 등 다양한 전략을 구사할 수 있습니다. 통짜 SQL 문을 실행하는 대신 PreparedStatement 를 쓰는 것과 같은 이야기입니다.
-> 앞으로 자바 플랫폼에서 많이 이용될 것으로 보입니다!

#### C2 컴파일러 개선
C2 컴파일러의 발전은 더이상 없을 거라고 보는 의견이 지배적이었지만 (많이 발전했기때문에)
하지만!! *SIMD(single instruction multiple data)* 라는 현대 CPU 에서 선보인 확장 기능을 응용하면 성능 개선을 기대할 수 있습니다.
JAVA/JVM 은 플랫폼 특성 때문에 SIMD 를 더 효과적으로 활용하기 좋은 조건을 가지고 있습니다.
- 바이트코드는 플랫폼과 무관
- JVM 은 시동시 CPU를 탐색하므로 어떤 하드웨어에서 실행되고 있는지 런타임에 파악 가능
- JIT 컴파일은 코드를 동적으로 생성하기 때문에 사용 가능한 모든 명령을 호스트에서 사용가능
또한 이러한 개선으로 VM 인트린직으로 구현할 수 있습니다. 인트린직은 범용 기술이 아닌 특정 해결책이므로 성능 엔지니어에게 강력하지만 유지보수에 많은 비용이 들 수도 있는 접근법입니다.

#### G1 새버전
자바9에서 부터는 default 수집기가 되었습니다. 하지만 수집기 알고리즘이 달라져서 자바8에서 9로 이전하면 애플리케이션에 영향을 줄 수 있습니다. 자바9의 G1 과 자바8의 G1은 버전도 다르므로 조심해야 합니다. 수집기 변경으로 크게 영향을 받으면 안되겠지만 자바9를 이용하는 경우 꼼꼼히 살펴볼 필요가 있습니다.

### 자바10과 그 이후 버전
#### 새로운 릴리즈 절차
*특성릴리즈* 앞으로 6개월 마다 자바 새 버전이 릴리즈됩니다. 기존의 메이저릴리즈와 같은 개념입니다. 기존처럼 많은 부분이 변경되지는 않을 예정입니다.
*장기지원* 일부 특성 릴리즈에 대해서 오라클에서 장기지원할 예정입니다. 상용JDK 릴리즈만 해당됩니다.

#### 자바10
*자바 개선 프로세스* 를 통해 새로운 특성 및 개선사항을 관리합니다. (http://openjdk.java.net/jeps/1)
JDK 개선제안서(JEP) 마다 관리번호 부여하여 관리합니다. _참고. list in p 473_
이 중에서 JEP307 - G1에서 풀 병렬 GC 구현, JEP310 - 애플리케이션 클래스 데이터 공유, JEP312 - 스레드 로컬 핸드쉐이크 는 미미하게나마 성능에 영향을 줄 수 있습니다.


### 자바9 Unsafe 그 너머
#### 핵심 내부 API
*sun.misc.Unsafe* 클래스는 표준 API 는 아니지만 사실상 자바8부터 표준이 됐습니다. *핵심 내부 API* 라고도 부릅니다. 자바9에서는 전혀 안전하지 않은 기능을 다른데에서는 얻을 수 없기 때문에 아직 쓰고 있는 이 핵심 내부 API 를 대체할만한 대안을 만들어 교체하려고 했지만 자바9출시 전에 끝내지 못해 클래스들이 그대로 유지되었습니다. 이들 API 는 자바9에서 *jdk.unsupported* 라는 JDK에 종속된 모듈에 정의되어 있습니다.

#### 자바9의 VarHandle
*가변핸드* (JEP193) 이라는 메서드 핸들까지 포괄하도록 확장됐습니다. Unsafe 에 있는 일부 API 를 안전하게 대체하여 간극을 메우자는 취지입니다.
- 기능 : CAS, volatile 및 메모리 순서 모드로 저수준 액세스 제공
- AutomicInteger example _p 478_


### 발할라 프로젝트와 값 타입
*발할라 프로젝트* 고급 자바 VM 및 언어의 특성 후보를 탐구/배양하는 장으로 아래와 같은 목표를 가지고 있습니다.
- JVM 메모리 레이아웃을 최신 하드웨어 비용 모델에 맞게 조정한다
- 제네릭스가 기본형, 값, 심지어 void 까지 포함하도록 모든 타입에 추상화한다
- 기존 라이브러리, 특히 JDK가 이러한 특성을 최대한 활용하는 방향으로 호환성을 유지하며 진화한다

JVM에서 *값타입* 을 사용 가능성을 모색하는 것이 이 프로젝트에서 가장 역점을 둔 연구분야입니다.
자바9이전에서 값타입은 기본형, 참조형 두가지 뿐입니다. (ex. int vs Integer)
임의의 타입을 만들경우 해당 타입을 하나의 객체로 보는 것이 아닌 내부필드를 모두 하나의 값으로 보며 값타입으로 표현하자는 것이 값타입의 취지입니다. _p 481_
현재 거론되고 있는 타입의 형태는,
- 참조형
- 값타입
- 만능타입입니다.
하지만 이 마저도 고려할 사항이 많고 이러한 논의의 장이 발할라 프로젝트입니다.
(https://wiki.openjdk.java.net/display/valhalla/Main)

### 그랄과 트러플
핫스팟에 내장된 C2 컴파일러의 성공 이후 현재는 더이상 개선할 여지가 크게 없어 대체품으로 *그랄과 트러플* 이 진행되고 있습니다.
- 그랄 : 특수한 JIT 컴파일러
- 트러플 : JVM 런타임에 호스팅된 언어에 대한 인터프리터 생성기

#### C2 의 문제점
C++ 로 작성되어 메모리 누수의 심각한 이슈가 발생할 수 있습니다. 언제라도 VM 을 멎게할 위험성이 있습니다. 또 C2 코드를 수차례 고치는 과정을 거듭하며 관리/확장이 아주 힘들어졌습니다.

#### 그랄의 접근법
JVM 용 JIT 컴파일러를 자바언어로 개발하는 것입니다. JVMCI 라는 JVM 컴파일러 인터페이스 덕분에 자바 인터페이스를 JVM 에 JIT 컴파일러로 탈착할 수 있습니다. 그랄 프로젝트의 사상은 JIT 컴파일러는 JVM 바이트코드를 기계어로 생성하기만 하면 된다는 역할에 집중하고 있고 자바를 자바로 접근하는 단순함, 메모리 보안 등 여러면에서 장점이 있습니다. 또한 난해한 C++ 기술을 개발자가 몰라도 IDE, 디버거 등의 표준 자바 툴 세트를 이용할 수 있는 장점이 있습니다.
이에 C2 에서 그랄로 부분탈출 등의 새로운 최적화 기법이 가능하고 개발자가 자신의 애플리케이션에 맞게 그랄의 일부를 수정할 수 있는 유연함도 있습니다. (커스텀 인트린직 또는 최적화 패스)

#### 트러플
JVM 에 기반한 언어 전용 인터프리터를 개발하는 프레임워크입니다. 입력 언어에 대한 고성능 JIT 컴파일러를 인터프리터에서 자동 생성하는 라이브러리입니다. 제이루비, 제이썬, 내시혼 등의 JVM 기반으로 움직이는 기존 언어 구현체는 런타임에 바이트코드를 생성하는데, 트러플은 이런 방식의 대안입니다. 그랄과 찰떡궁합으로 작동하도록 설계되었고 함께 사용하면 성능이 대폭 향상될 것입니다.

#### 메트로폴리스 프로젝트
위 모든 것이 *메트로폴리스 프로젝트* 라는 새로운 프로젝트의 일부로 진행중입니다. VM의 더 많은 부분을 자바로 다시 작성하려는 것입니다. 메트로폴리스/그랄 기술은 자바9에 탑재되었지만 아직 베타버전입니다. 사용하려면 옵션을 설정할 수 있습니다. _p 484_

#### 서브스트레이트VM
자바 애플리케이션과 JVM을 통째로 컴파일하여 정적으로 링크된 하나의 네이티브 실행코드를 생성하려는 연구 프로젝트입니다. JVM을 설치할 필요없이 수 KB 에 불과할 만큼 작은 네이티브 바이너리를 수 밀리초만에 시동할 수 있는 환경을 구축하려는 것입니다.


### 바이트코드의 향후 발전 방향
*invokedynamic* 명령어의 등장은 VM의 가장 큰 변화입니다. invokedynamic 은 JVM 바이트코드 작성 방법을 다시 생각하게 만든 도화선 역할을 했습니다. 앞으로도 더욱 유연하게 사용될 전망입니다. 궁극적인 목적은 invokedynamic을 사용하더라도 기존 invokevirtual 처럼 JIT 컴파일하기 좋고 성능도 좋게 나올 수 있게 만드는 것입니다.

### 동시성의 향후 발전 방향
*자동메모리 관리기능* 은 자바가 일으킨 가장 큰 혁신 중 하나 입니다. 자바 스레딩 모델도 초기에는 개발자가 모든 스레드를 직접 관리하고 가변 상태를 lock으로 보호해야 한다는 설계를 가지고 있었습니다. 해서 정확한 개발이 필요했습니다. 하지만 최근 자바는 좀 더 개발자의 손이 덜가고 사실상 런타임이 동시성을 관리하는 방향으로 진화했ㅅ브니다.
*룸프로젝트* 는 이러한 노력의 일환입니다. 스레드 추상화 장치들은 선점형이 아니라 반드시 협동형으로 동작해야 하고 JVM은 7버전부터 이런 추상화 장치를 스케쥴링하는 컴포넌트를 가지고 있었습니다.
```
자바7부터 등장한 포크/조인 API 는 실행 가능한 태스크의 재귀 분해와 작업 훔쳐오기, 두가지 개념에 기초합니다. ForkJoinPool 실행자가 바로 이 두 개념의 핵심으로 작업 훔쳐오기 알고리즘 구현을 담당합니다. 재귀 분해가 모든 태스크에 다 유용한 것은 아니지만 작업 훔쳐오기 알고리즘이 가미된 실행자 스레드 풀은 다양한 상황에 적용할 수 있습니다. ForkJoinPool 실행자는 경량급 실행 객체용 스케쥴링 컴퓨넌트로 활용될 가능성이 크고 이런 기능을 VM 및 코어 라이브러리와 더불어 표준화하면 굳이 외부 라이브러리를 쓰지 않아도 될 것입니다.
```

### 마치며
아직도 여러분야에서 자바의 진보를 위해 노력중입니다. 이곳에서 언급된 내용이 자바 성능의 세계로 친절하고 훌륭한 이정표가 되기를...
