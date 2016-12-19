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
# 갑자기 왠 서버 모델 타령이야?
우리가 뭐가 그리 부족하다고?

--

사실 우리는 꽤 잘하고 있음.

[KGC 2016: HTTPS 로 모바일 게임 서버 구축한다는 것](http://www.slideshare.net/XionglongJin1/https-kgc-2016-korea-games-conference)

여기서 '이렇게 하자'는거 보면 다 우리가 이미 하고 있는 것임.

--

그럼에도 ...
---
# 넘치는 서버 수

- 게임 서버가 >130대
  - 메모리 32기가씩 박은...
--

- 64GB씩 박은 DB 서버가 
  - 게임 16세트 x 3 (**비싼** SSD 씀)
  - 로그 4세트 x 3
  - 기타 1세트 x 3
--

- 64GB씩 박은 Redis 장비도...
  - 8대 이상
  - HA 구성은 하지도 않았는데?
--

- 그나마도 SEA DB 장비 수는 한국의 절반 ^^
---

background-size: contain
background-image: url(./images/gamedb_ops.png)
???
곱하기 8 ^^
요새 몽고를 다시 보고 있음. 이래도 버티는구나 하고...
---

background-size: contain
background-image: url(./images/redis_ops.png)
---

class: center
# 아마도 개발팀은 크게 관심없겠지만


장비만 늘린다고 모든게 바로 해결되지 않습니다

--

장비가 늘어나면 고장도 늘어납니다

--

세상에는 장비 관리도 알아서 못하는 퍼블리셔도 있습니다<br>
--
(말로만 들었는데 내가 겪을 줄이야)

--

항상 돈으로 해결할 수 있는 것도 아닙니다
???
물론 돈을 부어서 장비 사양을 올리면 파멸의 시점을 늦춰주는 효과가 있긴 합니다.
---
class: center
# 신작 서비스에 바란다

서버 성능 좀 높이고 DB 액세스 좀 줄여줬으면...!

---
class: center
# 현실은

---

background-size: contain
background-image: url(./images/img_4180.png)

---

background-size: contain
background-image: url(./images/twemproxy.png)

---

class: center
## 누가 이런 무시무시한 혼종을 자꾸 ...

..왜 갈수록 복잡해지는걸까..

이거 내가 나이를 너무먹었나 도저히 못따라가겠다..

틀렸어 이제 꿈도 희망도 없어

--

.image[![the_end](./images/the_end.jpg)]
???
..어쨌든 제 느낌은 저랬다는 겁니다.
---
class: center
# ..어쨌든

모든 것을 포기했지만,

왜 액터 모델이 도움을 줄 거라고 생각했는지,

그 이야기를 해보겠습니다.

---
template: monokai

# An actor
- is lightweight
- can be **stateful**
- **never shares** state
- communicates through **messages**
- **has a mailbox** to buffer messages
- processes **one message a time**
- is a **single threaded object**

--

**Everything is an Actor**<br>
cf. *Everything is an Object*

???
가볍고, 상태를 가질 수 있고, 다른 액터와 상태를 공유하지 않고,
다른 액터와 메시지를 주고 받으면서 통신하고, 우편함이 있고,
한번에 하나를 처리하고, ...
---
# 생각보다 오래된 개념

1973년에 Carl Hewitt 외 2명에 의해 만들어진 개념이라고..

---
class: center

# An Actor Cannot Exist on Its Own

Erlang 같이 언어가 그렇게 생겨먹었다면 모를까 <br>
위의 규칙을 지키도록 구현된 프레임워크가 필요함
---
# 지금 쓰고 있는 것은 어떤데?
Route된 연결이 Controller의 한 메소드를 거치며 모든 것을 처리함

- 요청이 들어오면 다른 함수를 호출해서 처리하고 반환함
- ○○○ 정보가 필요하다 → 1회용 ○○○ 개체를 생성해서 만들어!<br>
- DB에서 필요한 정보를 읽는다
- 뭔가를 처리하고 DB에 다시 쓴다
- 생성된 개체는 함수의 Scope를 벗어나면 폐기됨 (GC가 하든 사용자가 하든)
- 이걸 모든 요청에 대해 반복

--

Worker Thread를 여러 개 쓰는 게임 서버도 기본적으로 비슷함

- 하는 짓이 똑같은 스레드만 많았다 이거임
---
class: middle
# Actor Model?
모든 것을 Actor로 만들 **수도** 있음. 꼭 그럴 필요는 없지만.

* 행위 (Match, Login, ...)
* 데이터 (User, Inventory, ...)

--

Actor Client, 또는 Actor Method는 다른 Actor에 메시지를 보냄

* 함수 리턴값 대신 다른 액터의 응답을 기다림 (async)
* 아니면 보내기만 할 수도 있고 (fire-and-forget)
* 우린 이미 비동기 프로그래밍을 지겹게 해봐서 익숙한 패턴

--

**메시지를 받은 Actor**는 요청을 처리하고 송신자에게 회신함
* 그리고 다른 요청이 있다면 **그걸 계속 처리하고 있을 것**임

이게 한 대, 또는 여러 대의 머신에서 **동시에** 병렬로 돌아감.
???
처리하는 Context가 컨트롤러 메소드에 있느냐, 각 액터에 있느냐...
---
# Login 처리

기존 코드 | Actor Model
--- | ---
Controller Method | LoginActor
Session 데이터를 **불러옴** | **SessionActor**에게 메시지를 보내고 대기
퍼블리셔 연동 서비스에 요청함 | 퍼블리셔 연동 서비스에 요청함
User DB에서 유저 정보를 **읽어옴** 　　　| **UserActor**에게 메시지를 보내고 대기
**Session 데이터를 생성함** | **SessionActor**에게 메시지를 보내고 대기
세션키를 반환 | 로그인 성공 메시지를 반환

뭔가 비슷한 듯 다른 듯...
---
class: center
# Actor는 '상태'(State)를 가질 수 있다

--

그게 뭐 어쨌다고..?
---
class: center

## 어떤 데이터를 갱신할 때, <br>보통 이런 단계를 거친다면

Query Data<br>
↓<br>
Execute Logic<br>
↓<br>
Update Data

--

## **매번** 1 Read, 1 Update가 필요

그리고 관련 모델은요... MatchEnd라도 되면 건드리는 게 대체...?

---
class: center

## 각각을 액터로 짰다고 생각하면

데이터 액터에게 조회 요청<br>
↓<br>
Execute Logic<br>
↓<br>
데이터 액터에게 갱신 요청<br>

--

### 이렇게 봐선 뭐가 다른지 모르겠다

---
# Actor State

- 생성될 때 DB에서 불러올 수 있다
- 요청을 받으면 그대로 복사본을 만들어 전달한다
- 상태가 변경될 때 DB에 저장할 수 있다
  - 변경이 잘 일어나지 않는다면?

--

오오 공짜 캐시 오오
???
'불러올 수 있다', '저장할 수 있다'인 까닭은 이걸 사용자가 해줄 수도, Persistence 기능이 해줄 수도 있기 때문.
---

Actor Framework은 다음을 보장한다.

* 싱글 스레드로 동작한다
* **한번에 하나씩** 처리한다
* 특별히 지정하지 않는다면 한 액터는 한번에 하나의 동작만 처리한다 <br>(non-reentrant)

Race Condition이 발생하기가 더 어렵다.
---
# 다음 동작은 동시에 일어날 수 있다
.left[
* User **A**는 선수 카드를 까는 중이고,
* User **B**는 매치가 끝나서 보상을 받고 있다
]

--

User A와 User B는 서로 다른 액터가 처리한다
---
# 다음은 동시에 일어날 수 없다
.left[
* User **A**가 매치가 끝나고 보상을 받는다
* User **A**가 자기가 가진 선수 카드를 까고 있다
]

--

카드를 까고 보상을 받든 보상을 받고 카드를 까든 차례대로
---
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
# 데이터 관리 주체가 명확해짐

각 모델을 액터로 만들었다면
* 생성될 때 데이터를 읽어온다
* 상태가 변경되었을 때만 저장하면 됨
* Getter / Setter를 적절히 만들어주자<br>
  다른 곳에서는 복사본을 가져가서 쓸 뿐!

특정 모델의 데이터는 그 모델 액터에서만 처리하게 됨.
---
여기에 Transaction이 나오면 좀 복잡해집니다만 ^_^... <br>
프레임워크에 따라서 서로 다른 방법으로 처리합니다.

* Orbit은 EventSourceActor 기반으로 Transaction을 처리함
* Service Fabric도 자체 트랜잭션 기능이 있음
* 여러가지 패턴으로 구현하려는 시도가 Orleans에도 있는 듯

요는 변경된 내용을 어떻게, 한꺼번에 처리하느냐니까요...

---
class: center
# 그렇게 좋다는데 왜 난 처음 들어봤지

그것이 말입니다...

---
# 일단 언어에 따른 제약이...

* 동시/병렬 처리를 언어에서 지원하거나, 런타임에서 잘 지원하거나
* Python, ECMAScript (node), Ruby 등은 그렇지 못한 예

.footnote[
multiprocess로 구현하지 못할 것은 아니지만 왜 그런 짓을
]

---
# 그럼 JVM이나 CLR로는?

Akka의 .NET 포팅인 Akka.NET 샘플 코드를 잠깐 봅시다

---
```C#
public OrderProcessorActor() {
	Receive<PlaceOrder>(placeOrder => PlaceOrderHandler(placeOrder));
	Receive<OrderPlaced>(orderPlaced => OrderPlacedHandler(orderPlaced));
	Receive<AccountCharged>(accountCharged => AccountChargedHandler(accountCharged));
}
private void PlaceOrderHandler(PlaceOrder placeOrder) {
	var orderActor = Context.ActorOf(
		Props.Create(
			() => new OrderActor(
				(int)DateTime.Now.Ticks)),
		"orderActor" + DateTime.Now.Ticks);
	orderActor.Tell(placeOrder);
}
private void OrderPlacedHandler(OrderPlaced orderPlaced) {
	var accountActor = Context.ActorOf(
		Props.Create(
			() => new AccountActor(
				orderPlaced.OrderInfo.AccountId)),
		"accountActor" + orderPlaced.OrderInfo.AccountId);
	accountActor.Tell(new ChargeCreditCard(orderPlaced.OrderInfo.ExtPrice));
}```
---
class: center
# 엄.. 내가 방금 뭘 본거지

--

생소한 개념이 너무 많아!
???
뭔가 시스템의 날것을 그대로 본 기분! 
콜백을 대신하겠다고 제너레이터를 그대로 쓰는 기분!
---
# 생소하다

* 함수 호출 대신 **메시지 전달** (Ask, Tell)
* 메소드 정의 대신 **메시지 처리기** 정의 (Receive)
* 파라미터 대신 **Data Transfer Object**

*하나하나 까보면 이해가 가지 않는건 아닌데, 뭔가 알아야 하는게 너무 많아...*

--

메시지 전달은 Objective-C 같은 언어를 쓰셨던 분들에게는 익숙할지도?

패턴 매칭 등을 지원하는 함수형 언어와는 매우 잘 맞는다고 합니다

---
# 설정할 것, 만들 것이 너무 많다

* 여러 서버에 배치하려면?
  * Akka Clustering을 또 익혀야 함
* Actor의 생성, 소멸, 에러 처리 등의 관리가 필요
  * 이런 것을 제어하고 싶은 사람에게는 강점일지도
  * Actor Tree, State 같은 개념은 정말 강력해 보입니다
---
# Virtual Actor Framework
.center[
소개합니다
]
--

- [MS Orleans](https://dotnet.github.io/orleans/) 
  - Microsoft Research에서 2011년부터 개발, 2014년 공개
  - Halo 4, Halo 5 등의 서비스를 담당함
--

- [MS Azure Service Fabric](https://azure.microsoft.com/en-us/services/service-fabric/)
  - MS Azure의 서비스에 사용됨, 2015년 GA
--

- [**EA** (Bioware) Orbit](https://github.com/orbit)
  - Orleans에 영향을 받아 JVM으로 작성됨
  - Slack 채널도 있습디다
???
AKKA 스타일이 주류였으나 이걸 한번 뒤집었다고 할까
---
# 가상 액터 (Virtual Actors)

액터의 수명을 관리할 필요가 없음

- Actor instances always exist, virtually
- 각 Actor는 구별을 위해 고유 Key를 가짐
- 이미 생성되어 있으면 불러오고, 없으면 프레임워크에서 생성

--

프레임워크가 다 해주십니다

- 생성 / GC 모두 런타임이 알아서

---

# Clustering 기본 지원

클러스터 안에서, 어느 Actor가 어느 호스트에 있는지 알 필요가 없음

--

어려운 말로 **위치 투명성** (Location Transparency) 

--

클라이언트는 액터 참조를 가져와서 메시지를 보내고 응답을 기다린다
- 비동기!

---

# Clustering 기본 지원

호스트가 다운되면 어떻게 하나요?

- 바로 그 작업을 호출했던 클라이언트는 에러를 받을 수도 있음
  - 이것도 테스트해서 데모하면 좋을거라 생각은 했습니다...

- 다음 호출은 (이론상) 정상적으로 처리됨
  - Cluster 어딘가에서 새로 짜잔하고 뜬 액터가...

--
  - *난 아마 세 번째일거야*

*테스트를 안해봤으니 일단은 믿거나 말거나*

.image[![rei_third](./images/rei_third.jpg)]

???
마치 복제된 것처럼 죽으면 다른 곳에 뜬다..고 하는군요
---

background-size: contain
background-image: url(./images/grain_reactivation.png)

---
class: middle, center
이런 작업을 최대한 어색하지 않은(!) 구문으로 구현할 수 있는 것이 특징

샘플 코드를 잠시 보실까요

---
```C#
public class MatchGrain : Grain, IGrainWithIntegerKey, IMatchGrain
{
	List<IUserGrain> users;
	Dictionary<string, List<PlayerData>> userTeamData;
	Random r = new Random();

	private IUserTeamPlayersGrain GetTeamPlayersById(string id)
	{
		return GrainFactory.GetGrain<IUserTeamPlayersGrain>(id);
	}

	public override Task OnActivateAsync()
	{
		users = new List<IUserGrain>();
		userTeamData = new Dictionary<string, List<PlayerData>>();
		return base.OnActivateAsync();
	}

	// join match
	public async Task<bool> JoinMatch(string userId)
	{
		users.Add(GrainFactory.GetGrain<IUserGrain>(userId));
		// 아무것도 하지 않지만, 매치 전에 Player Data를 읽어들인다는 상징적인 의미로
		var teamData = await GetTeamPlayersById(userId).GetTeamPlayers();
		userTeamData.Add(userId, teamData);
		return true;
	}

```
---
```C#
	public async Task<bool> EndMatch()
	{
		// 돈과 경험치 보상
		users.ForEach(async user =>
		{
			var moneyToAdd = r.Next(10, 100) * 100;
			var expToAdd = moneyToAdd / 10;
			await user.AddMoneyExp(dmoney: moneyToAdd, dexp: expToAdd);
		});

		// 선수들의 경험치 업데이트
		foreach (var kvp in userTeamData)
		{
			var userid = kvp.Key;
			var teamPlayers = kvp.Value;
			teamPlayers.ForEach(player =>
			{
				player.Exp += r.Next(10, 100) * 5;
			});
			await GetTeamPlayersById(userid).UpdatePlayers(teamPlayers);
		}

		this.DeactivateOnIdle();
		return true;
	}
}
```
???
예제 코드로 짰던 것을 가져와봤는데, 아까에 비하면 훨씬 읽기 편한 코드라고 발표자는 생각함
---

# 프로젝트 기본 구성 요소

Orleans에서는 가상 Actor을 Grain, 런타임이 도는 호스트를 Silo라고 함

- Grain Interfaces Project
  - 인터페이스만 담음. 클라이언트와 공유함
- Grain Implementation Classes Project
  - 실제 코드는 여기에 몰리게 됨. Interface를 참조함
- Silo Host Project
  - Interface, Implementation 모두 참조함

--

외부 연동시 Orleans Client로 접근하게 됨
- REST API를 구현하는 웹 프레임워크를 앞에 놓고
- Controller 메소드에서 Client를 가져다가 접근하는 식

---

class: center
background-size: contain
background-image: url(./images/actor_model_as_stateful_middleware.png)
???
캐시고 뭐고 다 알아서 하겠다는 의지의 표현이 잘 보이는 그림.
---

# 주의할 점

--

모든 Actor 호출은 RPC Call임

- Local이든, Remote든 Serialize & Deserialize 비용이 듬
- 너무 주고받는 대화가 잦은 (*Chatty*) 액터들은 통합하는게 맞음

--

Stateful Actor는 메시지를 '*차례대로 하나씩*' 처리함

- 1000 : 1 집계 같은 사용에는 적합하지 않음
- StatelessWorker로 구현하고 내부에 적절히 락을 쓴다든지

--

여럿이 공유해야 하는 State가 있는 경우 적합하지 않음

- MMORPG 서버가 이런 구조가 많다고 함
- 우리랑은 크게 관계 없는 이야기

---

# 실제로 구현해보니

- 역시 프로젝트가 여러 개인 점은 좀 불편했음
- FO3처럼 게임 서버만 달랑 띄우고 디버깅하기는 어려울 듯

--

  - 이건 아일랜드 구조를 봐도 암담하겠던데...

- 벼락치기였지만, 개발 속도는 꽤 빨랐음
  - 기존 C# 문법과 크게 다르지 않고 유사하게 짤 수 있었음

--- 

# 그밖의 특징

실행 단위가 Silo인 점
- Grain (Actor) 은 Silo 안에서 무수히 많이 만들어지고 실행됨
- 밖에서 보면 외부로의 연결은 Silo Process가 담당함

--

즉

- 게임 서버를 열개를 띄울지 열다섯개 띄울지 쓸데없이 고민하지 않아도 됨
- mongos를 Silo 하나당 하나씩 띄워도 연결이 폭주하지 않을 듯

--

사실 이 점은 언어가 병렬처리만 지원해도 해결되는 점이지만...

<s>우리는 아니지!</s>

---

# 왜 이걸 쓰자고 주장하지 못했나

- 스튜디오의 ECMAScript 사랑
  - TypeScript까지 이어지다
  - ES 2016이나 TS를 보니 이쯤이면 그냥 C#을 써도 될 것 같은데
--

- Windows + .NET Framework용 
  - .NET Core을 지원하는 2.0이 이제 **Preview**가 나옴
  - 왠지 분위기상 Windows Server를 쓰자고 하면 돌맞을 것 같았음
--

- MS의 늦은 행보
  - .NET Core 1.0 발표를 앞두고 오락가락함
  - .NET Framework에 비해 **빠진 API가 잔뜩** <s>제정신인가..</s> 
  - 아직도 SDK는 Preview 상태. <s>이놈들 의지가 있는건가..</s>

---