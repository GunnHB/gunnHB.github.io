---
title:  "[Accelerated C++] 007. 연관 컨테이너"
excerpt: "효율적인 프로그램을 위한 방법을 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2024-03-16
last_modified_at: 2024-03-16
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
이전 포스팅들에서 살펴봤던 순차 컨테이너는 요소가 많을수록 특정 요소를 찾을 때
느려진다는 단점이 있습니다. 물론 속도를 높일 수 있는 알고리즘을 작성할 수 있지만
쉽지 않은 것이 사실입니다.

이번 포스팅에서는 이를 해결하기 위해 라이브러리에서 제공하는 효율적인 방법에 대해 알아봅시다.
</div>

## 효율적인 탐색을 위한 컨테이너
데이터를 저장하는 데 순차 컨테이너 대신 `연관 컨테이너associative container`를 이용해봅시다. 연관 컨테이너는 삽입한 순서가 아닌
값 순서로 순차열을 자동으로 정렬하기 때문에 직접 컨테이너를 조작할 필요없이 요소를 찾을 수 있습니다.

효율적인 탐색에 사용할 수 있는 각 요소의 일부분을 `키key`라고 합니다. 만약 특정 학생을 검색한다면 학생의 이름을 키로 사용하여
쉽게 찾을 수 있습니다.

연관 데이터 구조의 가장 일반적인 형태는 키-값 쌍들을 저장하고 각 키와 값을 연관시킨 후, 키를기반으로 요소들을 빠르게 삽입하고 탐색할 수 있는 구조입니다.
이를 데이터 구조에 넣으면 키-값 쌍을 제거할 때까지 해당 키는 같은 값과 연관되어 있습니다. 이러한 데이터 구조를 `연관 배열accosiative array`이라고 합니다.

c++에서 사용하는 가장 일반적인 연관 배열은 `맵map`이며 \<map> 헤더에 정의됩니다.
{: .notice--warning}

연관 컨테이너는 자동으로 요소들을 정렬하므로 작성한 프로그램에서 요소 순서를 바꾸는 어떠한 동작도 실행하면 안됩니다. 때문에
컨테이너 내용을 변경하는 알고리즘은 종종 연관 컨테이너에서 동작하지 않습니다.

## 단어의 빈도
간단한 예로 단어를 입력받으면서 개별 단어의 빈도를 구하는 방법을 생각해봅시다.

```c++
int main()
{
    string s;

    // 각 단어와 빈도를 저장하는 맵
    // 입력을 읽으면서 각 단어와 빈도를 저장
    map<string, int> counters;

    while (cin >> s)
        ++counters[s];

    // 단어와 빈도를 출력
    for (map<string, int>::const_iterator it = counters.begin();
        it != counters.end(); ++i)
    {
        cout << it->first << "\t" << it->second << endl;   
    }

    return 0;
}
```

다른 컨데이너와 마찬가지로 맵에 있는 객체 타입을 지정해야 합니다. 맵은 키-값 쌍이 있으므로 다음처럼 값의 타입뿐만
키의 타입도 명시해야 합니다.

```c++
map<strin, int> counters;
```

위 코드는 문자열 타입의 키와 int 타입의 값이 있는 맵인 counters를 정의합니다. 문자열 타입의 키를 주고 int 타입의 연관된
데이터를 받는 방식으로 맵을 사용할 수 있습니다.

흔히 이러한 컨테이너를 '문자열 타입에서 int 타입으로의 맵'이라고 합니다.
{: .notice--warning}

이후 반복문에서는 표준 입력을 이용해 한 번에 한 단어 씩 s로 읽어 들입니다.

```c++
++counters[s];
```

여기서는 직전에 읽어 들인 단어를 키로 사용해 counters에 접근합니다. counters\[s]는 s에 저장한 문자열과 연관된 정숫값입니다.
그리고 ++ 연산자를 사용해 정숫값을 증가시킵니다. 이는 단어가 한번 더 등장한 것을 나타냅니다.

만약 직전에 읽어 들인 단어가 처음 등장한 단어라면 counters에는 해당 단어를 키로 갖는 요소가 아직 존재하지 않는 상태입니다.
아직 존재하지 않는 키로 맵에 접근할 때 멥은 해당 단어를 키로 갖는 새로운 요소를 자동으로 만듭니다. 이를 `값 초기화vlaue-initialize`되었다고 합니다.

