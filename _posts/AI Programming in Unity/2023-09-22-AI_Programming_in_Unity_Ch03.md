---
title:  "[유니티 게임 AI 프로그래밍] 003. 센서 구현"
excerpt: "센서를 구현해봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-09-22
last_modified_at: 2023-09-24
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
3장에서는 살아있는 생명체와 유사한 감각 시스템 개념을 사용해 인공지능 행동을 구현해볼 예정입니다.

NPC의 인공지능 수준은 전적으로 환경으로부터 얻는 정보의 종류와 양에 의해 결정됩니다.
인공지능 캐릭터는 이런 정보를 토대로 어떤 로직을 실행할지 결정하는데,
만일 정보가 충분하지 않으면 캐릭터는 이상한 행동을 하게 될 것입니다.

모든 환경 매개변수를 검사하고 그에 대응하도록 만들 수도 있지만 좀 더 효율적인 디자인 패턴을 사용하면
코드의 유지보수와 확장이 쉬워집니다.

3장에서는 센서 시스템을 만들 때 유용하게 사용할 수 있는 디자인 패턴을 소개할 예정입니다.

- 센서 시스템의 의미
- 다양한 센서 시스템 소개
- 센서를 사용해 탱크 예제 설정하기
</div>

## 기본 센서 시스템
인공지능 센서 시스템은 시각이나 청각 후각 등을 흉내내는 형태로 사물을 인지합니다.
센서 시스템에서 에이전트는 자신이 관심을 두는 감각에 대해 주기적인 환경 검사를 수행합니다.

기본 센서 시스템 개념에는 2개의 컴포넌트가 존재합니다.

- Aspect
- Sense

우리의 인공지능 캐릭터는 지각이나 후각, 촉각 같은 감각기관을 통해 적을 감지할 것입니다.
기본적으로 구현하려는 베이스 인터페이스 `Sense`로 다른 커스텀 센스가 상속받아서 구현하게 됩니다.

만일 인공지능 캐릭터가 적을 발견하면 알림을 받아 어떤 행동을 처리하면 됩니다.

가장 먼저 최소한의 형태를 가지는 `Aspect` 클래스를 만들어봅시다.

## 콘 형태의 시야
2장에서 적의 시야에 플레이어가 있는지 감지하는 기능을 구현하기 위해 유니티의 
`레이캐스트` 방식을 사용했습니다. 이는 매우 간단하고 효율적인 모델이지만 시야를 처리하는데
그리 정확하진 않습니다.

이를 보완하기 위해 콘`Corn` 모양의 시야를 사용하는데, 이 방식은 2D, 3D 모두에서 사용 가능합니다.

![Ch003_001](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/21816567-8a2c-4b99-bfb4-0b7a8d975e79){: .align-center}{: width="70%" height="70%"}

위 그림처럼 에이전트의 눈으로부터 콘이 시작되어 커지며 거리가 멀어질수록 정확도는 낮아집니다.

💡색상의 농도는 거리에 따라 정확도가 낮아지는 모습입니다.
{: .notice--info}

간단한 모델에서는 거리가 주변을 곻려하지 않고 단지 겹치는지만 확인하면 됩니다. 좀 더 복잡한 구현에서는
콘이 원점으로부터 멀어짐에 따라 볼 수 있는 영역도 늘어나지만 콘의 가장자리로 갈수록 중앙부에 비해
볼 수 있는 가능성은 줄어듭니다.

## 구를 사용해서 듣기, 느끼기, 냄새 맡기
구를 활용하면 매우 간단하면서도 효과적인 소리와 촉각, 냄새를 모델링할 수 있습니다.

소리를 예로 들면 중앙 부분을 소리의 근원지로 생각할 수 있고 청자가 중심으로부터 멀어질수록 소리의 크기는 줄어듭니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/da7afb15-e97a-45ba-ae76-73726a2626ff){: .align-center}{: width="70%" height="70%"}

시야를 어떻게 설정하느냐에 따라서 거리에 따른 인지 확률이 달라집니다. 물론 단순하게 겹치는지만 검사하는
형태를 취하면 거리에 무관하게 모두 인지됩니다.

