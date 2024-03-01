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

### 회문
문자를 다루는 문제 중에 라이브러리를 사용하여 간단히 해결할 수 있는 또 다른 예는 단어가 회문인지 판단하는 것입니다.

회문은 거꾸로 읽어도 철자가 같은 단어입니다.
{: .notice--warning}

라이브러리를 사용한 간단한 해결책은 다음과 같습니다.

```c++
bool is_palindrome(const string& s)
{
    return equal(s.begin(), s.end(), s.rbegin());
}
```

rbegin 함수가 반환하는 반복자는 컨테이너의 마지막 요소에서 시작하여 컨테이너를 거꾸로 통과합니다.

eqaul 함수는 순서가 반대인 두 문자열이 같은 값인지 판별하려고 두 순차열을 비교합니다. 인수로 전달하는 첫 2개의 반복자는 첫 번째 순차열의 범위를 나타내고,
세 번째 인수는 두 번째 순차열의 시작점을 나타냅니다. eqaul 함수는 두 번째 순차열이 첫 번재 순차열과 같은 크기라고 가정합니다.

equal 함수는 s의 첫 번째 문자와 마지막 문자를 비교한 후, 두 번째 문자와 마지막에서 두 번째 문자를 비교하는 순서로 비교하는 순서입니다.

### URL 찾기
문서 전체 내용을 담은 하나의 문자열을 만들어 함수를 사용할 수 있습니다. 이 함수는 문서를 훑고 그 안에 있는 모든 URL을 찾습니다. URL은 다음의 형태를 갖는
문자들의 순차열입니다.

```
프로토콜 이름 :// 리소스 이름
```

- 프로토콜 이름
    - 문자로만 구성
- 리소스 이름
    - 문자, 숫자, 특정 구두 문자

인수로 전달된 문자열에 있는 모든 URL을 찾아야 합니다. 따라서 함수는 URL 각각을 요소 하나로 갖는 vector<string>을 반환할 것입니다.

함수에서는 반복자 b를 사용하여 인수로 전달한 문자열의 내부를 이동하며 URL의 일부인 ://에 해당하는 문자들을 찾습니다.
문자들을 발견하면 '프로토콜 이름'을 찾기 위해 ://의 앞쪽을 확인하고 '리소스 이름'을 찾기 위해 ://의 뒤쪽을 확인합니다.

```c++
vector<string> find_urls(const string& s)
{
    vector<string> ret;                             // 찾을 URL을 저장할 ret 벡터
    typedef strin:const_iterator iter;              // URL에 해당하는 문자열의 범위를 나타내는 반복자

    iter b = s.begin(), e = s.end();                // URL의 시작과 끝을 찾기 위한 함수

    // 인수로 전달받은 문자열 전체를 탐색
    while (b != e)
    {
        // ://의 앞쪽에서 하나 이상의 문자를 탐색
        b = url_beg(b, e);

        if (b != e)
        {
            // 해당 문자를 찾았다면 URL의 나머지 부분을 탐색
            iter after = url_end(b, e);

            // URL을 저장
            ret.push_back(string(b, after));

            // 인수로 전달받은 문자열에서 또 다른 URL을 찾기 위해 b를 증가
            b = after;
        }
    }

    return ret;
}
```

코드의 url_beg 함수는 유효한 URL이 있는지 식별하여 유효한 URL이면 프로토콜 이름의 첫 번째 문자를 참조하는 반복자를 반환합니다.

URL을 식별하지 못했으면 두 번째 인수인 e를 반환합니다.
{: .notice--warning}

url_end 함수는 인수로 주어진 지점부터 find_urls의 인수로 전달된 문자열의 끝이나 URL의 일부가 아닌 문자에 도달할 때까지 URL을 찾아서 URL의
마지막 문자 이후를 참조하는 반복자를 반환합니다.

이후 b를 방금 찾은 URL의 마지막 문자 이후로 할당하고 find_urls의 인수로 전달된 문자열을 모두 살펴볼 때까지 while문을 실행합니다.

```c++
string::const_iterator url_end(string::const_iterator b, string::const_iterator e)
{
    return find_if(b, e, not_url_char);
}
```

상대적으로 더 간단한 url_end 함수를 먼저 살펴봅시다.

이 함수는 앞에서 확인한 find_if 함수로 실행할 동작을 전달합니다.
not_url_char 함수는 인수로 전달된 문자가 url에 속할 수 없는 문자일 때 참을 반환합니다.

