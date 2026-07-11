---
title: '게임핵의 원리 (3) - ESP 핵'
published: 2026-06-10
description: 'View Matrix와 WorldToScreen 변환을 이용한 ESP 표시 원리를 분석합니다.'
image: '/assets/img/GameHacking/ESP/ESPall.jpg'
tags:
  - 'ESP'
  - 'ExternalOverlay'
  - 'Direct3D9'
  - 'WorldToScreen'
  - 'ViewMatrix'
  - 'EntityList'
category: 'Game Hacking'
draft: false
---


해당 문서는 게임에서 사용되는 불법 프로그램들의 동작 원리와 구현 구조를 분석하는 기술 문서입니다.  
본 문서는 보안 연구 및 교육 목적으로 작성되었으며 실제 게임에 대한 악용을 권장하지 않습니다.

---

이전 문서에서는 Direct3D9의 렌더링 함수 후킹을 이용하여 Z-Buffer를 조작하는 WallHack의 동작 원리를 살펴보았다.  

- [게임핵의 원리 (2) - Direct3D 기반 WallHack](/posts/game-hacking-direct3d-wallhack/)

이번 문서에서는 WallHack과 함께 자주 언급되는 ESP의 동작 원리를 분석한다.  

---

## ESP란 무엇인가?

ESP는 단순히 벽 뒤의 모델을 보이게 만드는 방식과는 다르게 게임 내부의 데이터를 읽고 해당 정보를 화면 위에 별도로 표시하는 방식이다.  

ESP는 Extra Sensory Perception의 약자로 게임 내부의 정보를 사용자 화면에 추가로 표시하는 기능을 의미한다.  
<figure>
  <img src="/assets/img/GameHacking/ESP/ESPall.jpg" alt="ESP">
  <figcaption>ESP</figcaption>
</figure>


예를 들어 게임 내부에는 다음과 같은 데이터가 존재한다.

- 플레이어 위치
- 체력
- 팀 정보
- 이름

ESP는 이러한 데이터를 읽어와서 화면 위에 시각적으로 표시한다.

```text
게임 내부 데이터
    -> 플레이어의 3D 월드 좌표 획득
    -> 2D 화면 좌표로 변환
    -> 오버레이 또는 렌더링 후킹을 통해 화면에 표시
```

핵심은 3D 좌표를 2D 좌표로 변환하는 과정이다.  
게임 내부의 위치 정보는 `{ x, y, z }` 형태의 월드 좌표이지만  
화면에 선이나 문자를 그리기 위해서는 `{ x, y }` 형태의 픽셀 좌표가 필요하다.  

---

## Z-Buffer WallHack과 ESP의 차이

Z 버퍼를 조작하여 벽 뒤의 오브젝트가 보이게 하는 방식이다.

```text
Z-Buffer WallHack
-> 렌더링 상태를 변경한다.
-> 깊이 검사를 조작하여 벽 뒤의 모델을 그대로 렌더링한다.
-> 게임 렌더링 파이프라인에 직접 개입한다.
```

ESP는 모델 자체를 벽 너머로 보이게 만드는 것이 아니라 게임 내부 데이터를 읽어 별도의 정보를 화면 위에 그린다.

```text
ESP
-> 엔티티 정보를 읽는다.
-> 월드 좌표를 화면 좌표로 변환한다.
-> 이름, 체력, 박스, 뼈대 등을 따로 그린다.
```

즉 ESP는 렌더링 상태를 직접 변조하는 방식이라기보다 게임 내부 데이터를 기반으로 부가적인 UI를 그리는 방식에 가깝다.

---

## ESP에 필요한 정보

ESP에서 가장 먼저 필요한 정보는 엔티티 리스트이다.  

FPS 게임에서 ESP를 구현하려면 결국 상대 플레이어의 좌표가 필요하다.  
상대 플레이어의 좌표, 체력, 이름, 뼈대 정보는 일반적으로 엔티티 객체 또는 그와 연결된 객체에서 얻을 수 있다.  

일반적인 게임에서는 엔티티 리스트가 단순한 배열처럼 구성되어 아래와 같은 경우로 되어있다.  

```text
EntityList[0]
EntityList[1]
EntityList[2]
...
```

