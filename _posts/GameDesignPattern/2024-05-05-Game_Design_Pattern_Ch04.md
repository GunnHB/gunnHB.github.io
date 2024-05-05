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
알지 못해도 됩니다. 올바른 명력이 실행되도록 커맨드 패턴 메커니즘이 처리하도록 하면 됩니다.

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