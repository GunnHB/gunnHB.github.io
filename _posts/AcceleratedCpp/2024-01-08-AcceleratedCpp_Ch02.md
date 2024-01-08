---
title:  "[Accelerated C++] 002. 반복문과 카운팅"
excerpt: "C++에서의 반복문을 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2024-01-08
last_modified_at: 2024-01-08
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
이전 포스팅에서 테두리로 둘러쌓인 인사말을 출력하는 프로그램을 만들었습니다.
이번 포스팅에서는 반복문을 통해 사용자가 테두리의 크기를 변경할 수 있는
프로그램을 만들어봅시다.
</div>

## 해결해야 하는 문제
```c++
****************
*              *
* Hello, noob! *
*              *
****************
```

위 출력문은 이전 포스팅에서 출력했던 테두리로 둘러쌓인 인사말의 결과입니다.
해당 결과를 출력하는 프로그램은 한 번에 한 행씩 출력합니다.

위 프로그램의 방식은 아주 큰 단점이 있습니다. **<u>각 행의 간단한 수정이 필요할 때마다
프로그램을 새롭게 만들어야한다는 점</u>**입니다. 그렇기에 이번에는 각 행을 변수에 저장하지 않는
출력 형식으로 만들어봅시다. 

## 전체적인 구조
```c++
// 이름을 입력받아 테두리로 묶인 인사말을 생성
#include <iostream>
#include <string>

int main() 
{
  // 이름을 물음
  std::cout << "Please enter your name: ";

  // 이름을 입력
  std::string name;
  std::cin >> name;

  // 출력하려는 메시지를 구성
  const std::string greating = "Hello, " + name + "!";

  // 다시 작성할 부분
  return 0;
}
```

위의 스크립트는 이전 스크립트와 동일하여 다시 작성하지 않아도 되는 부분입니다.
새로운 프로그램에 필요한 부분을 하나씩 구현하여 완성시켜봅시다.

## 주어진 개수만큼 행 출력하기
인사말과 위아래 테두리는 각각 한 행을 차지하므로 행의 개수는 총 3개입니다. 여기에
인사말과 한 쪽 테두리 사이에 남겨 둘 공백 개수의 2배를 더한 것으로 행의 총 개수를 구할 수 있습니다.

```c++
...

// 인사말과 한 쪽 테두리 사이의 공백 개수
const int pad = 1;

// 출력할 행의 전체 개수
const int rows = pad * 2 + 3;

...
```

- `pad` : 인사말과 한 쪽 테두리 사이의 공백 개수를 저장하는 상수
- `row` : pad를 사용하여 출력하고자 하는 행의 전체 개수를 제어

두 값은 const를 이용해 정의되었으므로 상수입니다.
{: .notice--warning}

pad 값의 변경만으로 공백 개수가 변경되어 행의 개수 역시 변경됩니다.

```c++
...

// 입력과 출력을 구분
std::cout << std::endl;

// rows만큼의 행을 출력
int r = 0;

// 불변성 : 지금까지 r개 행을 출력
while (r != rows)
{
    // 하나의 행을 출력
    std::cout << std::endl;
    ++r;
}

...
```