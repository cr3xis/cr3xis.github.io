---
title: '게임핵의 원리 (2) - Direct3D 기반 WallHack'
published: 2026-05-25
description: 'Direct3D9 후킹과 렌더링 상태 변경을 이용한 WallHack 동작 원리를 분석합니다.'
image: '/assets/img/GameHacking/DirectX/Dll.png'
tags:
  - 'Direct3D9'
  - 'D3D9'
  - 'WallHack'
  - 'Chams'
  - 'Wireframe'
  - 'ZBuffer'
  - 'DrawIndexedPrimitive'
  - 'Hooking'
category: 'Game Hacking'
draft: false
---


해당 문서는 게임에서 사용되는 불법 프로그램들의 동작 원리와 구현 구조를 분석하는 기술 문서입니다.  
본 문서는 보안 연구 및 교육 목적으로 작성되었으며, 실제 게임에 대한 악용을 권장하지 않습니다.  

---

이번 문서에서는 Direct3D 9 기반 게임에서 사용되는 월핵의 구현 방식을 분석합니다.  

WallHack의 기본 개념과 깊이 버퍼(Z-Buffer)의 동작 원리에 대한 설명은 이전 문서에서 자세히 다루었으므로  
해당 개념이 익숙하지 않다면 먼저 아래 문서를 읽고 오는 것을 권장합니다.

- [게임핵의 원리 (1) - OpenGL 기반 WallHack](/posts/game-hacking-opengl-wallhack/)

---

## D3D9 Hook

D3D9 함수를 후킹하기 위해서는 내부적으로 어떤식으로 동작을 하는지를 파악해야 한다.  

- Create a function to initialize Direct3D and create the Direct3D Device

```cpp
// this function initializes and prepares Direct3D for use
void initD3D(HWND hWnd)
{
    d3d = Direct3DCreate9(D3D_SDK_VERSION);    // create the Direct3D interface

    D3DPRESENT_PARAMETERS d3dpp;    // create a struct to hold various device information

    ZeroMemory(&d3dpp, sizeof(d3dpp));    // clear out the struct for use
    d3dpp.Windowed = TRUE;    // program windowed, not fullscreen
    d3dpp.SwapEffect = D3DSWAPEFFECT_DISCARD;    // discard old frames
    d3dpp.hDeviceWindow = hWnd;    // set the window to be used by Direct3D

    // create a device class using this information and information from the d3dpp stuct
    d3d->CreateDevice(D3DADAPTER_DEFAULT,
                      D3DDEVTYPE_HAL,
                      hWnd,
                      D3DCREATE_SOFTWARE_VERTEXPROCESSING,
                      &d3dpp,
                      &d3ddev);
}
```

위 코드는 Direct3D9 환경에서 렌더링에 사용되는 `IDirect3DDevice9` 객체를 생성하는 예제이다.  
Direct3D9의 렌더링 함수들은 Export 함수 형태로 제공되지 않고 COM 인터페이스 기반으로 구현되어 있다.  

따라서 `GetProcAddress()`를 이용하여 함수 주소를 획득할 수 없어  
`CreateDevice()`를 통해 생성되는 `IDirect3DDevice9` 객체를 기준으로 접근해야 한다.  

이때 `IDirect3DDevice9` 객체가 참조하는 VTable을 통해 렌더링 함수들의 주소를 확인할 수 있다.  
이번 예제에서는 렌더링 과정에서 호출되는 `DrawIndexedPrimitive()` 함수를 대상으로 분석을 진행한다.  

해당 함수는 3D 모델을 화면에 출력할 때 사용되는 대표적인 렌더링 함수 중 하나로 많은 D3D9 기반 월핵이 이 함수를 후킹하여 구현된다.  

정리하면 전체 과정은 다음과 같다.

1. `IDirect3DDevice9`의 VTable을 찾는다.  
2. `DrawIndexedPrimitive()` 함수 주소를 획득한다.  
3. Inline Hook을 설치한다.  
4. 렌더링 과정에 개입하여 벽 뒤의 오브젝트를 화면에 표시한다.  

---

### Internal & External

해당 예제는 외부 프로세스를 이용하는 `External` 방식이 아닌 `DLL Injection`을 통해 게임 프로세스 내부에서 동작하는 `Internal` 방식으로 구현한다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/Dll.png" alt="Dll Injection" style="display: block; margin: 0 auto;">
  <figcaption>Dll Injection</figcaption>
</figure>

--- 

### VTable 주소 찾기 (Pattern)

앞서 설명한것처럼 Direct3D9의 렌더링 함수는 Export 형태로 제공되지 않는다.  

