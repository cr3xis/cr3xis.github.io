---
title: 'CreateProcessW 이해하기'
published: 2026-03-15
description: 'CreateProcessW의 인자와 프로세스 생성 흐름을 예제 코드와 함께 정리합니다.'
image: '/assets/img/Win32/03/Result.png'
tags:
  - 'CreateProcessW'
  - 'STARTUPINFO'
  - 'PROCESS_INFORMATION'
category: 'Win32'
draft: false
---



WinAPI를 공부하다 보면 어느 순간부터 문서를 읽는 시간보다 이해하려고 머리를 굴리는 시간이 더 길어질 때가 있다.  
CreateProcessW를 처음 봤을 때가 그런 느낌이었다. 
인자는 많고 설명은 길고 흐름은 한 번에 잡히지 않고 뭔가 전체 구조가 제대로 정리돼 있어야 이해가 될 것 같은 함수였다.  

그래서 이번에는 이 함수를 조금 더 편안하게 바라볼 수 있는 흐름으로 정리해 보려고 한다.  
내가 공부하면서 느낀 생각이나 감정이 자연스럽게 스며 있는 방식으로 기록해 본다.

---

## CreateProcessW는 어떤 구조인가

CreateProcessW는 말 그대로 새로운 프로세스를 만들어 달라고 운영체제에게 요청하는 함수이다.  
설명만 보면 복잡해 보이지만 하나씩 뜯어보면 결국 프로세스를 생성하는 데 필요한 정보들을  
운영체제에 넘겨주는 역할을 한다는 느낌에 가까웠다.  

