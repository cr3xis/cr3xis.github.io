---
title: 'Win32 그리고 User Mode-Kernel Mode 이해하기'
published: 2026-02-20
description: 'WinAPI 호출이 User Mode에서 Kernel Mode로 이어지는 전체 흐름을 정리합니다.'
image: '/assets/img/CS/03/1.png'
tags:
  - 'WinAPI'
  - 'Syscall'
  - 'Kernel'
  - 'UserMode'
  - 'KernelMode'
  - 'Ring3'
  - 'Ring0'
  - 'DLL'
category: 'CS'
draft: false
---


Windows 내부 구조를 공부하면서 가장 크게 와닿았던 점은 우리가 평소에 호출하는 WinAPI 함수들이  
단순한 기능 호출이 아니라 User Mode에서 Kernel Mode까지 이어지는 흐름 속에 있다는 사실이었다.  

이 구조를 이해하면 WinAPI가 왜 이렇게 설계되었는지, DLL과 Syscall이 어떤 역할을 하는지,  
그리고 User Mode 프로그램이 Kernel을 직접 건드릴 수 없는 이유가 자연스럽게 연결된다.

아래는 내가 이해한 내용들을 전반적으로 설명한 글이다.

---

## 전체 흐름을 먼저 떠올려보기

우리가 호출하는 WinAPI는 User Mode에서 시작해 DLL, NTDLL, Syscall을 거쳐 Kernel로 내려간다.  

하나의 큰 그림으로 보면 이런 구조다.

<figure>
  <img src="/assets/img/CS/03/1.png" alt="Big Picture">
  <figcaption>Big Picture</figcaption>
</figure>

---

## Win32와 WinAPI의 역할

개발자가 자주 사용하는 `MessageBox`, `CreateFile`, `ReadFile` 같은 함수들은 Win32 API라고 부른다.  
이 함수들은 보기엔 단순한 함수 호출이지만, OS 깊은 영역과 통신하기 위한 첫 번째 관문이 된다.

WinAPI의 목적은 운영체제의 복잡한 내부 구조를 드러내지 않고 개발자가 이해할 수 있는 방식으로 기능을 제공하는 것이다.  

예를 들어 파일을 읽기 위해 NTFS 구조를 직접 다룰 필요도 없고, 커널 내부 자료구조를 건드릴 필요도 없다.  
WinAPI는 이런 복잡함을 완전히 숨겨준다.

---

## user32.dll과 kernel32.dll의 역할 차이

WinAPI는 대부분 DLL 안에 구현되어 있다.

<figure>
  <img src="/assets/img/CS/03/2.png" alt="user32 &amp; kernel32">
  <figcaption>user32 &amp; kernel32</figcaption>
</figure>

### user32.dll  
사용자 인터페이스 관련 기능을 제공한다.  
창, 버튼, 키 입력 처리 등 눈에 보이는 UI 기능들이 여기에 있다.

### kernel32.dll  
시스템 자원과 관련된 기능을 제공한다.   
파일 시스템, 프로세스, 스레드, 메모리 관리 등 운영체제의 핵심 기능과 연결되는 API들이 들어 있다.

---

## DLL에서 NTDLL로 이어지는 구조

WinAPI는 실질적인 작업을 직접 수행하지 않는다.  
특히 시스템 자원에 접근하는 기능은 Kernel Mode에서만 가능하기 때문에 WinAPI는 내부적으로 **NTDLL.dll**의 함수를 호출한다.

NTDLL은 User Mode에서 Kernel Mode로 넘어가는 마지막 단계다.

예를 들면 CreateFile → kernel32.dll → NTDLL → NtCreateFile → Syscall 이런 흐름으로 이어진다.

NTDLL에는 `NtReadFile`, `NtCreateFile` 같은 Kernel과 직접 연결되는 함수들이 존재한다.


---

## 시스템콜

User Mode는 Kernel 내부에 직접 접근할 수 없다.  
이 제한은 단순한 규칙이 아니라 OS의 안정성과 보안을 유지하기 위한 구조적 장치다.  

그래서 User Mode가 Kernel에 요청을 보내는 유일한 경로가 바로 시스템콜이다.

시스템콜은 User Mode에서 온 요청을 안전하게 Kernel로 전달하고, Kernel은 해당 요청을 수행한 뒤 다시 User Mode로 결과를 돌려준다.

이 흐름 덕분에 운영체제는 잘못된 코드가 Kernel 영역을 건드려 전체 시스템을 망가뜨리는 상황을 방지할 수 있다.

---

## User Mode와 Kernel Mode의 차이

Windows는 실행 계층을 두 가지로 나누어 운영한다.

### User Mode  
애플리케이션이 실행되는 공간이다.  
직접 메모리를 제어하거나 CPU의 모든 명령을 사용할 수 없도록 제한되어 있다.  
오류가 발생해도 해당 프로세스만 종료되고 시스템 전체는 영향을 받지 않는다.

### Kernel Mode  
운영체제가 동작하는 공간이다.  
메모리, 프로세스, 스케줄러, 드라이버 등 모든 자원에 접근할 수 있다.  
여기서 발생한 오류는 시스템 전체에 영향을 미칠 수 있다.

이 구분은 Windows가 안정성을 유지하는 핵심 원리다.

---

## Ring3 & Ring0

Windows는 두 단계(Ring3, Ring0)만 사용한다.

<figure>
  <img src="/assets/img/CS/03/Ring.png" alt="CPU 보호 링 구조">
  <figcaption>CPU 보호 링 구조</figcaption>
</figure>

- Ring3는 User Mode  
- Ring0은 Kernel Mode  

EXE, DLL 등 일반 프로그램은 Ring3에서 실행되며 드라이버와 커널 코드는 Ring0에서 실행된다.

이 구조 덕분에 User Mode 프로그램이 실수로 Kernel의 중요한 메모리를 망가뜨리는 상황을 막을 수 있다.

---

## 정리

Win32 → WinAPI → DLL → NTDLL → Syscall → Kernel로 이어지는 흐름은 Windows가 안정적으로 동작하기 위해 만든 계층 구조다.  

조금 풀어서 이야기하자면 Winapi는 커널 기능을 직접 호출하는 것이 아니라, Kernel32/user32가
ntdll에게 특정API에대한 요청을 하면 NT 함수들을 호출하고 syscall이 커널에게 요청을 전달하는 구조를 하고 있다는 말이다.

처음에는 복잡하게 보일 수 있지만 한 흐름으로 바라보면  
왜 User Mode에서 Kernel을 직접 건드릴 수 없는지, 왜 DLL이 필요한지, 왜 Syscall이 중요한지 자연스럽게 이해된다.  

이 구조를 이해하면 리버싱, 프로세스 분석, 메모리 구조 분석 같은 작업도 훨씬 선명하게 보이기 시작한다.