## 전지적 능력을 활용한 인공지능 확장
사실 전지적인 능력을 잘 활용하면 인공지능을 크게 향상시킬 수 있습니다. 이는 에이전트가 모든 걸 알아야 한다는 건 아니며
필요에 따라 모든 걸 알아낼 수 있다는 것을 의미합니다. 물론 이는 현실적이지 않지만 때론 최고의 해법이 될 수 있습니다.

게임에서는 추상적인 개념을 구체적인 값으로 모델링하는 경향이 있습니다. 
예를 들어 체력을 0부터 100 사이의 숫자로 표현합니다. 이런 정보에 직접 접근할 수 있는 것이 현실적으로 보이진 않지만
좀 더 실용적인 의사결정을 내리는 데는 큰 도움이 됩니다. 이런 부분에 대해서는 에이전트가 게임 내 설정상 전지적인
능력을 갖추고 있다고 이해해도 큰 무리는 없습니다.

## 플레이어 탱크와 특성 설정
`Target` 오브젝트는 메시 렌더러를 사용하지 않는 간단한 구 형태의 오브젝트입니다. 또한 점 광원`point light`도 하나
만들어서 `Target` 오브젝트의 자식으로 만듭니다. 다음은 `Target.cs` 파일 내의 코드입니다.

```c#
public class Target : MonoBehaviour
{
    public Transform targetMarker;

    private void Update()
    {
        int button = 0;

        // 마우스를 클릭하면 충돌 지점을 얻는다.
        if(Input.GetMouseButtonDown(button))
        {
            Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);

            RaycastHit hitInfo;

            if(physics.Raycast(ray.origin, ray.direction, out hitInfo))
            {
                Vector3 targetPosition = hitInfo.point;
                targetMarker.position = targetPosition;
            }
        }
    }
}
```

해당 스크립트는 마우스 클릭 이벤트를 감지한 후 레이캐스팅 기법을 사용해서 3D 공간의 평면상에서 마우스가 클릭된 곳을 찾아냅니다.
그 후 Target 오브젝트를 해당 위치로 이동시킵니다.

## 플레이어 탱크 구현
우리가 사용하는 탱크는 단순한 모델로 인공지능 캐릭터와 충돌 감지 및 처리를 위해서는 강체 컴포넌트가 필요합니다.
이를 위해 `PlayerTank.cs` 를 생성해 맵 위에서 타겟 위치를 가져와서 목적지와 방향을 갱신합니다.

``` c#
using UnityEngine;
using System.Collections;

public class PlayerTank : MonoBehaviour
{
    public Transform targetTransform;
    private float movementSpeed;
    private float rotSpeed;

    private void Start()
    {
        movementSpeed = 10f;
        roSpeed = 2f;
    }

    private void Update()
    {
        // 타켓 위치 근처에 도달하면 일단 정지
        if(Vector3.Distance(transform.position, targetTransform.position) < 5f)
            return;

        // 현재 위치로부터 타겟 위치로의 방향 벡터 계산
        Vector3 tarPos = targetTransform.position;
        tarPos.y = transform.position.y;
        Vector3 dirRot = tarPos - transform.position;

        // LookRotation 메소드를 사용해 이 새로운 회전 벡터를 위한 Quaternion 구성
        Quaternion tarRot = Quaternion.LookRotation(dirRot);

        // 보간법을 사용해서 이동하고 회전
        transform.rotation = Quaternion.Slerp(transform.rotation, tarRot, rotSpeed * Time.deltaTime);

        transform.Translate(new Vector3(0, 0, monvementSpeed * Time.deltaTime));
    }
}
```

이 스크립트는 맵상에서 `Target` 오브젝트의 위치를 가져와서 그것에 맞게 목적 지점과 방향을 갱신합니다.

💡Target 오브젝트를 targetTransform 변수에 지정하는 것을 잊지 맙시다.
{: .notice--info}

