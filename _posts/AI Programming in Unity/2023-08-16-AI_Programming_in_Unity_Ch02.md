---
title:  "[유니티 게임 AI 프로그래밍] 002. 유한 상태 기계"
excerpt: "유한 상태 기계를 알아봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-08-16
last_modified_at: 2023-09-10
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
2장에서는 유니티를 이용하여 FSM 사용 방법을 다룰 예정입니다.

- 유니티의 상태 기계 기능 이해
- 자신만의 상태와 전이 만들기
- 예제를 사용해서 간단한 씬 만들기

우리의 게임에서 플레이어는 탱크를 조종할 수 있습니다. 적 탱크는 4개의 웨이포인트를 참조하면서
씬 위를 돌아다니다가 플레이어 탱크가 자신의 시야에 들어오면 추격을 시작합니다.
그리고 충분한 사거리 내로 진입하면 플레이어 탱크를 공격하게 만들 것입니다.

적 탱크의 인공지능을 구현하기 위해 FSM을 사용해 봅시다!
</div>

## FSM 활용
일상생활을 하면서 우리는 많은 것들을 상태에 따라 구분합니다. 프로그래밍할 때 가장 효율적인 패턴은
실세계의 설계를 단순화해 흉내내는 것인데 이때 사용할 수 있는 기법이 FSM입니다.

예를 들어 전등은 켜지거나 꺼지는 두 상태 중 하나이고, 물은 고체이거나 액체, 기체 상태일 수 있는 것처럼입니다.

프로그래밍 패턴을 구현하는 데 엄격한 규칙이 있는 것은 아니지만, FSM은 기본적으로 특정 순간에 하나의 상태만
가지는 특성이 있습니다. 그렇지만 상태 전이는 두 상태 사이에서의 `핸드오프`를 허용합니다.

이제 상태 기계가 어떤 것인지 대강 알았으니 본격적으로 자신만의 상태 기계를 만들어봅시다!

## AnimatorController 에셋 생성
`AnimatorController` 에섯은 유니티 내장 에셋 타입으로 상태와 전이를 제어합니다. 기본적으로 FSM이지만
훨씬 더 많은 기능을 제공합니다. 그렇지만 우리는 FSM 기능에만 집중하도록 합시다.

애니메이터 컨트롤러를 생성하면 프로젝트 에셋 폴더에 나타나며 이름을 정할 수 있는데,
여기서는 TankFSM으로 하겠습니다. 해당 에셋을 선택하면 다른 에셋 타입과 달리 계층 구조가 비어 있는데,
이는 애니메이션 컨트롤러가 자신만의 윈도우(Animator)를 사용하기 때문입니다.

💡Animation이 아닌 Animator를 선택했는지 확인합시다. 이 둘은 완전히 다른 기능입니다.
{: .notice--warning}

## 레이어와 매개변수
`Layer`라는 이름이 암시하듯, 이 기능을 사용하면 서로 다른 상태 기계를 쌓을 수 있습니다.
이 패널을 사용하면 레이어를 쉽게 정리할 수 있으며 시각적 표현을 얻을 수 있습니다.

`Parameter` 패널은 상태 변경 시점을 결정하는 변수로 스크립트를 통해 접근할 수 있습니다.
매개변수 타입은 4종류로 `float`, `int`, `bool`, `trigger` 가 있습니다. 앞의 세 개는
익숙하지만 trigger는 애니메이터 컨트롤러에 특화된 타입으로 명시적으로 상태를 전환하는 수단입니다.
`물리 트리거`와 혼동하지 않도록 주의합시다.

## 애니메이션 컨트롤러 인스펙터
애니메이션 컨트롤러 인스펙터는 유니티의 일반적인 인스펙터와 약간 다른 모습인데,
일반적인 인스펙터에서는 게임 오브젝트에 컴포넌트를 추가할 수 있지만 애니메이션 컨트롤러 인스펙터는
`Add Behaviour` 버튼이 있어 `StateMachineBehaviour`를 추가할 수 있습니다. 

