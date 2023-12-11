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