그래서 `DrawIndexedPrimitive()`와 같은 렌더링 함수의 주소를 얻기 위해서  
먼저 `IDirect3DDevice9` 객체가 참조하는 VTable의 위치를 찾아야 한다.  

이번 예제에서는 `CreateDevice()` 함수를 기준으로 VTable 초기화 과정을 추적하였다.  

#### STEP 1. CreateDevice 함수 진입 

`CreateDevice()` 함수의 6번째 인자는 생성된 `IDirect3DDevice9` 객체를 반환받기 위한 출력 인자로 사용된다.  
해당 시점에서도 출력 인자는 아직 초기화 되지 않은 상태이며 실제 객체 생성 및 초기화는 내부 함수에서 수행된다.   

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/createdevice.png" alt="CreateDevice" style="display: block; margin: 0 auto;">
  <figcaption>CreateDevice</figcaption>
</figure>

#### STEP 2. 내부 객체 초기화 함수 추적

`CreateDevice()` 내부를 분석하면 Device 객체를 생성하고 초기화 하는 하위 함수를 확인할 수 있다.

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/subfunc.png" alt="Sub Function" style="display: block; margin: 0 auto;">
  <figcaption>Sub Function</figcaption>
</figure>

#### STEP 3. VTable 초기화 코드 확인 

내부 함수를 추적하다 보면 아래와 같이 VTable 주소를 객체에 기록하는 코드를 확인할 수 있다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/vftable.png" alt="VTable access" style="display: block; margin: 0 auto;">
  <figcaption>VTable access</figcaption>
</figure>

```asm
6CEEB022 | C706 3826E96C            | mov dword ptr ds:[esi],<d3d9.sub_6CE92638>        |
```

해당 코드는 `IDirect3DDevice9` 객체가 생성되는 과정에서 VTable 주소를 설정하는 부분이고 이를 기반으로 Byte Pattern을 작성한다.  

처음에는 아래와 같은 형태의 패턴을 얻을수 있다.
- `C7 06 24 1D 00 10 89 86 50 32 00 00 89 86 ` 

D3D9 버전이나 빌드 환경에 따라 주소나 일부 구조체 Offset 값이 달라질 수 있으므로  
가변적인 와일드 카드 처리로 치환하면 최종적으로 아래와 같은 패턴이 남게 된다.
- `C7 06 ?? ?? ?? ?? 89 86 ?? ?? ?? ?? 89 86` 

```cpp
DWORD FindPattern( DWORD dwAddress, DWORD dwSize, const std::vector<int>& pattern )
{
    BYTE* pData = (BYTE*)dwAddress;
    for (DWORD i = 0; i < dwSize; i++)
    {
        bool found = true;
        for (size_t j = 0; j < pattern.size(); j++)
        {
            if (pattern[j] != -1 && pData[i + j] != pattern[j])
            {
                found = false;
                break;
            }
        }

        if (found) return (DWORD)(dwAddress + i);
    }
    return 0;
}

DWORD addr = FindPattern( (DWORD)hD3D, 0x128000, { 0xC7,0x06,-1,-1,-1,-1,0x89,0x86,-1,-1,-1,-1,0x89,0x86 } );

DWORD vtableAddress = *(DWORD*)(addr+2);

```

위 코드를 통해 패턴이 발견되면 mov [esi], <vtable> 명령어의 주소가 반환된다.  

이를 통해 VTable 주소를 얻어올수 있게 된다.  

---

### DrawIndexedPrimitive  함수 주소 구하기

VTable 주소를 구했으니 VTable index를 이용하여 `DrawIndexedPrimitive` 함수 주소를 얻을수 있다.  
컴파일된 d3d9.dll 바이너리는 index 가 고정되어 있기 때문에 그 부분을 참고하면 된다.  

```cpp
#define DRAWINDEXEDPRIMITIVE    82
```

공식은 다음과 같다. 
- `vtable + 82 * 4(32bit) = DIP Address`

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/DIP.png" alt="DIP" style="display: block; margin: 0 auto;">
  <figcaption>DIP</figcaption>
</figure>

메모리에서 확인 `DrawIndexedPrimitive` 주소와 코드로 계산한 결과가 일치한것을 확인할수 있다.  

### Inline Hook

Inline Hook에 대한 간략한 설명을 하자면 함수 시작 부분을 JMP로 덮어써서 원본 함수가 호출되더라도 먼저 우리가 작성한 함수로 진입하게 만든다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/trampoline.png" alt="Detours" style="display: block; margin: 0 auto;">
  <figcaption>Detours</figcaption>
</figure>

