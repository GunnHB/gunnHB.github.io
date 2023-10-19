---
title:  "[유니티 게임 AI 프로그래밍] 004. 길 찾기"
excerpt: "길 찾기 알고리즘에 대해 알아봅시다!"

author: gunnHB
categories: 
 - AI Programming
tags: 
 - [Github, Git, Unity, Programming, AI Programming, C#]

toc: true
toc_sticky: true
 
date: 2023-10-18
last_modified_at: 2023-10-19
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

## A* 길 찾기
다음으로 유니티 환경에서 C# 을 사용해 A* 알고리즘을 구현해볼 예정입니다.
길 찾기에는 다양한 방법이 존재하지만 A* 알고리즘이 갖는 단순함과 높은
효율성의 장점 덕분에 게임이나 상호작용이 많은 애플리케이션에서 A* 길 찾기
알고리즘을 널리 사용하고 있습니다.

## A* 알고리즘 재확인
A* 알고리즘을 적용하기 위해서는 일단 맵을 운행 가능한 데이터 구조로 표현해야 합니다.
다양한 구조를 사용할 수 있지만, 이번 예제에서는 2차원 격자 배열을 사용합니다.

두 개의 변수를 사용해 하나는 이미 처리한 노드를 저장하도록 하고 다른 하나는
앞으로 처리해야 하는 노드를 저장하도록 한 후 이를 각각 닫힌 리스트와 열린 리스트로 
부르도록 합시다.

- `시작 노드`에서 시작하고 이를 `열린 목록`에 넣는다.
- 열린 목록이 노드를 가지고 있는 한, 다음 과정을 수행한다.
    1. 열린 리스트의 첫 노드를 가져와 이를 `현재 노드`로 유지한다.
        - 이는 열린 리스트가 정렬된 상태며 첫 노드는 가장 비용이 낫다고 가정
    2. 현재 노드의 이웃 중 `벽`이나 `대포`같이 통과할 수 없는 장애물 타입이 아닌 통과 가능한 노드를 가져온다.
    3. 각 이웃 노드에 대해 이미 닫힌 리스트에 포함된 상태인지 확인한 후 닫힌 리스트에 포함된 상태가 아니면 다음 수식을 사용해서 이웃 노드의 총비용을 계산한다.
        - `F = G + H`
    4. 이 수식에서 `G`는 `이전 노드에서 현재 노드까지`의 총비용이며, `H`는 `현재 노드에서 목적 지점 노드까지`의 총비용이다.
    5. 이웃 노드 오브젝트에 비용 데이터를 저장하고 현재 노드를 부모 노드로 저장한다. 나중에 이 부모 노드 데이터를 사용해서 실제 경로를 역추적하게 된다.
    6. 이 이웃 노드를 열린 리스트에 넣고 열린 리스트를 목적 노드에 도달하는데 필요한 총비용 순으로 오름차순 정렬한다.
    7. 처리할 이웃 노드가 더 없다면 현재 노드를 닫힌 리스트에 넣고 열린 리스트에서 제거한다.
    8. `2 단계`로 돌아간다.

이 과정을 마치면 현재 노드는 대상 목표 지정 노드의 위치에 놓이게 되는데 이는 시작 지점에서 목표 지점으로 향하는 길이 장애물로
원천봉쇄되지 않았다는 가정하에만 유효합니다.

## 구현
### 노드 클래스 구현
`Node` 클래스는 각 타일 오브젝트를 2차원 격자로 다루면서 맵을 표현합니다.
```c#
using UnityEngine;
using System.Collections;
using System;

public class Node : IComparable
{
    public float _nodeTotalCost;
    public float _estimatedCost;
    public bool _bObstacle;
    public Node _parent;
    public Vector3 _position;

    public Node()
    {
        this._nodeTotalCost = 0.0f;
        this._estimatedCost = 1.0f;
        this._bObstacle = false;
        this._parent = null;
    }

    public Node(Vector3 pos)
    {
        this._nodeTotalCost = 1.0f;
        this._estimatedCost = 0.0f;
        this._bObstacle = false;
        this._parent = null;
        this._position = pos;
    }

    public void MarkAsObstacle()
    {
        this._bObstacle = true;
    } 
}
```

Node 클래스는 비용과 장애물 여부 플래그, 위치, 부모 노드 등의 정보를 다루기 위한 속성을 포함합니다.
`_nodeTotalCost`는 G로 시작 위치에서 현재 노드까지의 이동 비용이며, `_estimatedCost`는 H로
현재 노드에서 대상 목표 노드까지의 총 추정 비용입니다. 그리고 두 개의 간단한 생성자 메소드와 
해당 노드의 장애물 여부 설정을 위한 `wrapper` 메소드를 하나 가집니다. 그 후 다음 코드처럼 `CompareTo`
메소드를 구현합니다.

```c#
public int CompareTo(object obj)
{
    Node node = (Node)obj;
    
    // 음수 값은 오브젝트가 정렬된 상태에서 현재보다 앞에 있음을 의미
    if(this._estimatedCost < node._estimatedCost)
        return -1;
    
    // 양수 값을 오브젝트가 정렬된 상태에서 현재보다 뒤에 있음을 의미
    if(this._estimatedCost > node._estimatedCost)
        return 1;

    return 0;
}
```

`CompareTo` 메소드를 `오버라이드` 하기 위해 Node zmffotmsms `IComparable`을 상속받았습니다.
이것은 총 예상 비용을 기준으로 노드 배열의 목록을 정렬해야 하기 때문입니다.
`ArrayList` 타입은 `Sort` 메소드를 포함하는데, 이는 기본적으로 리스트 내의 모든 오브젝트 내에 
구현된 `CompareTo` 메소드를 사용합니다.

### 우선순위 큐 구성
`PriorityQueue`는 짧고 간단한 클래스로 노드와 ArrayList를 쉽게 다루고자 할 때 사용합니다.

```c#
using UnityEngine;
using System.Collections;

public class PriorityQueue
{
    private ArrayList _nodes = new ArrayList();

    public int Length => this._nodes.Count;

    public bool Contains(object node)
    {
        return this._nodes.Contains(node);
    }

    public Node First()
    {
        if(this._nodes.count > 0)
            return (Node)this._nodes[0];

        return null;
    }

    public void Push(Node node)
    {
        this._nodes.Add(node);
        this._nodes.Sort();
    }

    public void Remove(Node node)
    {
        this._nodes.Remove(node);
        
        // 리스트를 확실하게 정렬
        this._nodes.Sort();
    }
}
```
코드는 간단합니다. 한가지 주의해야할 점은 ArrayList에 노드를 추가하거나 제거할 때 
`Sort` 메소드를 실행해주어야 합니다. 이는 내부적으로 Node 오브젝트의 CompareTo 메소드를
호출해 `_estimatedCost` 값에 따라 노드를 정렬합니다.

### 그리드 매니저 설정
`GridManager` 클래스는 맵을 표현하는 모든 격자의 속성을 다룹니다.
맵을 표현하는데는 하나의 오브젝트만 필요하므로 `GridManager` 클래스를 `싱글톤Singleton`
인스턴스로 유지합니다.

```c#
using UnityEngine;
using System.Collections;

public class GridManager : MonoBehaviour
{
    private static GRidManager _instance = null;

    // 씬에서 GridManager 오브젝트를 찾아본 후 이미 있으면 이를 _instance 스태틱 변수에 할당해 관리한다.
    public static GridManager Instance
    {
        get
        {
            if(_instance == null)
            {
                _instance = FinObjectOfType(typeof(GridManager)) as GridManager;

                if(_instance == null)
                    Debug.Log("Could nor locate a GridManager object.\n" +
                              "You have to gave exactly one GridManager in the scene");
            }

            return _instance;
        }
    }

    // 열과 행의 수, 각 격자 타일의 크기, 격자와 장애물을 시각화하는 데 필요한 bool 변수
    // 격자에 있는 모든 노드 등 맵을 표현할 때 필요한 모든 변수를 선언
    public int _numOfRows
    public int _numOfColumns;
    public float _gridCellSize;
    public bool _showGrid;
    public bool _showObstacleBlocks = true;
    
    private Vector3 _origin = new Vector3();
    private GameObject[] _obstacleList;

    public Node[,] _nodes {get; set;}
    public Vector3 Origin
    {
        get => _origin;
    }

    private Awake()
    {
        _obstacleList = GameObject.FindGameObjectsWithTag("Obstacle");
        CalculateObstacles();
    }

    private void CalculateObstacles()
    {
        _nodes = new Node[_numOfColumns, _numOfRows];

        int index = 0;

        for(int col = 0; col < _numOfColumns; col++)
        {
            for(int row = 0; row <_numOfRows; row++)
            {
                Vector3 cellPos = GetGridCellCenter(index);
                Node node = new Node(cellPos);
                nodes[col, row] = node;
                index++;
            }
        }

        if(_obstacleList != null && _obstacleList.Length > 0)
        {
            // 맵에서 발견한 각 장애물을 리스트에 기록한다.
            foreach(GameObject data in _obstacleList)
            {
                int indexCell = GetGridIndex(data.transform.position);
                int col = GetColumn(indexCell);
                int row = GetRow(indexCell);
                nodes[row, col].MarkAsObstacle();
            }
        }
    }

    public Vector3 GetGridCellCenter(int index)
    {
        Vector3 cellPosition = GetGridCellPosition(index);
        cellPosition.x += (_gridCellSize / 2.0f);
        cellPosition.z += (_gridCellSize / 2.0f);

        return cellPosition;
    }

    public Vector3 GetGridCellPosition(int index)
    {
        int row = GetRow(index);
        int col = GetColumn(index);
        float xPosInGrid = col * _gridCellSize;
        float zPosinGrid = row * _gridCellSize;

        return Origin + new Vector3(xPosInGrid, 0.0f, zPosinGrid);
    }

    public int GetGridIndex(Vector3 pos)
    {
        if(!IsInBound(pos))
            return -1;

        pos -= Origin;
        int col = (int)(pos.x / gridCellSize);
        int row = (int)(pos.z / gridCellSize);

        return (row * _numColumns + col);
    }

    public bool IsInBounds(Vector3 pos)
    {
        float width = _numOfColumns * _gridCellSize;
        float height = _numOfRows * gridCellSize;

        return (pos.x >= Origin.x && pos.x < Origin.x + width && posx <= Origin.z + height && pos.z >= Origin.z);
    }

    public int GetRow(int index)
    {
        int row = index / _numOfColumns;

        return row;
    }

    public int GetColumn(int index)
    {
        int col = index % _numOfColumns;

        return col;
    }

    public void GetNeighbours(Node node, ArrayList neighbors)
    {
        Vector3 neighborPos = node.Position;
        int neighborIndex = GetGridIndex(neighborPos);

        int row = GetRow(neighborIndex);
        int column = GetColumn(neighborIndex);

        // 아래
        int leftNodeRow = row + 1;
        int leftNodeColumn = column;
        AssignNeighbour(leftNodeRow, leftNodeColumn, neighbors);

        // 위
        leftNodeRow = row + 1;
        leftNodeColumn = column;
        AssignNeighbour(leftNodeRow, leftNodeColumn, neighbors);

        // 오른쪽
        leftNodeRow = row;
        leftNodeColumn = column + 1;
        AssignNeighbour(leftNodeRow, leftNodeColumn, neighbors);

        // 왼쪽
        leftNodeRow = row;
        leftNodeColumn = column - 1;
        AssignNeighbour(leftNodeRow, leftNodeColumn, neighbors);
    }

    private void AssignNeighbour(int row, int column, ArrayList neighbors)
    {
        if(row != -1 && column != -1 && row < _numOfRows && column <_numOfColumns)
        {
            Node nodeToAdd = nodes[row, column];

            if(!nodeToAdd.bObstacle)
                neighbors.Add(nodeToAdd);
        }
    }
}
```