그러나 분석한 게임은 단순히 `EntityList[i]`로 접근하는 구조가 아니었다.  
엔티티 리스트가 청크 단위로 관리되고 있었고 Controller와 Pawn객체가 분리되어 있었다.  

분석한 결과 아래와 같은 형식으로 되어 있었다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/Controller.png" alt="Controller">
    <figcaption>Controller</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/Pawn.png" alt="Pawn">
    <figcaption>Pawn</figcaption>
  </figure>
</div>

이를 코드로 표현하면 아래와 같다.  

```cpp
uint32_t pawnHandle = 0;
uintptr_t pawnListEntry = 0;
uintptr_t C_CSPlayerPawn = 0;

ReadMemory(Client + Offsets::dwEntityList, dwEntityList);
ReadMemory(dwEntityList + (8 * ((i & 0x7FFF) >> 9)) + 0x10, controllerListEntry);
ReadMemory(controllerListEntry + 0x70 * (i & 0x1FF), CCSPlayerController);
ReadMemory(CCSPlayerController + Offsets::m_hPlayerPawn, pawnHandle);

// entity list table + pawn handle chunk index -> pawn이 들어있는 list entry(chunk)
ReadMemory(dwEntityList + (8 * ((pawnHandle & 0x7FFF) >> 9)) + 0x10, pawnListEntry);
// pawn list entry + pawn handle slot index -> C_CSPlayerPawn
ReadMemory(pawnListEntry + 0x70 * (pawnHandle & 0x1FF), C_CSPlayerPawn);
```

Pawn 핸들은 단순 주소가 아니라 엔티티 리스트 내부에서 실제 객체를 찾기 위한 값으로 사용된다.  
분석한 게임은 엔티티를 하나의 배열로 관리하지 않고 메모리 성능을 위해 여러 개의 청크로 분할하여 저장하기에  
핸들에는 어느 청크에 위치하는지와 청크 내부의 몇 번째 슬롯인지를 함께 저장하며 이를 이용해 실제 객체를 찾는다.  

예를 들어 분석한 구조에서는 다음과 같은 형태의 계산이 사용되었다.  

```cpp
chunk = (handle & 0x7FFF) >> 9;
slot  = handle & 0x1FF;
```

- `0x1FF` 는 하위 9비트만 추출하여 청크 내부의 슬롯 번호를 구한다.
- `0x7FFF` 는 Handle의 유효한 하위 15비트만 사용한다.
- `>> 9` 는 하위 9비트를 제외하여 청크 번호를 얻는다.

계산된 Chunk Index를 이용해 dwEntityList에서 해당 Pawn이 저장된 Pawn List Entry를 찾고 이후 Slot Index를  
이용해 해당 Entry 내부의 C_CSPlayerPawn을 얻는다.   

이를 통해 Pawn에서 위치, 체력, 팀 등의 정보를 읽어 ESP를 구현한다.  

---

## 카메라와 좌표 변환이 필요한 이유

게임 내부의 좌표는 3D 월드 좌표를 이용한다.  
예를 들어 플레이어의 머리 위치는 아래와 같은 값으로 표현된다.  

```text
Vector3 worldPos;
worldPos.x = 좌우 위치
worldPos.y = 전후 위치
worldPos.z = 높이
```

하지만 내 화면에 그리기 위해서는 2D 픽셀 좌표가 필요하다.  

```cpp
Vector2 screenPos;
```

이 변환 과정을 일반적으로 `WorldToScreen`이라고 부른다.  

```text
3D 월드 좌표 { x, y, z } -> 2D 화면 좌표 { x, y }
```

좌표 변환을 이해하기 위해서는 먼저 카메라에 대한 개념이 필요하다.  
현실에서 사람이 눈을 통해 물체를 보는 것처럼 게임 내부에서도 물체를 화면에 보여주기 위한 기준점이 필요하다.  
이 기준점이 바로 카메라이다.  

<figure>
  <img src="/assets/img/GameHacking/ESP/ViewFrustum.png" alt="View Frustum">
  <figcaption>View Frustum</figcaption>
</figure>

위 그림에서 카메라 위치를 눈이라고 생각하면 된다.   
게임 월드에 존재하는 물체들은 실제 위치 그대로 화면에 표시되는 것이 아니라 카메라를 기준으로 어디에 보이는지 계산된 뒤 화면에 출력된다.  

