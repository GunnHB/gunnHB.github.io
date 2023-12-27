---
title:  "[Accelerated C++] 001. 문자열 사용"
excerpt: "C++을 이용한 문자열 사용을 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP]

toc: true
toc_sticky: true
 
date: 2023-12-27
last_modified_at: 2023-12-27
---

🔔 \[AcceleratedC++\] 서적을 정리한 내용입니다. 🔔
{: .notice}

<div class="notice--info" markdown="1">
해당 포스팅에서는 문자열을 사용하는 간단한 프로그램을 통해
기본적인 `C++`의 개념을 살펴보고 `선언`, `변수`, `초기화`, `입력`, `string 라이브러리`를
살펴볼 예정입니다.
</div>

## 입력
```c++
// 이름을 묻고, 인사를 건넴
#include <iostream>
#include <string>

int main()
{
	// 이름을 물음
	std::cout << "Please enter your first name : ";

	// 이름을 읽어 들임
	std::string name;
	std::cin >> name;

	// 인사말 작성
	std::cout << "Hello, " << name << "!" << std::endl;

	return 0;
}
```

위 프로그램은 간단한 인사말을 출력하는 프로그램입니다.

```c++
Please enter your first name : 
```

프로그램을 실행하면 콘솔창에 위의 메시지가 출력되고, 이름을 입력하면

```c++
Hello, noob!
```

위와 같은 메시지가 출력된 후 종료되는 간단한 프로그램입니다.

이와 같이 입력한 내용을 전달받기 위해서는 저장할 장소가 있어야 합니다.
이를 `변수Variable`라고 하고, 변수는 이름이 있는 `객체Object`입니다.

이름 없는 객체의 경우가 존재하기에 객체와 변수는 구분되어야 합니다.
{: .notice--warning}

