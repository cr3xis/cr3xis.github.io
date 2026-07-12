---
title: '게임핵의 원리 (5) - Silent Aimbot'
published: 2026-07-01
description: 'CreateMove 입력 흐름과 인라인 후킹을 이용한 Silent Aimbot의 동작 원리를 분석합니다.'
image: '/assets/img/GameHacking/Silent/silent.png'
tags:
  - 'SilentAimbot'
  - 'CreateMove'
  - 'UserCmd'
  - 'TrampolineHook'
  - 'MASM'
category: 'Game Hacking'
draft: false
---


해당 문서는 게임에서 사용되는 불법 프로그램들의 동작 원리와 구현 구조를 분석하는 기술 문서입니다.  
본 문서는 보안 연구 및 교육 목적으로 작성되었으며 실제 게임에 대한 악용을 권장하지 않습니다.

---

이전 문서에서는 ViewAngle을 수정하는 Aimbot의 기본 원리를 살펴보았다.  

- [게임핵의 원리 (4) - Aimbot 핵](/posts/game-hacking-aimbot/)

이번 문서에서는 화면은 움직이지 않고 발사에 사용되는 각도만 바꾸는 `Silent Aimbot`의 동작 원리를 분석한다.  

---

## Silent Aimbot이란 무엇인가?

기본적으로 Aimbot은 계산된 목표 각도를 현재 VeiwAngle에 적용하여 사용자의 화면을 실제 타겟 방향으로 이동한다.  

하지만 Silent Aimbot은 화면에 보이는 시야각을 바꾸지 않고 발사나 입력 처리에 사용되는 각도만 타겟 방향으로 바꾸는 것이다.  
<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Silent/silent.png" alt="Silent Aimbot 동작 결과" style="display: block; margin: 0 auto;">
  <figcaption>Silent Aimbot</figcaption>
</figure>


그러므로 Silent Aimbot에서 중요한 것은 화면에 보이는 `dwViewAngles`가 아니다.  
공격 입력이 처리되는 순간 어떤 각도가 사용되는지 찾는 것이다.    

---

## 입력 명령 흐름

공격 입력이 처리될 때 어떤 입력 명령이 만들어지고 그 안에 어떤 각도가 들어가는지 봐야 한다.  

Source SDK의 `CInput::CreateMove` 흐름에서 확인할 수 있다.  
`in_main.cpp` 파일의 목적 주석에는 서버로 보낼 movement command를 만든다고 되어 있다.  
실제 함수 내부에서도 `CUserCmd`를 가져온 뒤 입력값을 채운다.  

```cpp
CUserCmd *cmd = &m_pCommands[sequence_number % MULTIPLAYER_BACKUP];

cmd->command_number = sequence_number;
cmd->tick_count = gpGlobals->tickcount;
cmd->buttons = GetButtonBits(1);
VectorCopy(viewangles, cmd->viewangles);
```

`CUserCmd`는 한 tick에서 사용자의 입력 상태를 담는 구조체이다.  

`usercmd.h` 기준으로 아래와 같은 정보를 담게 된다.  

- viewangles: 사용자가 바라보는 시야각 
- buttons: 공격, 점프, 앉기 같은 입력 버튼 비트 플래그
- command_number: 현재 입력 명령의 번호
- tick_count: 해당 입력 명령이 어느 tick 기준으로 생성되었는지
- forwardmove: 앞뒤 이동 값
- sidemove: 좌우 이동 값
- mousedx / mousedy: 마우스 이동량

정리하자면 아래와 같이 생각할수 있다.  

```text
CInput::CreateMove 호출
-> 현재 입력값을 CUserCmd에 저장
-> ClientMode CreateMove 흐름에서 cmd 수정 가능
-> 최종 UserCmd가 네트워크 메시지로 직렬화
-> 서버로 전송
```

이 흐름 때문에 일반적인 Silent Aim은 `CreateMove` 계열 흐름에서 `CUserCmd` 또는 Subtick 입력 명령의 각도를 수정하는 방식으로 설명된다.  

이번 구현에서는 `CUserCmd`나 Subtick 구조체 전체를 먼저 복원하지 않았다.  
대신 `CreateMove` 관련 입력 처리 흐름을 따라가며 실제 Pitch와 Yaw가 기록되는 지점을 찾는 방식으로 접근했다.  

---

## CreateMove

