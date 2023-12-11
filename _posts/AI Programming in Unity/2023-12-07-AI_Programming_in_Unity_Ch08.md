---
title:  "[유니티 게임 AI 프로그래밍] 008. 통합"
excerpt: "책의 내용들을 기반으로 간단한 게임을 만들어 봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-12-07
last_modified_at: 2023-12-11
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
지난 포스팅까지 인공 지능을 구현하는 데 필요한 기본적인 도구들을 정리했습니다.
이 서적의 포스팅을 마무리하는 의미에서 간단한 게임을 하나 만들어봅시다.
</div>

## 규칙 설정
포스팅의 게임은 간단한 디팬스 장르입니다. 다만 적을 막기 위해 포탑을 설치하는 방식이 아닌
에이전트가 포탑의 공격을 피해 목표 지점까지 이동하는 게임입니다.

에이적트에 어떤 로직과 행동을 적용할지 결정할 때는 단순한 아이디어가 아닌 구체적인
규칙을 세우는 것이 중요합니다. 다양한 기능을 구현함에 따라 구칙은 변경되기도 하지만
정리된 개념을 세우고 있어야만 각 작업에 가장 적합한 방식을 적용할 수 있습니다.

우리의 에이전트가 무사히 목표에 도달할 수 있도록 2가즹 능력을 제공해줍시다.

- Boost
    - 잠깐 동안 이동속도 2배
- Shield
    - 에이전트 주변에 보호막 생성

## 타워 생성
![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/2438a140-290d-4aab-85ea-66012bb3f679){: width="70%" height="70%"}

타워의 총은 자유롭게 축을 따라 회전하면서 플레이어를 겨냥하고 발포할 수 있지만 이동은 불가능합니다.
만약 에이전트가 멀어지면 타워는 발사를 멈추고 다시 원위치로 돌아옵니다.

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/a7fd8924-c116-41d7-867c-f91242938523)

타워 프리팹의 계층 구조는 다음과 같습니다.

- Tower
    - 특별한 기능을 가지고 있지 않은 타워의 기반
- Gun
    - 에이전트를 따라 이동하며 추적하는 역할
- Barrel / Muzzle
    - 총알이 재장전되는 위치

![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/02f0e1eb-276e-4a3e-8360-eadc08273342){: width="70%" height="70%"}

다음은 로직에 영향을 주는 컴포넌트들을 살펴 봅시다.

- Sphere Collider
    - 에이전트가 해당 구의 영역 이내로 진입하면 타워는 이를 감지하고 발사를 시작
- RigidBody
    - 강체 컴포넌트를 가지지 않은 오브젝트에 대해서는 움직이지 않을 때 유니티가 충돌이나 트리거 이벤트롤 보내지 않기 때문에 필요
- Tower
    - 타워의 로직 스크립트
- Animator
    - 타워의 상태 기계. 실제 애니메이션을 처리하지는 않는다.

여기서 Animator 를 살펴보면 `Idle` 상태와 `LockedOn` 상태는 `TankInRange` 파라미터에 따라 
서로 전이되는 형태를 띕니다.

LoackedOn 상태는 연결된 `StateMachineBehaviour`클래스를 가지는데 내용을 살펴봅시다.

```c#
using UnityEngine;
using System.Collections;

public class LockedOnState : StateMachineBehaviour
{
    private GameObject _player;
    private Tower _tower;

    public override void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // base.OnStateEnter(animator, stateInfo, layerIndex);

        _player = GameObject.FindWithTag("Player");
        _tower = animator.gameObject.GetComponent<Tower>();
        _tower.LockedOnState = true;
    }

    public override void OnStateUpdate(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // base.OnStateUpdate(animator, stateInfo, layerIndex);

        animator.gameObject.transform.LookAt(_player.transform);
    }

    public override void OnStateExit(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // base.OnStateExit(animator, stateInfo, layerIndex);

        animator.gameObject.transform.rotation = Quaternion.identity;
        _tower.LockedOnState = false;
    }
}
```