## 행동 적용
상태 기계 행동은 유니티 5에서 지원하는 독특하고 새로운 개념입니다. 이름만 봐서는 마치
`MonoBehaviour`와 관련이 있어보이지만 전혀 그렇지 않으므로 주의합시다.

사실 행동은 `MonoBehaviour`가 아닌 `ScriptableObject`를 상속받기 때문에 애셋으로만 존재하므로
씬에 위치하거나 컴포넌트 형태로 `GameObject`에 추가할 수 없습니다.

## 첫 상태 생성
엄밀히 말해 유니티가 `New State`, `Any State`, `Entry`, `Exit` 같은 몇 개의 기본 상태를 만들어 두기에
첫 상태는 아니지만, 우리가 직접 생성하는 첫 상태를 만들어봅시다.

첫 상태를 만들기 위해 캔버스 아무 곳에서나 오른쪽 클릭을 하고 `Create State`를 선택합시다.
몇 개의 선택이 나오는데 그 중 `Empty`를 선택하면 됩니다. 나머지 두 선택들은 지금 진행하는
프로젝트에 바로 적용할 수 없으므로 일단 넘어갑시다.

## 상태 간 전이
직접 상태를 만들면 `Entry` 상태에서 나오는 연결 화살표가 생성되며 노드는 `오랜지색`입니다.
하나의 상태만 가지면 자동으로 기본 상태가 되며 자동으로 엔트리 상태에 연결됩니다. 특정 상태를
오른쪽 클릭으로 선택한 후 `Set as Layer Default State`를 클릭하면 기본 상태로 변경할 수 있습니다.

연결 화살표는 상태 전이 연결자로 이를 활용하면 상태 전이의 시점과 형태를 일부 제어할 수 있습니다.
하지만 상태 전이는 자동으로 발생하므로 엔트리 상태에서 기본 상태로 전이되는 건 변경할 수 없습니다.

상태 간의 전이는 수동으로 지정할 수 있습니다. 상태 노드에서 오른쪽 클릭 후 `Make Transition`을 
선택하면 됩니다. 그러면 전이 화살표가 선택한 상태로부터 생성되어 마우스 커서로 이어지며,
전이하고자 하는 대상 노드를 간단히 클릭하면 연결이 이루어집니다.

## 탱크 생성
이제 애니메이터 컨트롤러를 애셋 폴더에 생성해봅시다. 이 컨트롤러는 적 탱크의 상태 기계 역할을
수행합니다. 이름은 `EnemyFsm`으로 정합시다.

이 상태 기계는 탱크의 기본 행동을 제어합니다. 앞에서 설명한 것처럼 우리의 적은 플레이어에
대한 정찰과 추격, 사격을 할 수 있다. 이제 상태 기계를 설정해봅시다.

EnemyFsm 애셋을 선택하고 Animator 창을 열어 3개의 빈 상태를 생성합시다. 이들은 개념적/기능적으로
적 탱크의 상태를 나타냅니다. 이름은 `Patrol`, `Chase`, `Shoot` 으로 정합시다. 그리고 Patrol 를
기본상태로 지정해줍니다.

## 상태 전이 선택
이제는 상태의 흐름에 대한 설계와 로직을 결정해야 합니다. 이때 논리적이고 디자인 관점에서 명확한
형태로 만들어야합니다.

- Patrol
    - 정찰 상태에서는 추격 상태로 전이가 가능하며, 조건 체인을 사용해 변경할 상태를 선택할 예정입니다.
      만일 적 탱크가 플레이어를 볼 수 있다면 상태를 변경하고 다음 단계로 진행하고 보지 못한다면 정찰을 계속 합니다.
- Chase
    - 이 상태에서는 계속 추격이 가능한지를 검사하는 동시에 사격이 가능한 범위에 들어왔는지를 검사하거나
      시야에서 놓쳤는지를 검사합니다. 시야에서 놓치면 다시 정찰 상태로 돌아갑니다.
- Shoot
    - 이 상태에서도 마찬가지로 시야를 확인하며 시야에 들어오면 추격 상태로 변경하고 다시 정찰 상태로 돌아간다.

