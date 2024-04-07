---
title:  "[유니티로 배우는 게임 디자인 패턴] 002. 상태 패턴으로 캐릭터 상태 관리하기"
excerpt: "상태 패턴을 알아봅시다!"

author: gunnHB
categories: 
 - Design Pattern
tags: 
 - [Github, Git, Unity, Programming, Design Pattern, C#]

toc: true
toc_sticky: true
 
date: 2024-04-07
last_modified_at: 2024-04-07
---

🔔 유니티로 배우는 게임 디자인 패턴 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
게임에서는 플레이어의 입력이나 이벤트에 따라 하나의 상태에서 다른 상태로 끊임없이 전환됩니다.

이러한 캐릭터의 유한 상태를 관리하고자 `상태 패턴`을 사용합니다. 

- 상태 패턴의 개요
- 메인 캐릭터의 상태를 관리하기 위한 상태 패턴의 구현
</div>

## 상태 패턴의 개요
상태 패턴의 구조에는 세 가지 핵심 요소가 있습니다.

- Context
    - 클라이언트가 객체의 내부 상태를 변경할 수 있도록 요철하는 인터페이스를 정의
- IState
    - 구체적인 상태 클래스로 연결할 수 있도록 설정
- ConcreateState
    - IState 인터페이스를 구현하고 Context 오브젝트가 상태의 동작을 트리거하기 휘해 호출하는
    public 메서드 Handle()를 노출

![스크린샷 2024-04-07 114138](https://github.com/GunnHB/ProjectG/assets/117302300/450a3983-361e-47a4-9a82-65a324aa9ad4){: width="75%" height="75%"}{: .align-center}

클리이언트는 객체의 상태를 업데이트하고자 Context 객체를 활용해 원하는 상태로 설정하거나
새로운 상태로 전환할 것을 요청합니다.

상태 패턴은 하나의 클래스에서 모든 상태를 정의하고 switch case문으로 전환을 관리하는 것보다 확장성이 뛰어납니다.
{: .notice--info}

## 캐릭터 상태 정의하기
다음 이미지는 레이싱 게임에서 오토바이 객체의 유한 상태를 보여주는 다이어그램입니다.

![image](https://github.com/GunnHB/ProjectG/assets/117302300/2b73a7fe-a216-4384-b20d-b6e279d6ffd9){: width="75%" height="75%"}{: .align-center}

- 정지
    - 오토바이가 움직이지 않는다.
    - 속도는 0이며 기어는 중립
- 시작
    - 최고 속도롤 움직이기 시작하며, 바퀴는 전진 모션에 맞춰 회전
- 회전
    - 플레이어의 입력에 따라 왼쪽이나 오른쪽으로 회전
- 충돌
    - 충돌 상태라면 불이 붙고 옆으로 쏠림
    - 플레이어의 입력에 더 이상 응답하지 않는다.

네 가지의 상태는 오토바이의 상태를 간략하게만 정의한 것입니다. 원하는 상태를 추가하거나 빼는 것은 개발자의 몫입니다.
{: .notice--warning} 

## 상태 패턴 구현하기
정의한 오토바이의 유한 상태에서 예상할 수 있는 동작을 캡슐화한다는 목표로 상태 패턴을 구현합시다.

간결함과 명확성을 위해 최소한의 스켈레톤 코드로 작성됩니다.
{: .notice--info}

### 상태 패턴 구현하기
```c#
public interface IBikeState
{
    void Handle (BikeController controller);
}
```

일반적으론 Context 오브젝트를 상태에 전달합니다. 이러한 방식보다 쉽기 때문에 해당 방식을 사용합니다.
{: .notice--warning} 

```c#
public class BikeStateContext
{
    public IBikeState CurrentState { get; set; }

    private readonly BikeController _bikeController;

    public BikeStateContext(BikeController bikeController) 
    {
        _bikeController = bikeController;
    }

    public void Transition()
    {
        CurrentState.Handle(_bikeController);
    }

    public void Transition(IBikeState state)
    {
        CurrentState = state;
        CurrentState.Handle(_bikeController);
    }
}
```

위 코드를 보면 알 수 있는 것처럼 Transition() 함수를 통해 상태를 전환할 수 있습니다.

```c#
using UnityEngine;

public class BikeController : MonoBehaviour
{
    public float _maxSpeed = 2f;
    public float _turnDistance = 2f;

    public float CurrentSpeed { get; set; }

    // enum
    public Direction CurrentTurnDirection { get; private set; }

    private IBikeState _startState, _stopState, _turnState;

    private BikeStateContext _bikeStateContext;

    private void Start()
    {
        _bikeStateContext = new BikeStateContext(this);

        _startState = gameObject.AddComponent<BikeStartState>();
        _stopState = gameObject.AddComponent<BikeStopState>();
        _turnState = gameObject.AddComponent<BikeTurnState>();

        _bikeStateContext.Transition(_stopState);
    }

    public void StartBike()
    {
        _bikeStateContext.Transition(_startState);
    }

    public void StopBike()
    {
        _bikeStateContext.Transition(_stopState);
    }

    public void Turn(Direction direction)
    {
        CurrentTurnDirection = direction;
        _bikeStateContext.Transition(_stopState);
    }
}
```

개별적인 상태 클래스 내부의 오토바이 동작을 캡슐화하지 않았다면 BikeController 내부에 구현했을 것입니다.
그렇게 되면 유지 및 관리가 힘든 장황한 controller 클래스를 사용해야 합니다.

```c#
public class BikeStopState : MonoBehaviour, IBikeState
{
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
        if (!_bikeController)
            _bikeController = bikeController;
        
        _bikeController.CurrentSpeed = 0f;
    }
}
```

```c#
public class BikeStartState : MonoBehaviour, IBikeState
{
    private BikeController _bikeController;

    public void Handle(BikeController bikeController)
    {
        if (!_bikeController)
            _bikeController = bikeController;

        _bikeController.CurrentSpeed = _bikeController._maxSpeed;
    }

    private void Update()
    {
        if (_bikeController)
        {
            if (_bikeController.CurrentSpeed > 0f)
                _bikeController.transform.Translate(Vector3.forward * (_bikeController.CurrentSpeed * Time.deltaTime));
        }
    }
}
```

```c#
using UnityEngine;

public class BikeTurnState : MonoBahviour, IBikeState
{
    private Vector3 _turnDirection;
    private BikeController _bikeController;

    public void Handle(BikeController _bikeController)
    {
        if (!_bikeController)
            _bikeController = bikeController;
        
        _turnDirection.x = (float)_bikeController.CurrentTurnDirection;

        if (_bikeController.CurrentSpeed > 0f)
            transform.Translate(_turnDirection * _bikeController.turnDistance);
    }
}
```

```c#
public enum Direction
{
    Left = -1,
    Right = 1,
}
```

## 상태 패턴의 장단점
상태 패턴의 장점은

- 캡슐화
    - 상태가 변할 때 개체에 동적으로 할당할 수 있는 컴포넌트의 집합이기에 개체의 상태별 행동을 구현할 수 있다.
- 유지 및 관리
    - 긴 조건문이나 방대해진 클래스의 수정 없이도 새로운 상태를 구현할 수 있다.

하지만 상태 패턴은 애니메이션 캐릭터를 관리하기에는 한계가 있다.

- 블렌딩
    - 기본 형태에서 상태 패턴은 애니메이션 블렌드 방법을 제공하지 않는다. 시각적으로 어색한 전환이 일어난다.
- 전환
    - 상태 간의 전환에 대한 코드를 추가적으로 작성해야 한다. 대기 상태에서 걷기 상태로 전환하고 걷기에서 달리기로 전환하는 경우이다.

## 정리
이번 포스팅에서는 상태 패턴으로 메인 캐릭터의 상태별 행동을 정의하고 구현해보았습니다.

다음 포스팅에서는 게임의 전역 상태를 정의하고 이벤트 버스로 관리하는 법을 알아보겠습니다.