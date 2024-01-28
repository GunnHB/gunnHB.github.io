---
title:  "[Accelerated C++] 003. 데이터 일괄 처리"
excerpt: "여러 개의 비슷한 데이터를 처리하는 법을 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2024-01-20
last_modified_at: 2024-01-27
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
프로그램을 복잡하게 만드는 가장 일반적인 이유 중 하나는 여러 개의 비슷한 데이터를 처리하려는 것입니다.

이 장에서는 학생의 시험 및 과제 점수를 입력받고 최종 점수를 계산하는 프로그램을 작성하여 데이터를 일괄 처리하는 방법을
살펴봅시다.
</div>

## 학생의 최종 점수 계산하기
한 과목에서 중간시험 20%, 기말시험 40%, 평군 과제 점수 40%의 가중치를 두어 각 학생의 최종 점수를 구성하는 프로그램을
작성하면 다음과 같습니다.

```c++
#include <iomanip>
#include <ios>
#include <iostream>
#include <string>

using std::cin;
using std::cout;
using std::endl;
using std::setprecision;
using std::streamsize;
using std::string;

int main() {
  // 학생의 이름을 묻고 입력받음
  cout << "Please enter your first name: ";
  string name;
  cin >> name;
  cout << "Hello, " << name << "!" << endl;

  // 중간시험과 기말시험의 점수를 묻고 입력받음;
  cout << "Please enter your midterm and final exam grade: ";
  double midterm, final;
  cin >> midterm >> final;

  // 과제 점수를 물음
  cout << "Enter all yout homework grades,"
    "followed by end-of-file: ";

  // 지금까지 입력된 과제 점수의 개수와 합
  int count = 0;
  double sum = 0;

  // 입력을 위한 변수
  double x;

  // 불변성 : 지금까지 count개 점수를 입력받았으며 입력받은 점수의 합은 sum
  while (cin >> x) {
    ++count;
    sum += x;
  }

  // 결과를 출력
  streamsize prec = cout.precision();
  cout << "Your final grade is " << setprecision(3)
       << 0.2 * midterm + 0.4 * final + 0.4 * sum / count
       << setprecision(prec) << endl;

  return 0;
}
```

- \<ios>
  - 입출력 라이브러리에서 전송된 문자 수나 버퍼 크기를 나타내려고 사용하는 타입인 `streamsize`를 정의
- \<iomanip>
  - 출력 결과의 유효 자릿수를 결정하는 조작어인 `setprecision`을 정의

endl은 자주 사용하므로 \<iomanip>가 아닌 \<iostream>이 정의합니다.
{: .notice--warning}

프로그램은 이름, 중간 및 기말고사 점수를 묻고 입력받습니다. 그다음 학생의 과제 점수를 묻고 EOF 신호를 만날 때까지
계속 값을 입력받습니다.

<div class="notice--info" markdown="1">
EOF(End Of File)
- 윈도우
  - Ctrl + Z
- 유닉스 / 리눅스
  - Ctrl + D
</div>

과제 점수를 입력받는 동안 count를 사용해서 입력한 점수 개수를 추적하고 누적 점수를 sum에 저장합니다.

```c++
count << "Enter your midterm and final exam grades";
double midterm, final;
cin >> midterm >> final;
```

중간 / 기말고사 점수를 입력받기 위해 double형의 midterm과 final을 정의합니다. 부동 소수점을 나타내는 타입은
float형(4바이트, 32비트)과 double형(8바이트, 64비트)이 있습니다. 부동 소수점 계산은 double형을 사용하는 것이 낫습니다.

최근 컴퓨팅 환경에서는 일반적으로 double이 float보다 훨씬 정확하며 연산 속도도 큰 차이가 없습니다.
{: .notice--info}

그다음 실행문에서는 과제 점수를 입력받습니다.

```c++
cout << "Enter all your homework grades, "
    "followed by end-of-file: ";
```

사실 이 부분을 주의깊게 보면 2개의 문자열 리터럴을 출력하지만, 단 하나의 <<를 사용하는 것을 알 수 있습니다.
**<u>공백으로 분리된 2개 이상의 문자열 리터럴은 자동으로 결합</u>**되기 때문입니다. 따라서 다음의 코드와 같은 의미입니다.

```c++
cout << "Enter all your homework grades, followed by end-of-file: ";
```