상태에 진입하고 `OnStateEnter`가 호출됐을 때 플레이어에 대한 참조를 찾습니다. 이후 `LockedOn` 변수를 true로 설정해줍니다.

상태에 머무르는 동안 `OnStateUpdate`는 매 프레임 호출됩니다. 이 메소드 내에서 `Animator` 참조를 통해 `Gun GameObject`에
대한 참조를 얻습니다. 이 참조를 통해 `Transform.LookAt`을 사용해서 총이 플레이어를 추적하도록 합니다.

마지막으로 상태를 빠져나올 때 `OnStateExit`가 호출됩니다. 이 메소드는 일부 정리 작업도 수행하는데,
총의 회전 상태를 초기화하고 `Tower`의 LockedOn 변수를 false로 설정합니다.

다음은 Tower 스크립트입니다.

```c#
using UnityEngine;
using System.Collections;

public class Tower : MonoBehaviour
{
    [SerializeField]
    private Animator _animator;

    [SerializeField]
    private float _fireSpeed = 3f;
    private float _fireCounter = 0f;
    private bool _canFire = true;

    [SerializeField]
    private Transform _muzzle;

    [SerializeField]
    private GameObject _projectile;

    // 일반적으로 변수는 외부 접근을 제한하는 형태가 좋습니다.

    // 따라서 _lockedOnState을 그대로 노출하지 말고 외부에서 접근할 수 있는
    // 프로퍼티를 제공하는 편이 낫습니다.
    private bool _lockedOnState = false;
    public bool LockedOnState
    {
        get => _lockedOnState;
        set => _lockedOnState = value;
    }

    private void Update()
    {
        if (_lockedOnState && _canFire)
            StartCoroutine(Fire());
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.tag == "Player")
            _animator.SetBool("TankInRange", true);
    }

    private void OnTriggerExit(Collider other)
    {
        if (other.tag == "Player")
            _animator.SetBool("TankInRange", false);
    }

    private void FireProjectile()
    {
        GameObject bullet = Instantiate(_projectile, _muzzle.position, _muzzle.rotation);

        bullet.GetComponent<Rigidbody>().AddForce(_muzzle.forward * 300);
    }

    private IEnumerator Fire()
    {
        _canFire = false;
        FireProjectile();

        while (_fireCounter < _fireSpeed)
        {
            _fireCounter += Time.deltaTime;
            yield return null;
        }

        _canFire = true;
        _fireCounter = 0f;
    }
}
```

목표물을 발견하고 목표물에 총을 발사할 수 있다면 `Fire()` 코루틴을 호출합니다.
여기서 왜 코루틴을 사용해서 발사하는지 알아봅시다.

우리는 타워가 탱크를 향해 미친듯이 발사하길 원하진 않기 때문에 각 발사 간에 간격을 두기로 합니다.
`FireProjectile()`을 호출한 후 `_canFire`를 false로 설정하고 0부터 `_fireSpeed`까지 카운트를 센 후
다시 `_canFire`를 다시 true로 설정합니다. FireProjectile() 메소드는 포탄의 인스턴스화와 `RigidBody.AddForce`를
사용한 발사를 처리합니다.