```c++
bool not_url_char(char c)
{
    // URL에 들어갈 수 없는 알파벳과 숫자 이외의 문자
    static const string url_ch = "~;?:@&$-_+~*(),";

    // c가 URL에 들어갈 수 있는 문자인지 확인하면 음수를 반환
    return !(isalnum(c) || find(url_ch.begin(), url_ch.end(), c) != url_ch.end());
}
```

먼저 살펴볼 것은 `스토리지 클래스 지정자storage class specifier`인 static입니다. static으로 선언한 지역 변수는 함수를 반복해서 호출하더라도
값이 보존됩니다. 따라서 not_url_char 함수의 첫 번째 호출에서만 문자열 url_ch를 만들고 초기화합니다.

url_ch는 const 문자열이므로 초기화하면 값을 바꿀 수 없습니다.
{: .notice-warning}

다음으로 살펴볼 것은 isalnum입니다. 이는 /<cctype> 헤더가 정의하며 인수로 전달한 문자가 알파벳이나 숫자인지 여부를 판별합니다.

마지막으로 find 함수입니다. 이는 find_if 와 비슷하지만 서술 함수를 호출하는 대신 세 번째 인수로 주어진 특정 값을 찾습니다.
find_if와 마찬가지로 찾으려는 값이 존재하면 주어진 순차열에서 해당 값의 첫 번째 출현 지점을 가리키는 반복자를 반환합니다.

다음 url_beg 함수를 구현해봅시다.

```c++
string::const_iterator url_beg(string::const_iterator b, string::const_iterator e)
{
    static const string sep = "://";
    typedef string::const_iterator iter;

    // i는 구분 기호를 발견한 위치를 표시
    iter i = b;

    while((i = search(i, e, sep.begin(), sep.end())) != e)
    {
        // 구분 기호가 현재 탐색 범위의 처음 또는 마지막에 있는지 확인
        if (i ! = b && i + sep.size() != e)
        {
            // beg는 프로토콜 이름의 시작 위치를 표시
            iter beg = i;

            while (beg != b && isalpha(beg[-1]))
                --beg;

            // 구분 기호 앞뒤로 URL의 일부에서 유효한 문자가 하나라도 있는지 확인
            if (beg != i && !not_url_char(i[sep.size()]))
                return beg;
        }

        // 발견한 구분 기호는 URL 일부가 아니므로 해당 구분 기호 이후를 표시하도록 i를 증가시킴
        i += sep.size();
    }

    return e;
}
```

url_beg 함수는 탐색할 범위를 나타내는 2개의 반복자를 인수로 두며 해당 범위 안에서 찾은 첫 번째 URL의 시작점을 나타내는 반복자를 반환합니다.
반복자 i는 URL 구분 기호의 시작점을 가리키며 반복자 beg는 프로토콜 이름의 시작점을 기리킵니다.

내부의 while문에서는 알파벳이 아닌 문자나 현재 탐색 범위의 처음에 도달할 때까지 반복자 beg를 거꾸로 이동시킵니다.

- 컨테이너가 인덱스를 지원하면 반복자도 인덱스를 지원
- /<cctype> 헤더가 정의하는 isalpha 함수

코드는 위 두 가지의 개념을 사용합니다. isalpha 함수는 전달된 인수가 문자인지 판별합니다.

## 성적 산출 방식 비교
앞선 성적 산출 프로그램에서는 일부 학생들이 고의적으로 과제를 제출하지 않고 채점 방식을 악용할 수 있습니다. 과제 점수의 절반 가까이는
최종 점수에 아무런 영향을 주지 않으므로 점수를 잘 받을 만큼의 과제 점수를 확보했다면 더 이상 과제를 할 필요가 없습니다.

이를 해결하기 위해

- 중앙값 대신 평균값을 사용
    - 학생이 제출하지 않은 과제는 0점으로 처리
- 학생이 실제로 제출한 과제 점수들만으로 중앙값을 추출

1. 모든 학생의 정보를 읽고 과제를 모두 제출한 학생과 그렇지 않은 학생을 분류
2. 두 집단에 각각의 성적 산출 방식을 적용한 후 각 집단의 중앙값을 추출

### 학생 분류
```c++
bool did_all_hw(const Student_info& s)
{
    return ((find(s.homework.begin(), s.homework.end(), 0)) == s.homework.end());
}
```

먼저 학생 정보를 읽고 분류하는 것이 필요합니다.

did_all_hw 함수는 s.homework 벡터에 저장된 값 중 0이 있는지 확인합니다. 제출한 과제는 최소한의 부분 점수를 부여하므로 0점은 과제를
제출하지 않는 것을 의미합니다.

