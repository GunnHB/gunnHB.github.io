---
title:  "[자료구조] 002. LinkedList"
excerpt: "LinkedList의 이론과 예제 코드입니다!"

author: gunnHB
categories: 
 - DataStructure
tags: 
 - [Github, Git, Programming, C++, DataStructure]

toc: true
toc_sticky: true
 
date: 2024-06-15
last_modified_at: 2024-06-15
---

## 개요
배열은 <u>같은 타입의 데이터 여러 개를 사용하고 싶을 때</u> 사용하는 경우가 많습니다. 만약 10개의 정수형
변수를 사용하려고 할 때 각각의 변수 10개를 선언하는 것보다 크기 10의 정수형 배열을 선언해 사용하는 것이
더욱 효율적입니다.

하지만 크기가 고정적이라는 점으로 인해 추가적인 데이터를 삽입하는 것이 불가능합니다. 이를 해결하기 위한 방법 중
하나가 바로 `LinkedList`입니다.

## LinkedList
LinkedList는 말 그대로 <u>연결된 리스트</u>입니다. 여기서 연결이라는 의미는 현재의 데이터에서 `다음 데이터의 주소`를
알고 있는 것을 의미합니다. 이렇게 되면 가변적인 크기의 배열을 사용할 수 있는 것입니다.

LinkedList는 여러 `노드Node`가 연결된 구조입니다. 여기서 노드란 `데이터 부분`과 `다음 노드의 주소`로 이루어진 요소입니다.

여기서 만약 노드에 다음 노드의 주소만을 가지고 있다면 `Single Linked List`라고 하고 이전 노드의 주소까지 알고 있다면
`Double Linked List`라고 합니다.

이번 포스팅에서는 Double Linked List를 구현할 것입니다.
{: .notice--info}

## LinkedList 구현해보기
### 1. 기본 구조
그렇다면 LinkedList를 구현해봅시다. 위에서 작성한 것처럼 LinkedList는 여러 노드들이 연결된 구조입니다.
하지만 첫 데이터를 추가할 때 하나의 노드만이 덩그러니 생성되어 해당 이 노드의 주소를 알 수 있는 방법이 없습니다.
이렇게 되면 사용하는 것은 물론이고 삭제 시에도 문제가 발생합니다.

그렇기 때문에 LinkedList를 생성할 때 가상의 시작 노드와 끝 노드를 미리 생성해줍니다. 이를 이용하면
추가된 신규 노드를 쉽게 찾아갈 수 있게 됩니다.

```c++
template<typename T>
class CListNode
{
    // friend 키워드를 이용해 해당 클래스에서 현재 클래스의 private이나 protected로
    // 선언된 멤버를 사용할 수 있게 한다.
    template<typename T>
    friend class CLinkedList;

    // 외부에서 CListNode의 생성/소멸 불가능
private:
    CListNode()
    {

    }

    ~CLintNode()
    {

    }

private:
    T mData;                              // 현재 노드의 데이터
    CListNode<T>* mPrevNode = nullptr;    // 이전 노드 주소
    CListNode<T>* mNextNode = nullptr;    // 다음 노드 주소
};

template<typename T>
class CLinkedList
{
public:
    CLinkedList()
    {
        // LinkedList가 생성되면 시작 노드와 끝 노드를 동적할당
        mBeginNode = new Node;
        mEndNode= new Node;

        // 시작 노드와 끝 노드를 서로 연결시켜준다.
        // 시작 노드의 이전과 끝 노드의 다음은 존재하지 않는다.
        mBeginNode->mNextNode = mEndNode;
        mEndNode->mPrevNode = mBeginNode;

        mSize = 0;
    }

    ~CLinkedList()
    {
        // LinkedList가 소멸되면 모든 노드를 삭제해준다.
        Node* deleteNode = mBeginNode;

        while (deleteNode != nullptr)
        {
            // 삭제할 노드를 그 다음 노드로 갱신해주게 되면
            // 최종적으로 끝 노드의 다음 노드 값이 들어가게 되면서
            // 반복이 종료됨
            Node* nextNode = deleteNode->mNextNode;
            delete deleteNode;
            deleteNode = nextNode;
        }
    }

private:
    // CLinkedList<T> 타입을 Node로 정의
    // Node a; 라고 선언한 것은 곧 CLinkedList<T> a; 라고 선언한 것과 같다.
    typedef CLinkedList<T> Node;

private:
    // 시작 노드와 끝 노드를 선언
    // 해당 노드는 시작과 끝만을 알려주기 위한 노드로
    // 데이터를 가지지 않는다.
    Node* mBeginNode = nullptr;   
    Node* mEndNode = nullptr;
    int mSize;          // LinkedList의 크기. 노드가 추가되거나 삭제되면 값이 증감한다.
};
```

