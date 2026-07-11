---
title: '[Cheat Engine] Games: Step 2'
published: 2026-02-03
description: 'Cheat Engine Games Step 2에서 플레이어와 적 구조체를 구분하는 과정을 정리합니다.'
image: '/assets/img/CheatEngine/Games2/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
  - 'Games'
category: 'Cheat Engine'
draft: false
---


Games Step 2는 단순히 플레이어의 체력을 고정시키는 수준을 넘어서,  
**플레이어와 적의 구조체를 구분하고, 오직 적에게만 데미지가 들어가도록 코드를 조작하는 과정**을 다루고 있다.  
튜토리얼 파트에서 배웠던 "값 스캔 → 포인터 추적 → 코드 인젝션" 흐름을 실제 게임에 그대로 적용해보는 단계라고 보면 된다.

---

## 1. 게임 실행과 환경 파악

게임을 실행하면 플레이어와 두 개의 타깃(Target 1, Target 2)이 등장한다.  
각 개체는 체력을 가지고 있고, 총알 또는 충돌에 의해 체력이 깎이거나 파괴된다.
<figure>
  <img src="/assets/img/CheatEngine/Games2/1.png" alt="게임 실행 시 화면">
  <figcaption>게임 실행 시 화면</figcaption>
</figure>


이 단계에서 우리가 목표로 하는 것은 다음과 같다.

- 플레이어와 적의 체력 주소를 모두 찾아낸다.  
- 체력을 감소시키는 공통 코드를 분석한다.  
- 구조체 내부에서 "누가 맞았는지" 구분할 수 있는 필드를 찾는다.  
- 플레이어는 데미지를 받지 않고, 적만 데미지를 받도록 코드 인젝션을 작성한다.

---

## 2. 플레이어 체력 주소 찾기

먼저 화면에 표시된 플레이어 체력을 기준으로 Exact Value 검색을 진행한다.  
체력이 변할 때마다 값을 갱신하면서 Next Scan을 반복하면, 플레이어 체력 주소 하나만 남게 된다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/2.png" alt="플레이어 체력 검색 후 테이블로 내린 결과">
  <figcaption>플레이어 체력 검색 후 테이블로 내린 결과</figcaption>
</figure>

이제 이 주소에 실제로 어떤 코드가 접근하는지 추적해볼 차례다.

---

## 3. Find out what writes로 체력 감소 코드 확인

플레이어 체력 주소를 오른쪽 클릭하고 **Find out what writes to this address**를 실행한다.  
그 상태에서 플레이어가 데미지를 받도록 일부러 맞아주면, 해당 주소에 값을 쓰는 명령어가 캡처된다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/3.png" alt="플레이어 체력 주소에 대한 Find out what writes 결과">
  <figcaption>플레이어 체력 주소에 대한 Find out what writes 결과</figcaption>
</figure>

이때 어셈블리를 보면 이런 형태의 명령어를 확인할 수 있다.

```asm
sub [eax+50], edx
```

위 어셈을 정리하자면 이런식으로 추측이 가능하다.

- `eax+50` 위치에는 체력 값이 저장되어 있다.  
- `edx`에는 이번에 감소될 데미지 값이 들어 있다.  
- 이 명령이 실행될 때마다 현재 체력에서 데미지만큼 빼는 동작이 이루어진다.

즉, 체력 자체는 `[eax+50]`에 있고, 데미지는 `edx`를 통해 전달되는 구조라고 이해할 수 있다.

---

## 4. sub [eax+50], edx가 접근하는 모든 주소 확인

이번에는 이 명령어가 **어떤 객체들의 체력에 영향을 주는지**를 확인해야 한다.  
이를 위해 해당 명령어에 대해 **Find out what addresses this instruction accesses** 기능을 실행한다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/6.png" alt="Find out what addresses this instruction accesses 실행 직전">
  <figcaption>Find out what addresses this instruction accesses 실행 직전</figcaption>
</figure>

이 상태에서 플레이어와 Target 1, Target 2가 서로 공격을 주고받도록 하면서 관찰하면  
`sub [eax+50], edx` 명령이 여러 개의 주소에 차례로 접근한다는 것을 확인할 수 있다.

---

## 5. 플레이어와 적의 체력 주소 구분

Find out what addresses 결과 창에는 세 개의 주요 주소가 나타난다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/7.png" alt="플레이어와 적의 체력 주소 확인">
  <figcaption>플레이어와 적의 체력 주소 확인</figcaption>
</figure>

다음과 같은 주소들이 등장한다.

- `01AEE910`  
- `01AECD80`  
- `01AEEC20`  

