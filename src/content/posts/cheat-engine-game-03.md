---
title: '[Cheat Engine] Games: Step 3'
published: 2026-02-05
description: 'Cheat Engine Games Step 3에서 생존 상태와 충돌 처리 코드를 분석합니다.'
image: '/assets/img/CheatEngine/Games3/1.png'
tags:
  - 'Reversing'
  - 'CheatEngine'
  - 'Tutorial'
  - 'Games'
category: 'Cheat Engine'
draft: false
---


Games Step 3는 튜토리얼에서 배웠던 내용을 실제 게임 상황에 가져와  
**충돌 판정과 생존 여부를 직접 건드려 보는 단계**다.  

이 스테이지의 목표는 단순하다.  
우리는 게임이 내부적으로 사용하는 **생존 상태 플래그와 충돌 처리 코드**를 건드려 조건을 우회해서 클리어해 본다.

---

## 1. 게임 진행 방식 살펴보기

게임을 실행하면 캐릭터와 여러 발판이 보인다.  
모든 빨간 발판을 밟아 초록 발판으로 변경 시킨뒤 문을 통과하면 게임이 통과되는 구조이지만  
모든 발판을 초록 발판으로 변경한다 한들 장애물들이 문을 가로막아 사실상 우회없이는 클리어가 불가능한 단계이다.
<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/1.png" alt="게임 실행 시 화면" style="display: block; margin: 0 auto;">
  <figcaption>게임 실행 시 화면</figcaption>
</figure>


발판에 대한 정보와 생존상태에 대한 정보는 게임 상에서 보여지는 정보가 없으므로  
처음부터 Unknown Initial Value를 사용하는 편이 자연스럽다.

---

## 2. Unknown Initial Value로 생존 상태 스캔

생존 여부를 직접 보여주는 값이 없기 때문에 Scan Type을 **Unknown Initial Value**로 두고 첫 스캔을 진행한다.  

그 이후에는 캐릭터를 일부러 여러 번 죽였다가 다시 시작하면서  

- 살아 있는 동안에는 **Unchanged Value**  
- 죽고 난 뒤에는 **Changed Value**  

이 두 가지를 번갈아 사용해 필터링한다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/2.png" alt="Unknown Initial Value로 생존 값 검색" style="display: block; margin: 0 auto;">
  <figcaption>Unknown Initial Value로 생존 값 검색</figcaption>
</figure>

이 과정을 수차례 반복하면 처음엔 수만 개에 달하던 후보가 **약 700개 정도**까지 줄어든다.

---

## 3. 후보 주소를 생존 플래그 관점에서 다시 좁히기

이제 남은 700개 중에서 “살았을 때 1, 죽었을 때 0처럼 보이는 플래그 형태의 값”만 골라봐야 한다.  

보통 이런 값은 `bool`이나 작은 `int`로 저장되기 때문에 0 또는 1 근처의 값들에 먼저 확인하는 편이 좋다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/3.png" alt="필터링 후 테이블에 내린 주소" style="display: block; margin: 0 auto;">
  <figcaption>필터링 후 테이블에 내린 주소</figcaption>
</figure>

실제로 살펴보면 대략 이런 패턴이 보인다.

017로 시작하는 주소들은 `igxelpgicd32.dll` 같은 그래픽 모듈에서 나온 값이라 생존 플래그와는 거리가 멀어 보인다.  
018 대역에 있는 값들 중 일부는 캐릭터가 죽을 때 1 → 0, 다시 시작할때 0 → 1로 바뀌는 형태를 보인다.

이런 식으로 상태 변화를 직접 확인해 보면서 후보를 몇 개로 압축할 수 있다.

---

## 4. 생존 플래그를 직접 고정해서 테스트해 보기

후보 중에서 두 개의 주소가 캐릭터가 죽을 때마다 함께 0으로 떨어지고,  
다시 살아날 때 1로 돌아오는 모습을 보인다.  

이 둘을 테이블로 내린 뒤, 둘 다 0으로 고정한 상태에서 발판을 밟아 본다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/4.png" alt="두 값 다 0 고정 후 이펙트 발생 장면" style="display: block; margin: 0 auto;">
  <figcaption>두 값 다 0 고정 후 이펙트 발생 장면</figcaption>
</figure>

