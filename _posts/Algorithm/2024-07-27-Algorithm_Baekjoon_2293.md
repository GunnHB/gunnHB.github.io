---
title:  "[알고리즘] 백준::2193 이친수"
excerpt: "'백준::2193 이친수'위 문제 풀이입니다!"

author: gunnHB
categories: 
 - Algorithm
tags: 
 - [Github, Git, Programming, C++, Algorithm]

toc: true
toc_sticky: true
 
date: 2024-07-27
last_modified_at: 2024-07-27
---

🔔 [백준::2293 동전 1 문제 풀러가기](https://www.acmicpc.net/problem/2293) 🔔
{: .notice}

## 문제
n가지 종류의 동전이 있다.각각의 동전이 나타내는 가치는 다르다.이 동전을 적당히 사용해서, 그 가치의 합이 k원이 되도록 하고 싶다.
그 경우의 수를 구하시오.각각의 동전은 몇 개라도 사용할 수 있다.

사용한 동전의 구성이 같은데, 순서만 다른 것은 같은 경우이다.

입력
첫째 줄에 n, k가 주어진다. (1 ≤ n ≤ 100, 1 ≤ k ≤ 10,000) 다음 n개의 줄에는 각각의 동전의 가치가 주어진다.
동전의 가치는 100,000보다 작거나 같은 자연수이다.

출력
첫째 줄에 경우의 수를 출력한다.경우의 수는 231보다 작다.

## 이해하기
문제에서 제공해주는 예제를 보면 다음과 같습니다.

- 입력
	- 3 10
	- 1
	- 2
	- 5

- 출력
	- 3

이를 이용해 테이블을 그려 경우를 생각해보면 다음과 같습니다.

|k|1|1, 2|1, 2, 5|
|--|--|--|--|
|0|0|0|0|
|1|1|1|1|
|2|1|2|2|
|3|1|2|2|
|4|1|3|3|
|5|1|3|4|
|6|1|4|5|
|7|1|4|6|
|8|1|5|7|
|9|1|5|8|
|10|1|6|10|

해당 테이블은 사용되는 동전((1) or (1,2) or (1,2,5))을 이용해 목표 액수(k)를 만들 수 있는 경우의 수를 나타냅니다.
{: .notice--info}

먼저 0원을 만드는 모든 경우는 <u>아무런 동전을 사용하지 않는 경우</u> 즉, `한 가지의 경우`입니다. 이를 메모용 벡터에 저장한다고 하면 다음과 같습니다.

```c++
// k == 10일 때
mMemoVector[11] = {};		// 데이터를 모두 0으로

mMemoVector[0] = 1;			// 0의 경우에 1가지만 존재
```

이를 이용해 1원으로 1원부터 10원까지 만들 수 있는 경우의 수를 구하는 식은 다음과 같습니다.

```c++
// index == 1 ~ 10
mMemoVector[index] += mMemoVector[index - 1];
```

다음 2원이 추가되었을 때 0원과 1원을 만들 수 없으므로 1원의 경우를 그대로 가져오고
2원부터 10원까지 만들 수 있는 경우의 수를 구하면 됩니다.

```c++
mMemoVector[index] += mMemoVector[index - 2];
```

마지막으로 5원이 추가되었을 때 마찬가지로 0원부터 4원까지는 이전의 경우를 사용하고
5원부터 10원까지의 경우의 수를 구하면 됩니다.

```c++
mMemoVector[index] += mMemoVector[index - 5];
```

이를 이용해 최종적인 점화식을 구해보면 다음과 같습니다.

```c++
// mCoinVector == 입력받은 동전의 벡터
// cIndex == 동전 벡터의 인덱스
mMemoVector[index] += mMemoVector[index - mCoinVector[cIndex]];
```

## 풀이
```c++
#include <iostream>
#include <vector>
#include <algorithm>

class CCoin
{
private:
	int mInput;
	int mTarget;
	std::vector<int> mCoinVector;
	std::vector<int> mMemoVector;

public:
	void Init()
	{
		std::cin >> mInput >> mTarget;

		mCoinVector.reserve(mInput);
		mMemoVector.reserve(mTarget + 1);		// 0 ~ mTarget

		for (int index = 0; index < mInput; ++index)
		{
			int temp;
			std::cin >> temp;

			mCoinVector.push_back(temp);
		}

		for (int index = 0; index < mTarget + 1; ++index)
			mMemoVector.push_back(0);

		std::sort(mCoinVector.begin(), mCoinVector.end());
		mMemoVector[0] = 1;
	}

	void GetCaseCount()
	{
		for (int cIndex = 0; cIndex < mInput; ++cIndex)
		{
			for (int index = mCoinVector[cIndex]; index < mTarget + 1; ++index)
				mMemoVector[index] += mMemoVector[index - mCoinVector[cIndex]];
		}

		std::cout << mMemoVector[mTarget];
	}
};

int main()
{
	std::ios::sync_with_stdio(0);
	std::cin.tie(0);
	std::cout.tie(0);

	CCoin _coin;

	_coin.Init();
	_coin.GetCaseCount();

	return 0;
}
```
## 느낀 점
규칙을 발견하기에 꽤나 까다로웠던 문제입니다.😓 그래도 DP적 사고력을 꽤나 올릴 수 있었던 문제 같습니다.:D