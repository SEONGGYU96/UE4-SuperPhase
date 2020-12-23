# UE4-SuperPhase
컴퓨터 게임 프로그래밍 기말 개인 과제

<img src="https://user-images.githubusercontent.com/57310034/102895335-da710800-44a7-11eb-8aa6-39ba49da027b.png"/>

## 주제
간단한 횡스크롤 게임 만들기
- 형식 : 슈퍼마리오 형식의 횡스크롤 액션 게임
- 볼륨 : 총 플레이타임 3 ~ 5분 이내

## 개발 요구사항
- [시작 지점에서 캐릭터가 횡스크롤로 이동하여 목표 지점에 도달하면 스테이지가 종료](#Goal)
- [캐릭터는 이동 및 점프가 가능](#이동과-점프)
- [목표 지점에 도달하기까지의 소요 시간과 획득한 점수를 화면에 표시](#HUD)
- [점수를 획득할 수 있는 아이템들(ex. 동전)을 지형에 배치](#동전coin)
- [점프하여 하단에 닿으면 아이템(ex. 더 점수가 높은 동전)이 생성되는 블록들 배치](#블록)
- [떨어지면 죽는 구멍을 배치](#죽음-Death)
- [문을 열고 들어가면 스테이지 내의 다른 위치로 이동하는 포탈 구현](#포탈)
- [상호작용 키를 이용하여 대화할 수 있는 NPC 하나 이상 구현 (스토리 진행이나 길을 알려주는 정도의 간단한 대화)](#NPC)
- [지정된 위치를 움직이는 적 구현](#몬스터-Wolf) 
    - [플레이어가 닿으면 게임 종료](#죽음-Death)
- [배경음악 재생](#사운드)
- [최소 하나 이상의 효과음 사용](#사운드)

# 에셋
- 캐릭터 : [EpicGames/Paragon - Phase](https://www.unrealengine.com/marketplace/ko/item/e62072ec7bd640b39224f7e47575dd3f)
- 배경 : [EpicGames/Soul: City](https://www.unrealengine.com/marketplace/ko/item/6a5d9e9fa07a456b9a594db0ceb016c0)
- NPC 및 몬스터 : [PROTOFACTOR INC/ANIMAL VARIETY PACK](https://www.unrealengine.com/marketplace/ko/item/c661d0a956454ea4ba6d12c09a687406)

# 구현
- 블루프린트를 사용하여 제작하였습니다.

## 캐릭터

### 이동과 점프
<img src="https://user-images.githubusercontent.com/57310034/102889176-83663580-449d-11eb-9e0e-18a97473cd18.png">

- `CharacterMovement` 를 활용하여 수업에서 배운 예제와 동일하게 이동과 점프를 구현하였습니다.

### 죽음 (Death)
<img src="https://user-images.githubusercontent.com/57310034/102889505-228b2d00-449e-11eb-99f3-d2dc509a43f3.png"/>

- 몬스터 오브젝트에 닿았을 때, 절벽에 위치한 `BoxTrigger` 오브젝트에 닿았을 때 죽는 모션을 재현한 `Death` 이벤트를 실행하고 `DeathDispather` 를 호출합니다.

<img src="https://user-images.githubusercontent.com/57310034/102889942-e60c0100-449e-11eb-96e4-afe807cbe727.png"/>

- 죽었을 때의 효과는 위와 같이 죽는 애니메이션을 실행하고 (`isDeath` 변수로 인해 `AnimBP`에서 실행) `Ragdoll`상태로 변환합니다.

### 상호작용
<img src="https://user-images.githubusercontent.com/57310034/102890126-48650180-449f-11eb-9b81-8dccbf2d3037.png">

- 상호작용 키를 누르면 현재 캐릭터와 `Overlap` 한 `Actor` 들을 순회하며 `Interaction` 인터페이스를 구현했는지 검사합니다. 구현한 경우, 해당 구현부를 호출하도록 하였습니다.
- 순회한 이유는, **NPC에게 말을 걸 때**, **포탈을 이용할 때** 상호작용 키를 모두 이용하기 때문에 이렇게 구현하였습니다.

### 텔레포트
<img src="https://user-images.githubusercontent.com/57310034/102890431-ca552a80-449f-11eb-9cec-854eb0cadfba.png">

- 텔레포트에 사용되는 포털의 경우, `Interaction` 인터페이스의 구현부에서 캐릭터의 위와 같이 구현된 `Teleport` 이벤트를 다시 호출합니다. 이 때 전달받은 `Location`으로 캐릭터는 순간이동하게 됩니다.

## 몬스터 (Wolf)
- 몬스터는 늑대로 구현했습니다.
- 스테이지에 여러 마리의 늑대를 놓고싶었고, 각 늑대의 움직임 패턴을 달리하고싶어서 `turnTime`, `attackTime`, `period`(패트롤 주기), `speed` 값을 `InstanceEditable`로 설정하여 인스턴스마다 다르게 적용할 수 있도록 하였습니다.
<img src="https://user-images.githubusercontent.com/57310034/102890864-831b6980-44a0-11eb-99d5-43ccc9fdaccc.png"/>

- 그리고 `State` 변수를 사용하는데 -1의 경우 일반 패트롤, 0은 방향 전환, 1은 공격 상태입니다. `isTimeOut` 함수를 사용하여 설정된 각 상태 시간동안 행위를 반복하고, 상태를 전환하는 방법으로 설계하였습니다. 이 상태 값은 `AnimBP`에서도 사용되어 애니메이션을 전환하게 됩니다.
- `isTimeOut` 함수는 `deltaSecond` 를 계속 모아 `maximum` 보다 커지는지 계속 검사하는 함수입니다.

## NPC
- NPC는 까마귀로 구현하였습니다.
- 상호작용키를 이용하여 말을 걸면 늑대를 조심하라는 문구와 함께 애니메이션을 변화하도록 하였습니다.

## 동전(Coin)
- 캐릭터가 동전을 먹으면, `GameMode` 에서 이벤트를 받아 점수를 연산합니다. 점수 연산을 `GameMode`로 위임한 까닭은, 블록을 사용해도 점수가 올라가기 때문에 중복 코드를 해소하기 위함입니다.
- 매 프레임마다 z축을 중심으로 3도씩 회전합니다.
<img src="https://user-images.githubusercontent.com/57310034/102891962-79930100-44a2-11eb-8f23-608f3986a8f7.png"/>

## 블록
- 캐릭터가 블록의 하단부를 `Hit`하면, 블록이 위아래로 움직이면서 점수가 카운트 됩니다.
- 이와 동시에 블록에서 나던 빛이 사라지고 매시가 단조롭게 바뀝니다.
<img src="https://user-images.githubusercontent.com/57310034/102892524-54eb5900-44a3-11eb-842b-9adc0ca5f14a.png">

- 블록 `Hit` 효과는 `Timeline` 을 이용하였습니다.

## 포탈
- 두 개의 포탈을 배치하여 상호작용 키를 누르면 서로의 위치로 캐릭터가 이동할 수 있도록 하였습니다.
<img src="https://user-images.githubusercontent.com/57310034/102892696-97ad3100-44a3-11eb-8625-7d2050477adf.png"/>

- 우선 액터 생성 시, 자신의 위치를 `startLocation` 변수에 초기화하고 `isCreated` 변수를 `true`로 초기화합니다.
- 그 다음, `InitTargetPortal` 이벤트를 실행하여 `targetPotal` 이 생성되었는지 `isCreated` 변수를 검사합니다.
- 각 액터는 순차적으로 생성되기 때문에 상대 포털이 아직 생성되지 않을 수 있기 때문에 위와 같은 검사를 진행합니다.
- 생성된 상태라면, `targetPotal`이 현재 포털보다 먼저 생성되었다는 뜻이므로, `targetPotal`은 현재의 포털의 위치를 모릅니다. 따라서 `targetPotal`의 `targetLocation`과 현재 포탈의 `targetLocation`을 초기화해줍니다.
- `targetPotal` 은 `instanceEditable` 이므로 액터마다 다르게 이동할 포털을 유동적으로 지정해줄 수 있습니다.

<img src="https://user-images.githubusercontent.com/57310034/102893108-579a7e00-44a4-11eb-80aa-084ded3f90a9.png"/>

- 포털 또한 캐릭터가 올라갔을 때 조금 아래로 내려가고, 캐릭터가 내려가면 다시 원위치로 올라가는 움직임을 `Timeline`을 통해 구현해두었습니다.

## Goal
- 목표 지점에 `BoxTrigger` 를 설치한 후 아래와 같은 로직으로 `HUD`를 바꾸고 게임을 `Pause` 함으로써 게임을 클리어하도록 하였습니다.
<img src="https://user-images.githubusercontent.com/57310034/102893250-9a5c5600-44a4-11eb-8fd2-979aa262c22a.png"/>

## HUD
- 게임 중에는 획득한 점수와 현재까지의 소요시간을, 클리어 시에는 해당 정보와 축하 텍스트를 보여줍니다. 다음은 게임 중에 보여지는 `HUD` 입니다.
<img src="https://user-images.githubusercontent.com/57310034/102894321-418dbd00-44a6-11eb-9076-474552bfeed0.png"/>

## 사운드
- 배경음악과 캐릭터의 죽음, 늑대의 울음소리를 삽입하였습니다.

# 회고
> 게임 프로그래밍 수업을 들으면서 게임 다운 게임을 만들어볼 수 있어서 정말 즐거웠습니다. 대부분의 개발자는 어렸을 때 게임으로 컴퓨터를 접해봤을 것이라고 생각합니다. 저도 플래시게임을 만들며 개발에 흥미를 가졌던 사람으로써 3D 게임을 직접 개발해본 경험은 감회가 새로웠습니다. 언리얼의 경우 구글에 참고할 자료가 그렇게 많지 않아 개발이 무척 어려웠는데, 이번 수업 자료가 아니었다면 이만큼 절대 만들지 못했을 것 같습니다. 하지만 역시 자료의 부족함으로 원하는 기능을 구현하지 못한 것도 많고, 엉뚱하게 구현한 것도 있습니다. 그 부분은 조금 아쉽습니다.