```c++
vector<Student_info> did, didnt;
Student_info student;

// 학생 정보를 읽고 과제를 모두 제출한 학생과 그렇지 않은 학생을 분류
while (read(cin, student))
{
    if (did_all_hw(student))
        did.push_back(student);
    else
        didnt.push_back(student);
}

// 두 집단에 데이터가 있는지 각각 확인
if (did.empty())
{
    cout << "No Student did all the homework!" << endl;
    return 1;
}

if(didnt.empty())
{
    cout << "Every student did all the homework!" << endl;
    return 1;
}
```

empty 함수는 해당 컨테이너가 비어 있으면 참을 반환하고 그렇지 않으면 거짓을 반환합니다.

컨테이너가 비어 있는지 확인하려면 size 함수의 반환 값이 0인 것을 확인하는 것보다 empty 함수를 사용하는 것이 낫습니다.
{: .notice--warning}

### 과제 점수의 평균값을 사용한 성적 산출
과제 점수의 중앙값이 아닌 평균값을 사용하여 확생들의 최종 점수를 계산하는 average_analysis 함수를 작성해봅시다. 최종 점수를
구하는 계산에서 중앙값 대신 평균값을 사용하료고 벡터의 평균값을 계산하는 함수를 작성하는 것입니다.

```c++
double average(const vector<double>& v)
{
    return accumulate(v.begin(), v.end(), 0, 0) / v.size();
}
```

accumulate 함수는 지금까지 살펴본 다른 라이브러리 알고리즘과는 달리 /<numeric> 헤더에 선언되어 있습니다.

/<numeric> 헤더는 숫자 계산에 필요한 도구를 제공합니다.
{: .notice--warning}

accumulate 함수는 첫 2개의 인수 범위 값들을 더하여, 세 번째 인수로 주어진 값을 합하는 과정을 시작합니다.

총합 타입은 세 번째 인수 타입으로 결정되므로 0이 아닌 0.0을 사용하는 것이 매우 중요합니다. 세 번째 인수로 0을 사용한다면
결과는 int 타입이 되어 소수 부분이 손실됩니다.

이를 이용하여 average_grade 함수를 구현해봅시다.

```c++
double average_grade(const Student_info& s)
{
    return grade(s.midterm, s.final, average(s.homework));
}
```

함수는 average 함수를 사용하여 종합 과제 점수를 계산한 다음, grade 함수로 전달합니다.

이러한 구조를 따르면 average_analysis 함수는 다음과 같이 작성할 수 있습니다.

```c++
double average_analysis(const vector<Student_info>& students)
{
    vector<double> grades;
    transfform(student.begin(), student.end(), beck_inserter(grades), average_grade);
    
    return median(grades);
}
```

### 제출한 과제 점수의 중앙값
마지막으로 살펴볼 것은 optimistic_median_analysis 함수입니다. 이 함수는 제출하지 않는 과제 점수가 제출한 과제 점수와 같았을 것이라는 낙관적 가정에서
이름을 딴 함수입니다. 이러한 가정 아래 각 학생이 제출한 과제 점수의 중앙값을 계산하고자 합니다.

```c++
// s.homework 벡터에서 0이 아닌 요소들의 중앙값을 구합니다.
// 0이 아닌 요소가 없다면 종합 과제 점수를 0점으로 처리
double optimistic_median(const Student_info& s)
{
    vector<double> nonzero;
    remove_copy(s.homework.begin(), s.hommework.end(), back_inserter(nonzero), 0);

    if (nonzero.empty())
        return grade(s.midterm, s.final, 0);
    else
        return grade(s.midterm, s.final, median(nonzero));
}
```

함수는 homework 벡터에서 0이 아닌 요소들을 추출해 새로 만든 nonzero 벡터에 저장합니다. 0이 아닌 과제 점수가 존재한다면
grade 함수를 호출해 실제로 제출한 과제 점수의 중앙값으로 학생의 최종 점수를 계산합니다.

## 학생 분류 다시 살펴보기
이전 포스팅에서 다루어야할 학생의 수가 많아질수록 성능의 저하가 나타나기 때문에 리스트를 사용하는 방식을 알아봤습니다.

알고리즘 라이브러리를 사용하면 두 가지의 다른 방식으로 문제를 해결할 수 있습니다.

- 한 쌍의 라이브러리 알고리즘을 사용하여 모든 요소를 두 번씩 탐색
- 특성화된 라이브러리 알고리즘으로 모든 요소를 한 번씩 탐색

### 벡터를 두 번 탐색하는 방식
```c++
vector<Student_info> extract_fails(vector<Student_info>& students)
{
    vector<Student_info> fail;
    remove_copy_if(students.begin(), students.end(), back_inserter(fail), pgrade);
    students.erase(remove_if(students.begin()), students.end(), fgrade, students.end());

    return fail;
}
```