### 2. 추가와 삭제
다음으로는 추가와 삭제를 구현해봅시다. 추가와 삭제 기능은 리스트의 `가장 앞`이나 `가장 뒤`에 이루워지게 됩니다.

```c++
template<typename T>
class CLinkedList
{
    ...

public:
    // 가장 뒤에 추가
    void push_back(const T& data)
    {
        // 신규 노드를 끝 노드와 이전 노드로 연결시켜주면 된다.

        Node* newNode = new Node;               // 신규로 추가되는 노드를 동적할당
        Node* prevNode = mEndNode->mPrevNode    // 가장 뒤에 추가되는 것이기에 끝 노드의 이전 노드를 교체해줘야 한다.

        newNode->mData = data;                  // 신규 노드의 데이터에 값을 할당

        newNode->mNextNode = mEndNode;          // 신규 노드의 다음 노드는 끝 노드
        prevNode->mNextNode = newNode;          // (노드 추가 전)이전 노드의 다음 노드는 신규 노드

        mEndNode->mPrevNode = newNode;          // 끝 노드의 이전 노드는 신규 노드
        newNode->mPrevNode = prevNode;          // 신규 노드의 이전 노드는 (노드 추가 전)이전 노드

        ++mSize;                                // 노드가 추가됐으므로 값 1 증가
    }

    // 가장 앞에 추가
    void push_front(const T& data)
    {
        // 가장 뒤에 추가하는 경우와 마찬가지로
        // 시작 노드와 다음 노드를 신규 노드와 연결시켜주면 된다.

        Node* newNode = new Node;
        Node* nextNode = mBeginNode->mNextNode;

        newNode->mPrevNode = mBeginNode;
        nextNode->mPrevNode = newNode;

        mBeginNode->mNextNode = newNode;
        newNode->mNextNode = nextNode;

        ++mSize;
    }

    // 가장 뒤의 값을 제거
    void pop_back()
    {
        // 끝 노드의 이전 노드를 지운 후
        // 지운 노드의 이전 노드와 끝 노드를 연결시켜주면 된다.

        Node* deleteNode = mEndNode->mPrevNode;     // 끝 노드의 이전 노드
        Node* prevNode = deleteNode->mPrevNode;     // 지울 노드의 이전 노드

        delete deleteNode;                          // 동적할당을 해제

        mEndNode->mPrevNode = prevNode;             // 끝 노드의 이전 노드를 갱신
        prevNode->mNextNode = mEndNode;             // 이전 노드의 다음 노드를 갱신

        --mSize;                                    // 노드가 삭제됐으므로 1 감소
    }

    // 가장 앞의 값을 제거
    void pop_front()
    {
        // 마찬가지로 가장 뒤의 값을 지우는 것을
        // 가장 앞의 값을 지우는 것으로 바꿔주면 된다.

        Node* deleteNode = mBeginNode->mNextNode;
        Node* nextNode = deleteNode->mNextNode;

        delete deleteNode;

        mBeginNode->mNextNode = nextNode;
        nextNode->mPrevNode = mBeginNode;

        --mSize;
    }
};
```

### 3. 검색
다음은 검색 기능을 추가해봅시다. 검색을 위해서는 `iterator`를 추가해야 합니다. iterator란 단순히
<u>특정 노드의 주소</u>를 의미합니다. 이를 이용하여 원하는 노드를 찾아 데이터를 얻을 수 있는 것입니다.

여기서 중요한 것은 <u>기존의 연산자를 재정의하는 것</u>입니다.