그러므로 특정 플레이어의 월드 좌표를 알고 있더라도 그 좌표만으로는 화면의 어느 픽셀에 표시해야 하는지 알 수 없다.   
카메라 위치, 카메라가 바라보는 방향, 시야각, 화면 크기 등을 반영하여 3D 좌표를 2D 좌표로 변환해야 한다.  

그림에서 카메라 앞쪽으로 펼쳐진 공간을 View Frustum이라고 한다.  
이는 카메라가 볼 수 있는 공간을 의미하며 이 범위 안에 있는 오브젝트만 화면에 보이게 된다.  

- `Near Plane`: 카메라와 가장 가까운 절단 평면
- `Far Plane`: 카메라가 렌더링하는 최대 거리
- `View Frustum`: 카메라가 실제로 볼 수 있는 3차원 공간

정리하면 ESP는 엔티티에서 얻은 월드 좌표를 카메라 기준으로 변환하고 최종적으로 화면 픽셀 좌표로 바꾸는 과정이 필요하다.  

이 좌표 변환 방법은 크게 두 가지 방식으로 이야기 할수 있다.  

```text
1. Direct3D 함수를 이용한 좌표 변환
   -> 게임의 D3D Device를 얻어 Viewport와 Matrix를 가져온다.
   -> D3DXVec3Project() 같은 함수를 이용해 좌표를 변환한다.

2. 행렬 계산을 이용한 좌표 변환
   -> 게임 메모리에서 ViewMatrix를 직접 찾는다.
   -> 월드 좌표와 행렬을 직접 곱해 화면 좌표를 계산한다.
```

---

## Direct3D 함수를 이용한 좌표 변환

(Internal 방식)  
D3D9 기반은 Direct3D에서 제공하는 `D3DXVec3Project()` 함수를 이용해 3D 좌표를 2D 화면 좌표로 변환할 수 있다.  

```cpp
D3DXVECTOR3* D3DXVec3Project(
    D3DXVECTOR3* pOut,
    const D3DXVECTOR3* pV,
    const D3DVIEWPORT9* pViewport,
    const D3DXMATRIX* pProjection,
    const D3DXMATRIX* pView,
    const D3DXMATRIX* pWorld
);
```

각 인자의 의미는 다음과 같다.

- `pV`: 변환할 3D 좌표
- `pOut`: 변환 결과가 저장될 화면 좌표
- `pViewport`: 화면의 위치와 크기 정보
- `pProjection`: 3D 공간을 화면에 투영하기 위한 Projection Matrix
- `pView`: 월드 좌표를 카메라 기준 좌표로 바꾸기 위한 View Matrix
- `pWorld`: 오브젝트의 로컬 좌표를 월드 좌표로 바꾸기 위한 World Matrix

여기서 중요한 것은 `View Matrix`, `Projection Matrix`, `World Matrix`의 역할이다.

### World Matrix

World Matrix는 오브젝트의 로컬 좌표를 월드 좌표로 변환하는 행렬이다.  

예를 들어 캐릭터 모델 내부에서 머리의 위치가 `(0, 70, 0)`이라면 이는 모델을 기준으로 한 로컬 좌표이다.  
이후 캐릭터가 게임 월드의 `(500, 200, 0)` 위치에 배치되면 World Matrix를 통해 머리의 위치도 실제 월드 좌표로 변환된다.  

하지만 대부분의 게임에서 ESP는 메모리에서 플레이어 위치나 Bone 위치를 직접 읽어온다.  
이러한 값들은 이미 게임 엔진에서 World Matrix 연산이 완료된 월드 좌표이기 때문에 별도로 World Matrix를 다시 적용할 필요가 없다.  

정리하자면 일반적인 ESP에서는 World Matrix를 직접 사용하는 경우는 별로없고 이미 계산된 월드 좌표를 View Matrix에 전달하여 화면 좌표로 변환하는 과정에서 주로 사용된다.  

```cpp
D3DXMatrixIdentity(&worldMatrix);
```

### View Matrix

View Matrix는 월드 좌표를 카메라 기준 좌표로 변환하는 행렬이다.  

