---
title:  "[유니티 게임 AI 프로그래밍] 004. 군집 처리"
excerpt: "군집 처리에 대해 알아봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-10-30
last_modified_at: 2023-11-01
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
소수의 무리에 대한 처리는 구현이 매우 간단하며 단 몇 줄의 코드만으로도 현실감을 부여할 수 있지만
군집은 이보다는 좀 더 복잡합니다. 하지만 유니티가 제공하는 도구의 도움을 받는다면 
조금은 쉽게 처리할 수 있습니다.

5장에서 다룰 내용입니다.

- 군집 처리의 역사
- 군집 개념의 배경 이해
- 유니티를 사용한 군집 처리
- 전통 알고리즘을 사용한 군집 처리
- 사실감 있는 군집 처리
</div>

## 군집 처리의 기원
군집 처리 알고리즘의 역사는 80년대 중반으로 거슬러 올라갑니다. 처음 개발한 사람은
`크레이그 레이놀즈Craig Reynolds`로 영화 제작을 목적으로 개발했습니다. 이후로 군집 처리 알고리즘은
영화와 게임, 과학 분야에 널리 적용되기 시작했습니다. 매우 효율적이고 정교함에도 불구하고
알고리즘은 이해하고 구현하기에 매우 간단합니다.

## 군집 처리 개념의 배경 이해
군집 처리 알고리즘은 새가 하나의 목적지를 향해 함께 날아갈 때 서로 간에 일정한 거리를 유지하는 것에 착안해
개발됐습니다. 그룹 형태를 처리할 때는 우연성이나 사전에 정의된 경로에 의존하지 않고 그룹 내의 개별 개체가
고유한 움직임을 보이도록 해야합니다. 이때 중요한 점은 처리할 개체가 많으므로 연산 효율이 높은 방식이어야 합니다.

우리는 두 가지 방식으로 군집 처리를 구현할 것입니다.

-`Tropical Paradise Island` 데모 프로젝트에 있는 예제
- 군집 처리 알고리즘

알고리즘은 다음의 3가지 주요 개념을 기반으로 동작합니다.

- 분산
 - 그룹 내의 개별 개체 간에 일정 거리를 유지하도록 해 충돌을 방지하는 개념
- 정렬
 - 무리와 같은 방향, 같은 속도로 이동하는 개념
- 응집
 - 무리의 중심에 대해 일정 거리를 유지하는 개념

## 유니티 예제를 사용한 군집 처리
이 절에서 우리는 직접 오브젝트의 군집 처리를 하는 씬을 생성하고 C#을 사용해서 군집 행동을 구현할 것입니다.
이 예제에는 2개의 주요 컴포넌트가 있는데 하나는 `개별 개체의 행동`이고 나머지는 `군집을 유지하고 끌고 가는 메인 컨트롤러`입니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/9dfd0d8e-ff18-4198-8319-eeeb557368c7){: width="70%" height="70%"}

`UnityFlockController` 하위에 여러 `UnityFlock`이 존재합니다. UnityFlock 개체는 독립적으로 움직이며 부모인 UnityFlockController
개체를 리더로서 참조합니다. UnityFlockController 개체는 현재 목적지에 도달하면 이어서 임의로 다음 목적지를 갱신합니다.

## 개별 행동 흉내내기
`Boid`는 크레이그 레이놀즈가 만들어 낸 용어로 새처럼 움직이는 오브젝트를 의미합니다.
이 용어는 군집 속에 속한 개별 오브젝트를 가리키는 용도로 사용합니다.

