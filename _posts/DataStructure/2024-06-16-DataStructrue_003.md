---
title:  "[자료구조] 003. Stack과 Queue"
excerpt: "Stack과 Queue의 이론과 예제 코드입니다!"

author: gunnHB
categories: 
 - DataStructure
tags: 
 - [Github, Git, Programming, C++, DataStructure]

toc: true
toc_sticky: true
 
date: 2024-06-16
last_modified_at: 2024-06-16
---

## 개요
### Stack
`스택Stack`은 단어 그대로 <u>데이터가 쌓인 것</u>을 의미합니다. 이렇게 저장된 데이터들은
사용 시 가장 나중에 들어온 값 부터 사용되는 구조를 띕니다. 이를 `후입선출`, `LIFO(Last In First Out)`이라고 합니다.

원통형 과자를 생각하면 이해가 쉽습니다.
{: .notice--info}

### Queue
`큐Queue`는 스택과 비슷한 구조를 가지지만 큰 차이점은 <u>입구와 출구가 구분되어있다</u>는 점입니다.
삽입된 데이터들 중 먼저 들어온 데이터가 먼저 삭제되는 구조입니다. 이를 `선입선출`, `FIFO(First In First Out)`이라고 합니다.

온라인 게임에서의 대기 큐가 이것입니다.
{: .notice--info}

## 구현해보기
### 0. 구현 전 정리
스택과 큐는 LinkedList처럼 <u>여러 노드들이 연결된 구조</u>입니다. 다만 Double Linked List와는 달리 <u>이웃 노드 중 하나만 알고 있어도 됩니다.</u>

추가적으로 스택과 큐는 검색에 특화된 자료구조가 아니므로 검색 기능은 구현하지 않을 것입니다.

### 1-1. Stack의 기본 구조
먼저 스택을 구현해봅시다. 스택은 구조 상 가장 마지막에 들어온 값을 먼저 꺼내기 때문에 가상의 마지막 노드만을
미리 생성해두면 됩니다.

```c++
template<typename T>
class CStackNode
{
    template<typename T>
    friend class CStack;

private:
    CStackNode()
    {

    }

    ~CStackNode()
    {

    }

private:
    T mData;
    CStackNode<T>* mPrevNode;   // 현재 노드의 이전 노드
};

template<typename T>
class CStack
{
public:
    CStack()
    {
        mLastNode = new Node;

        mSize = 0;
    }

    ~CStack()
    {
        Node* deleteNode = mLastNode;

        while (deleteNode != nullptr)
        {
            Node* prevNode = deleteNode->mPrevNode;
            delete deleteNode;
            prevNode = deleteNode;
        }
    }

private:
    typedef CStackNode<T> Node;

private:
    Node* mLastNode = nullptr;
    int mSize;
};
```

### 1-2. Stack의 삽입와 삭제
다음으로 스택에 값을 삽입하거나 삭제하는 코드를 작성해봅시다.

```c++
template<typename T>
class CStack
{
    ...

public:
    void push(const T& data)
    {
        Node* newNode = new Node;                   // 신규 노드
        Node* prevNode = mLastNode->mPrevNode;      // 신규 노드와의 연결을 위한 임시 포인터

        newNode->mData = data;

        mLastNode->mPrevNode = newNode;
        newNode->mPrevNode = prevNode;

        ++mSize;
    }

    void pop()
    {
        if (mSize == 0)
            return;

        Node* deleteNode = mLastNode->mPrevNode;    // 가장 마지막 노드가 삭제됨
        Node* prevNode = deleteNode->mPrevNode;     // 삭제될 노드의 이전 노드

        delete deleteNode;

        mLastNode->mPrevNode = prevNode;

        --mSize;
    }
};
```

### 1-3. Stack의 그 외 함수
마지막으로 Stack의 크기 확인, 비어있는지 여부, 초기화 함수를 작성해봅시다.

```c++
template<typename T>
class CStack
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

    void claer()
    {
        Node* deleteNode = mLastNode->mPrevNode;

        while (deleteNode != nullptr)
        {
            Node* prevNode = deleteNode->mPrevNode;
            delete deleteNode;
            deleteNode = prevNode;
        }

        mSize = 0;
    }
};
```

### 2-1. Queue의 기본 구조
다음으로 큐를 구현해봅시다. 큐는 스택과 달리 먼저 들어온 데이터가 먼저 빠져나가야 되므로
가상의 첫 노드와 마지막 노드를 생성해 연결시켜주면 됩니다.

```c++
template<typename T>
class CQueueNode
{
    template<typename T>
    friend class CQueue;

private:
    CQueueNode()
    {

    }

    ~CQueueNode()
    {

    }

private:
    T mData;
    CQueueNode<T>* mNextNode = nullptr;
};

template<typename T>
class CQueue
{
public:
    CQueue()
    {

    }

    ~CQueue()
    {

    }

private:
    typedef CQueueNode<T> Node;

private:
    Node* mFirstNode = nullptr;
    Node* mLastNode = nullptr;
    int mSize;
};
```

### 2-2. Queue의 삽입과 삭제
다음은 큐의 삽입과 삭제입니다.

```c++
template<typename T>
class CQueue
{
    ...

public:
    void enqueue(const T& data)
    {
		Node* NewNode = new Node;
		NewNode->mData = Data;

		if (mSize == 0)
			mFirstNode = NewNode;
		else
			mLastNode->mNextNode = NewNode;

		mLastNode = NewNode;

		++mSize;
    }

    void dequeue()
    {
		if (mSize == 0)
			return;

		Node* nextNode = mFirstNode->mNextNode;

		delete mFirstNode;

		mFirstNode = nextNode;

		--mSize;
    }
};
```

### 2-3. Queue의 그 외 함수
마지막으로 큐의 크기 확인, 비어있는지 여부, 초기화 함수를 작성해봅시다.

```c++
template<typename T>
class CQueue
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
        while (mFirstNode != nullptr)
		{
			Node* nextNode = mFirstNode->mNextNode;

			delete mFirstNode;

			mFirstNode = nextNode;
		}

		mLastNode = nullptr;
		mSize = 0;
    }
};
```