## 타워 발사 처리
![image](https://github.com/GunnHB/gunnHB.github.io/assets/117302300/fc79498a-72fd-4d97-bc9d-26a28e6f67f3){: width="70%" height="70%"}

`bullet` 프리팹을 살펴보면 구 형태의 콜라이더가 연결되어 있습니다. 그리고 충돌 로직의 처리를 연결된 `Projectile` 스크립트를 살펴봅시다.

```c#
using UnityEngine;
using System.Collections;

public class Projectile : MonoBehaviour
{
    [SerializeField]
    private GameObject _explosionPrefab;

    private void OnTriggerEnter(Collider other)
    {
        if (other.tag == "Player" || other.tag == "Environment")
        {
            if (_explosionPrefab == null)
                return;
        }

        GameObject explosion = Instantiate(_explosionPrefab, transform.position, Quaternion.identity);
        Destroy(this.gameObject);
    }
}
```

## 탱크 설정
탱크 자체는 목표 지점을 도달한다는 목적을 가진 간단한 에이전트입니다. 앞에서 언급한 것처럼 플레이어는 탱크가 길을
따라 이동하면서 능력을 활용해 타워의 포격으로부터 벗어나 안전하게 이동하도록 해야합니다.

`Tank.cs`를 살펴봅시다.

```c#
using UnityEngine;
using System.Collections;
using UnityEngine.AI;

public class Tank : MonoBehaviour
{
    [SerializeField]
    private Transform _goal;
    private NavMeshAgent _agent;

    [SerializeField]
    private float _speedBoostDuration = 3f;

    [SerializeField]
    private ParticleSystem _boostParticleSystem;

    [SerializeField]
    private float _shieldDuration = 3f;

    [SerializeField]
    private GameObject _shield;

    private float _regularSpeed = 5f;
    private float _boostedSpeed = 7f;
    private bool _canBoost = true;
    private bool _canShield = true;

    private bool _hasShield;

    private void Start()
    {
        _agent = GetComponent<NavMeshAgent>();

        _agent.SetDestination(_goal.position);
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.B))
        {
            if (_canBoost)
                StartCoroutine(Boost());
        }

        if (Input.GetKeyDown(KeyCode.S))
        {
            if (_canShield)
                StartCoroutine(Shield());
        }
    }

    private IEnumerator Shield()
    {
        _canShield = false;
        _shield.SetActive(false);
        float shieldCounter = 0f;

        while (shieldCounter < _shieldDuration)
        {
            shieldCounter += Time.deltaTime;
            yield return null;
        }

        _canShield = true;
        _shield.SetActive(false);
    }

    private IEnumerator Boost()
    {
        _canBoost = false;
        _agent.speed = _boostedSpeed;
        _boostParticleSystem.Play();
        float boostCounter = 0f;

        while (boostCounter < _speedBoostDuration)
        {
            boostCounter += Time.deltaTime;
            yield return null;
        }

        _canBoost = true;
        _boostParticleSystem.Pause();
        _agent.speed = _regularSpeed;
    }
}
```

두 능력의 로직은 비슷합니다. `shield`는 인스펙터에 정의한 변수인 shield 게임 오브젝트를 활성화하거나
비활성화합니다. 그리고 `shieldDuration`만큼의 시간이 흐른 후 이를 비활성화시키고 다시 플레이어가
shield를 사용할 수 있게 합니다.

`Boost` 코드의 가장 큰 차이는 게임 오브젝트를 활성화하거나 비활성화하는 것이 아니며,
Boost는 인스펙터를 통해 지정한 파티클 시스템의 Play를 호출하고 NavMeshAgent의 속도를 2배로 일정 시간 동안 유지한다는 점입니다.

마지막으로 카메라의 코드를 살펴봅시다. 해당 프로젝트에서는 카메라가 플레이어를 따라 z축을 기준으로
수평으로만 이동하길 원합니다. 다음과 같습니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HorizontalCam : MonoBehaviour
{
    [SerializeField]
    private Transform _target;

    private Vector3 _targetPosition;

    private void Update()
    {
        _targetPosition = transform.position;
        _targetPosition.z = _target.transform.position.z;

        transform.position = Vector3.Lerp(transform.position, _targetPosition, Time.deltaTime);
    }
}
```

보시다시피 단순하게 카메라의 목표 지점을 모든 축에 대해 현재 위치와 모두 동일하게 설정했습니다. 다만 이후
목표 지점의 z축을 우리의 타겟과 동일하게 재지정했습니다.

## 요약
드디어 마지막까지 왔습니다! 지금까지의 내용을 기반으로 간단한 디펜스 게임을 만들어봤습니다!