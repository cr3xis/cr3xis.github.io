---
title: '[Dreamhack] Rev-basic-1 Writeup'
published: 2026-01-03
description: 'Dreamhack Rev-basic-1의 문자 단위 비교 로직과 정답 도출 과정을 정리합니다.'
image: '/assets/img/Rev-basic/Rev-basic-1/Main_asm.png'
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


Rev-basic-1 문제는 문자열을 한 글자씩 순차적으로 검사하는 구조로 되어 있다.  
프로그램은 입력된 문자열을 특정 규칙에 따라 하나씩 비교하며, 모든 비교가 조건을 만족할 때만 Correct를 출력한다.  
조금 더 세밀하게 코드를 따라가야 하기 때문에, Rev-basic-0보다 한 단계 더 분석적인 사고가 필요한 문제라고 볼 수 있다.

---

## 문제 파일

<ul class="file-list">
  <li>
    <i class="fa-solid fa-file"></i> <strong>File Download:</strong> 
    <a href="https://dreamhack.io/wargame/challenges/15" target="_blank">Rev-basic-1</a>
  </li>
</ul>

---

## 1. 프로그램 실행 화면

프로그램을 실행하면 Rev-basic-0과 동일하게 문자열 입력을 요구하는 단순한 형태로 시작된다.

```text
C:\Users\cryptonite7777>chall1.exe
input :
```

입력한 문자열이 조건을 통과하면 Correct, 아니면 Wrong을 출력하는 구조다.

---

## 2. Main 함수 흐름

_x64 Main_
<figure>
  <img src="/assets/img/Rev-basic/Rev-basic-1/Main_asm.png" alt="IDA에서 확인한 main 함수">
  <figcaption>IDA에서 확인한 main 함수</figcaption>
</figure>


입력값을 받아 특정 조건에 따라 검증하는 구조라는 점에서는 Rev-basic-0과 동일하다.  
입력된 문자열은 버퍼에 저장된 뒤, 내부의 sub 함수를 호출하여 비교 과정을 진행하게 된다.

---

## 3. 비교 로직 분석

아래 이미지는 x64dbg로 살펴본 sub 함수 내부의 핵심 부분이다.

<figure>
  <img src="/assets/img/Rev-basic/Rev-basic-1/sub_function.png" alt="문자 비교를 수행하는 검증 함수">
  <figcaption>문자 비교를 수행하는 검증 함수</figcaption>
</figure>
_x64 Sub_

어셈블리를 따라가다 보면 반복문 형태로 문자열의 각 문자를 하나씩 가져와 비교하는 흐름이라는 것을 알 수 있다.  
rax 또는 eax 레지스터가 루프 인덱스처럼 사용되고, 입력 문자열 주소(rcx)에 인덱스를 더한 뒤 해당 문자를 가져와 비교한다.

입력된 문자를 한 글자씩 읽어 하드코딩된 값과 비교하는 방식으로 동작하며,  
중간에 하나라도 조건이 맞지 않으면 Wrong을 출력하고 종료된다.  
모든 문자가 조건에 맞아야만 Correct가 출력되는 구조다.

---

## 4. 비교 대상 문자열 확인

함수 내부에서 비교되는 값들을 확인해보면 최종적으로 요구되는 문자열을 유추할 수 있다.  
비교는 문자 단위로 이루어지지만, 전체 흐름을 따라가면 아래와 같은 플래그 형태로 구성되어 있음을 알 수 있다.

```text
Compar3_the_ch4ract3r
```

입력 문자열이 위 플래그와 정확히 일치해야 프로그램은 Correct를 출력한다.

---

## FLAG

```text
DH{Compar3_the_ch4ract3r}
```

---

## 마무리

Rev-basic-1 문제는 문자열을 순차적으로 비교하는 기본적인 루프 구조를 이해하는 데 도움이 되는 문제다.  
어셈블리에서 루프가 어떻게 구성되고, 인덱스 증가와 비교가 어떤 방식으로 이루어지는지를 직접 확인할 수 있다.  
이 과정을 익혀두면 이후 더 복잡한 문자열 처리 로직을 분석할 때도 큰 도움이 될 것이다.