간단하게 원본 함수의 일부 명령어는 JMP 패치 과정에서 덮어쓰여지게 된다.  

Trampoline은 해당 명령어들을 별도의 메모리 영역에 복사한 뒤 원본 함수의 다음위치로 복귀하여 원래 실행 흐름이 계속 이어질 수 있도록 한다.  

```cpp
void* DetourFunction(BYTE* src, const BYTE* dst, const int len) {

    BYTE* jmp = (BYTE*)malloc(len + 5);
    DWORD dwBack;
    VirtualProtect(src, len, PAGE_EXECUTE_READWRITE, &dwBack);
    memcpy(jmp, src, len);

    BYTE* jmpTarget = jmp + len;
    *jmpTarget = 0xE9;
    *(DWORD*)(jmpTarget + 1) = (DWORD)(src + len) - (DWORD)jmpTarget - 5; 

    *(BYTE*)src = 0xE9;  
    *(DWORD*)(src + 1) = (DWORD)dst - (DWORD)src - 5;

    for (int i = 5; i < len; i++) src[i] = 0x90;

    VirtualProtect(src, len, dwBack, &dwBack);
    VirtualProtect(jmp, len + 5, PAGE_EXECUTE_READWRITE, &dwBack);

    return jmp;

HRESULT __stdcall hkDrawIndexedPrimitive(
LPDIRECT3DDEVICE9 pDevice,
D3DPRIMITIVETYPE pType,
INT BaseVertexIndex,
UINT MinVertexIndex,
UINT NumVertices,
UINT startIndex,
UINT primCount
){
    printf("hooked!\n");
    return oDrawIndexedPrimitive(pDevice, pType, BaseVertexIndex, MinVertexIndex, NumVertices, startIndex, primCount);
}


void hookmain(){
    DWORD vtableAddress = *(DWORD*)(addr+2);
    BYTE* pDIP = (BYTE*)((DWORD*)vtableAddress)[82];
    oDrawIndexedPrimitive = (tDrawIndexedPrimitive)DetourFunction(pDIP, (BYTE*)hkDrawIndexedPrimitive, 5);
}

```

위 코드는 DIP에 Inline Hook을 설치하는 과정이다.  
먼저 `DetourFunction`은 DIP의 프롤로그를 JMP hkDIP 로 변경하여 함수가 호출 될 경우 가장 먼저 Hook 함수가 실행되도록 한다.  

이 과정에서 덮어쓰여진 원본 명령어는 Trampoline 영역에 복사되고 이후 원본 함수의 나머지가 코드가 계속 실행 될 수 있도록 Origin + 5 위치로 복귀하는 JMP 코드가 추가된다. 

`DetourFunction`이 반환한 Trampoline 주소는 oDIP에 저장이 된다.  
따라서 Hook 함수 내부에서 oDIP를 호출하면 유실된 프롤로그를 복구한 뒤 원본 DIP가 정상적으로 수행된다.

여기서 주의해야할 점은 후킹 함수의 원형과 호출 규약이 원본 함수와 동일해야 한다는것이다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/DirectX/hookbefore.png" alt="Before">
    <figcaption>Before</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/DirectX/hookafter.png" alt="After">
    <figcaption>After</figcaption>
  </figure>
</div>

### Depth Buffer Code

이제 `DrawIndexedPrimitive` 후킹에 성공했으므로 렌더링 상태를 직접 변경할 수 있다.  

이전 `OpenGL` 에서도 이야기 했지만 원리는 동일하다.  
깊이 버퍼를 일시적으로 비활성화한 상태에서 오브젝트를 다시 그리게 하는것이다.  

동작 순서는 다음과 같다.  

1. 깊이 버퍼를 비활성화 한다.  
2. 원본 DIP 를 호출한다.  
3. 깊이 버퍼를 다시 활성화 한다.  

깊이 버퍼가 비활성화되면 오브젝트가 벽 뒤에 있는지 검사하지 않고 원래 가려져 있던 물체도 화면에 그대로 렌더링된다.  

그리고 렌더링 상태는 전역적으로 적용되기 때문에 원본 함수를 호출한 뒤에는 반드시 깊이 버퍼를 다시 활성화 해야한다.  
그렇지 하지 않으면 이후에 그려지는 다른 물체들까지 모두 영향을 받게 되기 때문이다.  

```cpp
HRESULT SetRenderState(
  [in] D3DRENDERSTATETYPE State,
  [in] DWORD              Value
);
```

D3D9에서는 `SetRenderState()` 함수를 통해 다양한 렌더링 상태를 변경할 수 있다.  
그중 `D3DRS_ZENABLE`은 깊이 버퍼 사용 여부를 제어하는 옵션이다.  

