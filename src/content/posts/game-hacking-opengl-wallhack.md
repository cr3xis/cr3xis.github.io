---
title: '게임핵의 원리 (1) - OpenGL 기반 WallHack'
published: 2026-05-10
description: 'OpenGL의 깊이 버퍼와 렌더링 함수를 이용한 WallHack 동작 원리를 분석합니다.'
image: '/assets/img/GameHacking/OpenGL/wallhack1.png'
tags:
  - 'OpenGL'
  - 'wallhack'
  - 'z-buffer'
  - 'depth-buffer'
  - 'rendering-pipe'
  - 'counter-strike'
category: 'Game Hacking'
draft: false
---


해당 문서는 게임에서 사용되는 불법 프로그램들의 동작 원리와 구현 구조를 분석하는 기술 문서입니다.  
본 문서는 보안 연구 및 교육 목적으로 작성되었으며, 실제 게임에 대한 악용을 권장하지 않습니다.  

---

## 월핵이란 ? 
월핵은 벽이나 장애물 뒤에 있는 플레이어를 화면에 표시하여 비정상적인 플레이를 하는것을 의미합니다.  
해당 핵을 사용하는 악성 플레이어는 벽 뒤에 존재하는 적의 위치를 파악할수 있게 됩니다.  

월핵을 구현하는 방식은 게임마다 그래픽 라이브러리가 다르기 때문에 조금씩 달라지게 됩니다.  

<figure>
  <img src="/assets/img/GameHacking/OpenGL/wallhack1.png" alt="wallhack">
  <figcaption>wallhack</figcaption>
</figure>
---

## Z Buffer (Depth Buffer)

월핵을 구현하기 위해서는 먼저 `Z Buffer`의 개념을 알아야 합니다.  
게임은 화면에 수많은 오브젝트를 그리는데, 이 과정에서 어떤 물체가 앞에 있고 어떤 물체가 뒤에 있는지를  
판단해야 하는데 이를 위해 `Z Buffer`를 사용합니다.  

`Z Buffer`는 화면의 각 픽셀에 대해 깊이 값을 저장하는 버퍼로 렌더링 과정에서  
현재 픽셀과 새롭게 그려질 픽셀의 깊이를 비교하여 더 가까운 물체만 화면에 출력합니다.  

<figure>
  <img src="/assets/img/GameHacking/OpenGL/zbuffer.png" alt="Z-buffer">
  <figcaption>Z-buffer</figcaption>
</figure>

위 그림을 보면 렌더링 결과를 저장하는 Color Buffer와 별도로 깊이 정보를 저장하는 Depth Buffer가 존재하는 것을 알 수 있습니다.   
오브젝트가 그려질 때마다 해당 픽셀의 깊이 값이 함께 저장되며, 이후 다른 오브젝트가 같은 위치에 그려질 경우 저장된 깊이 값과 비교하여 어떤 픽셀이 화면에 표시될지 결정합니다.  

---

## 분석 환경 및 설명 

결과적으로 물체가 가려지는 이유가 `Z Buffer` 때문인 것을 이제 이해했기에 이런식으로 생각할수 있습니다.  
특정 오브젝트를 렌더링할 때 깊이 비교를 수행하지 않는다면 렌더링 엔진은 해당 오브젝트가 벽 뒤에 있는지  
여부를 판단할수 없게 되며 결과적으로 화면의 다른 오브젝트보다 항상 앞에 그려지게 되는겁니다.  

이것이 바로 `Z Buffer`를 이용한 가장 기본적인 월핵의 원리 입니다.  

대부분의 문서의 경우 `Assault Cube` 를 기준으로 분석하는 글들이 상당히 많았는데,  
필자의 경우 `Counter-Strike 1.6` 를 기준으로 분석하여 글을 작성하도록 하였습니다.  

---

### STEP - 1 렌더링 흐름 분석

해당 게임의 경우 `OpenGL` 이라는 라이브러리를 사용하고 있었습니다.  

`OpenGL`에서 깊이 버퍼 처리를 위해 `glDepthFunc()` 라는 함수를 호출하게 되는데,  
이 함수는 각 오브젝트의 깊이값을 비교해 화면에 그릴지 말지 결정하게 됩니다.  

glDepthFunc() 의 핵심적인 인자는 아래와 같습니다.  

