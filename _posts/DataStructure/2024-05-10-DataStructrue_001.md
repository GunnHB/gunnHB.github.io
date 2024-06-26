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
함수의 <u>실행 시간</u>을 나타내는 것입니다. 이는 `점근적 분석`을 통해 실행 시간을 `점근적 표기법`으로 표현합니다.

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

```c#
private int[] ExampleMethod(int[] input, int multiplier)
{                                                                  // cost          times
    int[] result = new int[input.Length];                          //  c1             1

    for (int index = 0; index < result.Length; index++)            //  c2           N + 1
        result[index] = input[index] * multiplier;                 //  c3             N

    return result;                                                 //  c4             1
}

* N = input의 길이
```

`cost`는 각 라인 별 수행되는 스텝입니다. 그리고 `times`는 각 스텝이 수행되는 횟수를 나타냅니다.
예를 들어 c1 라인의 수행 횟수는 1회, c3 라인의 수행 횟수는 N회인 것입니다.

c2의 경우 반복문을 빠져나갈 때에도 수행되기 때문에 N + 1회입니다.
{: .notice--info}

이를 이용하여 실행 시간을 구해봅시다. 실행 시간은 각 라인(cost)에 라인이 실행되는 횟수(times)를 곱한 값을 모두 더해주면 됩니다.

```c#
// T(N) = c1 * 1 + c2 * (N + 1) + c3 * N + c4 * 1
//     = c1 + c2 * (N + 1) + c3 * N + c4
//     = (c2 + c3) * N + c1 + c2 + c4
//     = a * N + b

// * a = (c2 + c3), b = (c1 + c2 + c4) 식을 간단하게 하기 위해 치환
```

위 식에서 더 정확한 계산은 어렵습니다. 또한 배열의 크기(N)가 작아질수록 실행 시간은 무의미해집니다.  
우리는 <u>배열의 크기가 무한대로 증가할 때</u> 실행 시간을 알아봐야 합니다.

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

## 점근적 표기법
```c#
private bool ExampleMethod(int[] input, int target)
{
    for (int index = 0; index < input.Length; index++)
    {
        if (input[index] == target)
            return true;
    }

    return false;
}
```

위 함수는 인자로 정수형 배열 `input`과 정수 `target`을 받아, 배열 안에 값이 존재하는지 확인하는 기능을 수행합니다.

해당 함수는 `인자 값`에 따라 실행 시간이 조금씩 달라집니다. 예를 들어 길이가 100인 배열에서 찾으려는 값이 앞 쪽에
위치해 있다면 실행 시간이 짧을 것이고 반대로 뒤 쪽에 위치하거나 아예 값이 없다면 시간은 길어질 것입니다.

이를 `최선best`과 `최악worst`의 경우에 따라 분류해보면

```c#
//      case                   when                  times
//  ----------------------------------------------------------
//      best                0번째에 존재                1
//      worst       마지막에 존재 || 존재하지 않음       N
```

위와 같이 나타낼 수 있습니다. 최선의 경우 배열의 0번째에 위치한 값을 찾기 때문에 수행 횟수는 1번이고,
최악의 경우 N - 1번째에 위치한 값이거나 존재하지 않는 값을 찾기 때문에 수행 횟수는 N번이 됩니다.

이를 문장으로 나타내보면

- `하한선lower bound`
    - 함수의 실행 시간은 아무리 빨라도 <u>상수 시간 이상</u>
- `상한선upper bound`
    - 함수의 실행 시간은 아무리 오래 걸려도 <u>N에 비례하는 정도 이하</u>

기준을 어디에 두느냐에 따라 문장은 달라지지만 의미는 동일합니다.
쉽게 말하자면 "아무리 빨라도 ~정도는 걸린다."와
"아무리 오래 걸려도 ~정도는 걸리지 않는다."라고 할 수 있습니다.
{: .notice--info}

이것을 점근적 표기법으로 표현하면 lower bound를 <u>Ω(1)</u>, upper bound를 <u>Ο(N)</u>로 표기합니다.

Ω(1)을 Big omega one, Ο(N)을 Big o n으로 읽을 수 있습니다.
{: .notice--info}

Ω(1)에서의 Ω는 `하한선`을 의미하며, 1은 N의 크기에 상관없이 <u>상수 시간만큼 걸린다</u>는 것을 의미합니다.
반대로 Ο(N)에서의 Ο는 `상한선`을 의미하며, N은 <u>N의 크기에 비례하는 만큼 걸린다</u>는 것을 의미합니다.

위의 함수를 점근적 표기법으로 표기하면 Ω(1), Ο(N) 둘 다 맞지만, <u>최악의 경우를 더 우선시</u>하기 때문에
<u>Ο(N)</u>이 더욱 맞는 표기입니다.

### 상황별 시간복잡도
코드의 상황별 시간복잡도를 구해봅시다.

```c#
//          best            worst           avg
//  ------------------------------------------------
//  Ω       Ω(1)             Ω(N)           
//  Ο       Ο(1)             Ο(N)           Ο(N)
//  Θ       Θ(1)             Θ(N)           
```

먼저 최선(best)과 최악(worst)으로 나누어 생각해봅시다. 

최선은 배열의 0번째에 찾으려는 값이 있는 경우이고, 최악의 경우는 찾으려는 값이 배열의 맨 끝에 위치하거나 없는 값을 경우입니다.
{: .notice--info}

최선의 경우 상한선과 하한선 모두 상수 시간 안에 끝나게 됩니다. 그렇기 때문에 이는 <u>Ω(1)</u>, <u>Ο(1)</u>으로 표기할 수 있습니다.
이렇게 upper bound와 lower bound의 크기가 같을 때는 Θ로도 나타낼 수 있는데, 이 경우에는 <u>Θ(1)</u>으로 표기합니다.