- `D3DZB_TRUE` : 깊이 버퍼 활성화  
- `D3DZB_FALSE` : 깊이 버퍼 비활성화

```cpp
HRESULT __stdcall hkDrawIndexedPrimitive(LPDIRECT3DDEVICE9 pDevice, D3DPRIMITIVETYPE pType, INT BaseVertexIndex, UINT MinVertexIndex, UINT NumVertices, UINT startIndex, UINT primCount)
{
 pDevice->SetRenderState(D3DRS_ZENABLE, D3DZB_FALSE); 
 // 호출 전
 oDrawIndexedPrimitive(pDevice, pType, BaseVertexIndex, MinVertexIndex, NumVertices, startIndex, primCount);
 // 호출 후
 pDevice->SetRenderState(D3DRS_ZENABLE, D3DZB_TRUE);
}
```

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/DirectX/ZBTRUE.png" alt="Before">
    <figcaption>Before</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/DirectX/ZBFALSE.png" alt="After">
    <figcaption>After</figcaption>
  </figure>
</div>

결과적으로 깊이 검사 없이 렌더링이 수행되니 벽이나 장애물 뒤에 있는 오브젝트도 화면에 표시되는것을 확인할수 있다.  

### Advanced Techniques

지금까지 구현한 방식은 깊이 버퍼만 비활성화한 상태이므로 DIP를 사용하는 모든 오브젝트가 벽을 통과해 보이게 된다.  

이 경우 플레이어 뿐만 아니라 다른 여러 오브젝트들까지 함께 보이게 되는데 흔히 이러한 형태를 **Glass Wall** 이라고 부른다.  

그래서 우리는 원하는 오브젝트만 표시하려면 렌더링되는 모델을 구분할 수 있는 기준이 필요하다.  
D3D9에서는 `Stride`, `NumVertices`, `PrimitiveCount` 값을 이용하여 특정 모델을 식별할 수 있다.  

- `Stride`: 정점 하나의 크기  
- `NumVertices`: 사용되는 정점 개수  
- `PrimitiveCount`: 렌더링되는 Primitive 개수

특히 `Stride`는 모델마다 다른 값을 가지는 경우가 많아 가장 많이 식별되는 기준 중 하나이다.  
이를 찾기 위해 일반적으로 `Stride Logger`를 구현하여 현재 렌더링 되는 모델들을 수집한다.  

```cpp
    if (pDevice->GetStreamSource(0, &Stream_Data, &Offset, &Stride) == D3D_OK)
    {
        bool Found = false;

        for (auto& Model : Logger)
        {
            if (Model.Stride == Stride && Model.NumVertices == NumVertices && Model.PrimitiveCount == primCount) { Found = true; break; }
        }
        if (!Found) { 
            Logger.push_back({ Stride, NumVertices, primCount }); 
            printf("Stride: %u NV: %u PC: %u\n", Stride, NumVertices, primCount);
        }
    }
```

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/Logger.png" alt="Stride Logger" style="display: block; margin: 0 auto;">
  <figcaption>Stride Logger</figcaption>
</figure>

Stride Logger를 통해 플레이어 모델의 Stride 값을 확인했다면 더 이상 모든 오브젝트를 렌더링할 필요는 없다.    
이제 해당 값을 조건으로 사용하여 플레이어 모델만 별도로 처리하면 된다.  

Stride 조건에 일치하는 모델에 대해서만 별도의 색상을 적용하면 **Chams**를 구현할 수 있다.  

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/DirectX/Chams.png" alt="Chams" style="display: block; margin: 0 auto;">
  <figcaption>Chams</figcaption>
</figure>

또한 `D3DRS_FILLMODE` 상태를 변경하면 모델을 폴리곤 형태로 렌더링하는 Wireframe 효과를 구현할 수 있다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/DirectX/wf1.png" alt="Wireframe">
    <figcaption>Wireframe</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/DirectX/wf2.png" alt="Wireframe">
    <figcaption>Wireframe</figcaption>
  </figure>
</div>

--- 

### 마무리  

이번 글에서는 Direct3D9의 `DrawIndexedPrimitive` 후킹을 통해 WallHack의 기본 원리를 구현해 보았다.  

깊이 버퍼 조작, 모델 식별, 렌더링 상태 변경을 조합하면 Chams와 Wireframe 같은 다양한 응용 기법도 구현할 수 있다.  

다음 글에서는 기존 월핵보다 더 많이 사용되고 있는 ESP 핵에 대해 분석해 보도록 하겠습니다.  