|값|의미|
|:------|:---|
| `GL_LESS` | 더 가까운 물체만 화면에 표시합니다. |
| `GL_LEQUAL` | 더 가깝거나 동일한 깊이의 물체를 표시합니다. |
| `GL_ALWAYS` | 깊이 버퍼를 무시하고 항상 표시합니다. |

핵심 인자를 알았으니 직접 패치 해보도록 하겠습니다.

---

### STEP - 2 glDpethFunc() 인자 패치

glDepthFunc() 함수를 호출하는 주소를 역으로 추적했고,  
기존에 사용하던 GL_LEQUAL을 GL_ALWAYS 로 패치해보았습니다.

<figure>
  <img src="/assets/img/GameHacking/OpenGL/memory1.png" alt="Memory">
  <figcaption>Memory</figcaption>
</figure>

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/beforedepth.png" alt="GL_LEQUAL">
    <figcaption>GL_LEQUAL</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/afterdepth.png" alt="GL_ALWAYS">
    <figcaption>GL_ALWAYS</figcaption>
  </figure>
</div>

해당 방식으로 패치를 하게되면, 모든사물이 뒤섞이게 되어버립니다.  
그렇다면 원하는 오브젝트만 선택적으로 보이게 할수는 없을까 라는 생각이 들게 됩니다.  

일반적으로 `glDrawElements()`나 `glDrawArrays()`와 같은 Draw Call 함수를 후킹하여 구현합니다.  

`glDrawElements()`의 경우 렌더링되는 인덱스 개수를 나타내는 `count` 인자를 제공하기 때문에  
이를 이용하여 특정 오브젝트만 필터링 하는 방식이 널리 사용이 되었지만  
분석 대상 게임에서는 `glDrawElements()` 및 `glDrawArrays()` 호출이 되지 않았습니다.  

---

### STEP - 3 렌더링 함수 추적 

해당 게임은 Draw Call 기반 렌더링이 아닌 구형 OpenGL의 Immediate Mode 방식을 사용하고 있었습니다.  

일반적으로는 `glDrawElements()` 와 같은 함수를 기준으로 렌더링 대상을 분석하지만  
해당 게임에서는 해당 함수의 호출을 확인할 수 없었습니다.  

따라서 실제 정점이 전달되는 위치를 추적하는 방향으로 분석을 진행하였습니다.  

`glVertex3f()` 와 `glVertex3fv()` 는 정점 좌표를 OpenGL 파이프라인에 전달하는 함수이며  
호출 지점을 역추적한 결과 다음과 같은 렌더링 흐름을 확인할 수 있었습니다.  

```asm
FF 15 3820BC1F        - call dword ptr [hw.dll+A92038] ; { ->OPENGL32.glBegin }
FF 15 9820BC1F        - call dword ptr [hw.dll+A92098] ; { ->OPENGL32.glTexCoord2f }
FF 15 9017BC1F        - call dword ptr [hw.dll+A91790] ; { ->OPENGL32.glColor4f }
FF 15 301FBC1F        - call dword ptr [hw.qwglDeleteContext+3C] ; { ->OPENGL32.glVertex3f }
FF 15 D417BC1F        - call dword ptr [hw.dll+A917D4] ; { ->OPENGL32.glEnd }
```

이는 `glBegin()` 과 `glEnd()` 사이에서 정점 정보를 직접 전달하는 전형적인 Immediate Mode 렌더링 구조입니다.  


<figure>
  <img src="/assets/img/GameHacking/OpenGL/glBegin1.png" alt="glBegin() 호출부">
  <figcaption>glBegin() 호출부</figcaption>
</figure>

동일한 렌더링 루틴 내부에서 `GL_TRIANGLE_STRIP(0x05)` 과 `GL_TRIANGLE_FAN(0x06)` 이 함께 사용되는 것을  
확인할 수 있었습니다.  

플레이어 모델 렌더링 과정에서 여러 Primitive 조합으로 구성되어 렌더링되고 있음을  
확인할 수 있었고 각 Primitive Type 인자를 `GL_LINE_STRIP(0x02)` 으로 패치하여 확인한 결과  
모두 플레이어 모델 렌더링에 관여하고 있음을 확인할 수 있었습니다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/beforebegin.png" alt="Before">
    <figcaption>Before</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/afterbegin.png" alt="After">
    <figcaption>After</figcaption>
  </figure>
</div>

이를 통해 플레이어 모델이 렌더링 되는 경로를 식별하였으므로 해당 렌더링 함수를 후킹하여 깊이 테스트를 비활성화 해보겠습니다.  

