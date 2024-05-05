---
title:  "[유니티로 배우는 게임 디자인 패턴] 004. 커맨드 패턴으로 리플레이 시스템 구현하기"
excerpt: "커맨드 패턴을 알아봅시다!"

author: gunnHB
categories: 
 - Design Pattern
tags: 
 - [Github, Git, Unity, Programming, Design Pattern, C#]

toc: true
toc_sticky: true
 
date: 2024-05-05
last_modified_at: 2024-05-05
---

🔔 유니티로 배우는 게임 디자인 패턴 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
`커맨드 패턴command pattern`의 메커니즘은 플레이어 컨트롤러 입력과 그에 해당하는 `타임스탬프timestamp`를 기록합니다.
이를 이용하여 올바른 순서와 타이밍을 기록한 입력을 재생할 수 있습니다.

커맨드 패턴은 액션을 수행하거나 상태 변경을 트리거하는 데 필요한 정보를 캡슐화하는 매커니즘을 제안하며,
액션을 수행할 대상 객체와 분리합니다. 액션 요청을 캡슐화 및 분리를 통해 나중에 실행할 수 있도록 큐에 넣습니다.

- 커맨드 패턴 이해하기
- 리플레이 시스템 설계하기
- 리플레이 시스템 구현하기
</div>

## 커맨드 패턴 이해하기
키보드의 스페이스 바를 누르면 점프하는 캐릭터가 있다고 생각해봅시다. 
이를 간단하게 작성해보면 다음과 같습니다.

```c#
using UnityEngine;
using System.Collections;

public class InputHandler : MonoBahaviour
{
    private void Update()
    {
        if (Input.GetKeyDown("space"))
            CharacterController.Jump();
    }
}
```

이런 방식은 작업을 완료할 수 있지만 이후 플레이어의 입력을 기록하고 되돌리며 리플레이할 때
복잡해질 수 있습니다. InputHandler는 플레이어가 스페이스 바를 누를 때 정확히 어떤 액션을 해야하는지
알지 못해도 됩니다. 올바른 명령이 실행되도록 커맨드 패턴 메커니즘이 처리하도록 하면 됩니다.

다음은 개선된 코드입니다.

```c#
using UnityEngine;
using System.Collections;

public class InputHandler : MonoBahaviour
{
    [SerializeField]
    private Controller _characterController;

    private Command _spaceButton;

    private void Start()
    {
        _spaceButton = new JumpCommand();
    }

    private void Update()
    {
        if (Input.GetKeyDown("space"))
            _spaceButton.Execute(_characterController);
    }
}
```

코드를 보면 알 수 있듯 플레이어가 스페이스 바를 누를 때 CharacterController를 직접 호출하지 않습니다.
실제 점프 액션을 수행하는 데 필요한 모든 정보를 객체 내에서 캡슐화하여 나중에 큐에 넣고 다시 실행할 수 있도록 합니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/cc2af2b5-fde9-452e-a2c6-d04c71d3bf59){: width="75%" height="75%"}{: .align-center}

커맨드 패턴에서 사용하는 기본 클래스로

- `invoker`(호출자)
    - 명령을 실행하는 방법을 알고 실행한 명령을 즐겨찾기할 수도 있는 객체
- `receiver`(수신자)
    - 명령을 받아서 수행할 수 있는 종류의 객체
- `commandBase`
    - 개별 `ConcreateCommand` 클래스가 무조건 상속해야 하는 추상 클래스
    - 호출자가 특정 명령을 실행하기 위해 호출할 수 있는 Execute() 메서드를 노출

커맨드 패턴에 사용하는 클래스는 각자의 책임과 역할이 있습니다. 커맨드 패턴을 구현하여 큐에 넣은 후 
즉시 혹은 나중에 실행할 수 있는 오브젝트로 작업 요청을 캡슐화할 수 있습니다.

커맨드 패턴은 행동 패턴의 일부입니다. 이외에 메멘토, 옵저버, 방문자가 있습니다. 책임을 할당하고 다른 객체와
통신하는 방법과 관련있습니다.
{: .notice--info}

### 커맨드 패턴의 장단점
커맨드 패턴의 장점은

- 분리
    - 커맨드 패턴은 실행하는 벙법을 아는 객체에게서 작업을 호출하는 객체를 분리할 수 있습니다. 이러한
    분리 계층으로 즐겨찾기와 시퀀스 작업을 수행하는 중개자를 추가할 수 있습니다.