```c#
using UnityEngine;
using System.Collections;
using Unity.VisualScripting;
using UnityEditor.SearchService;
using System.Collections.Generic;
using System.Timers;

public class UnityFlock : MonoBehaviour
{
    public float _minSpeed = 20.0f;         // 최소 이동 속도
    public float _turnSpeed = 20.0f;        // 회전 속도
    public float _randomFreq = 20.0f;       // 얼마나 자주 _randomForce 값에 기반해 _randomPush 값을 갱신하는지 결정
    public float _randomForce = 20.0f;

    // 정렬 관련 변수
    public float _toOriginForce = 50.0f;    // 군집의 흩어짐 정도
    public float _toOriginRange = 100.0f;   // 군집의 중심으로부터 일정 범위 내에 유지하도록

    public float _gravity = 2.0f;

    // 분산 관련 변수
    // 군집의 분산 처리 규칙에 적용됨
    public float _avoidanceRadius = 50.0f;  // 개별 개체 간의 최소 거리를 유지하는 용도
    public float _avoidanceForce = 20.0f;   // 개별 개체 간의 최소 거리를 유지하는 용도

    // 응집 관련 변수
    // 군집 처리의 응집 규칙에 적용됨
    public float _followVelocity = 4.0f;    // 군집의 리더 또는 군집의 중심 위치와의 최소 거리를 유지하는 용도
    public float _followRadius = 40.0f;     // 군집의 리더 또는 군집의 중심 위치와의 최소 거리를 유지하는 용도

    // 개별 재체의 이동과 관련된 변수
    private Transform _origin;              // 군집 오브젝트 전체 그룹을 제어하는 부모 오브젝트
    private Vector3 _velocity;
    private Vector3 _normalizedVelocity;
    private Vector3 _randomPush;
    private Vector3 _originPush;
    private Transform[] _objects;
    private UnityFlock[] _otherFlocks;
    private Transform _transformComponent;

    private void Start()
    {
        _randomFreq = 1.0f / _randomFreq;

        // Parent를 _origin에 할당
        // _origin이 컨트롤러 오브젝트의 역할을 한다.
        _origin = transform.parent;

        // 군집 트랜스폼
        _transformComponent = this.transform;

        // 임시 컴포넌트
        Component[] tempFlocks = null;

        // 그룹 내의 부모 트랜스폼으로부터 모든 유니티 군집 컴포넌트를 얻는다.
        if (transform.parent)
            tempFlocks = transform.parent.GetComponentsInChildren<UnityFlock>();

        // 그룹 내의 모든 군집 오브젝트를 할당하고 저장
        _objects = new Transform[tempFlocks.Length];
        _otherFlocks = new UnityFlock[tempFlocks.Length];

        for (int index = 0; index < tempFlocks.Length; index++)
        {
            _objects[index] = tempFlocks[index].transform;
            _otherFlocks[index] = (UnityFlock)tempFlocks[index];
        }

        // parent에 null을 지정하면 UnityFlockController 오브젝트가 리더가 된다.
        transform.parent = null;

        // 주어진 랜덤 주기에 따라 랜덤 푸시를 계산
        StartCoroutine(nameof(UpdateRandom));
    }

    // _randomFreq 변수의 시간 간격에 기반해 _randomPush 값을 갱신
    private IEnumerator UpdateRandom()
    {
        while (true)
        {
            // Random.insideUnitySphere
            // _randomForce를 반지름으로 하는 구 내에서 
            // 임의의 x, y, z 값으로 Vector3 오브젝트를 반환
            _randomPush = Random.insideUnitSphere * _randomForce;
            yield return new WaitForSeconds(_randomFreq + Random.Range(-_randomFreq / 2.0f, _randomFreq / 2.0f));
        }
    }

    private void Update()
    {
        // 내부 변수
        float speed = _velocity.magnitude;
        Vector3 avgVelocity = Vector3.zero;
        Vector3 avgPosition = Vector3.zero;
        int count = 0;
        float f = 0.0f;
        float d = 0.0f;
        Vector3 myPosition = _transformComponent.position;
        Vector3 forceV;
        Vector3 toAvg;
        Vector3 wantedVel;

        for (int index = 0; index < _objects.Length; index++)
        {
            Transform transform = _objects[index];

            if (transform != _transformComponent)
            {
                Vector3 otherPosition = transform.position;

                // 응집을 계산하기 위한 평균 위치
                avgPosition += otherPosition;
                count++;

                // 다른 군집에서 이 군집까지의 방향 벡터
                forceV = myPosition - otherPosition;

                // 방향 벡터의 크기(길이)
                d = forceV.magnitude;

                // 만일 벡터의 크기가 _foloowRadius보다 작다면 값을 늘린다.
                if (d < _followRadius)
                {
                    // 현재 벡터의 길이가 지정된 회피 반경보다 작으면
                    // 무리 간의 회피 거리에 기반해 오브잭트의 속도를 계산한다.
                    if (d < _avoidanceRadius)
                    {
                        f = 1.0f - (d / _avoidanceForce);

                        if (d > 0)
                            avgVelocity += (forceV / d) * f * _avoidanceForce;

                        // 리더와의 현재 거리를 유지한다.
                        f = d / _followRadius;
                        UnityFlock otherSealgull = _otherFlocks[index];

                        // otherSealgull 속도 벡터를 정규화해 이동 방향을 얻은 후,
                        // 새로운 속도를 설정한다.
                        avgVelocity += otherSealgull._normalizedVelocity * f * _followVelocity;
                    }

                    if (count > 0)
                    {
                        // 군집의 평균 속도를 계산(정렬)
                        avgVelocity /= count;

                        // 군집의 중간 값을 계산(응집)
                        toAvg = (avgPosition / count) - myPosition;
                    }
                    else
                        toAvg = Vector3.zero;

                    // 리더를 향한 방향 벡터
                    forceV = _origin.position - myPosition;
                    d = forceV.magnitude;
                    f = d / _toOriginRange;

                    // 리더에 대한 군집의 속도를 계산
                    if (d > 0)       // 만일 boid 무리의 중심에 있지 않다면
                        _originPush = (forceV / d) * f * _toOriginForce;

                    if (speed < _minSpeed && speed > 0)
                        _velocity = (_velocity / speed) * _minSpeed;

                    wantedVel = _velocity;

                    // 최종 속도 계산
                    wantedVel -= wantedVel * Time.deltaTime;
                    wantedVel += _randomPush * Time.deltaTime;
                    wantedVel += _originPush * Time.deltaTime;
                    wantedVel += avgVelocity * Time.deltaTime;
                    wantedVel += toAvg.normalized * _gravity * Time.deltaTime;

                    // 무리를 회전시키기 위한 최종 속도 계산
                    _velocity = Vector3.RotateTowards(_velocity, wantedVel, _turnSpeed * Time.deltaTime, 100.0f);

                    _transformComponent.rotation = Quaternion.LookRotation(_velocity);

                    // 계산한 속도에 기반해 군집 이동
                    _transformComponent.Translate(_velocity * Time.deltaTime, Space.World);

                    // 속도 정상화
                    _normalizedVelocity = _velocity.normalized;
                }
            }
        }
    }
}
```

