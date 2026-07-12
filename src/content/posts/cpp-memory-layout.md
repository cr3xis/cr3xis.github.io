---
title: '프로그램이 메모리에서 배치되는 방식 이해하기'
published: 2026-02-15
description: '프로그램의 Code, Data, Heap, Stack 영역과 메모리 배치 구조를 정리합니다.'
image: '/assets/img/CS/02/Area.png'
tags:
  - 'Memory Layout'
  - 'Stack'
  - 'Heap'
  - 'Data'
  - 'Text'
  - 'C'
  - 'C++'
category: 'CS'
draft: false
---


소스 코드가 실행 파일로 만들어지는 과정까지 이해하고 나면, 자연스럽게 다음 질문이 생긴다.  
**“그럼 이 실행 파일은 메모리에서 어떤 구조로 배치되어 실행될까?”**

Windows 기준으로 보면 프로그램은 실행될 때 운영체제가 정해 둔 형태에 따라 여러 영역으로 나뉘어 올라가게 된다.  
그 구조를 자연스럽게 따라가며 정리하고, 각 영역이 실제 코드에서 어떤 의미로 연결되는지도 함께 설명한다.

---
<figure style="text-align: center;">
  <img src="/assets/img/CS/02/Area.png" alt="메모리 배치 구조" style="display: block; margin: 0 auto;">
  <figcaption>메모리 배치 구조</figcaption>
</figure>


## 1. Code(Text) 영역

프로그램의 핵심 로직, 즉 CPU가 실제로 실행할 기계어 코드가 저장되는 곳이다.  
함수의 본문, 분기, 반복문 같은 모든 흐름이 이 영역에 들어간다.

아래 예제에서 `func()`와 `main()`의 본문은 모두 Text 영역에 저장된다.

```c
#include <stdio.h>

void func() {
    printf("Hello from func\n");
}

int main() {
    func();
    return 0;
}
```

이 코드를 컴파일하면 `func`와 `main`의 기계어 코드가 Text 영역으로 배치되고,  
CPU는 이 Code 영역의 내용을 순차적으로 읽으며 프로그램을 실행한다.

---

## 2. Data/BSS 영역

전역 변수와 정적 변수는 프로그램이 시작될 때 메모리에 배치되고, 프로그램이 종료될 때까지 유지된다.  
초기값이 있으면 Data, 초기값이 없으면 BSS에 들어간다.

```c
int global_init = 10;   // Data 영역
int global_uninit;      // BSS 영역

void func() {
    static int counter = 0; // Data 영역
    counter++;
}
```

- `global_init` → 초기값이 있으므로 Data  
- `global_uninit` → 초기값이 없으므로 BSS  
- `static counter` → 정적 변수이므로 Data  

이 변수들은 함수가 끝나도 사라지지 않는다.

---

## 3. Heap 영역

Heap은 프로그램 실행 중 필요한 만큼 메모리를 할당하고 자유롭게 사용할 수 있는 공간이다.  
필요할 때 확장되며, 개발자가 직접 관리해야 한다.

```c
#include <stdlib.h>

void func() {
    int* p = (int*)malloc(sizeof(int)); // Heap 할당
    *p = 42;

    printf("%d\n", *p);

    free(p); // 직접 해제
}
```

여기서 포인터 `p`는 스택에 저장되지만, `malloc`으로 생성된 실제 값은 Heap 영역에 존재한다.  
해제하지 않으면 계속 남아 메모리 누수가 발생한다.

---

## 4. Stack 영역

Stack은 함수 호출 시 자동으로 생성되는 임시 공간이다.  
지역 변수, 매개변수, 복귀 주소 등이 이 영역에 저장된다.

```c
void func(int x) {
    int temp = x + 1; // Stack 영역
    printf("%d\n", temp);
}

int main() {
    func(5); // 매개변수 또한 Stack에 저장
    return 0;
}
```

- `x` → 매개변수 → Stack  
- `temp` → 지역 변수 → Stack  
- 함수가 종료되면 둘 다 함께 정리된다.

---

## 마무리

이 구조를 이해하고 나면 포인터와 참조, 변수 생명 주기 같은 개념이 단순 문법처럼 보이지 않고,  
메모리 위에서 실제로 어떻게 동작하는지 자연스럽게 연결된다.
