---
title:  "[유니티로 배우는 게임 디자인 패턴] 003. 이벤트 버스로 게임 이벤트 관리하기"
excerpt: "이벤트를 알아봅시다!"

author: gunnHB
categories: 
 - Design Pattern
tags: 
 - [Github, Git, Unity, Programming, Design Pattern, C#]

toc: true
toc_sticky: true
 
date: 2024-04-15
last_modified_at: 2024-04-15
---

🔔 유니티로 배우는 게임 디자인 패턴 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
`이벤트 버스event bus`는 객체가 구독하거나 게시할 수 있는 특정한 전역 이벤트의 목록을 관리하는
중앙 허브 역할을 하며, 이벤트 관리와 관련되 가장 간단한 패턴입니다.

이번 포스트에서는 이벤트 버스로 경주의 전체 상태 변경을 수신해야 하는 구성 요소에 특정 경주 이벤트를
`브로드캐스트broadcast`합니다.

- 이벤트 패턴 개요
- 경주 이벤트 버스 구현
</div>

## 이벤트 버스 패턴 이해하기
객체(게시자)가 이벤트를 발생하게 하면 다른 객체(구독자)가 받을 수 있는 신호를 보냅니다. 오브젝트는 이벤트
시스템에서 이벤트를 브로드캐스트하고 구독하는 오브젝트만 알림을 받으며 어떻게 처리할지 결정합니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/f992f354-46b3-4073-8768-7a0e1bb6abad){: width="75%" height="75%"}{: .align-center}

이벤트 버스는 게시자와 구독자 사이의 중개자 역할을 합니다.
{: .notice--info}

- 게시자 (publisher)
    - 이벤트 버스에서 선언한 특정 종류의 이벤트를 구독자에게 게시할 수 있음
- 이벤트 버스
    - 구독자와 기세자 사이의 이벤트 전송을 조정
- 구독자 (subscriber)
    - 이벤트 버스를 통해 특정 이벤트의 구독자로 자신을 등록

### 이벤트 버스 패턴의 장단점
이벤트 버스의 장점은

- 분리
    - 오브젝트는 직접 서로를 참조하는 대신 이벤트로 통신할 수 있음.
- 단순성
    - 이벤트 버스는 이벤트의 구독 혹은 게시 메커니즘을 추상화하여 단순성을 제공

단점으로는

- 성능
    - 모든 이벤트 시스템의 내부에는 오브젝트 간 메시지를 관리하는 저수준 메커니즘이 있어
    약간의 성능 비용이 발생할 수 있음
- 전역
    - static 메서드와 변수로 구현 시 전역적 접근으로 프로젝트 관리를 어렵게 만듦

### 전역 레이스 이벤트 관리하기
일반적인 레이시 게임 단계는 아래와 같습니다.

- 카운트 다운
    - 오토바이는 카운트 다운 타이머가 끝날 때까지 출발선 뒤에 멈춰있다.
- 레이스 시작
    - 시계가 0이 되면 녹색 신호등이 켜지고 오토바이가 트랙을 따라 앞으로 이동한다.
- 레이스 끝
    - 플레이어가 결승선을 통과하는 순간 레이스는 끝난다.

레이스의 시작과 끝 사이에는 레이스의 현재 상태를 바꿀 수 있는 이벤트가 일어날 수 있습니다.

- 레이스 일시중지
    - 레이스 중 플레이어가 게임을 일시중지할 수 있다.
- 레이스 나가기
    - 플레이어는 언제든지 레이스를 나갈 수 있다.
- 레이스 중지
    - 플레이어가 치명적인 충돌을 당하면 레이스가 중단될 수 있다.

레이스의 전체 상태를 변경할 수 있는 각 단계의 중요한 이벤트 발생을 브로드캐스트하려고 합니다.
레이스의 상황에 따라 특정 방식으로 동작해야 하는 구성 요소들의 특정 동작을 트리거할 수 있습니다.

- HUD
    - 레이스 상태 표시기는 레이스 HUD의 상황에 따라 바뀐다.
- RaceTimer
    - 플레이어가 레이스를 시작할 때만 시작되고 경승선을 넘을 때만 멈춘다.
- BikeController
    - 레이스가 시작되면 오토바이의 브레이크를 해제한다.
- InputRecorder
    - 레이스가 시작할 때 플레이어의 입력을 기록하여 나중에 돌려볼 수 있도록 한다.

## 레이스 이벤트 버스 구현하기
지원하는 특정 레이스 이벤트 종류를 열거형으로 정리합니다.

```c#
public enum RaceEventType
{
    COUNTDOWN,
    START,
    RESTART,
    PAUSE,
    STOP,
    FINISH,
    QUIT,
}
```

실제 게임의 이벤트 버스 클래스입니다.

```c#
using UnityEngine.Event;
using System.Collections.Generic;

public class RaceEventBus
{
    private static readonly IDictionary<RaceEventType, UnityEvent> Events = new();

    public static void Subscribe(RaceEventType eventType, UnityAction listener)
    {
        UnityEvent thisEvent;

        if(Events.TryGetValue(eventType, out thisEvent))
            thisEvent.AddListener(listener);
        else
        {
            thisEvent = new UnityEvent();
            thisEvent.AddListener(listener);
            Events.Add(eventType, thisEvent);   
        }
    }

    public static void Unsubscribe(RaceEventType type, UnityAction listener)
    {
        UnityEvent thisEvent;

        if(Events.TryGetValue(type, out thisEvent))
            thisEvent.RemoveListener(listener);
    }

    public static void Publish(RaceEventType type)
    {
        UnityEvent thisEvent;

        if(Events.TryGetValue(type out thisEvent))
            thisEvent.Invoke();
    }
}
```

