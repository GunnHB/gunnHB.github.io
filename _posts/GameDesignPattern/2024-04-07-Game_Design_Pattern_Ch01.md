---
title:  "[유니티로 배우는 게임 디자인 패턴] 001. 싱글턴으로 게임 매니저 구현하기"
excerpt: "싱글턴 패턴을 알아봅시다!"

author: gunnHB
categories: 
 - Design Pattern
tags: 
 - [Github, Git, Unity, Programming, Design Pattern, C#]

toc: true
toc_sticky: true
 
date: 2024-04-07
last_modified_at: 2024-04-07
---

🔔 유니티로 배우는 게임 디자인 패턴 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
싱글턴 패턴은 게임의 모든 핵심 시스템을 래핑하고 관리하는 간단한 코드 아키텍쳐를 빠르게
만들 수 있다는 장점이 있지만 구성 요소 간의 결합을 강력하게 하여 유닛 테스트를 매우
어렵게 만든다는 단점이 있습니다.

- 싱글턴 패턴의 기본
- 유니티에서 재사용 가능한 싱글턴 클래스 작성
- GameManager 구현
</div>

## 싱글턴 패턴 이해하기
싱글턴 패턴의 주요 목표는 유일성을 보장하는 것입니다. 이는 초기화된 후에는 `런타임 동안 메모리에
오직 하나의 인스턴스만 존재`한다는 것을 의미합니다.

싱글턴 디자인은 매우 단순합니다. 싱글턴 클래스는 자신과 같은 유형의 개체 인스턴스를 발견하면 즉시 없앱니다.

싱글턴 패턴에서 가장 중요한 점은 구현이 잘 됐다면 오직 하나만 존재한다는 것으로,
그렇지 않다면 `목적 달성에 실패한 것`입니다.

### 싱글턴 패턴의 장단점
싱글턴 패턴의 장점은

- 전역 접근 가능
    - 실글턴 패턴을 사용하여 리소스나 서비스의 전역 접근점을 생성할 수 있다.
- 동시성 제어
    - 공유 자원에 동시 접근을 제한하고자 사용 가능하다

싱글턴 패턴의 단점은

- 유닛 테스트
    - 싱글턴 오브젝트가 다른 싱글턴에 종속될 수 있다. 하나가 누락되면 종속성이 끊어진다.
- 잘못된 습관
    - 쉬운 사용으로 인해 정교한 작업이 귀찮게 느껴진다.

디자인을 선택할 때 아키텍쳐의 유지 및 관리, 확장, 테스트 가능 여부를 항상 염두에 둡시다.
{: .notice--warning}

## 게임 매니저 디자인하기
일반적으로 게임 매니저는 개발자가 싱글턴으로 구현하지만 코드마다 그 관리 범위가 다릅니다.

프로그래머에 따라 최고 레벨의 게임 상태를 관리하거나 전역적으로 접근할 수 있는 게임 시스템의
전면 인터페이스로 싱글턴을 사용합니다.
{: .notice--warning}

게임 매니저는 게임의 전체 수명 동안 살아있어야 한다는 점을 명심해야합니다.

## 게임 매니저 구현하기

```c#
using UnityEngine;

public class Singletone <T> : MonoBehaviour where T : Component
{
    private static T _instance;

    public static T Instance
    {
        get
        {
            if(_instance == null)
            {
                // 지정한 타입의 첫 번째로 로드된 오브젝트를 검색
                _instance = FindObjectByType<T>();

                if(_instance == null)
                {
                    // 없으면 새로운 오브젝틀르 생성한 후 컴포넌트를 추가
                    GmaeObject obj = new GameObject();
                    obj.name = typeof(T).Name;
                    _instance = obj.AddComponent<T>();
                }
            }

            return _instance;
        }
    }

    public virtual void Awake()
    {
        if(_instance == null)
        {
            _instance = this as T;
            DontDestroyOnLoad(gameObject);
        }
        else
            Destroy(gameObject);
    }
}
```

엔진이 Awake()를 호출했을 때 싱글턴 컴포넌트는 메모리에 초기화된 자신의 인스턴스가 이미
있는지 확인한다는 것을 명심해야 합니다.

다음은 GameManager 클래스의 스켈레톤 코드입니다.

```c#
using System;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBahviour
{
    private DateTime _sessionStartTime;
    private DateTime _sessionEndTime;

    private void Start()
    {
        // TODO:
        // - 플레이어 세이브 / 로드
        // - 세이브가 없으면 플레이어 등록 씬으로 리다이렉션
        // - 백엔드를 호출하고 일일 챌린지와 보상을 얻음

        _sessionStartTime = DateTime.Now;
        Debug.Log("Game session start @: " + DateTime.Now);
    }

    private void OnApplicationQuit()
    {
        _seesionEndTime = DateTime.Now;
         
        TimeSpan timeDifference = _sessionEndTime.Subtract(_sessionStartTime);

        Debug.Log("Game session ended @: " + DatTime.Now);
        Debug.Log("Game session lasted: " + timeDifference);
    }

    private void OnGUI()
    {
        if (GUILayout.Button("Next Scene"))
        {
            SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + 1);
        }
    }
}
```

주석 처리된 todo의 내용은 게임 매니저의 잠재적인 작업 목록입니다.
{: .notice--info}

위 GameManager를 Singleton으로 변경한다면 다음과 같습니다.

```c#
public class GameManager : Singleton<GameManager>
{
    ...
}
```

## 정리
싱글턴 패턴은 유니티의 코딩 모델에 완벽한 패턴이지만 지나친 사용은 과한 의존을 불러온다는 것을 잊지 맙시다.

다음 포스팅에서는 상태 패턴에 대해 알아보겠습니다ㄴ.