```c++
template<typename T>
class CListNode
{
    // CListIterator에서 멤버를 알 수 있도록
    template<typename T>
    friend class CListIterator;

    ...
};

template<typename T>
class CListIterator
{
    // CLinkedList에서 멤버를 알 수 있도록
    template<typename T>
    friend class CLinkedList;

public:
    CListIterator()
    {

    }

    ~CListIterator()
    {

    }

private:
    CListNode<T>* mNode = nullptr;    // iterator가 바라보는 노드

public:
    // iterator에서의 연산을 위해 재정의

    // 인자 앞의 const는 참조하는 대상의 값을 변경할 수 없도록 하기 위함
    // 함수 뒤의 const는 멤버의 변경이 없다는 것을 명시적으로 표현하기 위함
    // const 함수에서는 const 함수만을 호출할 수 있다.
    bool operator == (const CListIterator<T>& iter) const
    {
        return mNode == iter.mNode;
    }

    bool operator != (const CListIterator<T>& iter) const
    {
        return mNode != iter.mNode;
    }

    // 전위연산
    void operator ++ ()
    {
        mNode = mNode->mNextNode;
    }

    // 후위연산
    void operator ++ (int)
    {
        mNode = mNode->mNextNode;
    }

    // 전위연산
    void operator -- ()
    {
        mNode = mNode->mPrevNode;
    }

    // 후위연산
    void operator -- (int)
    {
        mNode = mNode->mPrevNode;
    }

    // 역참조를 재정의
    T& operator * ()
    {
        return mNode->mData;
    }

    operator CListNode<T>* ()
	{
		return mNode;
	}

    bool
};
```

이렇게 작성된 iterator를 사용하는 함수를 작성해봅시다.

```c++
template<typename T>
class CLinkedList()
{
    ...

private:
    typedef CListIterator<T> iterator;  // 타입을 재정의

public:
    // 맨 앞 노드를 반환
    iterator& begin()
    {
        // static 변수로 선언해 추가적인 객체의 생성을 방지
        static iterator iter;
        iter.mNode = mBeginNode->mNextNode;
        return iter;
    }

    // 맨 뒤 노드를 반환
    iterator& end()
    {
        // static 변수로 선언해 추가적인 객체의 생성을 방지
        static iterator iter;
        iter.mNode = mEndNode->mPrevNode;
        return iter;
    }

    // 특정 데이터를 삭제
    bool erase(const T& data)
    {
        // 노드들을 반복문으로 순회
        iterator iter = begin();
        iterator iterEnd = end();

        for (; iter != iterEnd; ++iter)
        {
            if(*iter == data)
                break;
        }

        if(iter == iterEnd)
            return false;

        erase(iter);        // 실제 데이터를 지우는 함수

        return true;
    }

private:
    void erase(iterator& iter)
    {
        Node* prevNode = iter.mNode->mPrevNode;     // 삭제하려는 노드의 이전 노드
        Node* nextNode = iter.mNode->mNextNode;     // 삭제하려는 노드의 다음 노드

        delete iter.mNode;                          // 해당하는 노드를 삭제 

        // 이전 노드와 다음 노드를 연결
        prevNode->mNextNode = nextNode;
        nextNode->mPrevNode = prevNode;

        --mSize;                                    // 삭제됐으니 크기를 1 감소
    }
};
```

### 4. 그 외 함수
끝으로 리스트의 크기와 빈 리스트인지의 여부, 초기화 함수를 작성해보겠습니다. 굉장히 간단하므로
보면 바로 이해될 것입니다.

```c++
template<typename T>
class CLinkedList
{
    ...

public:
    int size()
    {
        return mSize;
    }

    bool empty()
    {
        return mSize == 0;
    }

    void clear()
    {
        // 시작 노드와 끝 노드를 제외한 모든 노드를 삭제
        Node* deleteNode = mBeginNode->mPrevNode;

        while (deleteNode != mEndNode)
        {
            Node* nextNode = deleteNode->mNextNode;
            delete deleteNode;
            deleteNode = nextNode;
        }

        // 시작 노드와 끝 노드를 연결
        mBeginNode->mNextNode = mEndNode;
        mEndNode->mPrevNode = mBeginNode;

        // 크기 초기화
        mSize = 0;
    }
};
```
