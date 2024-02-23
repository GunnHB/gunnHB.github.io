---
title:  "[Accelerated C++] 005. 순차 컨테이너와 문자열 분석"
excerpt: "라이브러리 컨테이너를 다루는 방법을 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2024-02-23
last_modified_at: 2024-02-23
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
표준 라이브러리는 유용한 데이터 구조와 함수를 제공할 뿐만 아니라 일관된 아키텍쳐를 반영합니다.
따라서 한 종류의 컨테이너가 어떻게 동작하는지 학습했다면 모든 라이브러리 컨테이너를 다루는 방법을
이해할 수 있습니다.
</div>

## 학생 분류
이번에는 학생읯 최종 점수를 계산할 뿐 아니라 어떤 과목의 기준 점수를 통과하지 못한 학생들의 정보를 추출하여
다른 벡터에 저장하고 어떤 학생들이 과락 대상인지 알아봅시다.

우선 과락 여부를 판단하는 함수입니다.

```c++
// 학생의 과락 여부를 결정하는 서술 함수
bool fgrade(const Student_info& s)
{
    return grade(s) < 60;
}
```

최종 점수를 계산하는 데 grade 함수를 사용하고 과락 기준을 60점 미만으로 정했습니다.

이를 해결하는 방법은

1. 기준 점수를 통과한 학생과 그렇지 못한 학생의 벡터를 만듦
2. 각 학생의 정보를 조사하여 두 벡터 중 하나에 넣음

```c++
// 첫 번째 버전 : 기준 점수를 통과한 학생과 통과하지 못한 학생의 정보를 분류
vector<Student_info> extract_fails(vector<Student_info>& students)
{
    vector<Student_info> pass, fail;

    for(vector<Student_info>::size_type i = 0; i != students.size(); ++i)
    {
        if(fgrade(student[i]))
            fail.push_back(students[i]);
        else
            pass.push_back (students[i]);
    }

    students = pass;
    
    return fail;
}
```

함수는 두 개의 벡터를 만듭니다.

- 함수가 반환하는 과락 학생의 정보가 있는 벡터
- 함수를 호출할 때 부수적으로 생성하는 통과 학생들의 벡터

함수의 매개변수는 참조타입이므로 매개변수를 바꾸면 인수에 반영됩니다. 함수가 끝나면
인수로 전달된 벡터는 해당 과목의 기준 점수를 통과한 학생의 정보만 있습니다.

### 요소 제거
for문을 실행한 직후에는 pass 벡터와 fail 벡터를 만들었어도 원본 벡터는 여전히 남아있으므로
각 학생의 정보를 2개씩 보관하려면 충분한 메모리 공간이 필요합니다.

이를 해결하기 위해 pass를 없애고 fail 벡터만 남겨두어 students에서 과락 학생을 fail 벡터에 추가한 후
해당 학생의 정보를 제거하면 됩니다.

단, 벡터에서 요소를 제거하는 작업은 대용량의 입력 데이터를 처리할 때 속도가 느리다는 단점이 있습니다.

```c++
// 두 번째 버전 : 원하는 결과를 얻을 수 있지만 성능 저하가 우려됨
vector<Student_info> extract_fails(vector<Student_info>& students)
{
    vector<Student_info> fail;
    vector<Student_info>::size_type i = 0;

    // 불변성 : students 벡터의 [0, i) 범위에 있는 요소들은 과목을 통과한 학생들의 정보
    while (i != students.size())
    {
        if(fgrade(students[i]))
        {
            fail.push_back(students[i]);
        };
        students.erase(students.begin() + i);
    }
    else
        ++i;

    return fail;
}
```

위 함수에서는 어떤 과목의 기준 점수를 통과하지 못한 학생들의 정보를 복사할 fail 벡터를 만들어
students의 모든 요소를 확인하면서 과락 학생의 정보를 fail 벡터에 복사한 후 해당 정보는 students 벡터에서 삭제합니다.