## Aspect 클래스 구현
Aspect 클래스는 매우 간단한 클래스로 단 하나의 퍼블릭 속성 `aspectName`을 가집니다. 인공지능 캐릭터가
무언가를 감지할 때는 항상 `aspectName`을 검사해 인공지능 캐릭터가 찾고 있는 대상인지 검사할 예정입니다.

```c#
using UnityEngine;
using System.Collections;

public class Aspect : MonoBehaviour
{
    public enum aspect
    {
        Player,
        Enemy,
    }

    public aspect aspectName;
}
```

해당 스크립트는 플레이어 탱크에 연결하고 `aspectName`을 `Enemy`로 설정합니다.

## 인공지능 캐릭터 생성
인공지능 캐릭터는 씬 내에서 임의의 방향으로 돌아다니는데 여기에서는 2가지의 감각을 사용합니다.

- 지정된 시야 범위와 일정 거리 이내에 적이 존재하는지 검사하는 시각
- 인공지능 캐릭터를 포위할 적 특성을 갖는 개체가 박스 콜라이더와 충돌할 때 이를 감지할 촉각

앞에서도 살펴본 것처럼 플레이어 탱크는 `Enemy` 특성을 가지므로, 인공지능 캐릭터는 플레이어 탱크를
감지할 수 있습니다.

```c#
using UnityEngine;
using System.Collections;

public class Wander : MonoBehaviour
{
    private Vector3 tarPos;

    private float movementSpeed = 5f;
    private float rotSpeed = 2f;
    
    private float minX;
    private float maxX;
    private float minZ;
    private float maxZ;

    // 초기화에 사용
    private void Start()
    {
        minX = -45f;
        max = 45f;

        minZ = -45f;
        maxZ = 45f;

        // 돌아다닐 위치 얻기
        GetNextPosition();
    }

    // Update 는 초당 1회 호출됨
    private void Update()
    {
        if(Vector3.Distance(tarPos, transform.position) <= 5f)
            GetNextPosition();  // Generate new random position

        // 목적지 방향으로의 회전을 위한 쿼터니온 설정
        Quaternion tarRot = Quaternion.LookRotation(tarPos - transform.position);

        // 회전과 트랜슬레이션 갱신
        transform.rotation = Quaternion.Slerp(transform.rotation, tarRot, rotSpeed * Time.deltaTime);
        Transform.Translate(new Vector3(0, 0, movementSpeed * Time.deltaTime));
    }

    private void GetNextPosition()
    {
        tarPos = new Vector3(Random.Range(minX, maxX), .5f, Random.Range(minZ, maxZ));
    }
}
```

`Wander` 스크립트는 인공지능 캐릭터가 현재 목적지에 도달할 때마다 새로운 임의의 지점을 
지정된 영역 내에서 생성합니다. 이후 `Update` 메소드는 적을 회전시키고 새로운 목적지를 향해
이동시킵니다. 이 스크립트를 인공지능 캐릭터에 연결하면 이제 씬 내에서 돌아다니는 모습을 볼 수 있습니다.

## Sense 클래스 사용
`Sense` 클래스는 다른 커스텀 감각을 구현할때 사용하는 인터페이스로, 2개의 가상 메소드
`Initialize` 와 `UpdateSense`를 정의하고 있습니다. 이들 메소드는 커스텀 클래스에서 내용을 구현하며
각기 `Start`와 `Update` 메소드에서 실행됩니다.

```c#
using UnityEngine;
using System.Collections;

public class Sense : MonoBehaviour
{
    public bool bDebug = true;
    public Aspect.aspect aspectName = Aspect.aspect.Enemy;
    public float detectionRate = 1f;

    protected float elapsedTime = 0f;
    
    protected virtual void Initialize() { };
    protected virtual void UpdateSense() { };

    private void Start()
    {
        elapsedTime = 0f;
        Initialize();
    }

    private void Update()
    {
        UpdateSense();
    }
}
```

기본 속성은 찾고자 하는 특성의 이름과 더불어 감지율도 포함하고 있습니다.
이 스크립트는 어떤 오브젝트에도 연결되진 않습니다.

