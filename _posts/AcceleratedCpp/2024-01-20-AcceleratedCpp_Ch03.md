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
  cout << "Enter all yout homework grades, followed by end-of-file: ";

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
       << 0.2 * midterm + 0.4 * final + 0.4 * sum / count << setprecision(prec)
       << endl;

  return 0;
}
```

- \<ios>
  - 입출력 라이브러리에서 전송된 문자 수나 버퍼 크기를 나타내려고 사용하는 타입인 `streamsize`를 정의
- \<iomanip>
  - 출력 결과의 유효 자릿수를 결정하는 조작어인 `setprecision`을 정의

endl은 자주 사용하므로 \<iomanip>가 아닌 \<iostream>이 정의합니다.
{: .notice--warning}