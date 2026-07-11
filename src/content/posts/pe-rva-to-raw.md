---
title: 'RVA to RAW 이해하기'
published: 2026-03-01
description: 'PE 파일에서 RVA를 RAW 파일 오프셋으로 변환하는 과정을 실습합니다.'
image: '/assets/img/CS/RVATORAW/info.png'
tags:
  - 'PE'
  - 'RVA'
  - 'RAW'
  - 'IMAGE_SECTION_HEADER'
category: 'CS'
draft: false
---


 
대부분에 글에서는 실습보다는 이론적으로만 많이 다루고 있길래  
이번 글에서는 Notepad.exe를 기준으로 RVA → RAW 변환 흐름을 하나씩 따라가며  
파일 내부에서 주소가 어떻게 매핑되는지 정리해 보려고 한다.  

---

## RVA to RAW의 정의

RVA to RAW는 간단히 말하면 메모리 기준 상대 주소(RVA)를 파일 기준 주소(RAW)로 변환하는 과정이다.  

메모리에서 보는 주소 체계(RVA와 VA)는 실행 환경을 기준으로 잡혀 있지만  
파일에서 보는 주소(RAW)는 디스크에 저장된 구조를 기준으로 한다.  
그래서 이 둘을 연결하려면 PE의 섹션 구조를 반드시 이해하고 있어야 한다.    

---

## 필수적으로 알아야 하는 용어들

RVA와 RAW를 구별하기 위해 최소한 아래 다섯 가지 개념은 알고 있어야 했다.  

`VA (Virtual Address)`: 메모리 상태에서의 실제 주소  
VA = ImageBase + RVA  

`RVA (Relative Virtual Address)`: ImageBase를 0으로 가정한 상대 주소    
PE OptionalHeader의 주소 정보는 RVA 기준으로 되어 있다.  

`RAW (File Offset)`: 파일에서의 오프셋   

`ImageBase`: 프로그램이 메모리에 로드될 기본 위치 ( 환경에 따라 주소가 달라진다. )  
파일 기준 ImageBase : **0x00400000**  
메모리에서 실제 로드된 ImageBase : **0x00950000**  

`EntryPoint`: 프로그램이 실행을 시작하는 지점 ( RVA로 저장됨 )   

---

## 섹션 헤더 속 필요한 멤버들

RVA → RAW를 계산하려면 IMAGE_SECTION_HEADER에서 아래의 필드만 이해해도 문제없다.  

```cpp
typedef struct _IMAGE_SECTION_HEADER {
    DWORD VirtualAddress;
    DWORD VirtualSize;
    DWORD PointerToRawData;
} IMAGE_SECTION_HEADER;
```

`VirtualAddress`는 메모리에서의 상대위치인 RVA를 의미하고  
`PointerToRawData`는 파일에서의 해당 섹션이 어디에 배치되는지를 알려주는 값이다.   

---

## File Offset(RAW)을 구하는 방법

## 1. 해당 RVA가 어느 섹션에 속하는지 판단  

```text
Section.VirtualAddress(RVA) < RVA < Section.VirtualAddress(RVA) + Section.VirtualSize
``` 

---

## 2. RAW 계산 공식  
```text
RAW = (RVA - Section.VirtualAddress(RVA)) + Section.PointerToRawData
```

이 순서만 지킨다면 RAW를 구하는것에 큰 어려움이 없다.  
하지만 우리가 일반적으로 알고 있던 VA는 절대주소라고 이해하고 있을텐데  
NT_HEADER 구조체의 VirtualAddress는 RVA인데 이름 떄문에 VA를 의미하는줄 알고 한참 해맸었다.  

---

## 실습

대부분 자료를 찾아보면 이론적으로만 설명하고 임의로 부여를 해서 하기에 잘 이해가 되지 않았다.  
나는 Notepad.exe를 기준으로 직접 EntryPoint의 RAW를 찾아보도록 하였다.  
<figure>
  <img src="/assets/img/CS/RVATORAW/info.png" alt="PE View">
  <figcaption>PE View</figcaption>
</figure>


- AddressOfEntryPoint : **0x21860 (RVA)**  
- .text    
  - VirtualAddress(RVA) : **0x1000**  
  - VirtualSize : **0x223B8**  
  - PointerToRawData : **0x400**  

PE view를 통해 나는 이러한 정보들을 찾을수 있었고 이를 찾는 공식에 대입을 해보도록 하였다.  

---

## 공식 적용하기

## 1. 어느 섹션인지 판단하기  

```text
1000 < 21860 < 1000 + 223B8
```

해당 조건을 만족하므로 EntryPoint는 **.text 섹션**에 속한다는것을 알수있다.    

---

## 2. RAW 계산 공식  

```text
RAW = (21860- 1000) + 400 
RAW = 20C60
```
해당 계산을 통해 EntryPoint의 RAW는 **0x20C60**이라는 것을 알수있다.

---

## 덤프에서 직접 확인해보기

이제 계산된 File Offset을 직접 확인해보았다.

<figure>
  <img src="/assets/img/CS/RVATORAW/dump.png" alt="Dump">
  <figcaption>Dump</figcaption>
</figure>

파일 시점의 EntryPoint가 시작하는 위치의 덤프와 메모리 시점의 EntryPoint가 시작하는 위치의 덤프가 동일하다는것을 알수있게 된다.
  
또 한가지 알수 있는게 있는데 파일에서의 보는 EntryPoint위치와 디버거를 통해 보이는 EntryPoint주소가  
서로 다른 이유는 결국 ImageBase 때문이다.  

PE파일은 디스크에 있을 때와 메모리에 로드되었을때 ImageBase가 달라질 수 있는데  
이 차이 때문에 같은 RVA라도 VA로 계산했을때 서로 다른 절대주소가 나오게 된다.  

여기서 하나 해볼수 있는게 VA = ImageBase + RVA 라고 했었다.  
디버거에서의 ImageBase는 0x00950000이고 EntryPoint의 RVA는 0x21860 이다.  
즉 두 값을 더하게 되면 0X00971860이 VA이며 실제 사진에서의 주소와 같은 값을 나타낸다.  

---

## 마무리

RVA to RAW에서는 필드 이름때문에 너무 헛고생을 많이한것 같다.  
결국 중요한건 저 공식만 이해하고 있다면 큰 문제는 없는거 같다.  

나는 구조체의 VirtualAddress를 VA라고 착각해서 시간을 많이 낭비했는데  
이걸 정확히 RVA로 이해하고 나니 전체 구조가 깔끔하게 정리됐다.  