MSDN :[CreateProcessW](https://learn.microsoft.com/ko-kr/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw)

```cpp
BOOL CreateProcessW(
  [in, optional]      LPCWSTR                   lpApplicationName,
  [in, out, optional] LPWSTR                    lpCommandLine,
  [in, optional]      LPSECURITY_ATTRIBUTES     lpProcessAttributes,
  [in, optional]      LPSECURITY_ATTRIBUTES     lpThreadAttributes,
  [in]                BOOL                      bInheritHandles,
  [in]                DWORD                     dwCreationFlags,
  [in, optional]      LPVOID                    lpEnvironment,
  [in, optional]      LPCWSTR                   lpCurrentDirectory,
  [in]                LPSTARTUPINFOW            lpStartupInfo,
  [out]               LPPROCESS_INFORMATION     lpProcessInformation
);
```

처음에는 이 구조체를 보고 막막했지만 하나씩 풀어 이해하고 나니 인자 하나하나가 훨씬 자연스럽게 보였다.  

---

## lpApplicationName & lpCommandLine

lpApplicationName과 lpCommandLine은 처음 보면 비슷해 보여서 헷갈리기 쉽다.  
실제로는 둘의 개념이 꽤 다르다.

lpApplicationName은 실행 파일 경로를 운영체제에 정확하게 전달하는 방식이다.  
lpCommandLine은 실행 파일 이름과 인자를 하나의 문자열로 묶어서 넘기는 방식이다.

Windows는 lpCommandLine을 통째로 받아 내부에서 실행 파일 부분을 다시 파싱한다.  
그래서 둘을 동시에 사용하는 경우도 있지만 문서를 읽다 보면 가능한 한 명확하게 하나만 사용하는 편이  
코드의 흐름을 더 깨끗하게 유지할 수 있다는 생각이 들었다.  


## dwCreationFlags

새로 만들어진 프로세스가 어떤 상태로 시작될지 정하는 옵션들이 dwCreationFlags에 들어간다.  
처음에는 해당 필드에 어떤 플래그가 들어가는지를 몰라 찾고자 하는데에 어려움이 있었다.  
하지만 그부분은 내가 문서를 집중해서 읽지 못해 생겼던 문제였었다.

MSDN :[dwCreationFlags](https://learn.microsoft.com/ko-kr/windows/win32/procthread/process-creation-flags)

문서를 보면 많은 옵션이 있지만 그 흐름은 단순했다.  
새 프로세스를 어떤 모습으로 시작시키고 싶은지 운영체제에게 알려주는 용도라 생각하면 된다.

---

## STARTUPINFO 구조체

새 프로세스가 어떤 초기 상태로 실행될지 담는 구조체가 STARTUPINFO였다.  
공부하면서 가장 먼저 눈에 들어왔던 부분은 cb라는 필드였다.

이 cb는 구조체의 크기를 운영체제에 알려주는 역할을 한다.  
처음에는 굳이 왜 이런 행동을 할까 생각해서 알아 봤더니 WinAPI 구조체들은 버전이 바뀌면서  
필드가 추가되는 경우가 있어서 cb 같은 값을 전달하지 않으면 운영체제가 어디까지 읽어야 할지 애매해질 수 있었다.

이런 구조 덕분에 WinAPI는 오래된 구조체들도 계속 호환을 유지할 수 있었던 것 같다.  
그리고 cb나 dwSize 같은 역할을 하는 애들이 이름이 왜 다른지 찾아보니 Winapi 초기 개발자들이   
팀을 나눠 부서별로 일을 했는데 팀 마다 이름을 작명하는 규칙이 달라 이런것들이 지금까지 이어져 오는것 같다.

---

## PROCESS_INFORMATION 구조체

CreateProcessW가 성공하면 이 구조체에 여러 값들을 채워 준다.  
프로세스 ID, 스레드 ID, 프로세스 핸들, 스레드 핸들 같은 값들이 들어간다.

결국 프로세스를 직접 제어할 때 필요한 정보가 모두 이 구조체 하나에 들어 있다고 보면 된다.  
이 구조를 처음 봤을 때는 뭔가 복잡해 보였지만 실제로는 필요한 정보를 한 곳에 모아둔 결과물이라는 느낌이었다.

---

## Example
MSDN :[Example Code](https://learn.microsoft.com/ko-kr/windows/win32/procthread/creating-processes)의 
기본 예제를 먼저 보고 난 뒤 아래와 같은 변형된 코드를 작성해 보았다.

```cpp
#include <windows.h>
#include <stdio.h>

int main()
{
    STARTUPINFO si = { 0 };
    PROCESS_INFORMATION pi = { 0 };
    wchar_t title[] = L"CreateProcessW - cmd.exe";
    si.cb = sizeof(si);
    si.lpTitle = title;
    
    

    if (CreateProcessW(
        L"C:\\Windows\\System32\\cmd.exe",
        NULL,
        NULL,
        NULL,
        FALSE,
        CREATE_SUSPENDED |CREATE_NEW_CONSOLE,
        NULL, 
        NULL,
        &si, &pi))
    {
       printf("프로세스 생성 성공 :  PID = %d\n", pi.dwProcessId);
    }
    int count = 0;
    while (count < 5) {
        printf("count : %d\n", count);
        Sleep(1000);
        count++;
    }

    printf("ResumeThread ! \n");
    if (count == 5) { ResumeThread(pi.hThread); }
 

    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);
    return 0;
}

```
<figure>
  <img src="/assets/img/Win32/03/Result.png" alt="Result">
  <figcaption>Result</figcaption>
</figure>


각 필드 분석을 통해 간단한 예제를 만들었는데 코드를 보면 프로세스를 만들고 일시 정지 상태로 시작했다가  
resume을 통해 실행시키는걸 한눈에 이해할수 있다.

---

## 마무리

대부분의 API는 처음 보면 부담스럽고 어렵게 느껴진다.   
하지만 구조가 왜 이렇게 되어 있는지 하나씩 이해하고 나니 전체 흐름이 자연스럽게 이어지는 느낌이 들었다.  

WinAPI는 공부할 때마다 새로운 벽이 생기는 것 같지만 이런 글 하나를 정리하고 나면 또 어느 순간 이해가 더 깊어져 있는 걸 느낀다.  
이해하는 과정들을 글로 작성하는 시간이 길거나 가끔은 혼란스럽고 버거운 순간도 있지만 그래도 이렇게 하나씩 쌓여 가는 과정이 꽤 즐겁기도 하다.  
시간이 지나면 문서만 봐도 흐름이 그려지는 날이 올 거라 생각하면서 앞으로도 계속 이런 정리를 이어가려고 한다.
