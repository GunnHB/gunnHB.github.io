---
title:  "[Accelerated C++] 004. 프로그램 및 데이터 구조화"
excerpt: "프로그램의 구조화를 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2024-02-06
last_modified_at: 2024-02-06
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
프로그램을 작성하다보면 의도치 않게 코드의 양이 많아집니다. 특히 라이브러리를 사용하지 않는다면
더욱 많아질 것입니다. 라이브러리의 기능들은 다음과 같은 몇 가지의 특징이 있습니다.

- 특정 문제를 해결
- 다른 기능들과 연계하지 않고 독립적
- 기능의 이름이 존재

작은 프로그램에서는 이러한 점들 때문에 문제가 생기지 않겠지만 크기가 커질수록 프로그램을 나누지 않으면
관리에 어려움이 생깁니다.

이번 포스팅에서는 프로그램을 구조화하여 개별적으로 컴파일하는 법을 알아봅시다.
</div>

## 연산 구조화
중간시험 점수(20%), 기말시험 점수(40%), 종합 과제 점수(40%)에서 최종 점수를 구하는 프로그램을 작성해봅시다.

프로그램에서 하나의 연산을 여러 지점에서 반복 사용한다면 해당 연산은 함수로 만드는 것을 고려합시다.
그렇게 하면 반복해서 작성하지 않아 코드의 양이 줄어들고 연산의 내용을 간단하게 수정할 수 있습니다.

이를 작성하면 다음과 같습니다.

```c++
double grade(double midterm, double final, double homework)
{
    return .2f * midterm + .4f + .4f * homework; 
}
```

함수는 반환 타입, 함수 이름, 소괄호로 묶인 `매개변수 목록parameter list`, 중괄호로 묶인 함수의 본문 순으로 작성합니다.

매개변수는 함수 내에서 지역 변수처럼 동작하기 때문에 함수를 호출할 때 생성되고 반환할 때 소멸됩니다.

다른 변수들처럼 매개변수 역시 사용하기 전에 정의해야 합니다. 하지만 함수를 호출할 때 변수가 만들어지기 때문에
초기화하는 데 사용하는 `인수argument`를 제공해야 합니다.

예를 들어 최종 점수를 구하는 코드를 확인해보면

```c++
cout << "Your final grade is " << setprecision(3)
    << 0.2 * midterm + 0.4 * final + 0.4 * sum / count
    << setprecision(prec) << endl;
```

이는 다음과 같이 표현할 수 있습니다.

```c++
cout << "Your final grade is " << setprecision(3)
    << grade(midterm, final, sum / count)
    << setprecision(prec) << endl;
```

호출하는 함수의 매개변수에 맞게 인수를 제공해야 할 뿐만 아니라 순서도 같아야 합니다.
{: .notice--warning}

인수는 sum / count처럼 변수가 아닌 표현식도 사용할 수 있습니다. 인수는 각각 대응하는 매개변수를 초기화하는 데 사용하기 때문에
매개변수들은 인수 값을 참조하는 것이 아닌 복사본으로 초기화됩니다. 이러한 동작을 `값에 의한 호출call by value`라고 합니다.

### median 함수
이제 벡터의 중앙값을 구하는 부분에 대해 살펴봅시다.

```c++
// vector<double>의 중앙 값을 구함
// 함수를 호출하면 인수로 제공된 벡터를 통째로 복사
double median(vector<double> vec)
{
    typedef vector<double>::size_type vec_sz;
    vec_sz = vec.size();

    if(size == 0)
        throw domain_error("median of an empty vecto");

    sort(vec.begin(), vec_end());
    vec_sz mid = size / 2;
    
    return size % 2 == 0 ? (vec[mid] + vec[mid - 1]) / 2 : vec[mid];
}
```

여기서 vector\<double>의 변수명 vec은 과제 점수의 중앙 값만이 아닌 모든 벡터의 중앙 값을 
구할 수 있기에 특정한 변수와 연관되지 않습니다.

코드를 보면 벡터가 비었을 때에 문제가 생겼음을 알립니다. 이는 누가 어떤 목적으로 프로그램을 사용하는지
모르므로 좀 더 일반적인 문제 알림 방식이 필요합니다. 즉, 벡터가 비었을 때 `예외exception를 던지는throw 것`입니다.

프로그램이 예외를 던지면 throw가 있는 부분에서 실행을 멈춘 후, 호출자에게 전달할 정보가 있는
`예외 객체exception object`와 함께 프로그램의 또 다른 부분으로 이동합니다.

여기서 던진 예외는 domain_error입니다. 이는 표준 라이브러리 \<stdexcept> 헤더에 정의한 형식이며,
함수의 인수가 함수가 받을 수 있는 값의 범위를 벗어났음을 보고할 때 사용합니다. 또한 던지려는 domain_error 객체를
생성할 때 무엇이 잘못되었는지 설명하는 문자열을 넣을 수 있습니다.

### 성정 산출 방식 다시 구현하기
종합 과제 점수를 구하는 부분을 함수 내부에 작성하면 다음과 같습니다.

```c++
// 중간시험 점수, 기말시험 점수, 과제 점수의 벡터로 학생의 종합 점수를 구함
// 이 함수는 인수를 복사하지 않고 median 함수가 해당 작업을 실행
double grade(double midterm, double final, const vector<double>& hw)
{
    if(hw.size() == 0)
        throw domain_error("student has done no homework");

    return grade(midterm. final, median(hw));
}
```

마지막 매개변수인 const vector\<double>&의 의미는 const double 벡터를 참조한다는 뜻입니다.

보통 어떤 변수나 이름이 객체를 참조한다는 것은 해당 이름이 객체의 또 다른 호칭이라는 의미이기도 합니다.

```c++
vector<double> homework;
vector<double>& hw = homework;  // hw는 homework의 동의어 
```

여기서 hw는 homework의 또 다른 이름입니다. hw로 실행하는 모든 작업은 homework와 같은 작업을 한다는 의미입니다.
반대일 때도 마찬가지입니다.

```c++
// chw는 조회만 가능한 homework의 동의어
const vector<double>& chw = homework;
```

만약 const로 정의된 상수의 경우 homework과 동의어인 것은 맞지만 chw로 값을 바꾸는 작업은 실행되지 않습니다.

매개변수가 const vector\<double>& 타입이라면 인수를 복사하지 않고 인수에 직접 접근할 수 있도록 구현체에 요청하고
매개변수의 값을 바꾸지 않습니다.

함수 내부를 보면 grade함수에서 다른 grade함수를 호출하고 있습니다. 두 함수는 같은 이름이지만 다른 매개변수를 필요로 하는데,
이를 `오버로딩overloading`이라고 합니다.