벡터 타입은 벡터에서 요소를 제거하는 역할인 erase라는 멤버 함수가 있습니다. 함수의 인수는 제거하려는 요소를 가리킵니다.

또한 begin 함수는 벡터의 첫 번째 요소를 가리키는 것을 반환합니다.


erase 함수는 벡터의 크기를 바꿀 뿐만 아니라 인덱스가 i인 요소를 제거하여 i가 순차열의 다음 요소를 가리키게 만듭니다.
i 이후의 모든 요소는 하나 앞의 인덱스로 각각 복사됩니다. 따라서 i를 바꾸지 않아도 다음 요소를 가리키도록 인덱스를 조정하는
효과를 얻을 수 있으므로 i를 증가시켜 while문의 다음 반복 처리를 실행하면 안됩니다.

### 순차적 접근 vs 임의적 접근
앞서 살펴본 두 가지 버전의 extract_fail 함수는 컨테이너 요소에 순차적으로 접근합니다. 즉, 두 extract_fails 함수는
학생 각각의 정보를 순서대로 탐색하여 처리합니다.

컨테이너 요소에 접근하는 순서를 주목해야 하는 이유는 다음과 같습니다.

- 서로 다른 타입의 컨테이너는 성능도 서로 다르다.
- 서로 다른 연산을 지원한다.

프로그램이 사용하려는 어떤 연산이 특정 타입의 컨테이너에서만 효율적이라면 해당 컨테이너를 사용하여 프로그램의 성능을
더 향상시킬 수 있습니다.

다시 말해 extract_fails 함수는 단지 순차적 접근만 필요로 하므로 임의로 요소에 접근하는 기능이 있는 인덱스를
사용할 필요가 없습니다. 대신 순차적 접근만을 지원하는 연산으로 컨테이너의 요소에 제한적으로 접근하도록 extract_fail 함수를
다시 작성할 것입니다.

## 반복자
첫 번째 연산은 인덱스 i를 사용하여 Student_info 타입의 값을 가져오는 것입니다. i로 실행하는 연산은 i와
벡터의 크기를 비교할 때와 i를 증가시킬 때 뿐입니다.

```c++
while (i != students.size())
{
    // 여기까지 i 값은 변하지 않음
    i++;
}
```

유감스럽게도 라이브러리가 이러한 의도를 알 방법은 없습니다. 이럴 때 인덱스 대신 반복자를 사용하여
라이브러리가 상황을 알게 할 수 있습니다. `반복자iterator`는 다음 같은 역할을 합니다.

- 컨테이너와 컨테이너 내부 요소를 식별
- 해당 요소에 저장된 값을 검사
- 컨테이너 내부 요소 사이의 이동 연산을 제공
- 컨테이너를 효율적으로 사용하도록 연산을 제한

반복자는 인덱스와 비슷하게 동작하므로 인덱스를 사용하는 프로그램에서 반복자를 사용하도록 수정할 수 있습니다.

```c++
for (vector<Student_info>::size_type i = 0; i != students.size(); ++i)
    cout << students[i].name << endl;
```

반복자를 사용한다면 다음처럼 수정할 수 있습니다.

```c++
for (vector<Student_info>::const_iterator iter = students.begin();
    iter != students.end(); ++iter)
{
    cout << (*iter).name << endl;
}
```

### 반복자 타입
벡터와 같은 모든 표준 컨테이너는 2개의 반복자 타입을 정의합니다.

```c++
container-type::const_iterator
container-type::iterator
```

container-type 자리에는 vector\<Student_info>처럼 컨테이너 요소 타입을 포함하는 컨테이너 타입이 들어갑니다.
반복자를 사용하여 컨테이너에 저장된 값을 바꾸려면 iterator 타입을 사용하고, 값을 읽기만  하려면 const_iterator 타입을 사용합니다.

```c++
vector<Student_info>::const_iterator iter = students.begin();
```

