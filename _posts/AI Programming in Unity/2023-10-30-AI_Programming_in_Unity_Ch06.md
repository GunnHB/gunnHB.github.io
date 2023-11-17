---
title:  "[유니티 게임 AI 프로그래밍] 006. 행동 트리"
excerpt: "행동 트리에 대해 알아봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-11-17
last_modified_at: 2023-11-17
---

🔔 유니티 게임 AI 프로그래밍 2/e 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
행동 트리는 게임 개발자들 사이에서 꾸준한 인기를 얻고 있습니다.
[헤일로]나 [기어스 오브 워] 같은 게임은 행동 트리를 본격적으로 이용한 유명 게임입니다.
PC의 성능이 좋아지고 게임 콘솔과 모바일 기기가 발전함에 따라 게임에서 인공지능을
구현할 수 있는 범위가 점차 넓어지고 있습니다.

- 행동 트리의 기초
- 기존 행동 트리 솔루션 사용의 장점
- 직접 행동 트리 프레임워크를 구현하는 방법
- 만든 프레임워크를 사용해 간단한 트리 구현
</div>

## 행동 트리의 기초
루트`root`로 알려진 최상위 부모 노드 아래에 계층 구조 형태로 가지가 존재하는
구조적 특성 때문에 트리라고 부릅니다.

당연히 행동 트리가 가질 수 있는 노드와 자식 노드의 수에는 제약이 없습니다.
계층 구조의 가장 말단에 있는 노드는 리프`leaf` 노드라고 부르며,
노드는 행동 또는 테스트 등을 표현할 수 있습니다. 상태 전이 규칙을 통해서 상태가
바뀌는 상태 기계와 달리 행동 트리의 흐름은 철저히 `각 노드의 순서`에 따라 정의됩니다.
행동트리는 항상 `최상위 루트 노드`에서부터 평가를 시작해 `원하는 조건을 만족할 때까지`
또는 `가장 마지막 노드에 도달할 때까지` 순회합니다.

## 다양한 노드 타입의 이해
노드 타입을 부르는 이름은 다양하며 때론 작업이라고 부르기도 합니다.
다음 내용은 노드의 타입과 무관하게 통용되는 내용이며 노드는 항상 다음 상태 중 하나를 반환합니다.

- Success
    - 노드가 조건을 만족할 때
- Failure
    - 노드가 조건을 만족하지 않을 때
- Running
    - 노드에서 검사하는 조건의 유효성이 정의되지 않았을 때

행동 트리가 가질 수 있는 잠재적 복잡도로 인해 대부분의 구현은 `비동기`로 이뤄집니다.
이는 유니티에서 트리의 평가가 `게임의 흐름을 방해하지 않는다`는 것을 의미하기도 합니다.

단순하게 노드의 조건 만족을 판단하게 된다고 가정했을 때 하나의 노드의 조건 확인을 위해
수 프레임 간 진행될 수 있습니다. 단일 객체라면 크게 문제되지 않겠지만 다수가 그렇다면
이는 성능 저하를 초래할 수도 있습니다. 이런 이유로 `Running` 상태가 중요한 것입니다.

### 합성 노드 정의
하나 이상의 자식 노드를 가지고 있을 때 `합성 노드`라 부릅니다. 부모 노드의 상태는
전적으로 자식 노드의 평가 결과로 결정됩니다. 만약 자식 노드의 평가가 진행 중일 땐
부모의 상태는 Running이 됩니다. 자식의 평가 방식에 따라 다음의 합성 노드가 존재합니다.

- 시퀀스`Sequences`
시퀀스는 자식 노드의 조건이 모두 만족할 때만 성공으로 인정됩니다. 일반적으로 왼쪽부터 시작해
오른쪽으로 검사를 진행하며, 진행 중 하나라도 조건을 만족하지 않으면 검사는 종료되어 실패를 반환합니다.

- 셀렉터`Selectors`
셀렉터는 시퀀스와는 다르게 자식 노드 중 하나라도 조건을 만족하면 성공을 반환합니다.
시퀀스와 마찬가지의 방향으로 검사를 진행하며 만족하는 조건이 나오면 검사가 종료되고 성공을 반환합니다.
그렇기 때문에 셀렉터가 실패를 반환할 때는 모든 자식들의 조건이 만족하지 않을 경우 뿐입니다.

### 데코레이터 노드의 이해
`데코레이터 노드`는 합성 노드와 달리 단 하나의 자식 노드만 가집니다. 사실 부모 자체에
조건을 포함시키는 것과 동일한 결과를 얻을 수 있어 해당 노드의 필요성에 대해 의문을 가질 수 있습니다.
하지만 데코레이터 노드는 자식 노드가 반환한 상태를 취하면서 스스로의 매개변수에
기반한 반응을 얻을 수 있기에 특별합니다. 심지어 자식이 평가되는 방식과 횟수를 결정할 수도 있습니다.