---

### STEP - 4 캐릭터 렌더링 함수 후킹 

OpenGL의 깊이 테스트는 전역 상태로 관리되기 때문에, 단순히 `GL_DEPTH_TEST`를 비활성화 할 경우  
다른 렌더링 물체에도 영향을 주게 되기 때문에 그 부분을 주의해야 합니다.  

따라서 플레이어 모델이 렌더링 되는 시점에만 깊이 테스트를 비활성화하고  
렌더링이 종료된 이후에는 원래 상태로 복원하는 방식으로 구현했습니다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/glBegin1.png" alt="Before">
    <figcaption>Before</figcaption>
  </figure>
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/memory2.png" alt="After">
    <figcaption>After</figcaption>
  </figure>
</div>

아래와 같이 플레이어 모델이 렌더링 되는 구간에만 깊이 테스트를 비활성화하여  
벽이나 장애물 뒤에 위치하더라도 정상적으로 표시되는 것을 확인할 수 있습니다.  

<div style="display:grid; grid-template-columns:repeat(auto-fit,minmax(240px,1fr)); gap:12px; align-items:start;">
  <figure style="margin:0; text-align:center;">
    <img src="/assets/img/GameHacking/OpenGL/hookok.png" alt="hook">
    <figcaption>hook</figcaption>
  </figure>
</div>

이로써 핵심은 특정 렌더링 구간에서 OpenGL 상태를 조작하는 것이라는 걸 알수 있게되고  
동일한 기법을 응용하게 되면 단순한 월핵 뿐만이 아니라 다양한 시각적 효과를 구현할 수 있게 됩니다. 

---

### STEP - 5 응용하기

지금까지 구현한 방식은 단순히 월핵을 구현한 예제에 불과합니다.  
실제로는 렌더링 파이프라인의 상태를 제어할 수 있기 때문에 다양한 형태로 응용이 가능합니다.  

대표적으로는 벽의 투명도를 조절하는 `Glass Wall`, 모델의 색상을 변경하는 `Chams(형광 월핵)`,    
모델의 구조를 확인할 수 있는 `WireFrame` 등이 있으며 모두 동일한 원리를 기반으로 구현됩니다.  


#### Glass Wall

<figure>
  <img src="/assets/img/GameHacking/OpenGL/GlassWall.png" alt="GlassWall">
  <figcaption>GlassWall</figcaption>
</figure>

간략하게 설명하자면 알파 블렌딩을 이용하여 오브젝트를 반투명하게 렌더링하는 방식입니다.   
`glBlendFunc()`와 `GL_BLEND`를 통해 블렌딩을 활성화하고 Alpha 값을 조절하여 투명도를 적용할 수 있습니다.   

#### Chams

<figure>
  <img src="/assets/img/GameHacking/OpenGL/chams.png" alt="Chams">
  <figcaption>Chams</figcaption>
</figure>

참스는 다양한 방식으로 구현할 수 있지만 예제에서는 `glColor4f()`의 색상 값을 패치하여 구현하였습니다.  

#### WireFrame

<figure>
  <img src="/assets/img/GameHacking/OpenGL/wireframe.png" alt="wireframe">
  <figcaption>wireframe</figcaption>
</figure>

`glPolygonMode()` 는 폴리곤의 렌더링 방식을 결정하는 함수입니다.  
기본적으로 `GL_FILL` 상태로 설정되어 있어 폴리곤 내부까지 채워 렌더링하지만   
`GL_LINE`으로 변경할 경우 폴리곤의 외곽선만 렌더링 됩니다.

#### Translucent

<figure>
  <img src="/assets/img/GameHacking/OpenGL/trans.png" alt="Translucent">
  <figcaption>Translucent</figcaption>
</figure>

동일한 원리를 응용하면 모델의 투명도를 조절하는 반투명 월핵 역시 구현할 수 있습니다.

---

## 마무리 

이번 글에서는 OpenGL 기반 게임을 대상으로 깊이 버퍼의 동작 원리를 살펴보고 렌더링 함수를 분석하여  
월핵을 구현하는 과정을 정리했습니다.  

결국 핵심은 특정 오브젝트가 렌더링 되는 시점을 식별하고 해당 구간에서 렌더링 상태를 제어하는 것입니다.  

다음 글에서는 Direct3D9 환경에서 동일한 개념이 어떻게 구현되는지 작성해보겠습니다.  