캐릭터가 분명히 위험한 발판을 밟았는데도 사망 처리 대신 특이한 이펙트만 발생하고 그대로 살아남는 것을 볼 수 있다.  

이걸로 이 두 주소가 실제 충돌 처리와 밀접하게 연결된 값이라는 점을 확인할 수 있다.  
단순히 값만 고정하는 방식으로도 클리어는 가능하지만, 이번 단계에서는 한 발 더 나아가 코드 자체를 건드려 본다.

---

## 5. Find out what writes로 생존 플래그를 갱신하는 코드 찾기

이제 두 주소 중 하나를 기준으로 **Find out what writes to this address**를 실행한다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/5.png" alt="Find out what writes to this address 결과" style="display: block; margin: 0 auto;">
  <figcaption>Find out what writes to this address 결과</figcaption>
</figure>

캐릭터가 위험한 발판과 충돌하거나, 게임이 “죽었다”고 판단하는 순간  
특정 명령어가 이 주소에 값을 쓰는 것을 확인할 수 있다.  

이 명령어가 실제로 생존 여부를 바꾸는 핵심 코드다.  
여기서 Show Disassembler로 넘어가 어셈블리 흐름을 자세히 본다.

---

## 6. 메모리 뷰에서 조건 분기 확인

Disassembler 화면에서 보면 해당 주소에 값을 쓰는 부분 바로 앞에 조건 분기 명령어가 붙어 있다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/6.png" alt="Show Disassembler로 확인한 메모리 뷰" style="display: block; margin: 0 auto;">
  <figcaption>Show Disassembler로 확인한 메모리 뷰</figcaption>
</figure>

구체적인 레지스터 이름이나 정확한 주소는 상황마다 다를 수 있지만, 흐름 자체는 대략 이런 형태다.

```asm
cmp  [생존_관련_값], 0
jne  short 죽음_처리_분기
...
; 이 구간에서 충돌, 이펙트, 사망 플래그 설정 등의 처리가 이루어진다
```

비교 후 **값이 0이 아니면(jne)** 죽음 처리 루틴으로 분기하는 조건이다.  
여기서 분기를 뒤집으면, 원래 죽어야 할 상황에서도 충돌 루틴이 건너뛰어져  캐릭터가 계속 살아남는 효과를 만들 수 있다.

---

## 7. 조건문 뒤집기 — jne를 je로 변경

이제 Auto Assembler나 CE의 간단한 패치를 이용해 `jne`를 `je`로 바꿔 준다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/7.png" alt="조건문 변경 (jne → je)" style="display: block; margin: 0 auto;">
  <figcaption>조건문 변경 (jne → je)</figcaption>
</figure>

이 한 줄을 바꾸면 흐름이 이렇게 바뀐다.

원래는 비교 결과가 다를 때(충돌 조건 만족 시)  죽음 처리 코드가 실행됐다.   
이제는 비교 결과가 같을 때만 죽음 처리 코드로 들어가기에 분기가 일어나지 않으므로 캐릭터가 죽지 않게 된다.

앞에서 생존 플래그를 0으로 고정했던 것과 비슷한 효과지만, 이번에는 값을 직접 만지는 것이 아니라  
조건문 자체를 손대서 게임 로직을 바꾼 것이다.

---

## 8. 실제 게임 진행과 Step 3 완료

조건을 바꾼 상태로 다시 게임을 진행해 보면 위험한 장애물을 밟아도 캐릭터가 계속 살아남는다.  
장애물에 닿게되면 기존에는 사망처리가 되어야 하지만 정상적으로 통과하여 클리어할 수 있다.

<figure style="text-align: center;">
  <img src="/assets/img/CheatEngine/Games3/8.png" alt="Step 3 완료 화면" style="display: block; margin: 0 auto;">
  <figcaption>Step 3 완료 화면</figcaption>
</figure>

---

## 마무리

Games Step3는 단순한 변경을 통한 클리어가 아닌 플래그, 조건 분기, 충돌 처리 로직을 생각하게 만든 문제인거 같다.  
Unknown Initial Value로 상태 값을 찾고, 플래그를 직접 고정해 보며 의미를 파악한 뒤,  
해당 값을 갱신하는 코드와 분기 조건까지 조작해 보는 흐름이다.  

이런 과정을 한번 경험해 두면 앞으로 다른 게임을 분석할 때도 유용하게 참고정도는 할수 있을것으로 생각이 된다. 