- 인버터`Inverter`
인버터는 자식이 반환한 상태의 반대를 취합니다. 자식이 true를 반환했다면 데코레이터 노드는 false를 반환하는 것입니다.

- 리피터`Repeater`
데코레이터에서 정의된 대로 true 또는 false가 될 때까지 평가를 반복하여 진행합니다.
예를 들어 '에너지가 충분히 참'의 조건을 만족할 때까지 평가를 반복하는 것입니다.

- 리미터`Limiter`
에이전트가 무한 루프에 빠지지 않도록 평가 횟수에 제한을 둡니다. 이는 리피터와 반대로 조건이 아닌
횟수를 제한할 때 유용합니다. 예를 들면 포기하기 전까지 문을 걷어차는 횟수를 정할 수 있습니다.

일부 데코레이터 노드는 디버깅과 테스트 용도로 사용할 수 있습니다.

- 가짜 상태`Fake state`
데코레이터에 정의된 대로 `항상 참이나 거짓을 반환`해 에이전트의 특정 행동을 검증할 때 유용합니다.
또는 주변의 다른 에이전트를 관찰하기 위해 가짜로 Running 상태만 유지시킬 수도 있습니다.

- 브레이크포인트`Breakpoint`
코드에서의 것과 마찬가지로 이 노드에 도달하면 로그나 다른 메서드를 통해 알림을 받을 수 있습니다.

이런 타입들은 상호 베타적인 단일 원형체가 아니며 각자의 목적에 맞게 조합해서 사용할 수 있습니다.
조심해야할 것은 너무 많은 기능을 하나의 데코레이터 노드에 결합하지 않도록 해야합니다.

### 리프 노드 표현
앞서 리프 노드는 `계층 구조의 가장 말단에 있는 노드`라고 했습니다. 사실 리프 노드 역시 어떤 종류의
행동이 될 수 있습니다. 에이전트가 가질 수 있는 모든 로직을 표현할 수 있다는 점에서 매우 유용합니다.
이는 걷기, 발사, 공격 등의 행동을 정의할 수 있으며, 어떤 제약도 가지지 않습니다.
단지 계층의 말단 노드라는 특성만 있을 뿐 3가지의 행동 중 하나를 반환합니다.

## 기반 Node 클래스 구현
모든 노드에서 필요로 하는 `기반 기능`이 존재합니다. 우리가 만들 간단한 프레임워크는
모둔 기반 추상 Node.cs 클래스를 상속받을 예정입니다. 이 클래스는 다른 모든 노드 타입의 기반입니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace BehaviorTree
{
    public enum NodeStates
    {
        None,
        Success,
        Failure,
        Running,
    }

    [System.Serializable]
    public abstract class Node : MonoBehaviour
    {
        // 노드의 상태를 반환하는 델리게이트
        public delegate NodeStates NodeReturn();

        // 현재 노드의 상태
        protected NodeStates _nodeState;
        public NodeStates NodeState => _nodeState;

        // 노드 생성자
        public Node() { }

        // 원하는 조건 세트를 평가하기 위해 이 메소드를 구현
        public abstract NodeStates Evaluate();
    }
}
```

## 노드를 셀렉터로 확장
```c#
using System.Collections;
using System.Collections.Generic;
using BehaviorTree;

using UnityEngine;

public class Selector : Node
{
    // 셀렉터의 자식 노드들
    protected List<Node> _nodes = new();

    // 셀렉터는 자식 노드의 목록을 필요로 합니다.
    public Selector(List<Node> nodes)
    {
        _nodes = nodes;
    }

    // 자식 중 하나가 성공을 보고하면 셀렉터는 즉시 상위로 성공을 보고합니다.
    // 만일 모든 자식이 실패를 보고하면 상위에 실패를 보고합니다.
    public override NodeStates Evaluate()
    {
        foreach (Node node in _nodes)
        {
            switch (node.Evaluate())
            {
                case NodeStates.Success:
                    _nodeState = NodeStates.Success;
                    return _nodeState;
                case NodeStates.Failure:
                    continue;
                case NodeStates.Running:
                    _nodeState = NodeStates.Running;
                    return _nodeState;
                default:
                    continue;
            }
        }

        _nodeState = NodeStates.Failure;
        return _nodeState;
    }
}
```
`Evaluate` 메서드의 구현부를 확인해봅시다. 모든 자식 노드를 돌며 개별 결과를 확인합니다.
만약 자식 중 하나가 실패를 반환한다고 해도 셀렉터는 검사를 계속 진행합니다. 진행 중
성공을 반환하는 노드가 있다면 그 즉시 검사를 종료하여 상위에 성공을 보고합니다.
만일 모든 자식 노드가 실패를 반환했다면 셀렉터 역시 상위에 실패를 반환합니다.

## 시퀀스
시퀀스는 셀렉터와 구현이 매우 유사합니다. 다만 앞서 배웠던 것처럼 결과의 반환 방식이
다르다는 것을 염두에 두고 코드를 확인합시다.

```c#
using System.Collections;
using System.Collections.Generic;
using BehaviorTree;
using UnityEngine;