Silent Aimbot은 발사에 사용되는 각도와 관련 있다.  
그러면 단순히 카메라 각도 주소를 볼 것이 아니라 입력 명령이 만들어지고 공격 입력이 처리되는 흐름을 봐야 한다.  

Source SDK의 `CInput::CreateMove`는 `CUserCmd`를 만들고 `cmd->viewangles`를 채운다.  
이후 `g_pClientMode->CreateMove(..., cmd)`를 호출하고 그 결과에 따라 `engine->SetViewAngles(cmd->viewangles)`와 prediction view angle을 갱신한다.  

CS2 내부 구조가 Source SDK와 완전히 같다고 단정할수는 없지만  
Source 계열에서 입력 명령과 ViewAngle이 `CreateMove` 흐름내에 있다고 생각할수 있다.  

그래서 IDA에서 `CreateMove` 문자열을 먼저 검색했다.  
문자열이 있다고 해서 그 위치가 곧바로 함수의 시작점이라는 뜻은 아니다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Silent/IDA.png" alt="CreateMove Strings" style="display: block; margin: 0 auto;">
  <figcaption>CreateMove Strings</figcaption>
</figure>

검색 결과 `CreateMove` 관련 문자열은 여러 개가 나왔다.  
그중 `dev_create_move_report`처럼 리포트 성격에 가까운 문자열도 있었고 `invalid player history`처럼 특정 검증 흐름에 가까운 문자열도 있었다.  

내가 우선적으로 본 것은 `cmd`, `tick`, `attack history` 같은 단어가 함께 등장하고 같은 함수 안에서 여러 번 참조되는 흐름이었다.  
이 단어들은 입력 명령과 공격 기록을 다루는 흐름일 가능성이 높기 때문이다.  

따라서 해당 문자열을 참조하는 함수의 XREF를 따라가고 함수 내부에서 실제 각도 값이 사용되는지 동적 분석으로 확인했다.  

---

## 동적 분석

후보 함수의 흐름을 따라가며 XMM 레지스터를 확인했다.  
각도 값은 float 형태로 처리되므로 XMM 레지스터에 현재 ViewAngle과 같은 값이 들어오는지 확인했다.   

분석 중 아래와 같은 구간을 확인할 수 있었다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Silent/infunc.png" alt="Pitch / Yaw" style="display: block; margin: 0 auto;">
  <figcaption>Pitch / Yaw</figcaption>
</figure>

```asm
movss xmm0, [rdi+10]
movss [rcx+18], xmm0

movss xmm1, [rdi+14]
movss [rcx+1C], xmm1
```

xmm0 에 저장된 Pitch와 xmm1에 저장된 Yaw가 각각 기록되는것을 확인할수 있다.  
해당 값을 수정하면 화면은 그대로지만 발사 방향이 다른곳으로 바뀌는 것을 확인했고  
해당 위치가 발사에 참조되는 각도 기록 지점이라고 판단했다.   

---

## 후킹

이제 필요한 것은 해당 위치에서 원본 Pitch와 Yaw 대신 내가 계산한 Pitch와 Yaw를 기록하는 것이다.  

이번 구현에서는 MASM으로 Stub을 작성했다.  
후킹 지점에는 Stub으로 이동하는 점프만 설치하고 실제 어셈블리 흐름은 ASM 파일에 작성하는 방식이다.  

```text
후킹 지점
-> MASM Stub으로 이동
-> Silent 온/오프 체크
-> 켜진 상태라면 Pitch / Yaw 덮어쓰기
-> 원본 명령어 실행
-> 원래 코드로 복귀
```

`FF 25 00 00 00 00`은 다음 8바이트에 들어있는 주소를 읽어 그 위치로 점프하는 형태이다.  
이 방식으로 64비트 주소를 직접 Stub으로 넘길 수 있다.  

```text
FF 25 00 00 00 00
8 byte address
```

---

## 인라인 훅 설치

후킹을 설치하는 과정은 단순하다.  
후킹할 코드 영역의 메모리 보호 속성을 변경한 뒤 그 위치에 점프 코드를 덮어쓴다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Silent/inlinehook.png" alt="JMP" style="display: block; margin: 0 auto;">
  <figcaption>JMP</figcaption>
</figure>

```cpp
VirtualProtect(hookAddress, hookSize, PAGE_EXECUTE_READWRITE, &oldProtect);

patch[0] = 0xFF;
patch[1] = 0x25;
memcpy(patch + 6, &stubAddress, sizeof(stubAddress));
memset(patch + 14, 0x90, hookSize - 14);

memcpy(hookAddress, patch, hookSize);

FlushInstructionCache(GetCurrentProcess(), hookAddress, hookSize);
VirtualProtect(hookAddress, hookSize, oldProtect, &temp);
```