플레이어와 두 개의 적이 각각 하나의 체력 주소를 가지는 형태라고 볼 수 있다.  
공격 상황을 바꿔가며 어떤 주소가 언제 줄어드는지 확인하면 각 주소가 어떤 객체에 해당하는지 감을 잡을 수 있다.

여기까지 정리하면 다음과 같이 이해할 수 있다.

- `sub [eax+50], edx`는 단일 객체 전용 코드가 아니라, `eax`가 가리키는 구조체의 체력을 줄이는 것을 알수있게 된다.  
- `eax` 값이 바뀌는 것에 따라 플레이어가 맞을 수도 있고, Target 1이나 Target 2가 맞을 수도 있다.  
- 결국 "누가 맞았는지"는 `eax`가 가리키는 구조체 내부의 필드를 보고 구분해야 한다.

---

## 6. 구조체 분석 — Dissect Data/Structures

이제 `eax`가 가리키는 구조체 내부에서 **팀/타입을 구분하는 필드**를 찾아야 한다.  
체력 감소 시점에 `eax` 레지스터에 들어 있는 주소를 기준으로 **Dissect Data/Structures** 기능을 실행한다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/8.png" alt="Dissect Data/Structures 실행 직전">
  <figcaption>Dissect Data/Structures 실행 직전</figcaption>
</figure>

구조체를 열어보면 다음과 같은 형태의 데이터가 나열되어 있는 것을 볼 수 있다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/9.png" alt="Dissect Data/Structures 결과">
  <figcaption>Dissect Data/Structures 결과</figcaption>
</figure>

각 필드를 하나씩 바꿔보거나, 플레이어와 적의 구조체를 비교해보면 특정 offset에서 차이가 난다는 것을 알 수 있다.  
예를 들어 다음과 같이 정리할 수 있다.

- `+0x50`: 체력 값 ( 앞에서 본 `[eax+50]` )  
- `+0x5C`: 팀 값 ( 플레이어는 0, 적은 1 )
- 그 외 일부 offset: 팀 정보 또는 역할 구분 값으로 사용되는 필드

플레이어와 적의 구조체를 나란히 비교해보면  
0x5C 주소에서 플레이어와 적이 서로 다른 값을 가지고 있는 패턴을 확인할 수 있다.  
이 필드를 이용해 플레이어와 적을 구분할 수 있다.

---

## 7. 코드 인젝션 스크립트 작성

이제 `sub [eax+50], edx` 명령이 실행될 때 `eax`가 가리키는 구조체의 팀 값에 따라 데미지를 줄지 말지 결정하면 된다.

이를 위해 CE에서 **Code Injection** 템플릿을 생성하고, 원래 명령어를 후킹한 뒤  
조건을 추가하는 방식으로 스크립트를 작성한다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/10.png" alt="코드 인젝션 스크립트 작성">
  <figcaption>코드 인젝션 스크립트 작성</figcaption>
</figure>

기본적인 흐름은 다음과 같이 잡을 수 있다.
여기서 중요한 부분은 다음 두 가지다.

- 체력 감소는 항상 `sub [eax+50], edx`로 이루어진다.  
- 누가 맞았는지는 구조체 내 팀/역할 필드를 통해 구분한다.

팀 offset의 실제 값은 게임마다 다르지만, Dissect Data/Structures 결과를 보며 직접 비교하면 쉽게 찾을 수 있다.

---

## 8. 스크립트 적용 및 동작 확인

스크립트를 적용한 뒤 게임으로 돌아가서 다음과 같이 테스트해본다.

<figure>
  <img src="/assets/img/CheatEngine/Games2/11.png" alt="Step 2 완료">
  <figcaption>Step 2 완료</figcaption>
</figure>

정상적으로 동작한다면 다음과 같은 결과를 확인할 수 있다.  
플레이어 체력은 공격을 받아도 감소되지 않고 그대로 유지가 되며  
적의 체력만 줄어들고, 두 타겟 모두 제거할수 있게 되어 Next 조건이 만족하게 된다.

---

## 마무리

Games Step 2는 **공용 데미지 처리 루틴을 후킹하고, 구조체 기반으로 대상을 선별하여  데미지를 선택적으로 적용하는 과정**을 알수 있게된다.

이 과정을 통해 자연스럽게 공용 처리 루틴이 어떻게 동작하는지,  
구조체 기반 게임 설계에서 팀/역할/체력과 같은 필드가 어떻게 연결되는지,  
조건 분기를 이용해 원하는 대상에게만 주는 코드인젝션 기법을 익힐수 있게 되었다.