다음은 입력받는 정보를 유지하는 데 사용할 변수를 정의합니다.

```c++
int count = 0;
double sum = 0;
```

위 코드에서 주목해야 할 점을 변수에 초깃값을 할당하는 일입니다. 변수의 초깃값을 저장하지 않으면 암묵적으로 
`기본초기화default-initialization`단계를 따르고, 이러한 초기화는 변수 타입에 따라 결과가 다릅니다.

명시적으로 초기화하지 않은 기본 타입 지역 변수는 `정의하지 않은undefined 상태`입니다. 이때 변수는 생성된 메모리 공간을
이미 차지하는 임의의 의미 없는 값입니다. 대부분의 구현체는 정의하지 않은 값에 접근하도록 허용합니다. 하지만 그렇게 되면
메모리에 있는 것이 무엇이든 정확한 값이 아니므로 충돌이 일어나거나 잘못된 결과를 얻게됩니다.

만약 sum과 count를 초기화하지 않았다면 프로그램은 제대로 동작하지 않았을 것입니다. 프로그램이 두 변수들로 실행하는
첫 번째 작업이 변수들의 값을 사용하는 것이기 때문입니다.

while문에서 새롭게 살펴볼 점은 조건문의 형태입니다.

```c++
// 불변성 : 지금까지 count개 점수를 입력받았으며 입력받은 점수의 합은 sum
while (cin >> x) 
{
    ++count;
    sum += x;
}
```

