---
title:  "[Accelerated C++] 001. 문자열 사용"
excerpt: "C++에서의 문자열 사용을 알아봅시다!"

author: gunnHB
categories: 
 - Accelerated Cpp
tags: 
 - [Github, Git, Programming, CPP, Accelerated Cpp]

toc: true
toc_sticky: true
 
date: 2023-12-27
last_modified_at: 2023-12-28
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
이를 `변수variable`라고 하고, 변수는 이름이 있는 `객체object`입니다.

이름 없는 객체의 경우가 존재하기에 객체와 변수는 구분되어야 합니다.
{: .notice--warning}

변수를 사용하려면 구현체에 변수의 `이름`과 `타입`을 알려줘야 합니다.
이는 구현체가 프로그램의 기계 코드를 더 효율적으로 만들어주며, 컴파일러가 
변수 이름 중에서 오탈자를 탐지할 수 있습니다.

위의 예제에서의 변수 이름은 `name`이고 타입은 `std::string`입니다.
여기서 std:: 뒤에 오는 이름 string은 프로그래밍 언어의 기본 영역이 아닌
**<u>표준 라이브러리의 일부</u>**라는 뜻입니다.

이제 첫번째 실행문부터 살펴보겠습니다.

```c++
std::cout << "Please enter your first name : ";
```

첫번째 실행문은 사용자의 이름을 묻는 메시지가 출력됩니다. 여기서 중요한 점은
`std::endl`을 사용하지 않았으므로 메시지를 출력한 후 행 바꿈을 하지 않으며
동일한 행에서 입력을 기다립니다.

```c++
std::string name;
```

다음의 실행문은 std::string 타입 변수 name의 `정의문definition`입니다. 정의문이
함수 본문에 있으므로 해당 변수는 중괄호 안에서만 존재하는 `지역 변수local variable`입니다.
컴퓨터가 \}에 도달하면 지역 변수는 `소멸destroy`되고 변수가 차지하던 메모리를 반환합니다.

객체의 타입에는 `인터페이스interface`의 개념이 포함되어 있습니다. 문자열 변수인 name의 동작 중에는
문자열을 `초기화initialize`하는 것이 있습니다.

인터페이스란 어떤 타입의 객체로 할 수 있는 동작들의 모음입니다.
{: .notice--warning}

문자열 변수를 정의하면 암묵적으로 변수가 초기화됩니다. 사용자가 원하는 값을 넣어 문자열을 만들 수 있지만,
그렇지 않다면 문자열은 문자를 전혀 가지지 않습니다. 이를 `빈empty 문자열` 또는 `null 문자열`이라고 합니다.

```c++
std::cin >> name;
```

메시지의 출력 시 << 연산자와 std::cout을 사용하는 것처럼 입력 시에는 >> 연산자와 std::cin을 사용합니다.
여기서 >> 연산자는 표준 입력으로 문자열을 읽어서 name이라는 객체에 저장하는 역할을 합니다. 문자열을 읽도록
라이브러리에 요청하면 `공백whitespace 문자`(스페이스, 탭, 백스페이스, 행 바꿈)를 무시하고 다음 공백 문자나
`EOF End Of File`이 나타날 때까지 문자들을 읽기 시작하여 변수 name에 넣습니다.

일반적으로 입출력 라이브러리는 출력 작업을 최적화하기 위해 `버퍼buffer`라는 내부 데이터 구조를 사용합니다.
라이브러리는 버퍼를 사용하여 출력할 문자를 모은 후 필요할 때에 버퍼에 저장한 내용을 출력하고 버퍼를 비우는
방식으로 출력 작업을 효율적으로 처리합니다.

시스템이 버퍼를 비우는 상황은 세 가지가 있습니다.

- 버퍼가 가득 차 있는 경우
- 라이브러리가 표준 입력 스트림으로 입력 내용을 요청받을 경우
- 개발자가 코드로 명시한 경우

이를 위의 코드에 적용시켜보면 cout를 사용하여 메시지를 출력할 때, 출력 내용은 표준 출력 스트림과
관련된 버퍼에 저장됩니다. 그 다음 cin을 사용하여 입력을 받는데, 이 작업으로 cout 버퍼를 비우면서
메시지가 출력되게 됩니다.