월드 좌표는 게임 맵 전체를 기준으로 한 위치이다.  
하지만 화면에 물체를 그리려면 카메라 기준으로 해당 물체가 앞에 있는지, 왼쪽에 있는지, 오른쪽에 있는지 판단해야 한다.  

즉 View Matrix는 다음 변환을 담당한다.  

```text
월드 기준 좌표 -> 카메라 기준 좌표
```

### Projection Matrix

Projection Matrix는 카메라 기준 3D 좌표를 2D 화면으로 투영하기 위한 행렬이다.  

3D 공간의 물체를 2D 화면에 표시할 때는 원근감이 반영되어야 한다.  
가까운 물체는 크게 보이고 먼 물체는 작게 보인다.  
이러한 원근 투영을 담당하는 것이 Projection Matrix이다.  

`D3DXVec3Project()`는 내부적으로 다음 흐름을 처리한다.  

```text
Local 좌표
-> World Matrix
-> View Matrix
-> Projection Matrix
-> Viewport 변환
-> Screen 좌표
```

이 함수에 필요한 Viewport와 Matrix는 `pDevice`에서 얻을 수 있다.

```cpp
D3DVIEWPORT9 viewport;
D3DXMATRIX viewMatrix;
D3DXMATRIX projectionMatrix;
D3DXMATRIX worldMatrix;

pDevice->GetViewport(&viewport);
pDevice->GetTransform(D3DTS_VIEW, &viewMatrix);
pDevice->GetTransform(D3DTS_PROJECTION, &projectionMatrix);
D3DXMatrixIdentity(&worldMatrix);

D3DXVec3Project(
    &screenPos,
    &worldPos,
    &viewport,
    &projectionMatrix,
    &viewMatrix,
    &worldMatrix
);
```

`GetViewport()`는 현재 렌더링 대상의 화면 크기와 위치를 가져온다.  
`GetTransform()`은 D3D9의 고정 함수 파이프라인에서 사용되는 View / Projection / World 행렬을 가져온다.  

이 방식은 게임에서 사용되는 `pDevice`에 접근해야하기 때문에 일반적으로 `EndScene()`이나 `DrawIndexedPrimitive()` 같은 렌더링 함수를 후킹하여 `pDevice`를 얻어야 한다.  

```text
EndScene / DIP Hook
-> 게임이 사용하는 pDevice 획득
-> GetViewport / GetTransform 호출 -> View, Projection 얻기
-> D3DXVec3Project로 3D 좌표를 2D 좌표로 변환
-> ID3DXFont, ID3DXLine 또는 Direct3D 렌더링 API를 이용해 화면에 출력
```

D3D11 기반 게임에서는 D3D9의 `EndScene()` 대신 `IDXGISwapChain::Present()`를 후킹하는 방식이 자주 사용된다.

---

## 행렬 계산을 이용한 좌표 변환

외부 오버레이 방식에서는 게임의 `pDevice`를 직접 사용하지 않는다.  
따라서 `pDevice->GetViewport()`나 `pDevice->GetTransform()`을 호출할 수 없다.  

게임 메모리를 분석하여 ViewMatrix를 찾아야 한다.  

```cpp
Matrix viewMatrix;
ReadMemory(client + dwViewMatrix, viewMatrix);
```

ViewMatrix를 찾는 방법은 게임마다 다르다.  
D3D 기반 게임에서는 `D3DXMatrixLookAtLH()`처럼 View Matrix를 생성하는 함수가 호출되는 지점을 후킹하여  
생성되는 값을 추적할 수도 있고 메모리 스캔을 통해 게임 엔진이 저장해둔 ViewMatrix를 직접 찾을 수도 있다.  

게임 메모리에 저장된 ViewMatrix를 찾아 읽고 이를 이용해 직접 `WorldToScreen()` 함수를 작성하였다.  

```cpp
bool WorldToScreen( const Matrix& view_matrix, const Vector3& world, int screen_width, int screen_height, Vector2& screen)
{
    const float* m = view_matrix.m;

    const float w = m[12] * world.x + m[13] * world.y + m[14] * world.z + m[15];
    if (w < 0.01f)  return false;

    float x = m[0] * world.x + m[1] * world.y + m[2] * world.z + m[3];
    float y = m[4] * world.x + m[5] * world.y + m[6] * world.z + m[7];

    const float inv_w = 1.0f / w;
    x *= inv_w;
    y *= inv_w;

    screen.x = (screen_width * 0.5f) * (1.0f + x);
    screen.y = (screen_height * 0.5f) * (1.0f - y);

    return true;
}
```