## 기능 연결
지금까지는 우리가 사용할 FSM에서는 논리적인 상태 연결에 관한 내용을 살펴봤는데, 이제부터는 본격적으로 `코드 작업`을
진행해봅시다. Patrol 상태를 선택하면 계층 구조에서 `Add Behaviour` 버튼을 볼 수 있습니다. 이 버튼을 클릭하면
상태 기계 행위에 특화된 내용을 보여줍니다. 이 행위의 이름을 `TankPatrolState`로 정하면 동일한 이름의
스크립트가 프로젝트에 생성됨과 동시에 상태에 연결됩니다.

```c#
using UnityEngine;
using System.Collections;

public class TankPatrolState : StateMachineBehaviour
{
    // // OnStateEnter는 상태 전이가 시작될 때 호출되고
    // // 상태 기계는 이 상태 평가를 시작한다.
    // override public void OnStateEnter(Animator animator,
    //     AnimatorStateInfo stateInfo, int layerIndex)
    //     {

    //     }

    // // OnStateUpdate는 OnStateEnter와 OnStateExit 콜백 사이
    // // Update 프레임마다 호출된다.
    // override public void OnStateUpdate(Animator animator,
    //     AnimatorStateInfo stateInfo, int layerIndex)
    //     {

    //     }
    
    // // OnStateExit는 상태 전이가 끝날 때 호출되며 이 상태 평가를 종료한다.
    // override public void OnStateExit(Animator animator,
    //     AnimatorStateInfo stateInfo, int layerIndex)
    //     {

    //     }

    // // OnStateMove는 Animator.OnAnimatorMove() 코드 직후 호출된다.
    // // 루트 모션 처리는 이곳에 구현해야 한다.
    // override public void OnStateMove(Animator animator,
    //     AnimatorStateInfo stateInfo, int layerIndex)
    //     {

    //     }
    
    // // OnStateIK는 Animator.OnAnimatorIK() 직후에 호출된다.
    // // 애니메이션 IK(inverse kinematics)는 이곳에 구현해야 한다.
    // override public void OnStateIK(Animator animator,
    //     AnimatorStateInfo stateInfo, int layerIndex)
    //     {

    //     }
}
```

해당 파일은 유니티가 생성했지만 모든 메소드는 주석 처리된 상태입니다. MonoBehaviour 가 제공하는
메소드와 유사하게 이들 메소드도 기반 로직에 의해 호출되며 동작 원리를 자세히 알 필요는 없습니다.
또한 `OnStateIK`, `OnStateMove` 메소드는 애니메이션 메시지로 지금은 신경 쓸 필요가 없어
삭제해도 무방합니다.

- OnStateEnter
    - 전이가 시작되어 상태에 진입할 때 호출됩니다.
- OnStateUpdate
    - MonoBehaivour 업데이트 후 매 프레임 호출됩니다.
- OnStateExit
    - 전이가 끝난 후 상태에서 벗어났을 때 호출됩니다.
- OnStateIK
    - IK 시스템이 업데이트되기 전에 호출됩니다. 이는 애니메이션과 리깅에 특화된 개념입니다.
- OnStateMove
    - 최상위 모션을 사용하기 위해 설정하는 아바타에서 사용합니다.

알아둬야 하는 중요한 또 하나의 정보는 메소드에 전달되는 매개변수입니다.

- Animator
    - 이 애니메이터 컨트롤러를 담고 있는 애니메이터에 대한, 즉 상태 기계입니다.
- AnimatorStateInfo
    - 현재 상태에 대한 정보를 제공하지만, 사실 주된 용도는 애니메이션 상태 정보 그 자체에 있으므로
      지금 사용하기엔 적합하지 않습니다.
- layerIndex
    - 이는 상태 기계의 어떤 레이어에 우리의 상태가 존재하는지 알려주는 정수로 기본 레이어의 
      인덱스는 0 입니다.

### 조건 설정
적 탱크의 상태 전이를 위해 몇 개의 조건을 제공해야 합니다.

먼저 Patrol 상태를 살펴봅시다. 적 탱크가 Patrol 상태에서 Shoot 상태로 변경되려면 플레이어가 일정
범위에 들어왔는지 알아야 합니다. 이를 위해서는 적 탱크와 플레이어 사이의 거리를 표현할 `float` 타입이 필요한데
`Parameter` 패널에서 float 을 추가해줍시다. 해당 매개변수는 Chase 상태로 변경할지 판단할 때도 사용 가능합니다.