- 시퀀싱
    - 커맨드 패턴은 되돌리기/다시 하기 기능, 매크로, 명령 큐의 구현을 허용하고 사용자 입력을 큐에 넣는
    작업을 용이하게 합니다.

단점으로는

- 복잡성
    - 각 명령이 그 자체로 클래스입니다. 커맨드 패턴을 구현하려면 수많은 클래스가 필요하며 패턴으로 만들어진
    코드의 유지 보수를 위해 패턴을 잘 이해해야 합니다.

### 커맨드 패턴을 사용하는 경우
- 실행 취소
    - 대부분 텍스트와 이미지 에디터에서 볼 수 있는 실행 취소 및 재실행 시스템을 구현합니다.
- 매크로
    - 공격 혹은 방어 콤보를 기록하고 자동으로 입력 키에 적용하여 실행할 수 있는 매크로 기록 시스템을 구현합니다.
- 자동화
    - 봇이 자동으로 그리고 순차적으로 실행할 명령 집합을 기록하는 자동화 과정 혹은 행동을 구현합니다.

## 리플레이 시스템 설계하기
시스템을 구현하기 위해 게임 구현 방식에 영향을 줄 수 있는 몇 가지를 선언해야 합니다.

- 결정론적
    - 게임의 모든 것은 결정론적입니다. 마음대로 행동하는 객체가 없습니다. 리플레이하는 동안
    같은 방식으로 움직이고 행동합니다.
- 물리
    - 객체의 움직임은 다른 어떤 물리적 요소나 상호작용으로 결정되지 않아 유니티의 Physics 기능을 최소한으로 사용합니다.
- 디지털
    - 모든 입력은 디지털입니다.
- 정확성
    - 입력을 리플레이하는 타이밍이 정확하지 않은 것을 허용합니다.
    `이 허용 수준은 리플레이 기능에 요구하는 정확성과 관련된 다양한 요소에 따라 변경할 수 있습니다.`

## 리플레이 시스템 구현하기
1. 먼저 단일 메서드인 Execute()를 가진 Command의 기본 추상 클래스를 구현합니다.

```c#
public abstract class Command
{
    public abstract void Execute();
}
```

2. 기본 클래스인 Command에서 파생되는 구체적인 세 가지 커맨드 클래스를 작성하고 Execute() 메서드를 구현합니다.
먼저 BikeController의 터보 차저를 토글합니다.

```c#
public class ToggleTurbo : Command
{
    private BikeController _controller;

    public ToggleTurbo(BikeController controller)
    {
        _controller = controller;
    }

    public override void Execute()
    {
        _controller.ToggleTurbo();
    }
}
```

3. TurnLeft와 TurnRight 커맨드입니다. InputHandler를 구현할 때처럼 각 커맨드는 다른 액션을 나타내며
특정 입력 키에 매핑됩니다.

```c#
public class TurnLeft : Command
{
    private BikeController _controller;

    public TurnLeft(BikeController controller)
    {
        _controller = controller;
    }

    public override void Execute()
    {
        _controller.Turn(BikeController.Direction.Left);
    }
}
```

4. 다음은 오른쪽으로 회전하는 액션을 나타냅니다.

```c#
public class TurnRight : Command
{
    private BikeController _controller;

    public TurnRight(BikeController controller)
    {
        _controller = controller;
    }

    public override void Execute()
    {
        _controller.Turn(BikeController.Direction.Right);
    }
}
```

각 명령을 개별 클래스로 캡슐화했으니 리플레이 시스템을 작동시키는 중요한 요소인
invoke를 작성합시다. invoker에서 호출하는 동작들은 C#의 키/값 구조의 정렬된 컬렉션인 SortedList 형식으로 표현합니다.

```c#
using UnityEngine;
using System.Linq;
using System.Collections.Generic;

class Invoker : MonoBahaviour
{
    private bool _isRecording;
    private bool _isReplaying;
    private float _replayTime;
    private float _recordingTime;

    private SortedList<float, Command> _recordedCommands = new();
    
    #region 입력 구현부
    public void ExecuteCommand(Command command)
    {
        command.Execute();

        if (_isRecording)
            _recordedCommands.Add(_recordingTime, command);
        
        Debug.Log("Recorded Time: " + _recordingTime);
        Debug.Log("Recorded Command: " + command);
    }

    public void Record()
    {
        _recordingTime = 0f;
        _isRecording = true;
    }
    #endregion

    #region 리플레이 구현부
    public void Replay()
    {
        _replayTime = 0f;
        _isReplaying = true;

        if (_recordedCommands.Count <= 0)
            Debug.LogError("No Commands to replay!");
        
        _recordedCommands.Reverse();
    }

    void FixedUpdate()
    {
        if (_isRecording)
            _recordingTime += Time.fixedDeltaTime;

        if (_isReplaying)
        {
            _replayTime += Time.deltaTime;

            if (_recordedCommands.Any())
            {
                if (Mathf.Approximatly(_replayTime, _recordedCommands.Keys[0]))
                {
                    Debug.Log("Replay Time: " + _replayTime);
                    Debug.Log("Replay Command: " + _recordedCommands.Values[0]);

                    _recordedCommands.Values[0].Execute();
                    _recordedCommands.RemoveAt(0);
                }
            }
            else
                _isReplaying = false;
        }
    }
    #endregion
}
```