여기서 `x`, `y`, `z`는 월드 좌표의 위치 값이다.  

그런데 좌표 변환 과정에서 `w`라는 값이 보이는데 해당 값은 동차 좌표계에서 사용하는 값이다.   
일반적으로 3D 좌표는 `{ x, y, z }`로 표현하지만 그래픽스에서는 행렬 변환과 원근 투영을 편하게 처리하기 위해 `{ x, y, z, w }` 형태를 사용한다.  

```text
일반 3D 좌표
{ x, y, z }

동차 좌표
{ x, y, z, w }
```

동차 좌표계를 사용하는 이유는 이동, 회전, 크기 변환, 투영 같은 여러 변환을 하나의 4x4 행렬 곱셈으로 처리할 수 있기 때문이다.  

좌표 변환 후에는 `x`, `y`를 `w`로 나누는 과정이 필요하다.  

```cpp
x *= 1.0f / w;
y *= 1.0f / w;
```

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/parallel.png" alt="parallel(평행)">
    <figcaption>parallel(평행)</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/perspective.png" alt="perspective(원근)">
    <figcaption>perspective(원근)</figcaption>
  </figure>
</div>


이 과정을 Perspective Divide라고 한다.  
쉽게 말해, 3D 공간의 좌표를 화면에 보이는 비율로 바꾸는 과정이다.

가까운 물체는 크게 보이고 먼 물체는 작게 보이는 원근감도 이 과정과 관련이 있다.  
카메라에서 멀리 있는 좌표일수록 `w`의 영향이 커지고 `x / w`, `y / w`를 거치면서 화면에서 더 중심에 가까운 작은 변화로 투영된다.  

이후 정규화된 좌표를 실제 화면 픽셀 좌표로 변환한다.  

```cpp
screen.x = (screen_width * 0.5f) * (1.0f + x);
screen.y = (screen_height * 0.5f) * (1.0f - y);
```

여기서 `screen_width`와 `screen_height`는 오버레이 창의 크기이다.  
오버레이 창은 게임 클라이언트 영역과 같은 위치와 크기로 맞춰두기 때문에 변환된 좌표에 그대로 선이나 문자를 그릴 수 있다.

---
## 외부 오버레이에 그리기

이번 구현에서는 Internal 후킹을 사용하지 않고 외부 오버레이 방식을 선택하였다.  
외부 오버레이 방식에서는 게임 창 위에 투명한 윈도우를 하나 더 생성한다.   
그리고 해당 윈도우에 D3D9 Device를 생성하여 선과 문자를 그린다.  

```text
게임 창 위치 획득
-> 투명 오버레이 창 생성
-> D3D9 Device 생성
-> 매 프레임 위치 동기화
-> ESP Render
```

투명 오버레이를 만들때는 아래와 같다.  

```cpp

Overlay::ctx.hwnd = CreateWindowEx(
    WS_EX_TOPMOST | WS_EX_LAYERED | WS_EX_TRANSPARENT | WS_EX_TOOLWINDOW,
    ...
);

SetLayeredWindowAttributes(Overlay::ctx.hwnd, RGB(0, 0, 0), 0, LWA_COLORKEY);

```

- `WS_EX_TOPMOST`: 오버레이 창을 항상 게임 창 위에 표시한다. 
- `WS_EX_LAYERED`: 투명도를 적용할 수 있는 Layered Window를 생성한다.
- `WS_EX_TRANSPARENT`: 마우스 입력을 오버레이가 받지 않고 아래의 게임 창으로 전달한다. 
- `WS_EX_TOOLWINDOW`: 작업 표시줄과 Alt + Tab 목록에 오버레이 창이 표시되지 않도록 한다.
- `SetLayeredWindowAttributes` : RGB(0, 0, 0)을 투명 색상으로 지정하겠다. 


선을 그릴 때는 `ID3DXLine`을 이용한다.  