입력을 모두 읽었다면 단어와 해당 단어의 빈도를 출력해야 합니다. 이는 리스트나 벡터의 내용을 출력하는 것과 흡사합니다. 맵 클래스에 정의된 반복자 타입의
변수를 사용해 for문에서 컨테이너를 탐색합니다. 리스트나 벡터와의 한 가지 차이점은 for문의 본문에서 데이터를 출력하는 방법입니다.

```c++
cout << it->first << "\t" << it->second << endl;
```

맵을 탐색하면서 해당 키에 연관된 값을 모두 얻는 방법이 있어야 합니다. 
이와 같이 2개의 서로 다른 타입을 저장하려고 맵 컨테이너는 라이브러리 타입인 pair를 제공합니다.

pair 타입은 2개의 요소로 first와 second가 있는 간단한 데이터 구조입니다. 각 요소는 키를 포함하는 멤버 변수 first와 해당 키와 연관된 
값을 포함하는 멤버 변수 second가 있습니다. pair 타입의 값은 맵 반복자를 역참조하여 얻습니다.

pair 클래스는 다양한 타입의 값을 포함할 수 있습니다. 즉, pair 클래스에 있는 값의 타입이 정해져 있지 않으므로 pair 타입의 객체를 만들 때
데이터 멤버 first와 second의 타입을 지정해야 합니다.

## 상호 참조 테이블
이번 단에서는 입력한 문자열에서 각 단어의 위치를 나타내는 상호 참조 테이블을 만들어봅시다.

우선 한 번에 한 단어씩 읽어 들이는 대신 한 번에 한 행씩 읽어 들여서 행 번호와 단어를 연관시킵니다. 그다음으로 각 행을 단어들로
나눌 방법이 필요합니다. 이는 split 함수를 사용하면 됩니다.

```c++
map<string, vectot<int>> xref(istream& in, vector<string> find_words(const string&) = split)
{
    string line;
    int line_number = 0;
    map<string, vector<int> > ret;

    while (getlin(in, line))
    {
        ++line_number;

        // 입력한 행을 단어로 나눔
        vector<string> words = find_words(line);

        // 현재 행에 등장한 모든 단어를 저장
        for (vector<string>::const_iterator it = words.begin(); it != words.end(); ++it)
            ret[*it].push_back(line_number);
    }

    return ret;
}
```

map<string, vector\<int> > ret; 처럼 공백을 넣어야합니다. >>으로 작성하면 컴파일러는 >> 연산자로 인식합니다.
{: .notice--warning}

```c++
xref(cin);                  // split 함수를 사용하여 입력 스트림에서 단어를 찾음
xref(cin, find_urls);       // find_urls 함수를 사용하여 단어를 찾음
```

먼저 입력물의 각 행을 저장할 문자열 타입의 변수 line과 현재 처리 중인 행의 범호를 저장할 int 타입 변수 line_number를 정의합시다.
입력 작업은 반복문으로 실행하며 getline 함수를 호출하여 한 번에 한 행씩 읽어 line에 저장합니다.

line에 저장한 행에서 추출한 모든 단어를 저장할 때는 지역 변수 words를 선언하고 find_words 함수를 호출해 초기화합니다.
find_words 함수는 행을 각 단어로 나누는 split 함수거나 문자열 인수를 전달받아 그 결과로 vector\<string>를 반환하는
또 다른 함수입니다. 이어서 for문으로 words 벡터의 각 요소를 탐색하면서 ret의 내용을 갱신합니다.

```c++
int main()
{
    // 기본 인수인 split 함수를 사용하여 xref 함수를 호출
    map<string, vector<int> > ret = xref(cin);

    // 결과를 출력
    for (map<string, vecotr<int> >::const_iterator it = ret.begin(); it != ret.end(); ++it)
    {
        // 단어를 출력
        cout << it->first << " occurs on line(s): ";

        // 이어서 하나 이상의 행 번호를 출력
        vector<int>::const_iterator line_it = it->second.begin();

        // 해당 단어가 등장한 첫 번째 행 번호를 출력
        cout << *line_it;
        ++line_it;

        // 행 번호가 더 있으면 마저 출력
        while (line_it != it->second.end())
        {
            cout << ", " << *line_it;
            ++line_it;
        }

        // 각 단어를 다음 단어와 구분하려고 새로운 행을 출력
        cout << endl;
    }

    return 0;
}
```