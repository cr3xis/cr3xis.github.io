---
title: 'Memory Module Scanner 리뷰'
published: 2026-03-20
description: '실행 중인 프로세스의 모듈과 메모리 패턴을 확인하는 Memory Module Scanner를 리뷰합니다.'
image: '/assets/img/Review/MemoryScanner/executable.png'
tags:
  - 'WinAPI'
  - 'Memory'
  - 'Scanner'
category: 'Review'
draft: false
---


PE 구조를 파일 기준으로만 보다가 실제 실행 중인 메모리에서는 어떤식으로 구성이 되어있는지 궁금해 만들게 되었다.   

자세한 소스코드와 내용은 [Github](https://github.com/Cryptonite7777/Memory-Module-Scanner) 을 참고하면 된다.

특정 프로세스에 붙어서 모듈을 기준으로 메모리를 파악하고 바이트 패턴이 실제로 어디에 존재하는지  
확인할 수 있는 아주 단순한 스캐너를 만들기로 했다.  
잘 생각해보면 YARA룰의 시초가 되는 프로그램이지 않을까 라는 생각도 했다.  

---

## 초기

가장 먼저 확인하고 싶었던 건 복잡한 구조가 아니었다.  
내가 만든 프로그램이 실제 프로세스에 붙고 실행 파일이 메모리에 올라간 상태를 제대로 보고 있는지가 중요했다.  
<figure style="text-align: center;">
  <img src="/assets/img/Review/MemoryScanner/executable.png" alt="실행 화면" style="display: block; margin: 0 auto;">
  <figcaption>실행 화면</figcaption>
</figure>


일단 정상적으로 PID를 출력시키고 내가 입력시킨 PID를 기준으로 정보를 잘 읽어오는지 파악하는게 중요했다.

---

## 모듈 기준으로 검사를 한 이유

프로세스를 정한 후 처음으로 생각한 것은 범위를 지정하는 것이었다.  
프로세스의 가상 메모리 주소를 전부 검사한다는것은 공간이 너무 크고 비효율적이라 판단해서  
특정 범위를 지정해줘야 한다고 판단했고 그래서 프로세스에 로드된 모듈을 기준으로 검사를 하는걸로 했다.  

<figure style="text-align: center;">
  <img src="/assets/img/Review/MemoryScanner/program_region.png" alt="프로그램 메모리 영역" style="display: block; margin: 0 auto;">
  <figcaption>프로그램 메모리 영역</figcaption>
</figure>
<figure style="text-align: center;">
  <img src="/assets/img/Review/MemoryScanner/my_region.png" alt="모듈 정보" style="display: block; margin: 0 auto;">
  <figcaption>모듈 정보</figcaption>
</figure>

Module32 함수를 사용해 프로세스에 로드된 모듈 정보를 수집하고 필요한 정보만 구조체로 정리해 벡터 형태로 관리했다.    
그리고 memory Region을 보여주는 툴과 비교해 정상적으로 출력이 되었는지 비교했었다.  

---

## 효율적으로 처리하기 위해

모듈의 Base 주소부터 Size만큼 범위를 기준으로 VirtualQueryEx를 호출해 메모리 영역 단위로 순회하며  
MEM_COMMIT 상태 + 접근 가능한 페이지만 처리하겠금 했다.  

패턴 검사 방식 또한 구현 과정에서 몇번이나 수정을 했는지 모르겠다..  
초기에는 main 함수에서 모듈 하나에 패턴 하나 이런식으로 구성해 전달하며 검사하는 구조였는데  
나중에 패턴 수가 늘어날수록 비효율적이며 패턴을 굳이 묶어서 같이 전달할 필요가 없다는 문제 였다.  

이를 해결하기 위해 패턴 관련 로직을 main함수에서 분리시키고 모든 패턴을 PatternDB에 모아두고  
이름:패턴 형태의 map 구조로 관리하도록 변경했고 검사하는 함수에서 받아와 바로 사용하겠금 처리했다.  

---

## 결과

구조를 정리한 뒤, 테스트 실행 파일에 심어둔 패턴들이 정상적으로 탐지되는 걸 확인할 수 있었다.  

<figure style="text-align: center;">
  <img src="/assets/img/Review/MemoryScanner/result.png" alt="패턴 탐지 결과 출력" style="display: block; margin: 0 auto;">
  <figcaption>패턴 탐지 결과 출력</figcaption>
</figure>

---

## 마무리

이 프로젝트를 하면서 느꼈던 점은 일단 가벼운 마음으로 만들면서 이해해보자 였는데  
실제로는 구현 과정에서 반복적으로 발생하는 오류와 수정 과정을 거치면서  
이 코드 하나를 제대로 이해하기 위해 꽤 오랜 시간을 썼던거 같다.  

단순한 기능처럼 보였던 메모리 스캐너도 실제로 구현해보니 메모리 범위 설정, Region 단위 접근,  
읽기 가능한 영역 처리 등 고려해야할 요소가 너무 많았다.  

하지만 그 과정 자체가 나한테는 좋은 학습 경험이 되었던거 같고 또 무엇인가를 만들면서  
나 자신이 성장해간다는것을 느끼게 되는것 같다.  

학습 목적으로 한 매우 단순한 수준의 프로젝트이지만 메모리를 직접  
다뤄보고 구조적으로 이해해본 경험은 확실히 남았다고 생각한다.