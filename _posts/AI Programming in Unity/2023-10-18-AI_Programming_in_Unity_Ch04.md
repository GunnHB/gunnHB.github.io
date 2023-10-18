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
인공지능 개체가 장애물을 피해가며 목적지에 도달하게 하는 일은 어렵지 않습니다.
예제를 구현할 때 최고의 효율성을 추구하거나 최단 거리 이동을 고려하지는 않을 것이며
최단 거리 탐색과 관련된 A* 길찾기 알고리즘은 다음 절에서 다룰 예정입니다.

다음은 4장에서 다룰 내용입니다.

- 길 찾기와 조향
- 커스텀 A* 길 찾기 구현
- 유니티 내장 NavMesh
</div>

## 경로 따라가기
경로는 일반적으로 웨이포인트를 연결해서 생성합니다. 경로를 만드는 방법은 다양한데
지금 사용할 방법이 아마도 가장 간단한 방법일 가능성이 큽니다.

`Path.cs` 스크립트를 생성하고 모든 웨이포인트를 `Vector3` 배열에 저장합니다.
그런 다음 에디터에서 각 지점을 직접 입력해주면 됩니다.

이 과정이 다소 지겹다면 빈 게임 오브젝트를 웨이포인트로 사용하거나 이런 처리를 자동으로
해주는 편집기 플러그인을 직접 만들 수 있습니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/76221be5-7117-4c78-bf44-78089d39b4ea){: width="70%" height="70%"}

그림에 있는 목록은 경로를 만드는 데 필요한 웨이포인트이고 그 외 두개의 속성
`debug mode` 와 `radius` 가 있습니다.

- debug mode
    - 에디터 윈도우 상에 경로 그려주기
- radius
    - 개체가 목적지에 도달했는지 검사하기 위한 반지름
    - 목적지로부터 지정한 반지름 이내에 들어오면 도착했다고 판단

## 경로 스크립트
```c#
using UnityEngine;
using System.Collections;

public class Path : MonoBehaviour
{
    public bool _bDebug = true;
    public float _radius = 2.0f;
    public Vector3[] _pointA;

    public float Length
    {
        get => _pointA.Length;
    }

    public Vector3 GetPoint(int index)
    {
        return _pointA[index];
    }

    private void OnDrawGizmos()
    {
        if(!_bDebug)
            return;

        for(int index = 0; index < _pointA.Length; index++)
        {
            Debug.DrawLine(_pointA[index], _pointA[index + 1], Color.red);
        }
    }
}
```
보이는 것처럼 매우 간단한 스크립트이고 `Length` 속성을 가지고 있으며 요청을 받으면
웨이포인트 배열의 길이를 반환합니다. `GetPoint` 메소드는 배열 내 특정 인덱스의
웨이포인트를 `Vector3` 위치로 반환합니다.

## 경로 추종자 사용
이제 `VehicleFollowing` 스크립트를 생성합니다. 해당 스크립트에는 몇 개의 변수가 있는데
일단 따라갈 경로 오브젝트의 찹조가 있으며, 가속도를 계산할 때 필요한 `Speed` 와 `Mass`
속성이 있습니다. `IsLooping` 플래그는 경로 이동을 반복할지 결정하는 데 사용합니다.

```c#
using UnityEngine;
using System.Collections;

public class VehicleFollowing : MonoBehaviour
{
    public Path _path;
    public float _speed = 20.0f;
    public float _mass = 5.0f;

    public bool _isLooping = true;

    // 차량의 실제 속도
    private float _curSpeed;

    private int _curPathIndex;
    private float _pathLength;
    private Vector3 _targetPoint;

    private Vector3 _velocity;

    // 속성을 초기화하고 속도 벡터의 방향을 개체의 전방 벡터로 설정
    private void Start()
    {
        _pathLength = _path.Length;
        _curPathIndex = 0;

        // 차량의 현재 속도를 얻는다.
        _velocity = transform.forward;
    }

    // 개체가 특정 웨이포인트에 도달했는지를 경로의 반지름값을 고려해 검사
    private void Update()
    {
        // 속도를 통일
        _curSpeed = speed * Time.deltaTime;

        _targetPoint = _path.GetPoint(_curPathIndex);

        // 목적지의 반지름 내에 들어오면 경로의 다음 지점으로 이동
        if(Vector3.Distance(transform.position, _targetPoint) < _path.radius)
        {
            // 경로가 끝나면 정지
            if(_curPathIndex < _pathLength - 1)
                _curPathIndex++;
            else if(_isLooping)
                _curPathIndex = 0;
            else
                return;
        }

        // 최종 지점에 도착하지 않았다면 계속 이동
        if(_curPathIndex >= _pathLength)
            return;
        
        // 경로를 따라 다음 Velocity 를 계산
        if(_curPathIndex >= _pathLength - 1 && !_isLooping)
            _velocity += Steer(_targetPoint, true);
        else
            _velocity += Steer(_targetPoint);

        // 속도에 따라 차량 이동
        transform.position += _velocity;
        // 원하는 Velocity 로 차량을 회전
        transform.rotation = Quaternion.LookRotation(_velocity);
    }
    
    // 목적지로 벡터의 방향을 바꾸는 조향 알고리즘
    public Vector3 Steer(Vector3 target, bool bFinalPoint = false)
    {
        // 현재 위치에서 목적지 방향으로 방향 벡터를 계산한다.
        Vector3 desiredVelocity = (target - transform.position);
        float dist = desiredVelocity.magnitude;

        // 원하는 Velocity 를 정규화
        desiredVelocity.Normalized();

        // 속력에 따라 속도를 계산
        if(bFinalPoint && dist < 10.0f)
            desiredVelocity *= (_curSpeed * (dis / 10.0f));
        else 
            desiredVelocity += _curSpeed;

        // 힘 Vector3 계산
        Vector3 steeringForce = desiredVelocity - _velocity;
        Vector3 acceleration = steeringForce / mass;

        return acceleration;
    }
}
```