```cpp
void DrawLine(float x1, float y1, float x2, float y2, D3DCOLOR color)
{
    D3DXVECTOR2 points[2]
    {
        { x1, y1 },
        { x2, y2 }
    };

    pLine->SetWidth(1.0f);
    pLine->Draw(points, 2, color);
}
```

문자는 `ID3DXFont`를 이용한다.

```cpp
void DrawString(float x, float y, LPCWSTR text, D3DCOLOR color)
{
    RECT rect;
    SetRect(&rect, x, y, x, y);
    pFont->DrawTextW(nullptr, text, -1, &rect, DT_CENTER | DT_NOCLIP, color);
}
```

---

## Advanced

엔티티의 월드 좌표를 화면 좌표로 변환이 가능하다면 이를 이용해 여러 형태의 ESP를 만들 수 있다.  

---

### Name / Health ESP

Name ESP는 플레이어 이름을 읽어 머리 위에 출력하면 된다.  
Health ESP는 플레이어 체력 값을 읽어 발 아래나 박스 하단 기준 위치에 출력하면 된다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/ESPname.jpg" alt="Name">
    <figcaption>Name</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/ESPhealth.jpg" alt="Health">
    <figcaption>Health</figcaption>
  </figure>
</div>

---

### Bone Logger / Skeleton

Bone Logger는 Bone Array를 순회하면서 각 Bone 번호를 화면에 출력해 필요한 뼈 번호를 찾을수 있다.  
이를 통해 머리, 목, 가슴, 팔, 다리 등에 해당하는 Bone Index를 확인할 수 있다.  

Skeleton은 이렇게 찾은 Bone 좌표들을 화면 좌표로 변환한 뒤 서로 선으로 연결하는 방식이다.  
머리에서 목, 목에서 가슴, 어깨에서 팔꿈치, 팔꿈치에서 손목처럼 주요 관절을 연결하면 캐릭터의 뼈대가 표시된다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/ESPboneLogger.jpg" alt="Bone Logger">
    <figcaption>Bone Logger</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/ESP/ESPskeleton.jpg" alt="Skeleton">
    <figcaption>Skeleton</figcaption>
  </figure>
</div>

---

### 2D Box

2D Box는 플레이어의 머리 좌표와 발 좌표를 기준으로 사각형을 그리면 된다.  
머리와 발의 월드 좌표를 각각 화면 좌표로 변환한 뒤 두 좌표의 높이 차이를 이용해 박스의 세로 길이를 구한다.  

이후 세로 길이에 일정 비율을 곱해 가로 폭을 정하고 계산된 네 점을 선으로 연결하면 된다.  

<figure>
  <img src="/assets/img/GameHacking/ESP/ESPbox.jpg" alt="2D Box">
  <figcaption>2D Box</figcaption>
</figure>

---

### Snap Line

Snap Line은 화면의 기준점에서 대상까지 선을 그리는 기능이다.  
보통 화면 하단 중앙을 시작점으로 잡고 플레어어의 발이나 몸 중앙까지 선을 연결한다.

<figure>
  <img src="/assets/img/GameHacking/ESP/ESPsnapline.jpg" alt="Snap Line">
  <figcaption>Snap Line</figcaption>
</figure>

---

## 마무리

<figure>
  <img src="/assets/img/GameHacking/ESP/ESPall.jpg" alt="All">
  <figcaption>All</figcaption>
</figure>

Internal 방식은 게임 내부의 렌더링 흐름을 후킹해 `pDevice`, `Viewport`, `Matrix`를 활용하는 방식이다.  
External 방식은 게임 외부에서 메모리를 읽고 별도의 투명 오버레이 위에 직접 그리는 방식이다.  

본 문서에서는 분석 대상 게임이 D3D11 기반이지만 이전 D3D9 문서와 연결되는 흐름을 만들기 위해 D3D9 기반 외부 오버레이를 사용하였다.  
좌표 변환은 게임 메모리에서 읽은 ViewMatrix를 이용해 직접 수행하고 변환된 좌표를 통해 여러가지 ESP 를 만들어 보았다.  

다음 글에서는 ESP에서 사용한 좌표와 각도 개념을 통해 Aimbot의 기본 원리를 알아보겠습니다.  