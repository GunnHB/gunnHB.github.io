---
title:  "[Accelerated C++] 006. 라이브러리 알고리즘"
excerpt: "라이브러리에서 제공해주는 알고리즘에 대해 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2024-03-01
last_modified_at: 2024-03-01
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
이전 포스팅에서는 다양한 타입의 컨테이너에 적용할 수 있는 함수들이 있는 것을 확인했습니다.
insert, erase 등의 함수들의 인터페이스는 이들을 지원하는 모든 타입에서 가능합니다.

이번 포스팅에서는 라이브러리가 표준 알고리즘을 제공할 때 공통의 인터페이스를 어떻게 활용하는지
살펴봅시다.
</div>

## 문자열 분석
``` c++
for (vector<string>::const_iterator it = bottom.begin(); it != bottom.end(); ++it)
    ret.push_back(*it);
```

해당 코드는 문자를 결합하는 반복문입니다. 이는 ret 벡터 끝에 bottom 벡터에 있는 요소들의 복사본을
삽입하는 것을 의미합니다.

```c++
ret.insert(ret.end(), bottom.begin(), bottom.end());
```

이는 다음의 코드처럼 컨테이너의 끝에 요소들을 삽입하는 개념과 요소들을 복사하는 개념을 따로 구분할 수 있습니다.

```c++
copy(bottom.begin(), bottom.end(), back_inserter(ret));
```

여기서 copy 함수는 제네릭 알고리즘이고 back_inserter 함수는 반복자 어댑터입니다.

`제네릭 알고리즘generic algorithm`이란 특정 종류의 컨테이너에 속하지 않으며, 인수의 타입으로 데이터에 접근하는
방법을 결정하는 알고리즘입니다. 표준 라이브러리의 제네릭 알고리즘은 일반적으로 컨테이너 요소를 조작하는데 사용하는 반복자가 인수로 있습니다.
copy 알고리즘은 [begin, end) 범위의 모든 요소를 out에서 시작하는 요소들의 순차열에 필요한 공간을 확장하여 복사합니다.

```c++
copy(begin, end, out);
```

위 코드는 while문에서 반복자의 값을 바꾼다는 점을 제외하면 다음 코드와 같은 의미입니다.

```c++
while (begin != end)
    *out++ = *begin++;
```

`후위postfix` 표기를 주목합시다. 이는 begin 값을 반환한 후 begin을 증가시킵니다.

증가 연산자는 * 연산자와 우선순위가 같고 둘 다 오른쪽 우선 결합성이 있으므로 *out++는
*(out++)와 같습니다.

```c++
{
    *out = *begin;
    ++out;
    ++begin++;
}
```

`반복자 어댑터iterator adaptor`를 알아봅시다. 반복자 어댑터는 인수와 관련된 속성이 있는 반복자를 반환하는 함수입니다.
일반적으로 사용하는 back_inserter 함수는 컨테이너를 인수로 사용하며 컨테이너에 값들을 추가할 목적지인 반복자를 반환합니다.

```c++
copy(bottom.begin(), bottom.end(), back_inserter(ret));
```

위 함수는 실행이 끝나면 ret의 크기가 bottom.size()만큼 증가합니다.

copy함수는 요소 복사와 컨테이너 확장을 개념적으로 분리하면 프로그래머는 자신에게 가장 알맞은 연산을 선택할 수 있습니다.
{: .notice--warning}

### 새로운 split 함수
```c++
// 인수가 공백일 때는 true, 그렇지 않을 때는 false를 반환
bool space(char c)
{
    return isspace(c);
}

// 인수가 공백일 때는 false, 그렇지 않을 때는 true를 반환
bool not_space(char c)
{
    return !isspace(c);
}

vector<string> split(cont stirng& str)
{
    typedef string::const_iterator iter;
    vector<string> ret;

    iter i = str.begin();

    while (i != str.end())
    {
        // 선행 공백을 무시
        i = find_if(i, str.end(), not_space);

        // 다음 단어의 끝을 찾음
        iter j = find_if(i, str.end(), space);

        // [i, j] 범위의 문자를 복사
        if (i != str.end())
            ret.push_back(string(i, j));

        i = j;
    }

    return ret;
}
```

위 코드에서 i와 j는 인덱스가 아닌 반복자입니다. typedef를 사용하여 반복자의 타입을 string::const_iterator에서 iter로
간략하게 줄입니다.

여기에서 사용한 알고리즘은 find_if 함수입니다. find_if 함수의 첫 두 인수는 순차열을 나타내는 반복자입니다. 세 번째 인수는 서술 함수며
인수를 검사하여 참 또는 거짓을 반환합니다. find_if 함수는 순차열의 각 요소에 서술 함수를 호출하다가 참을 반환하는 요소를 찾으면 동작을 멈춥니다.

표준 라이브러리는 문자가 공백인지 여부를 검사하는 isspace 함수를 제공합니다. 함수는 오버로드되었으므로 한국어를 나타내는 wchar_t 타입 같은
다른 문자 타입으로 동작할 수도 있습니다. 이처럼 오버로드된 함수를 템플릿 하수의 인수로 직접 전달하는 것은 원하는 동작을 한다고 보장할 수 없습니다.
find_if 함수는 컴파일러가 함수 형태를 선택하는데 사용하는 인수를 제공하지 않습니다. 따라서 사용하려는 isspace 함수 형태를 컴파일러에 명확하게 전달하려면
space와 not_space라는 서술 함수를 직접 작성해야 합니다.

find_if 함수는 첫 호출에서 단어를 시작하는 공백이 아닌 첫 번재 문자를 찾습니다. 첫 호출이 끝나면 i는 str에서 공백이 아닌 첫 번째 문자를 가리킵니다.
이어지는 호출에서는 [i, str.end()) 범위 안에서 첫 번재 공백을 찾는 데 i를 사용합니다. 함수가 만족하는 값을 찾지 못하면 find_if 함수의 두 번째 인수인
str.end()를 반환합니다. 따라서 j는 현재 단어와 다음 단어를 구분하는 첫 번째 공백을 가리키도록 초기화됩니다.

