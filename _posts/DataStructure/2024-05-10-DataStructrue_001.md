---
title:  "[자료구조] 001. 시간복잡도"
excerpt: "시간복잡도에 관한 정리입니다!"

author: gunnHB
categories: 
 - DataStructure
tags: 
 - [Github, Git, Programming, C#, DataStructure]

toc: true
toc_sticky: true
 
date: 2024-05-10
last_modified_at: 2024-05-10
---

🔔 유튜브 ['쉬운코드'님의 영상](https://www.youtube.com/watch?v=tTFoClBZutw)을 정리한 내용입니다. 🔔
{: .notice}

## 시간복잡도
함수의 <u>실행 시간</u>을 나타내는 것입니다. 이는 `점근적 분석`을 통해 실행시간을 `점근적 표기법`으로 표현합니다.

아래 함수는 인자값으로 정수형 배열 `input`과 정수 `multiplier`를 받아,
input 길이의 새로운 정수형 배열 `result`를 생성해 연산을 진행하여 결과를 반환합니다.

```c#
private int[] ExampleMethod(int[] input, int multiplier)
{
    int[] result = new int[input.Length];

    for (int index = 0; index < result.Length; index++)
        result[index] = input[index] * multiplier;

    return result;
}
```

이 함수를 실행했을 때 실행 시간이 어느 정도인지 표현해 보고 싶습니다. 그 전에 우리는 사전 정의해 둘 것이 필요한데,

- 실행 시간은 함수/알고리즘 수행에 필요한 `스텝 수`로 정의
- 각 라인을 수행하기 위해 필요한 스텝 수는 `상수`로 가정

이를 이용하여 정리해보면 다음과 같습니다.

```
private int[] ExampleMethod(int[] input, int multiplier)
{                                                                  cost          times
    int[] result = new int[input.Length];                           c1             1

    for (int index = 0; index < result.Length; index++)             c2           N + 1
        result[index] = input[index] * multiplier;                  c3             N

    return result;                                                  c4             1
}

* N = input의 길이
```

`cost`는 각 라인 별 수행되는 스텝입니다. 그리고 `times`는 각 스텝이 수행되는 횟수를 나타냅니다.
예를 들어 c1 라인의 수행 횟수는 1회, c3 라인의 수행 횟수는 N회인 것입니다.

c2의 경우 반복문을 빠져나갈 때에도 수행되기 때문에 N + 1회입니다.
{: .notice--info}

이를 이용하여 실행 시간을 구해봅시다. 실행 시간은 각 라인(cost)에 라인이 실행되는 횟수(times)를 곱한 값을 모두 더해주면 됩니다.

```
T(N) = c1 * 1 + c2 * (N + 1) + c3 * N + c4 * 1
    = c1 + c2 * (N + 1) + c3 * N + c4
    = (c2 + c3) * N + c1 + c2 + c4
    = a * N + b

* a = (c2 + c3), b = (c1 + c2 + c4) 식을 간단하게 하기 위해 치환
```

위 식에서 더 정확한 계산은 어렵습니다. 또한 배열의 크기(N)가 작아질수록 실행시간은 무의미해집니다.  
우리는 <u>배열의 크기가 무한대로 증가할 때</u> 실행시간을 알아봐야 합니다.

이것을 알아보기 위해서는 식을 단순화할 필요가 있습니다.

- 최고차항을 제외한 나머지 항은 삭제
    - N이 무한대로 커질수록 최고차항 외의 나머지 항들은 식에 큰 영향을 주지 못합니다.
- 최고차항의 계수 삭제
    - 최고차항의 계수 역시 같은 조건에서 무의미한 값입니다.

최종적으로 식을 정리하면 <u>Θ(N)</u>입니다.

Big theta n 또는 Theta n으로 읽을 수 있습니다.
{: .notice--info}

이렇게 임의의 함수가 <u>N이 무한대로 증가할 때</u> 어떤 함수의 형태에 근접해지는지 분석하는 것을 `점근적 분석asymptotic analysis`이라고 합니다.
또한 위 식의 결과인 Θ(N)같은 표기법을 `점근적 표기법`이라고 합니다.

