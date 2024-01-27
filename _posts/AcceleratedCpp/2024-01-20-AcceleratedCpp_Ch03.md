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
last_modified_at: 2024-01-20
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