## 약간의 시각 부여
시각은 특정한 특성을 가진 개체가 시야 범위 내에 있는지 검사합니다. 만일 찾던 개체를 발견하면 특정 동작을 수행합니다.

```c#
using UnityEngine;
using System.Collections;

public class Perspective : Sense
{
    public int FieldOfView = 45;
    public int ViewDistance = 100;

    private Transform playerTras;
    private Vector3 rayDirection;

    protected override void Initialize()
    {
        // 플레이어 위치 찾기
        playerTrans = GameObject.FinGameObjectWithTag("Player").transform;
    }

    // Update는 프레임 당 한번 호출
    protected override void UpdateSense()
    {
        elapsedTime += Time.deltaTime;

        // 검출 범위에 있으면 시각 검사를 수행
        if(elapsedTime >= detectionRate)
            DetectAspect();
    }

    // 인공지능 캐릭터에 대한 시야를 검사
    private void DetectAspect()
    {
        RaycastHit hit;

        rayDirection = playerTrans.position - transform.position;

        // 인공지능 캐릭터의 전방 벡터와 플레이어와 인공지능 캐릭터 사이의
        // 방향 벡터 간의 각도를 검사
        if(Vector3.Angle(rayDirection, transform.forward) < FieldOfView)
        {
            // 플레이어가 시야에 들어왔는지 검사
            if(Physics.Raycast(transform.position, rayDirection, out hit, ViewDistance))
            {
                Aspect aspect = hit.collider.GetComponent<Aspect>();

                if(aspect != null)
                {
                    // 특성 검사
                    if(aspect.aspectName == aspectName)
                        print("Enemy Detected");
                }
            }
        }
    }
}
```

플레이 테스트를 하는 동안 `OnDrawGizmos` 메소드는 시야 범위를 표시하는 선을 그려서 인공지능 캐릭터가
에디터 창에서 어디까지 볼 수 있는지를 표시합니다. 이 스크립트를 인공지능 캐릭터에 연곃라고 특성 이름을
잊지 말고 `Enemy`로 설정합시다.

```c#
private void OnDrawGizmos()
{
    if(playerTrans == null)
        return;

    Debug.DrawLine(transform.position, playerTrans.position, Color.red);

    Vector3 frontRayPoint = transform.position + (transform.forward * ViewDistance);

    // 대략적인 시야 범위 시각화
    Vector3 leftRayPoint = frontRayPoint;
    leftRayPoint.x += FieldOfView * .5f;

    Vector3 rightRayPoint = frontRayPoint;
    rightRayPoint.x += FieldOfView * .5f;

    Debug.DrawLine(transform.position, frontRayPoint, Color.green);

    Debug.DrawLine(transform.position, leftRayPoint, Color.green);

    Debug.DrawLine(transform.position, rightRayPoint, Color.green);
}
```

## 촉각 활용
플레이어 개체가 인공지능 개체 근처에 있을 때 알아채는 용도로 사용하는 촉각을 구현하기 위해
`Touch.cs`를 구현합시다.

`OnTriggerEnter` 이벤트를 구현해야 콜라이더 컴포넌트가 다른 콜라이더 컴포넌트와 충돌할 때
알림을 받을 수 있습니다.

```c#
using UnityEngine;
using System.Collections;

public class Touch : Sense
{
    void OnTriggerEnter(Collider other)
    {
        Aspect aspect = other.GetComponent<Aspect>();

        if(aspect != null)
        {
            if(aspect.aspectName == asepectName)
                print("Enemy Touch Detected");
        }
    }
}
```

`OnTriggerEnter` 메소드 냉에서 충돌한 다른 개체의 특성 컴포넌트에 접근해 인공지능 캐릭터가
찾고 있던 특성이 맞는지 이름을 확인합니다. 현재 단계에서는 적의 특성만을 출력하도록 했지만
플레이어가 갑자기 적에게 달려들어 추적과 공격을 하는 구현을 추가할 수 있습니다.

## 요약
3장에서는 인공지능 캐릭터의 감각을 구현하는 개념을 소개했고 2개의 감각으로 시각과 촉각을 구현해봤습니다.