public class Sequence : Node
{
    // 시퀀스에 속하는 자식 노드들
    private List<Node> _nodes = new();

    // 초기에 자식 목록을 제공해야합니다.
    public Sequence(List<Node> nodes)
    {
        _nodes = nodes;
    }

    // 하나의 자식 노드라도 실패를 반환하면 전체 노드는 실패합니다.
    // 모든 자식 노드가 성공을 반환하면 노드는 성공을 보고합니다.
    public override NodeStates Evaluate()
    {
        bool anyChildRunning = false;

        foreach (var node in _nodes)
        {
            switch (node.NodeState)
            {
                case NodeStates.Success:
                    continue;
                case NodeStates.Failure:
                    _nodeState = NodeStates.Failure;
                    return _nodeState;
                case NodeStates.Running:
                    anyChildRunning = true;
                    continue;
                default:
                    _nodeState = NodeStates.Success;
                    return _nodeState;
            }
        }

        _nodeState = anyChildRunning ? NodeStates.Running : NodeStates.Success;
        return _nodeState;
    }
}
```
시퀀스는 전체 자식이 성공을 반환해야 성공을 보고합니다. 검사 중 실패한 자식을 발견하면
즉시 검사를 종료하고 실패를 반환합니다. 만일 Running 상태인 자식이 있다면 부모 노드 역시
Running 상태를 반환하여 트리를 다시 평가합니다.

## 인버터 데코레이터 구현
Inverter.cs의 구조는 약간 다르지만 나머지 노드처럼 역시 Node를 상속받습니다.

```c#
using System.Collections;
using System.Collections.Generic;
using BehaviorTree;
using UnityEngine;

public class Inverter : Node
{
    // 평가할 자식 노드
    // 데코레이터 노드는 자식이 하나입니다.
    private Node _node;
    public Node ThisNode => _node;

    // 생성자는 이 인버터 데코레이터가 감쌀 자식 노드를 필요로 합니다.
    public Inverter(Node node)
    {
        _node = node;
    }

    // 자식이 실패하면 성공을 보고하고 자식이 성공하면 실패를 보고합니다.
    // Running은 그대로 보고합니다.
    // 금쪽이같습니다.
    public override NodeStates Evaluate()
    {
        switch (_node.NodeState)
        {
            case NodeStates.Success:
                _nodeState = NodeStates.Failure;
                return _nodeState;
            case NodeStates.Failure:
                _nodeState = NodeStates.Success;
                return _nodeState;
            case NodeStates.Running:
                _nodeState = NodeStates.Running;
                return _nodeState;
        }

        _nodeState = NodeStates.Success;
        return _nodeState;
    }
}
```

## 일반 액션 노드 생성
이제 구현할 ActionNode.cs는 로직을 델리게이트로 전달하기 위한 일반 리프 노드입니다.
현재의 예제는 델리게이트에 맞는 어떤 메서드라도 전달할 수 있다는 점에서 유연하지만
파라미터를 가지지 않는 델리게이트라는 점에 제약적입니다.

```c#
using System.Collections;
using System.Collections.Generic;
using BehaviorTree;
using UnityEngine;

public class ActionNode : Node
{
    // 액션에 대한 메서드 시그니처
    public delegate NodeStates ActionNodeDelegate();

    // 이 노드를 평가할 때 호출하는 델리게이트
    private ActionNodeDelegate _action;

    // 이 노드는 아무런 로직을 포함하지 않으므로
    // 델리게이트 형태로 로직이 전달돼야 합니다.
    // 시그니처에 나와 있듯이 액션은 NodeState 열거형을 반환해야 합니다.
    public ActionNode(ActionNodeDelegate action)
    {
        _action = action;
    }

    public override NodeStates Evaluate()
    {
        switch (_action())
        {
            case NodeStates.Success:
                _nodeState = NodeStates.Success;
                return _nodeState;
            case NodeStates.Failure:
                _nodeState = NodeStates.Failure;
                return _nodeState;
            case NodeStates.Running:
                _nodeState = NodeStates.Running;
                return _nodeState;
            default:
                _nodeState = NodeStates.Failure;
                return _nodeState;
        }
    }
}
```
이 노드의 동작을 책임지는 것은 `_action` 델리게이트입니다. 생성자는 `NodeStates`를 반환하는
스그니처에 맞는 메서드 전달을 요구합니다. 조건만 만족하면 어떤 로직이던 구현할 수 있습니다.

## 요약
6장에서는 행동 트리의 전체적인 동작 방식에 대해 살펴봤고 행동 트리를 구성하는 각 노드의
개별 타입에 대해서도 다뤘으며 상황에 따라 어떤 타입의 노드를 사용하는 것이 더 유용한지도 살펴봤습니다.

7장에서는 6장에서 배운 개념에 복잡도를 높이고 다양한 기능을 추가하는 방법을 다룰 예정입니다.