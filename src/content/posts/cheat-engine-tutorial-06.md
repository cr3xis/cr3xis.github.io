---
title: '[Cheat Engine] Tutorial: Step 6'
published: 2026-01-18
description: 'Cheat Engine Tutorial Step 6의 포인터 추적과 테이블 등록 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Step6/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 6에서는 단순히 값만 수정하는 것이 아니라,  
**값을 가리키는 포인터 구조를 추적해서 올바른 방식으로 수정하는 과정**을 다루게 된다.  
튜토리얼에서는 값의 주소가 고정되어 있지 않기 때문에, 값 자체만 수정하면 금방 다시 바뀌어버린다.  
따라서 이번 단계에서는 **포인터를 직접 찾아 CE 테이블에 등록하는 것**이 목표다.

---

## 1. 초기 값 검색

튜토리얼 창에서 보이는 값은 처음에 100으로 시작한다.  
따라서 CE에서도 100을 기준으로 첫 스캔을 진행한다.
<figure>
  <img src="/assets/img/CheatEngine/Step6/1.png" alt="초기 값 100 검색">
  <figcaption>초기 값 100 검색</figcaption>
</figure>


검색 결과가 여러 개 나오지만, 다음 단계에서 값이 변하는 순간을 이용하여 후보를 좁혀나가게 된다.

---

## 2. Change value 눌러 값 변경 후 다시 검색하기

튜토리얼 창에서 **Change value** 버튼을 누르면 값이 920으로 변경된다.  
CE에서도 검색 값을 920으로 바꾸고 *Next Scan*을 수행한다.

<figure>
  <img src="/assets/img/CheatEngine/Step6/2.png" alt="&quot;Change value&quot; 클릭 후 값 920으로 검색">
  <figcaption>&quot;Change value&quot; 클릭 후 값 920으로 검색</figcaption>
</figure>

여러 번 반복하면 결국 한 개의 주소만 남게 된다. 해당 주소를 테이블에 추가하여 다음 과정을 이어간다.

---

## 3. 해당 주소에 어떤 코드가 접근하는지 확인하기

남은 주소를 테이블에 추가한 뒤 오른쪽버튼을 클릭하여 **Find out what writes to this address**를 선택한다.

<figure>
  <img src="/assets/img/CheatEngine/Step6/2-1.png" alt="&quot;Find out what writes to this address&quot; 선택">
  <figcaption>&quot;Find out what writes to this address&quot; 선택</figcaption>
</figure>

이 상태에서는 아직 아무 코드도 보이지 않는다.  
값이 변경되는 기록이 남기에 다시 Change value 버튼을 눌러야 한다.

---

## 4. Change value 실행 시 나타나는 명령어 확인하기

튜토리얼에서 Change value를 다시 눌러보면 해당 주소에 값을 쓰는 명령어가 추적 창에 나타난다.

<figure>
  <img src="/assets/img/CheatEngine/Step6/3.png" alt="&quot;Change value&quot; 클릭 후 나타난 코드">
  <figcaption>&quot;Change value&quot; 클릭 후 나타난 코드</figcaption>
</figure>

명령어 형태를 보면 `[edx]`에 값을 쓰고 있다는 것을 알 수 있다.  
즉, `edx`가 이 값의 실제 주소를 가리키고 있다는 의미다.  
이제 **edx가 어떤 주소를 가리키는지**를 따라가는 것이 포인터 분석의 핵심이다.

---

## 5. 포인터 주소 검색하기

Advanced Options에 표시된 edx 값(실제 주소)을 복사한 뒤 CE에서 Hex 옵션을 체크하고 이 주소를 그대로 검색한다.

<figure>
  <img src="/assets/img/CheatEngine/Step6/4.png" alt="포인터 주소 검색 및 테이블 추가">
  <figcaption>포인터 주소 검색 및 테이블 추가</figcaption>
</figure>

검색되는 값들 중 exe 모듈 영역에 해당하는 녹색 주소가 실제 포인터일 가능성이 높다.  
해당 주소를 테이블에 추가하여 구조를 살펴본다.

---

## 6. 포인터 등록 및 오프셋 입력

테이블에 추가된 포인터 항목을 보면 해당 주소가 아까 찾았던 값의 위치를 가리키고 있음을 확인할 수 있다.  
이제 이를 CE 포인터로 등록하기 위해 **Add Address Manually → Pointer 체크** 후 아래와 같이 입력한다.

<figure>
  <img src="/assets/img/CheatEngine/Step6/5.png" alt="포인터 주소와 오프셋 추가">
  <figcaption>포인터 주소와 오프셋 추가</figcaption>
</figure>

명령어에서 `[edx]` 형태였기 때문에 오프셋은 0이다.

---

## 7. 포인터 값 수정 및 고정

포인터를 등록한 뒤 Value를 5000으로 수정하고 Active 체크를 하게되면 값이 고정된다.

<figure>
  <img src="/assets/img/CheatEngine/Step6/6.png" alt="포인터 값 5000으로 설정 및 고정">
  <figcaption>포인터 값 5000으로 설정 및 고정</figcaption>
</figure>

이제 Change pointer 버튼을 눌러도 값이 변경되지 않는다.  
포인터 기반으로 주소를 추적했기 때문에 튜토리얼에서 값이 이동해도 올바르게 따라가게 된다.

---

## 마무리

Step 6는 단순 주소 수정에서 벗어나  **값이 어떻게 가리켜지고 이동하는지**,  
그리고 **포인터를 통해 안정적인 접근이 왜 필요한지**를 알려주는 단계다.  
이 과정을 정확히 이해하면 이후 다단계 포인터 구조나 동적 메모리 구조도 훨씬 쉽게 분석할 수 있다.