해당 코드는 iter가 vector<Student_info>::const_iteractor 타입이라는 것을 알려줍니다. iter가 실제로 어떤 타입인지는 정확히
알 수 없고 알아햐 할 필요도 없습니다.

### 반복자 연산
변수 iter를 정의할 때는 students.begin()의 값을 할당합니다. 이는 컨테이너의 첫 번째 요소의 다음을 나타내는 값을 반환합니다.

반대로 end()는 마지막 요소의 다음을 나타내는 값을 반환합니다.
{: .notice--warning}

중요한 것은 begin함수와 end 함수가 컨테이너에 관한 반복자 타입 값을 반환한다는 것입니다.

```c++
iter != students.end()
```

앞에서 언급한 대로 end 함수는 컨테이너의 마지막 요소 이후를 나타내는 값을 반환합니다. iter와 students.end()의
반환 값이 같으면 for문 실행이 종료됩니다.

for문의 본문에서는 iter는 출력하려는 students 벡터 요소를 가리켜야 합니다. 따라서 해당 요소에 접근하려고
`역참조dereference 연산자 *`를 호출합니다.

'.' 연산은 '\*' 연산보다 우선순위가 높습니다. 만약 \*iter.name으로 작성하면 컴파일러는 \*(iter.name)으로
받아들여 처리합니다. 이는 iter 객체의 name 멤버를 가져와서 역참조 연산자를 적용하라는 의미입니다.
\*iter 객체의 name 멤버를 참조하려면 (\*iter).name으로 작성해야 합니다.
{: .notice--warning}

### 문법적 편의
앞서 살펴본 코드에서는 반복자를 역참조하여 반환된 값에서 구성 요소를 하나 가져왔습니다. 이러한 연산 조합은
일반적으로 사용하므로 약어로 표현할 수 있습니다.

```c++
(*iter).name
```

이는 다음과 같이 작성할 수 있습니다.

```c++
iter->name
```

이를 이용하여 코드를 수정하면 다음과 같습니다.

```c++
for (vector<Student_info>::const_iterator iter = students.begin();
    iter != students.end(); ++iter)
{
    cout << iter->name << endl;
}
```

### students.erase(students.begin() +i)
```c++
students.erase(students.begin() + i);
```

students.begin()의 반환 값이 students 벡터의 첫 번째 요소를 참조하는 반복자인 것과
students.begin() + i가 students 벡터의 i번째 요소를 참조하는 것임을 알 수 있습니다.

## 인덱스 대신 반복자 사용하기
반복자를 사용하면 인덱슬르 전혀 사용하지 않고 extract_fails 함수를 다시 구현할 수 있습니다.

```c++
// 세 번째 버전 : 인덱스 대신 반복자를 사용하지만 여전히 성능 저하가 우려됨
vector<Student_info> extract_fails(vector<Student_info>& students)
{
    // 과락 학생의 정보를 담을 벡터
    vector<Student_info> fail;
    // 인덱스 대신 students 벡터 요소를 탐색하는 데 사용할 반복자
    vector<Student_info>::iteractor iter = students.begin();

    while (iter != students.end())
    {
        if (fgrade(*iter))
        {
            fail.push_back(*iter);          // 요소를 얻기위해 반복자를 역참조
            iter = students.erase(iter);
        }
        else
            ++iter;
    }

    return fail;
}
```

## 더 나은 성능을 위한 데이터 구조
지금까지의 프로그램은 입력 데이터 양이 적을 때라면 제대로 동작합니다. 하지만 양이 많아지면 성능 저하의 문제가 발생합니다.

erase함수는 임의 요소를 빠르게 접근하도록 벡터의 데이터 구조를 정의합니다. 이는 새로운 요소를 한 번에 하나씩
벡터 끝에 추가하여 벡터의 크기가 증가하게 됩니다.

이러한 성능의 한계를 극복하려면 컨테이너의 어느 곳에서나 효율적으로 요소를 삽입 또는 제거할 수 있는 데이터 구조가 필요합니다.

