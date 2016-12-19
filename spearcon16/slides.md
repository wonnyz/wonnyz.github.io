name: inverse
layout: true
class: center, middle, monokai
---
name: monokai
layout: true
class: middle, monokai
---
template: inverse

# Actor Model로 서버를 짜보자
사실은 Microsoft Orleans 알아보기

---
사실은 처음 생각했던 것보다 많이 줄어든 것

도중에 의욕이 많이 도망가서 (..)

그냥 이런 것이 있고 알아보니 이랬다는 것만 알려드리는 것으로
???
어차피 안 바뀔 거잖아요?
---
template: monokai

## 생각보다 오래된 개념

1973년에 Carl Hewitt 외 2명에 의해 만들어진 개념이라고..

**Everything is an Actor**<br>
cf. *Everything is an Object*

--
.footnote[
모든 것을 액터로 처리할 필요는 없긴 함. <br>
기존 프로그래밍 모델과 섞어쓸 수도 있음
]
---

## An actor
- is lightweight
- can be **stateful**
- **never shares** state
- communicates through **messages**
- **has a mailbox** to buffer messages
- processes **one message a time**
- is a **single threaded object**
???
가볍고, 상태를 가질 수 있고, 다른 액터와 상태를 공유하지 않고,
다른 액터와 메시지를 주고 받으면서 통신하고, 우편함이 있고,
한번에 하나를 처리하고, ...
---
template: inverse

## An Actor Cannot Exist on Its Own

Erlang 같이 언어가 그렇게 생겨먹었다면 모를까 <br>
위의 규칙을 지키도록 구현된 프레임워크가 필요함
---
layout: false
class: middle
# 지금 쓰고 있는 것은 어떤데?
Route된 연결이 Controller의 한 메소드를 거치며 모든 것을 처리함
.left[
* 요청이 들어오면 다른 함수를 호출해서 처리하고 반환함
* ○○○ 정보가 필요하다 → 1회용 ○○○ 개체를 생성해서 만들어!<br>
  함수의 Scope를 벗어나면 폐기됨 (GC가 하든 사용자가 하든)
* 이걸 모든 요청에 대해 반복
]
Worker Thread를 여러 개 쓰는 게임 서버도 기본적으로 비슷함
.left[
* 하는 짓이 똑같은 스레드만 많았다 이거임
]
---
class: middle
# Actor Model에서는?
모든 것을 Actor로 만들 **수도** 있음

* 행위 (Match, Login, ...)
* 데이터 (User, Inventory, ...)

Actor Client, 또는 Actor Method는 다른 Actor에 메시지를 보냄

* 함수 리턴값 대신 다른 액터의 응답을 기다림 (async)
* 아니면 보내기만 할 수도 있고 (fire-and-forget)
* 우린 이미 비동기 프로그래밍을 지겹게 해봐서 익숙한 패턴

**메시지를 받은 Actor**는 요청을 처리하고 송신자에게 회신함
* 그리고 다른 요청이 있다면 **그걸 계속 처리하고 있을 것**임

이게 한 대, 또는 여러 대의 머신에서 **동시에** 병렬로 돌아감.
???
처리하는 Context가 컨트롤러 메소드에 있느냐, 각 액터에 있느냐...
---
class: middle
# Login 처리
.left[
기존 코드 | Actor Model
--- | ---
Controller Method | LoginActor
Session 데이터를 **불러옴** | **SessionActor**에게 메시지를 보내고 대기
퍼블리셔 연동 서비스에 요청함 | 퍼블리셔 연동 서비스에 요청함
User DB에서 유저 정보를 **읽어옴** 　　　| **UserActor**에게 메시지를 보내고 대기
**Session 데이터를 생성함** | **SessionActor**에게 메시지를 보내고 대기
세션키를 반환 | 로그인 성공 메시지를 반환
]
뭔가 비슷한 듯 다른 듯...
---
class: middle, center
Actor는 '상태'(State)를 가질 수 있다.

*? 그게 왜 중요하지*
---
class: middle, center
template: inverse

## 어떤 데이터를 갱신할 때, <br>보통 이런 단계를 거친다면

Query Data<br>
↓<br>
Execute Logic<br>
↓<br>
Update Data

--

## 모든 작업에 대해 <br>1 Read, 1 Update가 필요.

---
class: middle, center
template: inverse

## 각각을 액터로 짰다고 생각하면

데이터 액터에게 조회 요청<br>
↓<br>
Execute Logic<br>
↓<br>
데이터 액터에게 갱신 요청<br>

--

### 이렇게 봐선 뭐가 다른지 모르겠다

---
class: middle
# Actor State

* 생성될 때 불러올 수 있다
* 상태가 변경될 때 저장할 수 있다
* 변경이 잘 일어나지 않는다면?

오오 공짜 캐시 오오
???
게다가 또 있다
---
class: middle, center
# 복습

Actor Framework은 다음을 보장한다.

.left[
* 싱글 스레드로 동작한다
* **한번에 하나씩** 처리한다
* 특별히 지정하지 않는다면 한 액터는 한번에 하나의 동작만 처리한다 <br>(non-reentrant)
]

Race Condition이 발생하기가 더 어렵다.
---
class: middle
다음 동작은 동시에 일어날 수 있다
* User **A**는 선수 카드를 까는 중이고,
* User **B**는 매치가 끝나서 보상을 받고 있다

User A와 User B는 서로 다른 액터가 처리한다.
---
class: middle
다음은 동시에 일어날 수 없다.
* User **A**가 매치가 끝나고 보상을 받는다
* User **A**가 자기가 가진 선수 카드를 까고 있다

카드를 까고 보상을 받든 보상을 받고 카드를 까든 차례대로.
---
class: middle
# 액터 스스로가 캐시가 된다

* 자신의 상태를 유지하면서 메모리에 떠 있다
* 수명 관리는 프레임워크에서 해준다
* (클러스터) 어느 장비에 생성될지도 프레임워크에서 처리한다
* Race Condition도 어느 정도 막아준다<br>
  (비즈니스 로직상 막아야 하는 부분은 어쩔 수 없다)

손이 덜 가는 캐시 탄생.
???
물론 세션 데이터 등이라면 Redis를 백엔드로 삼든지 해야할 것이다<br>
Service Fabric같이 백엔드 스토어를 제공하는 프레임워크도 있음. 별도 저장소 불필요
---
class: middle
# 데이터 관리 주체가 명확해짐

각 모델을 액터로 만들었다면
* 생성될 때 데이터를 읽어온다
* 상태가 변경되었을 때만 저장하면 됨
* Getter / Setter를 적절히 만들어주자<br>
  다른 곳에서는 복사본을 가져가서 쓸 뿐!

특정 모델의 데이터는 그 모델 액터에서만 처리하게 됨.
---
class: middle, center
물론 분산 Transaction이 나오면 좀 복잡해집니다만 ^_^...

* 프레임워크에 따라서 서로 다른 방법으로 처리함<br>
  * Orbit은 EventSourceActor 기반으로 Transaction을 처리함
  * Service Fabric도 자체 트랜잭션 기능이 있음
* Interceptor / Actor Locking 등으로 처리 가능할 듯
