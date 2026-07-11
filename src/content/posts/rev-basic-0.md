---
title: '[Dreamhack] Rev-basic-0 Writeup'
published: 2026-01-01
description: 'Dreamhack Rev-basic-0의 문자열 검증 흐름을 IDA와 디버거로 분석합니다.'
image: '/assets/img/Rev-basic/Rev-basic-0/Main_asm.png'
tags:
  - 'Reversing'
  - 'dreamhack'
  - 'wargame'
  - 'writeup'
  - 'ida'
  - 'x64dbg'
category: 'Reverse Engineering'
draft: false
---


Rev-basic-0 문제는 입력된 문자열이 특정 조건과 일치하는지를 검사하는 형태로 구성되어 있다.  
겉으로 보기엔 간단한 프로그램이지만, 내부의 흐름을 따라가다 보면 문자열 비교가 어떤 방식으로 이루어지는지  
자연스럽게 파악할 수 있다.

---

## 문제 파일

<ul class="file-list">
  <li>
    <i class="fa-solid fa-file"></i> <strong>File Download:</strong> 
    <a href="https://dreamhack.io/wargame/challenges/14" target="_blank">Rev-basic-0</a>
  </li>
</ul>

---

## 1. 프로그램 실행 화면

프로그램을 실행하면 아래와 같이 입력을 기다리는 단순한 형태로 시작된다.

```text
C:\Users\cryptonite7777>chall0.exe
input :
```

이런 형태의 프로그램은 보통 입력된 값을 특정 기준과 비교한 뒤 결과를 출력하는 구조를 가지고 있다.

---

## 2. Main 흐름 살펴보기

아래 이미지는 분석 과정에서 확인한 메인 함수의 흐름이다.

_x64 Main_
<figure>
  <img src="/assets/img/Rev-basic/Rev-basic-0/Main_asm.png" alt="IDA에서 확인한 main 함수">
  <figcaption>IDA에서 확인한 main 함수</figcaption>
</figure>


디컴파일된 코드를 보면 전체적인 동작 방식이 단순하다는 것을 알 수 있다.

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char v4[256];

  memset(v4, 0, sizeof(v4));
  sub_1400013E0("Input : ", argv, envp);
  sub_140001440("%256s", v4);
  if ((unsigned int)sub_140001000(v4))
    puts("Correct");
  else
    puts("Wrong");
  return 0;
}
```

입력은 `v4`에 저장되고, 이후 `sub_140001000` 함수를 통해 문자열 검증 과정을 거치게 된다.  
반환값이 참이면 Correct, 거짓이면 Wrong을 출력하는 구조다.  
즉, 실제 비교 로직은 모두 `sub_140001000` 내부에 있다.

---

## 3. sub_140001000 함수 분석

아래 이미지는 해당 함수 내부의 핵심 부분을 보여준다.

<figure>
  <img src="/assets/img/Rev-basic/Rev-basic-0/Sub_asm.png" alt="IDA에서 확인한 검증 함수">
  <figcaption>IDA에서 확인한 검증 함수</figcaption>
</figure>
_x64 Sub_

어셈블리를 따라가다 보면 입력 문자열과 특정 하드코딩된 문자열을 직접 비교하는 구조라는 것을 확인할 수 있다.  
RCX에는 내가 입력한 문자열이 들어가고, RDX에는 비교할 하드코딩된 문자열이 들어간다.  
이후 strcmp 함수를 호출하여 서로 값이 같으면 Correct, 다르면 Wrong을 출력하는 방식으로 동작한다.

아래 어셈블리 부분이 비교 로직의 핵심이다.

```asm
lea rdx,qword ptr ds:[7FF749EB2220]  ; 하드코딩된 문자열 주소
mov rcx,qword ptr ss:[rsp+40]        ; 입력 문자열 주소
call <JMP.&strcmp>
```

strcmp 결과가 같은 경우에만 프로그램이 Correct를 출력하게 된다.

---

## 4. 비교 문자열 확인

rdx가 가리키는 주소를 확인해보면 프로그램이 원하는 문자열을 알 수 있다.

```text
Compar3_the_str1ng
```

사용자가 입력한 문자열이 이 값과 정확히 일치할 때만 Correct가 출력된다.

---

## FLAG

```text
DH{Compar3_the_str1ng}
```

---

## 마무리

Rev-basic-0은 구조가 단순하지만 리버싱에서 자주 등장하는 문자열 비교 패턴을 이해하기에 좋은 문제다.  
입력된 문자열이 어떻게 처리되고 어떤 조건을 만족해야 하는지를 확인하는 과정에서 흐름을 읽는 능력을 기를 수 있다.  
이 문제를 기반으로 다음 단계의 문제를 분석하면 한층 더 수월하게 따라갈 수 있다.