여기서 핵심은 Events 딕셔너리입니다. 이벤트 종류와 구독자 간 관계 목록을 유지 및 관리하는 역할을 하며,
private 및 readyonly로 외부에서 수정할 수 없게 합니다.

### 레이스 이벤트 버스 테스트하기
COUNTDOWN 레이스 이벤트 타입을 구독하는 카운트다운 타이머를 작성합니다. COUNTDOWN 이벤트가 게시되면 3초의
카운트다운이 레이스 시작까지 트리거되고, 카운트가 끝나는 순간 레이스의 시작을 알리는 START 이벤트를 게시합니다.

```c#
using UnityEngin;
using System.Collections;

public class CountdownTimer : MonoBehaviour
{
    private float _currentTime;
    private float _duration = 3f;

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.COUNTDOWN, StartTimer);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.COUNTDOWN, StartTimer);
    }

    private void StartTimer()
    {
        StartCoroutine(Countdown());
    }

    private IEnumerator Countdown()
    {
        _currentTime = _duration;

        while (_currentTime > 0)
        {
            yield return new WaitForSeconds(1f);
            _currentTime--;
        }

        RaceEventBus.Publish(RaceEventType.START);
    }

    private void OnGUI()
    {
        GUI.color = Color.blue;
        GUI.Label(new Rect(125, 0, 100, 20), "COUNTDOWN: " + _currentTime);
    }
}
```

CountdownTimer 오브젝트가 활성화될 때마다 Subscribe() 메서드가 호출되고 비활성화될 때마다
구독을 취소합니다.

다음은 START와 STOP 이벤트를 테스트하기 위한 BikeController 클래스입니다.

```c#
using UnityEngine;

public class BikeController : MonoBehaviour
{
    private string _status;

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.START, StartBike);
        RaceEventBus.Subscribe(RaceEventType.STOP, StopBike);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.START, StartBike);
        RaceEventBus.Unsubscribe(RaceEventType.STOP, StopBike);
    }

    private void StartBike()
    {
        _status = "Started";
    }

    private void StopBike()
    {
        _status = "Stopped";
    }

    private void OnGUI()
    {
        GUI.color = Color.green;
        GUI.Label(new Rect(10, 60, 200, 20), "BIKE STATUS: " + _status);
    }
}
```

마지막으로 레이스가 시작되면 레이스 중지 버튼을 표시하는 HUDController 클래스입니다.

```c#
using UnityEngine;

public class HUDController : MonoBahviour
{
    private bool _isDisplayOn;

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.START, DisPlayHUD);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.START, DisPlayHUD);
    }

    private void DisPlayHUD()
    {
        _isDisplayOn - true;
    }

    private void OnGUI()
    {
        if(_isDisplayOn)
        {
            if(GUILayout.Button("Stop Race"))
            {
                _isDisplayOn - false;
                RaceEventBus.Publish(RaceEventType.STOP);
            }
        }
    }
}
```

이벤트의 각 단계를 테스트하고 싶다면 다음의 클래스를 연결해 카운트다운 타이머를 트리거할 수 있습니다.

```c#
using UnityEngine;

public class ClientEventBus : MonoBahviour
{
    private bool _isButtonEnabled;

    private void Start()
    {
        gameObject.AddComponent<HUDController>();
        gameObject.AddComponent<CountdownTimer>();
        gameObject.AddComponent<BikeController>();

        _isButtonEnabled = true;
    }

    private void OnEnable()
    {
        RaceEventBus.Subscribe(RaceEventType.STOP, Restart);
    }

    private void OnDisable()
    {
        RaceEventBus.Unsubscribe(RaceEventType.STOP, Restart);
    }

    private void Restart()
    {
        _isButtonEnabled = true;
    }

    private void OnGUI()
    {
        if(_isButtonEnabled)
        {
            if(GUILayout.Button("Start Countdown"))
            {
                _isButtonEnabled = false;
                RaceEventBus.Publish(RaceEventType.COUNTDOWN);  
            }
        }
    }
}
```

클래스를 살펴보면 어떤 오브젝트도 서로 직접 통신하지 않는다는 것을 알 수 있습니다.

유일하게 공통되는 부분은 RaceEventBus입니다. 더 많은 구독자와 게시자를 쉽게 추가할 수 있습니다. 예를 들어
TrackController 오브젝트가 언제 레이스 트랙을 재설정하는지 알고 싶을 때 RESTART 이벤트를 수신할 수 있습니다.

Action은 하나 이상의 인자를 허용하지만 값을 반환하지 않는 메서드를 가리키는 델리게이트입니다.
UnityAction은 UnityEvents와 함께 사용하도록 설계됐다는 점만 제외하면 기본 C#의 Action처럼 작동합니다.
{: .notice--warning}

## 대안 살펴보기
이벤트 시스템은 방대한 주제이니 더 많은 패턴이 있다는 점을 명심해야합니다. 

- 옵저버
    - 오브젝트가 오브젝트 목록을 유지 및 관리하고 내부 상태 변경을 알리는 패턴
- 이벤트 큐
    - 게시자가 생성한 이벤트를 큐에 저장하고 편한 시간에 구독자에게 전달
- ScritableObject
    - 유니티의 ScriptableObject를 이용한 이벤트 시스템

## 정리
이번 포스팅에서는 유니티에서 이벤트를 게시하고 구독하는 과정을 단순화하는 간단한 패턴인 이벤트 버스를 살펴봤습니다.

다음 포스팅에서는 커맨드 패턴을 사용하여 플레이어의 입력을 리플레이할 수 있는 기능을 만들어봅시다.