Invoker 클래스에서는 Invoker가 새로운 커맨드를 실행할 때마다 _recordedCommands 목록에 새로운 명령을 추가합니다.

그리고 FixedUpdate()를 사용하여 커맨드를 기록하고 리플레이합니다. 플레이어 입력을 수신할 때는 일반적으로
Update()를 사용합니다. 그러나 FixedUpdate()는 고정된 시간을 간격으로 실행된다는 장점이 있으며 시간에 종속적이지만 프레임 속도와는
무관한 작업에 유용합니다.

기본 엔진 시간 간격은 0.02초입니다.
{: .notice--info}

### 리플레이 시스템 테스트하기
먼저 InputHandler 클래스를 구현합니다. 플레이어의 입력을 받고 적절한 커맨드를 호출합니다.

```c#
using UnityEngine;

public class InputHandler : MonoBahaviour
{
    private bool _isReplaying;
    private bool _isRecording;

    private Invoker _invoker;
    private BikeController _bikeController;
    private Command _buttonA, _buttonD, _buttonW;

    private void Start()
    {
        _invoker = gameObject.AddComponent<Invoker>();
        _bikeController = FindObjectOfType<BikeController>();

        _buttonA = new TurnLeft(_bikeController);
        _buttonD = new TurnRight(_bikeController);
        _buttonW = new ToggleTurbo(_bikeController);
    }

    private void Update()
    {
        if (!_isReplaying && _isRecording)
        {
            if (Input.GetKeyDown(KeyCode.A))
                _invoker.ExecuteCommand(_buttonA);
            
            if (Input.GetKeyDown(KeyCode.D))
                _invoker.ExecuteCommand(_buttonD);
            
            if (Input.GetKeyDown(KeyCode.W))
                _invoker.ExecuteCommand(_buttonW);
        }
    }

    // 테스트를 위한 GUI 버튼
    private void OnGUI()
    {
        if (GUILayout.Button("Start Recording"))
        {
            _bikeController.ResetPosition();
            _isReplaying = false;
            _isRecording = true;
            _invoker.Record();
        }

        if (GUILayout.Button("Stop Recording"))
        {
            _bikeController.ResetPosition();
            _isRecording = false;
        }

        if (!_isRecording)
        {
            if (GUILayout.Button("Start Replay"))
            {
                _bikeController.ResetPosition();
                _isRecording = false;
                _isReplaying = true;
                _invoker.Replay();
            }
        }
    }
}
```

마지막으로 테스트를 위한 BikeController의 스켈레톤 코드를 구현합니다. 커맨드 패턴에서 수신자처럼 작동합니다.

```c#
using UnityEngine;

public class BikeController : MonoBahaviour
{
    public enum Direction
    {
        Left = -1,
        Right = 1,
    }

    private bool _isTurboOn;
    private float _distance = 1f;

    public void ToggleTurbo()
    {
        _isTurboOn = !_isTurboOn;
        Debug.Log("Turbo Active: " + _isTurboOn.ToString());
    }

    public void Turn(Direction direction)
    {
        if (_direction == Direction.Left)
            transform.Translate(Vector3.Left * _distance);
        
        if (_direction == Direction.Right)
            transform.Translate(Vector3.Right * _distance);
    }

    public void ResetPosition()
    {
        transform.position = Vector3.zero;
    }
}
```

## 구현 검토하기
작성된 내용을 프로젝트에 바로 적용시키기엔 굉장히 미비합니다. 하지만 해당 포스트는 사용하는 방법에 대한 포스팅임을 잊지 맙시다.

## 정리
이번 포스팅에서는 커맨드 패턴을 사용하여 간단하고 기능적인 리플레이 기능을 구현해보았습니다.

다음 포스팅에서는 오브젝트 풀을 통해 코드를 최적화해봅시다.