최악의 경우 상한선과 하한선 모두 배열의 크기에 배례하는 시간이 걸리게 됩니다. 그렇기 때문에 이는 <u>Ω(N)</u>, <u>Ο(N)</u>으로 표기할 수 있습니다.
최선의 경우와 같이 상한선과 하한선의 크기가 같기 때문에 Θ로도 나타낼 수 있고, 이 경우에는 <u>Θ(N)</u>으로 표기합니다.

이처럼 best와 worst 모두 상한선과 하한선이 같을 때 둘은 굉장히 가까이 있다고 할 수 있습니다. 이를 Θ를 통해 표현하며 Θ는 `tight bound`에 사용됩니다.
예를 들어, Θ(N)으로 표기된 것은 worst 케이스에서 상한선과 하한선 모두 배열의 크기(N의 크기)에 비례한다는 것을 알 수 있습니다.

"상한선과 하한선이 가까이 있다"는 "같은 형태를 띈다"라고도 할 수 있습니다.
{: .notice--info}

추가로 평균(avg)의 경우는 best와 worst의 경우를 제외한 일반적인 경우입니다. 이는 배열의 중간 쯤에 값을 찾게 된다는 의미이기에,
배열의 절반인 N/2로 계산할 수 있습니다. 하지만 점근적인 측면에서 계수는 큰 역할을 하지 않는다고 했기 때문에 결과적으로 N의 값을 띄게 됩니다.
이를 점근적 표기법으로 나타내면 위와 같이 <u>Ο(N)</u>으로만 나타낼 수 있습니다. 

lower bound와 tight bound의 경우에는 표현하기에는 애매한 부분이 있어 Ο로만 나타냅니다.
{: .notice--info}

```c#
private int[] ExampleMethod(int[] input, int multiplier)
{
    int[] result = new int[input.Length];

    for (int index = 0; index < result.Length; index++)
        result[index] = input[index] * multiplier;

    return result;
}
```

이를 이용하여 위의 함수를 다시 살펴봅시다. 이는 상한선을 기준으로 `"아무리 오래 걸려도 N의 크기에 비례하게 걸린다."(Ο(N))`라고 표현할 수 있고, 하한선을 기준으로 `"아무리 빨라도 N의 크기에 비례하게 걸린다."(Ω(N))`라고 표현할 수 있습니다. 두 문장의 의미는 사실 같기 때문에 tight bound로
표현할 수 있으며, 결과적으로 <u>Θ(N)</u>으로 표기할 수 있습니다.

## 이진탐색의 시간복잡도
이번에는 `이진탐색binary search`의 시간복잡도를 구해봅시다.

```c#
private int ExampleMethod(int[] input, int target)
{
    ...
}
```

간단하게 작성한 위 함수는 정수형 배열 `input`과 정수 `target`을 받아, 찾고자 하는 값의 위치를 반환하는 기능을 수행합니다.
`이진탐색`이란 <u>'정렬된' 배열에서 찾으려는 값과 중간 값을 비교해가며 범위를 좁혀가는 값을 탐색</u>하는 방식입니다.

up & down 게임을 한다고 생각하면 됩니다.
{: .notice--info}

이를 경우에 따라 나눠보면

- worst (찾으려는 값이 배열에 없을 때)
    - 범위를 절반씩 줄여나가다보면 결국 크기는 1이 됩니다. 다시 생각해보면 몇 번만에 사이즈가 1이 되는지 알고 싶습니다.
    - 이를 수식으로 나타내면 N * (1/2)<sup>k</sup> = 1 이 됩니다. 이것을 k에 대한 식으로 나타내면 <u>k = log<sub>2</sub>N</u>이 됩니다.
    - 결과적으로 점근적 표기법은 <u>Θ(logN)</u>이 됩니다.

- best (한번에 찾았을 때)
    - 찾으려는 값이 배열의 중간 값과 같으면 그 경우는 best입니다.
    - 한번에 찾았기 때문에 점근적 표기법은 Θ(1)이 됩니다.

이를 정리하면 다음과 같습니다.

```c#
//          best            worst           avg
//  ------------------------------------------------
//  Ω       Ω(1)           Ω(logN)           
//  Ο       Ο(1)           Ο(logN)         Ο(logN)
//  Θ       Θ(1)           Θ(logN)           
```

## 시간복잡도의 속도 비교
Ο(1) < Ο(logN) < Ο(N) < Ο(N * logN) < Ο(N<sup>2</sup>) < Ο(2<sup>N</sup>) < Ο(N!)

오른쪽으로 갈수록 속도가 느려집니다.
{: .notice--info}

## Ο 표기법을 많이 쓰는 이유?
일반적으로 upper bound만 알아도 충분합니다. 예를 들어 친구와 약속을 잡는다고 생각해봅시다. 내가 아직 업무을 끝내지 못했고
해당 업무가 빠르면 1시간, 늦으면 2시간이 걸릴 것으로 예상된다면 보통은 2시간 이후 시간에 약속을 잡게됩니다.

이처럼 우리는 혹시 모를 상황에 대비하여 upper bound(상한선)를 기준으로 일을 계획하는 성향이 있습니다. 그렇기 때문에 점근적 표기법 역시
upper bound만 알고 있어도 무방한 경우가 많습니다.

Ω와 Θ는 키보드에 없기 때문에 Ο를 많이 사용한다는 우스갯소리도 있습니다.😅
{: .notice--info}