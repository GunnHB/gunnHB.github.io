---
title:  "[알고리즘] 백준::2193 이친수"
# excerpt:

author: gunnHB
categories: 
 - Algorithm
tags: 
 - [Github, Git, Programming, C++, Algorithm]

toc: true
toc_sticky: true
 
date: 2024-07-14
last_modified_at: 2024-07-14
---

🔔 [문제 풀러가기](https://www.acmicpc.net/problem/2193) 🔔
{: .notice}

## 문제
2193::이친수

0과 1로만 이루어진 수를 이진수라 한다.이러한 이진수 중 특별한 성질을 갖는 것들이 있는데, 이들을 이친수(pinary number)라 한다.이친수는 다음의 성질을 만족한다.

1. 이친수는 0으로 시작하지 않는다.
2. 이친수에서는 1이 두 번 연속으로 나타나지 않는다.즉, 11을 부분 문자열로 갖지 않는다.

예를 들면 1, 10, 100, 101, 1000, 1001 등이 이친수가 된다.하지만 0010101이나 101101은 각각 1, 2번 규칙에 위배되므로 이친수가 아니다.

N(1 ≤ N ≤ 90)이 주어졌을 때, N자리 이친수의 개수를 구하는 프로그램을 작성하시오.

입력
첫째 줄에 N이 주어진다.

출력
첫째 줄에 N자리 이친수의 개수를 출력한다.

## 이해하기
하나씩 경우를 적어가면서 풀면 생각보다 쉽게 풀 수 있는 문제였습니다. 

1. 첫 번째 자리에 올 수 있는 수는 항상 1 (1번 규칙)
2. 두 번째 자리에 올 수 있는 수는 항상 0 (2번 규칙)

위 조건을 생각해 자리 수 별 이친수를 구하면

1. 한 자리 이친수의 개수는 1개 (1)
2. 두 자리 이친수의 개수는 1개 (10)
3. 세 자리 이친수의 개수는 2개 (101, 100)
4. 네 자리 이친수의 개수는 3개 (1010, 1000, 1001)
5. 다섯 자리 이친수의 개수는 5개 (10101, 10100, 10001, 10000, 10010)

계산을 이어 나가보면 `피보나치 수열`임을 알 수 있습니다.

피보나치 수열이란 f(n) = f(n - 1) + f(n - 2) (n > 2)의 규칙을 가지는 수열을 뜻합니다.
{: .notice--info}

이를 단순히 재귀함수를 이용해 풀 수 있긴 하지만 그렇게 되면 N 크면 클수록
함수의 호출이 기하급수적으로 늘어나 시간이 굉장히 오래 걸리게 됩니다.

그렇기 때문에 동적 계획법(DP)를 이용해 연산의 횟수를 줄일 필요가 있습니다.

연산의 결과를 따로 저장해 중복 연산을 막을 수 있습니다.
{: .notice--info}

## 풀이
```c++
#include <iostream>
#include <vector>

long long GetPinary(const long long& input);

int main()
{
	std::ios::sync_with_stdio(0);
	std::cin.tie(0);
	std::cout.tie(0);

	long long _input = 0;

	std::cin >> _input;
	std::cout << GetPinary(_input);

	return 0;
}

long long GetPinary(const long long& input)
{
	std::vector<long long> pinaryVector;

	pinaryVector.push_back(1);		// 0번
	pinaryVector.push_back(1);		// 1번

    // 연산의 결과를 벡터에 저장하기 때문에 필요없는 연산을 막을 수 있다.
    // 그냥 값을 꺼내오면 되는 것
	for (int index = 2; index < input; ++index)
		pinaryVector.push_back(pinaryVector[index - 1] + pinaryVector[index - 2]);

	return pinaryVector[input - 1];
}
```