## 컨트롤러 생성
이 클래스는 자신의 위치를 갱신해 자신을 따르는 개별 boid 객체들이 어디로 가야할지 알 수 있게 합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class UnityFlockController : MonoBehaviour
{
    public Vector3 _offset;
    public Vector3 _bound;
    public float _speed = 100.0f;

    private Vector3 _initialPosition;
    private Vector3 _nextMovementPoint;

    // 초기화에 사용
    private void Start()
    {
        _initialPosition = transform.position;
        CalculateNextMovementPoint();
    }

    // update 는 매 프레임 호출됨
    private void Update()
    {
        transform.Translate(Vector3.forward * _speed * Time.deltaTime);
        transform.rotation = Quaternion.Slerp(transform.rotation,
                                              Quaternion.LookRotation(_nextMovementPoint - transform.position),
                                              1.0f * Time.deltaTime);

        if (Vector3.Distance(_nextMovementPoint, transform.position) <= 10.0f)
            CalculateNextMovementPoint();
    }

    private void CalculateNextMovementPoint()
    {
        float posX = Random.Range(_initialPosition.x - _bound.x, _initialPosition.x + _bound.x);
        float posY = Random.Range(_initialPosition.y - _bound.y, _initialPosition.y + _bound.y);
        float posZ = Random.Range(_initialPosition.z - _bound.z, _initialPosition.z + _bound.z);

        _nextMovementPoint = _initialPosition + new Vector3(posX, posY, posZ);
    }    
}
```

## 대체 구현 사용
좀 더 간단한 군집 알고리즘을 살펴봅시다. 구현을 위해서는 `개별 boid 동작`과 `컨트롤러 동작`을 처리하는
두 개의 컴포넌트가 필요합니다. 나머지 모든 boid는 컨트롤러를 따라 이동합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Flock : MonoBehaviour
{
    internal FlockController _controller;
    public Rigidbody _rigidBody;

    private void Update()
    {
        if (_controller)
        {
            Vector3 relativePos = Steer() * Time.deltaTime;

            if (relativePos != Vector3.zero)
                _rigidBody.velocity = relativePos;

            // boid의 최소와 최대 속도를 강제한다.
            float speed = _rigidBody.velocity.magnitude;

            if (speed > _controller._maxVelocity)
                _rigidBody.velocity = _rigidBody.velocity.normalized * _controller._maxVelocity;
            else if (speed < _controller._minVelocity)
                _rigidBody.velocity = _rigidBody.velocity.normalized * _controller._minVelocity;
        }
    }

    private Vector3 Steer()
    {
        Vector3 center = _controller._flockCenter - transform.localPosition;             // 응집
        Vector3 velocity = _controller._flockVelocity = _rigidBody.velocity;              // 정렬
        Vector3 follow = _controller._target.localPosition - transform.localPosition;    // 리더 추종
        Vector3 separation = Vector3.zero;

        foreach (Flock flock in _controller._flockList)
        {
            if (flock != this)
            {
                Vector3 relativePos = transform.localPosition - flock.transform.localPosition;

                separation += relativePos / (relativePos.sqrMagnitude);
            }
        }

        // 무작위화
        Vector3 randomize = new Vector3((Random.value * 2) - 1, (Random.value * 2) - 1, (Random.value * 2) - 1);
        randomize.Normalize();

        return (_controller._centerWeight * center +
                _controller._velocityWeight * velocity +
                _controller._separationWeight * separation +
                _controller._followWeight * follow +
                _controller._randomizeWeight * randomize);
    }
}
```