해당 씬을 실행하면 오브젝트가 경로를 따라가는 모습을 확인할 수 있습니다.

## 장애물 회피
이번에 사용할 알고리즘은 아주 간단한 레이캐스팅 방식을 사용하기 때문에 경로 이동 중
정면을 가로막는 장애물만 피해갈 수 있습니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/346a3aaa-fffb-4876-92b4-d2aa2375d5ef){: width="70%" height="70%"}
![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/1165f6e2-d5c6-40d0-bbe5-b9a0c5b0163b){: width="70%" height="70%"}

씬 생성을 위해 몇 개의 큐브 개체를 생성하고 이를 빈 오브젝트 `Obstacle` 아래로 묶습니다.
또한 `Agent` 라는 또 하나의 큐브 오브젝트를 생성하고 여기에 장애물 회피 스크립트를 연결합시다.

`Agent` 는 제대로 된 경로탐색기가 아니므로 너무 많은 벽을 세우면 `Agent` 는 경로탐색이 어려울 수 있습니다.
적절한 수의 벽을 세우고 `Agent` 의 움직임을 살펴봅시다.

그리고 새로운 레이어 `Obstacle` 을 생성하여 장애물 오브젝트의 레이어로 지정해줍니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/ddd6e4a1-9999-41b6-904d-e17f29c36fc0){: width="70%" height="70%"}

## 장애물 회피 로직 구현
이제 규브 개체가 실제로 장애물을 피하도록 스크립트를 작성해봅시다.

```c#
using UnityEngine;
using System.Collections;

public class VehicleAvoidance : MonoBehaviour
{
    public float _speed = 20.0f;
    public float _mass = 5.0f;
    public float _force = 50.0f;
    public float _minimunDistToAvoid = 20.0f;

    // 차량의 실제 속도
    private float _curSpeed;
    private Vector3 _targetPoint;

    private void Start()
    {
        _mass = 5.0f;
        _targetPoint = Vector3.zero;
    }

    private void OnGUI()
    {
        GUILayout.Label("Click anywhere to move the vehicle");
    }

    private void Update()
    {
        // 차량은 마우스 클릭으로 이동
        RaycastHit hit;
        var ray = Camera.main.ScreenPointToRay(Input.mousePoisition);

        if(Input.GetMouseButtonDown(0) && Physics.Raycast(ray, out hit, 100.0f))
            _targetPoint = hit.point;

        // 목표 지점을 향하는 방향 벡터
        Vector3 dir = (_targetPoint - transform.position);
        dir.Normalized();

        // 장애물 회피 적용
        AvoidObstacles(ref fir);

        // 목표 지점에 도착하면 차량을 멈춘다.
        if(Vector3.Distance(_targetPoint, transform.position) < 3.0f)
            return;
        
        // 속도에 델타 타임을 적용한다.
        _curSpeed = _speed * Time.deltaTime;

        // 목표 방향 벡터로 차량을 회전시킨다.
        var rot = Quaternion.LookRotation(dir);
        transform.rotation = Quaternion.Slerp(transform.rotation, rot, 5.0f * Time.deltaTime);

        // 차량을 전진시킨다.
        transform.position += transform.forward * _curSpeed;
    }

    // 장애물 회피를 위해 새 방향 벡터를 계산
    public void AvoidObstacles(ref Vector3 dir)
    {
        RaycastHit hit;

        // Obstacle 레이어 검사 (해당 프로젝트에서는 8번)
        // 비트마스크
        int layerMask = 1 << 8;

        // 회피 최소거리 이내에서 장애물과 차량이 출동했는지 검사 수행
        if(Physics.Raycast(transform.position, transform.forward, out hit, _minimumDistToAvoid, layerMask))
        {
            // 새 방향을 계산하기 위해 충돌 지점에서 법선을 구한다.
            Vector3 hitNormal = hit.normal;
            hitNormal.y = 0.0f;     // Don't want to move in Y-Space

            // 차량의 현재 전방 벡터에 force 를 더해 새로운 방향 벡터를 얻는다.
            dir = transform.forward += hitNormal * _force;
        }
    }
}
```