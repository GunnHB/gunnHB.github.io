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
last_modified_at: 2024-02-11
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

### 성적 산출 방식 다시 구현하기
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

### read_hw 함수
벡터에 과제 점수를 넣는 부분을 함수로 만든다고 생각해보면

- 입력된 과제 점수
- 입력이 제대로 일어났는지 나타내는 값

위 2개의 값을 한번에 반환해야 합니다.

두 개 이상의 값을 반환하는 방법은 없습니다. 간접적인 방법으로 반환하려는 값 2개 중 하나를 담은 객체의 참조를 함수의
매개변수로 지정하는 것이 있습니다.

```c++
// 입력 스트림에서 과제 점수를 읽어서 vector<double>에 넣음
istream& read_hw(istream& in, vector<double>& hw)
{
    return in;
}
```

const가 없는 매개변수는 보통 함수의 인수로 사용하는 객체를 수정하겠다는 의미입니다.

```c++
vector<double> homework;
read_hw(cin, homework)
```

read_hw 함수의 두 번째 매개변수가 참조 타입일 때는 read_hw 함수를 호출하면 homework의 값이 변경될 수 있음을 예상해야 합니다.

함수에서 인수를 바꿀 것이 예상되므로 표현식 형태의 인수로 함수를 호출할 수는 없습니다.
대신 1value 인수를 참조 매개변수로 전달합니다. 1value라는 것은 `비일시적nontemporary 객체`를 나타내는 값입니다.

read_hw 함수는 참조 타입인 in을 반환합니다. 결론적으로 객체를 복사하지 않고 받은 후, 복사하지 않은 채로 반환할 것입니다.
입력 스트림을 반환하므로 다음처럼 코드를 작성할 수 있습니다.

```c++
if(read_hw(cin, homework))
{ /* 생략 */ }
```

이는 다음과 같은 의미입니다.

```c++
read_hw(cin, homework);
if(cin)
{ /* 생략 */ }
```

다음은 모든 과제 점수를 입력받아야 하므로 다음 형태의 코드를 작성합니다.

```c++
// 아직 완성된 코드가 아님
double x;

while(in >> x)
    hw.push_back(x);
```

해당 코드는 정상적으로 동작하지 않습니다.

- hw에 대한 정의가 없습니다.
- 반복문을 언제 멈출지 알 수 없습니다.

먼저 hw 안에 이미 어떤 데이터가 있을지도 모릅니다. 그렇기 때문에 입력받은 작업을 시작하기 전
hw.Clear()를 호출하여 문제를 해결할 수 있습니다.

다음으로 반복문의 경우 점수를 더 이상 읽을 수 없을 때까지 실행할 수 있지만 두 가지의 문제점이 있습니다.

- EOF에 도달했을 경우
- 입력 스트림에 점수가 아닌 다른 무언가가 있을 경우

보통 EOF 표시는 입력 시도가 실패했음을 의미하기 때문에 모든 데이터를 성공적으로 읽은 후 나타나는 것은 문제가 있습니다.

또한 점수가 아닌 무언가를 마주쳤을 경우 라이브러리에서 입력 스트림을 `실패 상태failure state`로 표시하게끔 합니다.
이것은 EOF에 도달한 것과 같은 의미로, 앞으로의 입력 요청이 실패할 것입니다.

이를 위해서는 라이브러리에서 입력 시도를 할 수 없는 모든 상태(EOF 또는 유효하지 않은 입력)를 무시하도록 지시하는 것입니다.
in 내부의 오류 상태를 재설정 하려면 in.Clear()를 호출해야 합니다. in.Clear()는 입력이 실패하더라도 계속 코드를 실행할 것을
라이브러리에 지시합니다.

read_hw 완성하면 다음과 같습니다.

```c++
// 입력 스트림에서 과제 점수를 읽어서 vector<double>에 넣음
istream& read_hw(istream& in, vector<double>& hw)
{
    if (in)
    {
        // 이전 내용을 제거
        hw.Clear();

        // 과제 점수를 읽음
        double x;

        while(in >> x)
            hw.push_back(x);
    
        // 다음 학생의 점수 입력 작업을 고려해 스트림을 지움
        in.Clear();
    }

    return in;
}
```

멤버 함수 Clear가 istream 객체와 벡터 객체 사이에서 완전히 다른 동작을 실행하는 것에 주의합시다.
{: .notice--warning}

### 함수를 사용하여 학생 성적 구하기
지금까지의 내용을 이용해 다시 구현하면 다음과 같습니다. 코드는 main 함수만 구현합니다.