이어지는 실행문에서는 std::endl을 통해 라이브러리가 버퍼를 비우도록 명시적으로 지시합니다.
std::endl 값이 출력되면 이 행에서 해야할 작업이 끝나므로 버퍼를 비웁니다. 이 때 시스템은
버퍼의 내용을 출력 스트림에 강제로 즉시 반영합니다.

## 테두리
위의 코드를 약간 수정하여 아래와 같이 나타내도록 해봅시다.

```c++
****************
*              *
* Hello, noob! *
*              *
****************
```

출력될 내용은 다음과 같습니다.

- 첫 번째 행은 입력한 이름의 문자 개수, 인사말("Hello, ")의 문자 개수 양 끝에 하나씩 존재하는
*과 공백의 개수만큼의 길이를 갖습니다.
- 두 번째 행은 양 끝에 하나씩 존재하는 *과 첫 번째 행과 같은 길이가 되도록 공백을 줍니다.
- 세 번째 행은 *, 공백, 메시지, 공백, * 순서입니다.
- 마지막 두 행은 각각 두 번째와 첫 번째 행의 반복입니다.

이를 구현하면 다음과 같습니다.

```c++
// 이름을 입력받아 테두리로 묶인 인사말을 생성
#include <iostream>
#include <string>

int main() 
{
  std::cout << "Please enter your name: ";
  std::string name;
  std::cin >> name;

  // 출력하려는 메시지를 구성
  const std::string greating = "Hello, " + name + "!";

  // 인사말의 두 번째 행과 네 번째 행
  const std::string spaces(greating.size(), ' ');
  const std::string second = "* " + spaces + " *";

  // 인사말의 첫 번째 행과 다섯 번째 행
  const std::string first(second.size(), '*');

  // 모두 출력
  std::cout << std::endl;
  std::cout << first << std::endl;
  std::cout << second << std::endl;
  std::cout << "* " << greating << " *" << std::endl;
  std::cout << second << std::endl;
  std::cout << first << std::endl;

  return 0;
}
```

위 코드를 분석해보면

```c++
std::string name;
std::cin >> name;
```

프로그램은 사용자의 이름을 입력받아 변수 name에 넣습니다.

```c++
const std::string greating = "Hello, " + name + "!";
```

그런 다음 출력 메시지인 상수 greating을 정의합니다.

```c++
const std::string spaces(greating.size(), ' ');
```

다음으로 greating의 문자 개수만큼 공백을 넣는 상수 spaces를 정의합니다.

```c++
const std::string second = "* " + spaces + " *";
```

인사말의 두 번째 행을 포함하는 상수 second는 상수 spaces를 사용하여 정의합니다.

```c++
const std::string first(second.size(), '*');
```

그리고 second의 문자 개수만큼 *을 포함하는 상수 first를 정의합니다.

마지막으로 한번에 한 행씩 인사말을 출력합니다.

인사말을 정의하는 부분에서 세 가지의 개념을 확인합시다.

- 변수를 정의하면서 변수에 값을 넣을 수 있습니다.
  - 변수와 값이 서로 다른 타입이면 구현체는 넣은 값을 변수의 타입으로 `변환convert`합니다.
- \+ 기호를 사용하여 1개의 문자열과 1개의 문자열 리터럴을 `결합concatenate`하거나
2개의 문자열끼리 결합할 수 있습니다.
  - \+ 연산자는 피연산자의 타입에 따라 실행하는 연산이 다릅니다.
  - 이를 연산자의 `오버로드overload`라고 합니다.
- 상수를 정의할 때는 const를 사용합니다.
  - const는 소멸할 때까지 값을 변경하지 않겠다는 약속입니다.

const로 정의한 상수는 정의할 때 반드시 초기화해주어야 합니다. 위 예제에서는 name 값을 입력한 뒤에
greating을 정의합니다. name은 프로그램을 실행하기 전까지는 정해져 있지 않으므로 상수가 아닙니다.

다음 실행문에서의 세 가지 개념을 살펴봅시다.

```c++
const std::string spaces(greating.size(), ' ');
```

상수 greating을 정의할 때는 \= 기호를 사용하여 초기화했습니다. \=기호를 사용하면 변수가 가져야 하는 값을
명시적으로 지시할 수 있습니다. 상수 spaces는 쉼표로 구분하고 괄호로 묶인 2개의 표현식을 사용하여 초기화합니다.
괄호를 사용함으로써 표현식을 거쳐 상수를 생성하도록 구현체에 지시합니다.

