---
title: '[Dreamhack] Rev-basic-2 Writeup'
published: 2026-01-05
description: 'Dreamhack Rev-basic-2의 배열 기반 문자열 검증 구조를 분석합니다.'
image: '/assets/img/Rev-basic/Rev-basic-2/Main_asm.png'
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


Rev-basic-2 문제는 입력된 문자열을 배열 형태로 저장된 값들과 비교하는 구조를 가지고 있다.  
Rev-basic-1에서 한 글자씩 비교하는 방식이었다면, 이번 문제는 배열 인덱싱과 반복 구조를 조금 더 복잡하게 다루고 있어서 한 단계 심화된 형태라고 볼 수 있다.

---

## 문제 파일

<ul class="file-list">
  <li>
    <i class="fa-solid fa-file"></i> <strong>File Download:</strong> 
    <a href="https://dreamhack.io/wargame/challenges/16" target="_blank">Rev-basic-2</a>
  </li>
</ul>

---

## 1. 프로그램 실행 화면

프로그램을 실행하면 Rev-basic 시리즈와 같은 형태로 입력을 받는 화면이 먼저 나타난다.

```text
C:\Users\cryptonite7777>chall2.exe
input :
```

입력된 값을 내부 배열과 비교하여 조건을 모두 만족시키면 Correct를 출력한다.

---

## 2. Main 함수 흐름

_x64 Main_
<figure>
  <img src="/assets/img/Rev-basic/Rev-basic-2/Main_asm.png" alt="IDA에서 확인한 main 함수">
  <figcaption>IDA에서 확인한 main 함수</figcaption>
</figure>



메인 함수에서 입력된 문자열을 버퍼에 저장한 뒤, sub 함수로 넘겨 검증하는 구조는 동일하다.  
rev-basic 시리즈의 공통된 흐름으로 볼 수 있다.

---

## 3. 비교 로직 분석

아래 이미지는 sub 함수 내부의 핵심 비교 과정이다.

<figure>
  <img src="/assets/img/Rev-basic/Rev-basic-2/Sub_asm.png" alt="배열 값을 비교하는 검증 함수">
  <figcaption>배열 값을 비교하는 검증 함수</figcaption>
</figure>
_x64 Sub_

코드를 자세히 보면 배열 기반 반복 구조라는 것을 확인할 수 있다.  
루프 변수를 증가시키며 배열의 특정 인덱스로 접근하고,  
입력된 문자열의 각 문자를 가져와 배열에 저장된 값과 비교하는 방식으로 동작한다.

입력 문자열은 1바이트씩 가져오며,  
배열은 4바이트 단위로 정렬되어 있기 때문에 인덱스 계산 시 `i * 4` 형태로 접근하는 흐름을 보게 된다.  
이 계산 방식만 이해하면 전체 비교 구조는 어렵지 않게 따라갈 수 있다.

---

## 4. 비교 배열 분석

배열에 저장된 값을 따라가다 보면 최종적으로 어떤 문자열이 필요한지 확인할 수 있다.  
문자 단위로 비교되지만 배열 구조를 그대로 따라가면 다음과 같은 플래그 형태가 구성된다.

```text
Comp4re_the_arr4y
```

---

## FLAG

```text
DH{Comp4re_the_arr4y}
```

---

## 마무리

Rev-basic-2 문제는 배열 기반 문자열 비교 흐름을 이해하는 데 도움이 되는 문제다.  
루프 구조, 인덱스 계산, 배열 접근 방식 등을 자연스럽게 익힐 수 있으며  
이후 더 복잡한 문자열 변환 문제를 분석할 때 기반이 되는 개념들이다.  
Rev-basic-0과 1을 기반으로 차근차근 접근하면 무리 없이 해결할 수 있는 문제다.