```c++
int main()
{
    // 학생의 이름을 묻고 읽음
    cout << "Please enter your first name: ";
    string name;
    cin >> name;
    cout << "Hello, " << name << "!" << endl;

    // 중간시험과 기말시험의 점수를 묻고 읽음
    cout << "Please enter your midterm and final exam grades: ";
    double midterm, final;
    cin >> midterm >> final;

    // 과제 점수를 물음
    cout << "Enter all your homework grades, "
            "followd by end-of-file: ";
    vector<double> homework;

    // 과제 점수를 읽음
    read_hw(cin, homework);

    // 종합 점수를 계산
    try
    {
        double final_grade = grade(midtermm final, homework);
        streamsize prec = cout.precision();
        cout << "Your final grade is " << setprecision(3)
            << final_grade << setprecision(prec) << endl;
    }
    catch (domain_error)
    {
        cout << endl << "You must enter your grades. "
                        "Please try again." << endl;
        return 1;
    }

    return 0;
}
```

## 데이터 구조화
앞선 프로그램은 학생 1명의 성적을 구하는 데 유용합니다. 수강하는 모든 학생의 성적을 구할 수 있도록 프로그램을 수정합시다.

학생의 점수를 일일이 입력받는 대신 이름과 점수가 포함된 파일을 제공한다고 가정해봅시다.

```
Smith 93 91 47 90 92
Carpenter 75 90 87 92 93
```

학생 1명의 데이터를 저장할 공간이 있다면 벡터를 사용하여 모든 학생의 데이터를 저장할 수 있습니다. 학생의 데이터를 저장하는
데이터 구조를 만들고 저장한 데이터를 읽고 처리하는 몇 가지 보조 함수를 만들어 이를 해결해봅시다.

### 학생 데이터 통합하기
먼저 학생 1명에 관련된 모든 정보를 하나의 공간에 저장할 수 있는 방법이 필요합니다. 

이를 정의하면 다음과 같습니다.

```c++
struct Student_info
{
    string name;
    double midterm, final;
    vector<double> homework;
};
```

`구조체struct`인 Student_info는 4개의 데이터 멤버가 있는 타입입니다. Student_info 자체가 타입이므로
4개의 데이터 멤버가 있는 Student_info 타입의 객체를 각각 정의할 수 있습니다.

### 학생 점수 관리
학생들의 점수를 다루는 개념을 처리하기 쉽게 나누면 세 단계의 개별 함수로 표현할 수 있습니다.

- Student_info 객체에 데이터를 입력
- Student_info 객체의 종합 점수를 생성
- Student_info 객체들의 요소를 갖는 벡터를 정의

```c++
istream& read(istream& is Student_info& s)
{
    // 학생의 이름, 중간시험 점수, 기말시험 점수를 읽어 저장
    is >> s.name >> s.midterm >> s.final;
    read_hw(is, s.homework);    // 학생의 모든 과제 점수를 읽어 저장
    return is;
}
```

read 함수는 2개의 참조 매개변수를 가집니다.

- 읽어야 할 istream
- 읽은 것을 저장할 객체

매개변수 s는 참조 타입일므로 함수 안에서 s를 사용하면 전달할 인수의 상태에 영향을 줍니다.

```c++
double grade(const Student_info& s)
{
    return grade(s.midterm, s.final, s.homework);
}
```

위 함수는 Student_info 타입의 객체를 사용하여 double 타입의 종합 점수를 구해 반환합니다. const Student_info&이므로 함수를 호출할 때
Student_info 객체를 통째로 복사하면서 오버헤드를 피한다는 점을 유의합시다.

마지막으로 Student_info 객체의 벡터를 sort로 정렬합시다.

```c++
sort(student.begin(), student.end());   // 틀림
```

sort는 함수가 벡터의 요소들을 순서대로 비교할 때 < 연산자를 사용합니다. vector<double> 타입인 벡터에 sort 함수를 호출하면
< 연산자는 2개의 double 타입 값을 비교하여 알맞은 결과를 제공합니다.

그렇기에 위의 코드는 Student_info 타입의 값을 비교하기 위한 명확한 기준이 없기 때문에 컴파일러는 에러를 반환합니다.

이를 위해 sort는 세 번째의 선택 인수로 `서술predicate 함수`가 있습니다. 서술 함수는 bool 타입인 진릿값을 반환하는 함수입니다.

```c++
bool compare(const Student_info& x, const Strudent_info& y)
{
    return x.name < y.name;
}
```

위 함수는 문자열을 비교하려고 < 연산자를 제공하는 문자열 클래스에 Student_info를 비교하는 작업을 맡깁니다.

왼쪽 피연산자가 오른쪽 피연산자보다 알파벳순으로 작으면 온쪽 피연산자보다 작다고 간주합니다.
{: .notice--warning}

이를 세 번째 인수에 넣으면 벡터 정렬이 가능합니다.

```c++
sort(student.begin(), student.end(), compare);
```