코드 영역을 쓰기 가능상태로 변경한뒤 후킹 지점에서 ASM Stub 으로 점프시키고 남는 바이트는 정리하면 된다.

이렇게 하면 게임 코드가 해당 위치에 도달했을 때 원래 명령어를 실행하기 전에 내가 작성한 Stub으로 먼저 이동한다.  

---

## MASM Stub

MASM을 사용한 이유는 x64 MSVC에서는 C++ 인라인 어셈블리를 사용할 수 없고  
ASM Stub으로 분리하면 실제 어셈블리 흐름을 그대로 보면서 작성할 수 있다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Silent/asmstub.png" alt="ASM" style="display: block; margin: 0 auto;">
  <figcaption>ASM</figcaption>
</figure>

```asm
cmp byte ptr [g_SilentActive], 0
je originalcode

movss xmm0, dword ptr [g_SilentPitch]
movss xmm1, dword ptr [g_SilentYaw]

movss dword ptr [rcx+18h], xmm0
movss dword ptr [rcx+1Ch], xmm1
```

`g_SilentActive`가 꺼져 있다면 원본 코드로 바로 이동한다.  
켜져 있다면 `g_SilentPitch`와 `g_SilentYaw` 값을 이용해 각도를 덮어쓴다.  

그 다음에는 후킹으로 덮어쓴 원본 명령어를 다시 실행해야 한다.  

```asm
originalcode:
    movss xmm0, dword ptr [rdi+18h]
    mov dword ptr [rcx+10h], eax
    movss dword ptr [rcx+20h], xmm0

    mov rcx, qword ptr [g_SilentLeaTarget]
    jmp qword ptr [g_SilentReturn]
```

마지막에는 `g_SilentReturn`으로 점프해 원래 코드 흐름으로 돌아간다.  

---

## Silent Angle 적용

아래와 같이 해당 기능이 켜져있다면 아래와 같이 각도만 수정해주면 된다.

```cpp
 if (Options::SilentAimbot)
 {
     g_SilentPitch = targetAngle.x;
     g_SilentYaw = targetAngle.y;
     g_SilentActive = 1;
 }
```

사용자의 화면은 움직이지 않지만 공격은 타겟 방향으로 처리되는 형태가 된다. 

---

## 결과 

적을 바라보지 않고 아무곳이나 바라보더라도 적이 죽는 `Silent Aim` 이 완성되었다.  
Fov 조건을 함께 적용하면 특정 범위 안의 대상만 선택하도록 할수 있다.  

<iframe
        width="100%"
        height="450"
        src="https://www.youtube.com/embed/chfWA_pIAbU"
        title="Aimbot(Silent)"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        referrerpolicy="strict-origin-when-cross-origin"
        allowfullscreen>
    </iframe>

---

## 마무리

Silent Aimbot은 화면 ViewAngle을 수정하는 것이 아닌 입력 명령이나 발사 처리에서 사용되는 각도를 찾아  
그 지점에서 값을 변경하는 방식이다.  

해당 시리즈를 작성하기 위해 관련 자료를 찾아보고 직접 분석하며 하나씩 동작을 확인하는 과정을 반복했다.  
하지만 시간이 지나면 구조와 오프셋이 변경되어 다시 분석해야하는 경우가 많았고 생각보다 많은 시간과 노력이 필요하다.  

그래서 앞으로도 새로운 내용을 정리할 생각은 있지만 분석 난이도와 유지보수 부담을 고려하면 이 시리즈를 계속 쓰는것은 고민할것 같다.  

---

## 참고

- [Valve Source SDK 2013 - in_main.cpp](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/game/client/in_main.cpp)
- [Valve Source SDK 2013 - usercmd.h](https://github.com/ValveSoftware/source-sdk-2013/blob/master/src/game/shared/usercmd.h)

`in_main.cpp`에서는 `CInput::CreateMove`가 `CUserCmd`를 만들고 `command_number`, `tick_count`, `buttons`, `viewangles` 같은 입력 값을 채우는 흐름을 확인할 수 있다.  
`usercmd.h`에서는 `CUserCmd` 구조체가 실제로 입력 명령에 필요한 값을 담는 구조체임을 확인할 수 있다.  

---