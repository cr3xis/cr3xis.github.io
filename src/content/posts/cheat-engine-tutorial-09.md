---
title: '[Cheat Engine] Tutorial: Step 9'
published: 2026-01-24
description: 'Cheat Engine Tutorial Step 9의 구조체 구분과 코드 인젝션 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Step9/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
category: 'Cheat Engine'
draft: false
---


Step 9에서는 지금까지의 모든 내용을 종합하여  
**나와 팀·적의 체력을 구분한 뒤, 오직 적의 체력만 감소하도록 코드 인젝션을 적용하는 과정**을 다룬다.  
이전 단계들보다 확실히 실전적인 구조이며, 게임 구조를 이해하는 능력과 코드 조작 능력을 동시에 요구한다.

튜토리얼의 목표는 다음과 같다:

- 모든 플레이어의 체력을 찾는다.  
- 체력 감소 코드를 분석해 어떤 플레이어에 적용되는지 파악한다.  
- 팀 정보(아군/적군)를 확인한다. 
- 적군에게만 체력 감소가 적용되도록 코드 인젝션을 작성한다.

---

## 1. 모든 플레이어 체력 찾기

우선 튜토리얼 창에 있는 모든 플레이어 목록을 확인한 후 각 플레이어의 체력을 CE에서 검색해준다.
<figure>
  <img src="/assets/img/CheatEngine/Step9/1.png" alt="각 플레이어들의 체력 주소">
  <figcaption>각 플레이어들의 체력 주소</figcaption>
</figure>


한 번 검색으로 여러 개의 Health 값이 나오게 되며, 이를 하나하나 테이블에 내리면서 누구의 체력인지 매칭한다.

---

## 2. 특정 플레이어의 체력에 접근하는 코드 분석

체력 주소 중 하나를 선택하여 **Find out what writes to this address**를 수행하면  
해당 플레이어의 체력을 조작하는 코드를 확인할 수 있다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/2.png" alt="&quot;Find out what writes to this address&quot;로 Dave의 체력 접근 코드 확인">
  <figcaption>&quot;Find out what writes to this address&quot;로 Dave의 체력 접근 코드 확인</figcaption>
</figure>

이 코드는 Hit me 버튼을 눌렀을 때 Dave의 체력을 감소시키는 역할을 수행한다.

---

## 3. 명령어 구조 분석 — 어떤 필드가 어떤 정보를 담고 있는가

이제 해당 명령어의 주소를 기준으로 **Show Disassembler**를 열어 레지스터와 구조체 필드를 분석한다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/3.png" alt="Show Disassembler로 명령어 분석">
  <figcaption>Show Disassembler로 명령어 분석</figcaption>
</figure>

Health 감소 로직은 보통 `mov [reg+offset], value` 형태로 구성된다.  
여기서 reg가 가리키는 구조체 내부에 플레이어의 정보(팀, 체력 등)가 저장되어 있다.  
따라서 이 구조체를 분석해야 정확한 조건 분기가 가능해진다.

---

## 4. 플레이어 구조체 분석 — 팀 정보 찾기

구조체 내부 필드들을 확인하기 위해 **Dissect Data/Structures** 기능을 사용한다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/4.png" alt="Dissect Data/Structures 실행 직전">
  <figcaption>Dissect Data/Structures 실행 직전</figcaption>
</figure>

이 기능을 통해 reg가 가리키는 구조체의 필드들이 정리되어 나타난다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/5.png" alt="Dissect Data/Structures로 팀 정보 확인">
  <figcaption>Dissect Data/Structures로 팀 정보 확인</figcaption>
</figure>

여기에서 플레이어의 팀 정보를 포함한 필드(offset)가 확인되며,  
이를 기준으로 "아군인지, 적군인지"를 구분할 수 있다.

---

## 5. 코드 인젝션 템플릿 생성

체력 감소 명령어를 선택한 뒤 **Code Injection** 기능을 통해 인젝션용 템플릿을 생성한다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/6.png" alt="코드 인젝션 템플릿 추가">
  <figcaption>코드 인젝션 템플릿 추가</figcaption>
</figure>

이 템플릿 안에 조건문을 작성하여 적군에게만 체력이 감소하도록 조작할 수 있다.

---

## 6. 코드 인젝션 스크립트 작성

생성된 스크립트에는 아래 조건을 반영한다.

- 플레이어 구조체의 팀 정보 필드를 확인한다.  
- 팀값이 "적군"을 의미하는 값일 때만 원래의 감소 코드를 실행한다.  
- 나머지(아군, 나 자신)는 체력이 감소하지 않도록 우회한다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/7.png" alt="코드 인젝션 스크립트 작성">
  <figcaption>코드 인젝션 스크립트 작성</figcaption>
</figure>

이 부분이 Step 9의 핵심이며, 정확한 오프셋과 원본 코드를 이해해야만 제대로 동작한다.

---

## 7. 스크립트 적용 및 테스트

작성한 스크립트를 적용한 뒤 Hit me 버튼을 눌러 테스트해 보면  
적군의 체력만 줄어들고 아군과 나의 체력은 그대로 유지된다.

<figure>
  <img src="/assets/img/CheatEngine/Step9/8.png" alt="스크립트 적용 후 테스트 및 Next 버튼 활성화">
  <figcaption>스크립트 적용 후 테스트 및 Next 버튼 활성화</figcaption>
</figure>

정상적으로 동작하면 Next 버튼이 활성화된다.

---

## 마무리

Step 9는 CE 튜토리얼 중 가장 실제 게임 구조에 가까운 내용을 다룬다.  
플레이어 구조체를 분석하고, 그 안의 팀/체력/상태 등 다양한 정보를 이해하며  
그에 따라 조건 분기를 만들어내는 과정은 게임 메모리 해킹에서 매우 자주 사용되는 기법이다.

이 단계를 정확히 이해하면 다중 포인터, 구조체 기반 로직, 조건부 코드 인젝션 등  
중급 난이도 이상의 작업도 훨씬 자연스럽게 다룰 수 있게 된다.