## 리스트 타입
이제 컨테이너 내부에서 효율적으로 요소를 제거하는 데이터 구조를 사용하도록 프로그램을 다시 만들어봅시다.

데이터 구조 내부에서 요소 삽입 / 제거는 매우 자연스업게 일어날 수 있는 작업입니다. 라이브러리는 이러한 종류의 접근에
최적화된 리스트list라는 타입을 제공합니다.

벡터가 빠른 임의적 접근에 최적화된 것처럼 리스트는 컨테이너의 내부 어디에서나 빠르게 데이터를 삽입하고 제거하는 데
최적화되어 있습니다.

리스트가 더 복잡한 구조이므로 컨테이너가 순차적 접근만 지원할 때는 리스트가 벡터보다 느립니다.
컨테이너의 중간에서 많은 요소를 제거할 때는 리스트가 더 성능이 우수합니다.
{: .notice--warning}

```c++
// 네 번째 버전 : 벡터 대신 리스트를 사용
list<Student_info> extract_fails(list<Student_info>& students)
{
    list<Student_info> fail;
    list<Student_info>::iteractor iter = students.begin();

    while (iter != students.end())
    {
        if(fgrade(*iter))
        {
            fail.push_back(*iter);
            iter = students.erase(iter);
        }
        else
            ++iter;
    }

    return fail;
}
```

프로그램의 논리적인 구조는 같습니다. 다만 함수를 호출할 때 리스트를 전달해야 하고
함수는 리스트를 반환하게 되는 차이점이 있습니다.

### 리스트 vs 벡터
리스트와 벡터의 주요 차이점 중 하나는 반복자를 사용하는 연산입니다. 예를 들어 erase함수를 사용하여
벡터에서 요소 하나를 제거하면 제거한 요소와 순서상 이어지는 요소들을 참조하는 모든 반복자를 무효로 만듭니다.

반면 리스트에서 erase함수나 push_back함수를 사용할 때는 다른 요소를 참조하는 반복자를 무효로 만들지 않습니다.

### 리스트를 사용하는 이유
과락 학생들의 정보를 추출하는 코드는 선택 가능한 데이터 구조 사이의 성능을 비교하는 좋은 예입니다.

입력 데이터의 양이 적다면 리스트는 벡터보다 느립니다. 입력 데이터의 양이 많다면 벡터를 사용하는 프로그램은
리스트를 사용하는 프로그램보다 훨씬 더 느리게 실행될 수 있습니다.

## 문자열 분할
문자열을 특별한 종류의 컨테이너로도 생각할 수 있습니다. 문자열은 문자로만 구성되어 있고 전부는 아니지만 컨테이너
연산 몇 가지를 지원합니다.

예를 들어 문자열 하나를 공백으로 구분하여 각각의 단어로 분리할 수 있습니다. 이를 통해 입력 연산자는 공백까지
문자로 취급한다는 것을 알 수 있습니다.

문자열을 분리하는 동작은 자주 사용될 수 있으므로 함수로 만들 필요가 있습니다.

```c++
vector<string> split(const string& s)
{
    vector<string> ret;
    typedef string::size_type string_size;
    string_size i = 0;

    // 불변성 : 지금까지 [원래의 i, 현재의 i) 범위에 있는 문자들을 처리
    while (i != s.size())
    {
        // 선행하는 공백들을 무시
        // 불볌성 : [원래의 i, 현재의 i) 범위에 있는 문자들은 모두 공백
        while (i != s.size() && isspace(s[i]))
            ++i;
            
        // 순서상 다음 단어의 끝을 찾음
        string_size j = i;

        // 불변성 : [원래의 j, 현재의 j) 범위에 있는 문자들은 공백이 아님
        while (j != s.size() && !isspace(s[j]))
            j++;
    
        // 공백이 아닌 문자들을 찾았을 때
        if (i != j)
        {
            // i에서부터 j - i개의 문자들을 s에 복사
            ret.push_back(s.substr(i, j - i));
            i = j;
        }
    }

    return ret;
}
```