상수나 변수의 생성 방법은 타입에 따라 다릅니다. 무엇으로 문자열을 만드는 것이며 2개의 표현식은 어떤 의미가
있는지 살펴봅시다.

- greating.size()는 `멤버 함수member function`를 호출합니다.
  - 상수 greating은 std::string 타입이며 greating.size()로 greating의 문자 개수를 나타내는 정수를 얻을 수 있습니다.
- ' '은 `문자 리터럴character literal`입니다.
  - 문자열 리터럴과는 완전히 다릅니다.
  - 문자열 리터럴은 항상 큰따옴표로 묶고 문자 리터럴은 항상 작은따옴표로 묶습니다.
  - 문자 리터럴은 리본 타입인 char입니다.
- 상수 spaces의 정의는 정숫값과 문자값을 받아 정숫값만큼의 횟수로 문자값을 복사해 문자열을 만듭니다.

## 정리
다음의 사항들을 잘 기억해둡시다.

### 문자 타입
- `char`
  - 구현제가 정의한 기본 타입이며 일반 문자를 저장합니다.
- `wchar_t`
  - 한국어 같은 언어의 문자를 표현할 만큼 크기가 큰 '확장 문자'를 저장하는 기본 타입입니다.

### 문자열 타입
- 표준 헤더인 \<string>에 정의되어 있습니다.
- 문자열 타입 객체는 0개 이상의 문자로 이루어진 순차열을 갖습니다.
- 정수를 n, 문자를 c, 입력 스트림을 is, 출력 스트림을 os라고 한다면...
    - `std::string s`
      - std::string 타입의 변수 s를 정의하며 변수 s는 빈 문자열로 초기화됩니다.
    - `std::string t = s`
      - std::string 타입의 변수 t를 정의하며 변수 t는 s에 있는 문자들의 복사본으로 
        초기화됩니다. s는 문자열 또는 문자열 리터럴입니다.
    - `std::string z(n, c)`
      - std::string 타입의 변수 z를 정의하며 z는 문자 c의 복사본 n개를 포함하여
        초기화됩니다. c는 문자열 또는 문자열 리터럴이 아닌 char 타입이어야만 합니다.
    - `os << s`
      - os가 가리키는 출력 스트림에  s에 포함된 문자들을 서식 변경 없이 출력합니다.
      - 표현식의 결과는 os입니다.
    - `is >> s`
      - 공백 외의 문자가 등장할 때까지 is가 가리키는 스트림에서 문자들을 읽고 버립니다.
      - is가 가리키는 스트림에서 새로운 공백이 등장할 때까지 연속된 문자들을 s로 읽어 들입니다.
      - 기존의 s 값이 무엇이든 상관없이 덮어씁니다.
      - 표현식의 결과는 is입니다.
    - `s + t`
      - s에 있는 문자들의 복사본과 t에 있는 문자들의 복사본을 결합한 std::string 객체입니다.
      - s 또는  t 중 하나가(둘 다는 아님) 문자열 리터럴이거나 char 타입의 값일 수 있습니다.
    - `s.size()`
      - s에 있는 문자들의 개수입니다.

### 변수
변수는 다음 세 가지 방법 중 한 가지로 정의할 수 있습니다.

```c++
std::string hello = "Hello";   // 초깃값을 명시하여 변수를 정의
std::string stars(100, '*');   // 타입과 주어진 표현식에 따라 변수를 생성
std::string name;              // 타입에 따라 암묵적인 초기화로 변수를 정의 
```

한 쌍의 중괄호 내부에 정의된 변수는 중괄호 내부의 코드를 실행할 동안에만 존재하는
지역 변수입니다. 구현체가 \}에 도달하면 구현체는 변수를 소멸시키고 변수가 차지하던
메모리를 시스템에 반환합니다.

const로 상수를 정의하면 소멸할 때까지 값을 바꾸지 않겠다는 의미입니다. 나중에 값을
바꾸지 못하므로 정의하면서 반드시 초기화해야 합니다.

### 입력
std::cin >> v를 실행하면 표준 입력 스트림에 있는 모든 공백을 무시한 후에 표준 입력의 내용을
변수 v에 넣습니다. 연속적인 입력 동작을 하려는 목적으로 istream 타입인 std::cin을 반환합니다.