Shoot 상태와 Chase 상태는 플레이어가 시야에 들어왔는지에 대한 조건을 공통적으로 사용합니다. 이를 위해 간단한 
`레이캐스트raycast` 방식을 사용해 직선 방향에 플레이어가 보이는지 판단합니다. 

### 코드로 매개변수 제어
다음으로, `EnemyTankPlaceHolder` 게임 오브젝트에 TankAi.cs 스크립트를 추가합니다.

```c#
using UnityEngine;
using System.Collections;

public class TankAi : MonoBehaviour
{
    // 범용 상태 기계 변수
    private GameObject player;
    private Animator animator;
    private Ray ray;
    private RayCastHit hit;
    private float maxDistanceToCheck = 6.0f;
    private float currentDistance;
    private Vector3 checkDirection;

    // Patrol 상태 변수
    public Transform pointA;
    public Transform pointB;
    public NavMeshAgent navMeshAgent;

    private int currentTarget;
    private float distanceFromTarget;
    public Transform[] wayPoints = null;

    private void Awake()
    {
        player = GameObject.FindWithTag("Player");
        animator = gameObject.GetComponent<Animator>();
        pointA = GameObject.Find("p1").transform;
        pointB = GameObject.Find("p2").transform;
        navMeshAgent = gameObject.GetComponent<NavMeshAgent>();
        wayPoints = new Transform[2]{ pointA, pointB };
        currentTarget = 0;
        navMeshAgent.SetDestination(wayPoints[currentTarget].position);
    }

    private void FixedUpdate()
    {
        // 일단 플레이어와의 거리를 검사한다.
        checkDistance = Vector3.Distance(player.transform.position, transform.position);
        animator.SetFloat("distanceFromPlayer", currentDistance);

        // 그런 다음 시야에 들어왔는지 확인한다.
        checkDirection = player.transform.position - transform.position;
        ray = new Ray(transform.position, checkDirection);

        if(Physics.Raycast(ray, out hit, maxDistanceToCheck))
        {
            if(hit.collider.gameObject == player)
                animator.setBool("isPlayerVisible", true);
            else
                animator.SetBool("isPlayerVisible", false);
        }
        else
            animator.SetBool("isPlayerVisible", false);

        // 마지막으로 다음 웨이포인트 대상까지의 거리를 구한다.
        distanceFromTarget = Vector3.Distance(wayPoints[currentTarget].position, transform.position);
        animator.SetFloat("distanceFromWayPoint", distanceFromTarget);
    }

    public void SetNextPoint()
    {
        switch(currentTarget)
        {
            case 0:
                currentTarget = 1;
                break;
            case 1:
                currentTarget = 0;
                break;
        }

        navMeshAgent.SetDestination(wayPoints[currentTarget].position);
    }
}
```

### 적 탱크 이동
눈치챘겠지만 아직 우리의 탱크는 이동할 수 없는데 `서브 기계 상태Sub State Machine`를 사용하면 쉽게 처리할 수 있습니다.
서브 기계 상태는 상태 내에 존재하는 상태 기계를 말합니다. 약간은 이상할 수 있지만 잘 생각해보면
정찰 상태는 2개의 서브 상태로 나눌 수 있습니다.

- 하나의 웨이포인트로 이동하는 상태
- 다음 웨이포인트를 찾는 상태

서브 상태를 추가하기 위해 다시 상태 기계로 돌아갑시다.

캔버스 내의 빈 곳을 클릭하고 `Create Sub-State Machine`을 선택하면 서브 상태를 만들 수 있습니다.

## 요약
2장에서는 간단한 탱크 게임을 통해 유니티에서 제공하는 애니메이터 컨트롤러 기반의 상태 기계를 사용한
구현을 상펴봤습니다.

3장에서도 계속 탱크 게임을 만들어가면서 주변을 둘러싼 세상에 대한 정보를 감각을 이용해 인지하고 이를
응용하는 방법에 대해 살펴보겠습니다.