while문의 조건인 cin >> x에서 입력이 발생하는 한 계속 실행됩니다. 그렇기에 새로운 입력 요청이 발생하면
조건을 만족시키게 됩니다. 자세한 내용은 [[다음]](#입력-종료-판별)에서 살펴봅시다.

이제 결과를 출력하는 부분입니다.

```c++
streamsize prec = cout.precision();
cout << "Your final grade is " << setprecision(3)
    << 0.2 * midterm + 0.4 * final + 0.4 * sum / count 
    << setprecision(prec) << endl;
```

목표는 최종 점수를 세 자리 숫자로 출력하는 것입니다. 이 때문에 setprecision을 사용합니다.
setprecision은 이어서 발생하는 출력을 주어진 숫자만큼의 유효 자릿수로 나타내려고 스트림을 조작합니다.
setprecision(3)이라고 작성하면 점수를 3개의 유효 자릿수로 출력하도록 구현체에 요청하라는 의미입니다.

일반적으로 소수점 앞 두 자리, 소수점 뒤 한 자리를 출력합니다.
{: .notice--warning}

이후 다시 setprecision을 사용해서 cout에 있을지도 모르는 후속 출력의 정밀도를 바꿉니다.

사실 해당 부분은 프로그램의 마지막이므로 후속 출력을 없습니다. 하지만 cout의 정밀도를 원래
상태로 돌려놓는 것은 좋은 프로그래밍 습관입니다.
{: .notice--warning}

### 입력 종료 판별
프로그램에서 새로 등장한 부분은 while문의 조건입니다. 조건의 암묵적인 대상은 istream입니다.

```c++
while(cin >> x) {/* 생략 */}
```

먼저 cin을 이용해 입력을 받습니다. 입력받은 값을 x에 넣으면 while문이 실행됩니다.
입력이 일어나지 않거나 x의 타입과 맞지 않는 값이 입력된다면 while문은 실행되지 않습니다.

해당 코드의 이해가 어렵다면 다음과 같이 이해할 수 있습니다. 예를 들어 x에 임의 값을 하나 넣고
조건의 참, 거짓 여부를 판별한다고 했을 때 코드는 다음과 같습니다.

```c++
if(cin >> x) {/* 생략 */}
```

해당 코드는 다음과 같은 의미입니다.

```c++
cin >> x;

if(cin) {/* 생략 */}
```

cin >> x를 조건으로 사용하는 것은 조건을 판별하는 것만이 아니라 입력받은 값을 x에 넣는 부수적인 효과를 얻습니다.

istream 클래스에서는 cin을 숫자 값으로 바꿀 수 있습니다. 또한 해당 숫자 값을 bool 타입 값으로 바꿔 조건 판별에
사용합니다. 값을 바꿔 반환하는 값은 최근에 입력을 받았는지에 따라 달라지는 istream 객체의 내부 상태를 반영합니다.
따라서 조건 판별에서 cin을 사용하는 것은 cin을 이용해 최근에 입력을 받았는지 판별하는 것과 같습니다.

스트림에서 입력값을 읽을 수 없는 몇 가지 상황은 다음과 같습니다.

- EOF에 도달했을 때
- 변수의 타입과 맞지 않는 값을 입력받았을 때
- 시스템이 입력 장치에서 하드웨어 정애를 감지했을 때

## 평균값 대신 중앙값 사용하기
현재의 프로그램은 평균값을 계산하는 과정에서는 괜찮지만 중앙값을 사용할 때에 문제가 발생합니다.

값을 집단에서 중앙값을 찾는 가장 간단한 방법은 값을 오름차순이나 내림차순으로 정렬한 후, 가운데 값을 선택하는 것입니다.
엉망인 점수 몇 개 때문에 결과가 왜곡되는 일이 없으므로 중앙값이 평균값보다 더 유용할 때가 종종 있습니다.

그러기 위해선 입력받는 모든 값을 유지해야 합니다.

### 데이터 집단을 벡터에 저장하기
중앙값을 계산하려면 모든 과제 점수를 입력받아 저장한 후 정렬하고 마지막으로 가운데 하나의 값을 선택해야 합니다.

- 얼마나 많은 값이 있을지 모르지만 한 번에 하나씩 입력받고 저장합니다.
- 입력이 끝나면 값들을 정렬합니다.
- 가운데에 위치한 값을 효율적으로 찾습니다.

이러한 문제를 위해 표준 라이브러리는 `벡터Vector`타입을 제공합니다.

- 벡터 Vector
  - 주어진 타입 값들의 순차열이 있으며 새로운 값을 저장할 필요에 따라 늘어남
  - 개별적인 접근이 가능

입력받은 점수를 더한 후 버리는 기존의 코드를 벡터에 저장하는 방식으로 바꾸면 다음과 같습니다.

```c++
double x;
vector<double> homework;

while(cin >> x)
    homework.push_back(x);
```

homework는 vector<double> 타입으로 정의합니다. 벡터는 집단이 있는 `컨테이터container`입니다. 각 벡터의 값을 모두 같은 타입이며,
벡터를 정의할 때마다 벡터가 감을 값을 타입을 지정해야 합니다.

벡터 타입은 c++의 특징이 `템플릿 클래스template class`로 정의합니다. vector\<double> 타입의 객체는 double타입의 객체를 담은 벡터이고
vector\<string> 타입의 객체는 문자열을 담은 벡터입니다.

```c++
homework.push_back(x);
```

이전 포스팅에서 살펴본 greeting.size()와 마찬가지로 push_back은 멤버 함수입니다. push_back은 벡터의 마지막에 새 요소를 추가하는
작업을 실행합니다. push_back은 해당 인수를 벡터의 뒤쪽으로 밀어내며, 이때 발생하는 부수적인 효과 때문에 벡터의 크기는 1씩 증가합니다.

### 출력 생성하기
```c++
streamsize prec = cout.precision();
cout << "Your final grade is " << setprecision(3)
    << 0.2 * midterm + 0.4 * final + 0.4 * sum / count
    << setprecision(prec) << endl;
```

midterm과 final은 시험 점수를 나타내고, sum과 count는 모든 과제 점수의 합과 입력된 과제 점수의 개수를 나타냅니다.

중앙값을 찾으려면 벡터 homewok의 크기를 최소 두 번 알아야 합니다.

- 1. 벡터의 크기가 0인지
- 2. 가운데 요소 위치를 계산

```c++
typeof vector<double>::size_type vec_sz;
vec_sz size = homework.size();
```

벡터 타입은 vector\<double>::size_type라는 타입과 size라는 함수를 정의합니다. size_type은 크기가 큰 벡터도 충분히
담을 수 있는 부호 없는 타입이고, size()는 벡터의 요소 개수를 나타내는 size_type 값을 반환합니다.

vector\<double>::size_type 타입은 이름이 길어 다루기 불편합니다. 이를 단순화하기 위해 typeof를 사용할 수 있습니다.

- typeof
  - 변수를 정의할 때 변수 타입의 동의어로 이름을 지정할 수 있습니다.

typeof로 정의한 이름은 다른 이름들과 같은 범위를 갖습니다. 즉, size_type과 같은 범위와 의미로 vec_sz를 사용할 수 있습니다.
