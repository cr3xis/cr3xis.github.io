---
title: '[Cheat Engine] Tutorial: Step 7'
published: 2026-01-20
description: 'Cheat Engine Tutorial Step 7의 코드 인젝션과 Auto Assembler 사용법을 정리합니다.'
image: '/assets/img/CheatEngine/Step7/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 7에서는 **코드 인젝션(Code Injection)**을 이용해 기존 프로그램의 동작을 직접 수정하는 과정을 다루게 된다.  
이전 단계들은 특정 값을 찾고 수정하는 방식이었지만, 이번 단계는 **게임 로직 자체를 원하는 방향으로 바꾼다**는 점에서  
이전과 확실히 차별화된다.

튜토리얼에서 요구하는 것은  **체력이 감소하는 대신 증가하도록 코드를 수정하는 것**이며,  
이를 위해 CE의 **Auto Assembler** 기능을 사용하게 된다.

---

## 1. 초기 값 검색

튜토리얼 창에서 보이는 값은 처음에 100이며, 이를 기준으로 CE에서 Exact Value 스캔을 진행한다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/1.png" alt="초기 값 100 검색">
  <figcaption>초기 값 100 검색</figcaption>
</figure>

---

## 2. 값이 변경된 뒤 다시 스캔하기

Hit me 버튼을 누르면 값이 99가 된다.  
CE에서도 값을 99로 바꿔 Next Scan을 수행한다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/2.png" alt="&quot;Hit me&quot; 클릭 후 값 99로 검색">
  <figcaption>&quot;Hit me&quot; 클릭 후 값 99로 검색</figcaption>
</figure>

이 과정을 통해 Health 값을 수정하는 메모리 주소를 특정하게 된다.

---

## 3. 해당 주소에 값을 쓰는 코드 확인하기

찾은 주소를 테이블에 추가한 뒤 **Find out what writes to this address** 메뉴를 통해  
Health 값을 감소시키는 명령어를 추적한다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/3.png" alt="&quot;Find out what writes to this address&quot;로 명령어 확인">
  <figcaption>&quot;Find out what writes to this address&quot;로 명령어 확인</figcaption>
</figure>

이 명령어가 실행되는 순간에 Health가 감소하므로  
이 코드를 수정하면 체력을 증가시키는 로직으로 바꿀 수 있다.

---

## 4. 메모리 뷰에서 해당 코드 위치 확인하기

Show Disassembler를 통해 해당 명령의 메모리 위치를 확인한다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/4.png" alt="Show Disassembler로 메모리 뷰 확인">
  <figcaption>Show Disassembler로 메모리 뷰 확인</figcaption>
</figure>

이 위치가 코드 인젝션을 적용할 지점이다.

---

## 5. 코드 인젝션 템플릿 열기

해당 명령어를 선택한 상태에서 **"Replace with code that does nothing"** 또는 **"Code Injection"** 기능을 선택한다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/5.png" alt="Code Injection 템플릿 선택 직전">
  <figcaption>Code Injection 템플릿 선택 직전</figcaption>
</figure>

이 기능은 CE가 자동으로 인젝션용 코드 템플릿을 생성해준다.

---

## 6. Auto Assembler 템플릿 생성

템플릿이 생성되면 아래와 같은 기본 구조가 만들어진다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/6.png" alt="Code Injection 기본 템플릿">
  <figcaption>Code Injection 기본 템플릿</figcaption>
</figure>

이제 기존 코드를 내가 원하는대로 커스터마이징이 가능해진다.

---

## 7. 체력 감소 코드를 증가 코드로 수정하기

원래 코드는 체력을 감소시키는 명령어였다.  
해당 줄을 주석 처리하고 그 자리에 증가 코드를 작성한다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/7.png" alt="원래 코드 주석 처리 후 증가 코드 추가">
  <figcaption>원래 코드 주석 처리 후 증가 코드 추가</figcaption>
</figure>

이렇게 하면 Hit me 버튼을 눌러도 Health가 줄지 않고 오히려 계속 증가하게 된다.

---

## 8. 코드가 정상적으로 적용되었는지 확인하기

코드를 적용한 뒤 다시 메모리 뷰를 보면 인젝션된 코드가 정상적으로 삽입된 것을 확인할 수 있다.

<figure>
  <img src="/assets/img/CheatEngine/Step7/8.png" alt="코드 주입 후 변경된 메모리 뷰">
  <figcaption>코드 주입 후 변경된 메모리 뷰</figcaption>
</figure>

튜토리얼 창에서 Hit me 버튼을 눌러보면 이제 Health 값이 감소하지 않고 증가한다.

---

## 마무리

Step 7은 이전 단계들과 달리 **프로그램의 동작을 직접 조작하는 단계**다.  
Code Injection을 통해 실제 게임 로직을 원하는 방향으로 바꾸는 과정은 실전에서 매우 자주 활용되는 기법이기도 하다.

이 단계만 정확히 이해해도 CE의 Auto Assembler 기능을 활용하는 데 큰 어려움이 없어지며  
이후의 더 복잡한 메모리 조작 과정에도 자연스럽게 응용할 수 있다.