## FlockController 구현
`FlockController`는 런타임에 boid를 생성하고 군집의 평균 속도와 중심의 위치를 갱신합니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FlockController : MonoBehaviour
{
    public float _minVelocity = 1;          // 최저 속도
    public float _maxVelocity = 8;          // 최고 군집 속력
    public int _flockSize = 20;             // 그룹 내에 있는 군집의 수

    // boid가 중앙에서 어느 정도까지 떨어질 수 있는지 지정
    // weight가 클수록 중앙에 근접
    public float _centerWeight = 1;

    public float _velocityWeight = 1;       // 정렬 동작

    // 군집 내에서 개별 boid 간의 거리
    public float _separationWeight = 1;

    // 개별 boid와 리더 간의 거리 (weight가 클수록 가깝게 따라감)
    public float _followWeight = 1;

    // 추가적인 임의성 제공
    public float _randomizeWeight = 1;

    public Flock _prefab;
    public Transform _target;               // 리더 저장

    // 그룹 내 군집의 중앙 위치
    internal Vector3 _flockCenter;
    internal Vector3 _flockVelocity;        // 평균 속도

    public ArrayList _flockList = new();

    private void Start()
    {
        for (int index = 0; index < _flockSize; index++)
        {
            Flock flock = Instantiate(_prefab, transform.position, transform.rotation) as Flock;

            flock.transform.parent = transform;
            flock._controller = this;
            _flockList.Add(flock);
        }
    }

    private void Update()
    {
        // 전체 군집 그룹의 중앙 위치와 속도를 계산
        Vector3 center = Vector3.zero;
        Vector3 velocity = Vector3.zero;

        foreach (Flock flock in _flockList)
        {
            center += flock.transform.localPosition;
            velocity = flock._rigidBody.velocity;
        }

        _flockCenter = center / _flockSize;
        _flockVelocity = velocity / _flockSize;
    }
}

```