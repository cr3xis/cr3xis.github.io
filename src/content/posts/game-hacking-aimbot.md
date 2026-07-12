---
title: '게임핵의 원리 (4) - Aimbot 핵'
published: 2026-06-20
description: '각도 계산, FOV, Smooth 처리를 중심으로 Aimbot의 동작 원리를 분석합니다.'
image: '/assets/img/GameHacking/Aimbot/aimbot.png'
tags:
  - 'Aimbot'
  - 'ViewAngle'
  - 'CalcAngle'
  - 'FOV'
  - 'Smooth'
category: 'Game Hacking'
draft: false
---


해당 문서는 게임에서 사용되는 불법 프로그램들의 동작 원리와 구현 구조를 분석하는 기술 문서입니다.  
본 문서는 보안 연구 및 교육 목적으로 작성되었으며 실제 게임에 대한 악용을 권장하지 않습니다.

---

이전 문서에서는 게임 내부 데이터를 읽고 화면 위에 표시하는 ESP의 동작 원리를 살펴보았다.  

- [게임핵의 원리 (3) - ESP 핵](/posts/game-hacking-esp/)

이번 문서에서는 ESP에서 사용했던 좌표와 각도 개념을 이어서 Aimbot의 기본 원리를 분석한다.  

---

## Aimbot이란 무엇인가?
<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Aimbot/aimbot.png" alt="Aimbot" style="display: block; margin: 0 auto;">
  <figcaption>Aimbot</figcaption>
</figure>


Aimbot은 사용자가 직접 조준점을 움직이지 않아도 특정 대상을 자동으로 조준하게 만드는 기능이다.  

FPS 게임에서 총알은 일반적으로 사용자가 바라보는 방향을 기준으로 발사된다.  
그러므로 에임봇의 핵심은 내가 바라봐야 하는 방향을 계산하고 그 방향으로 시점을 이동시키는 것이다.  

---

## Aimbot의 구현 방식

### 1. 각도를 계산하여 회전하는 방식

내 위치와 적 위치를 이용해 적을 바라보기 위한 각도를 계산한다.  
그 다음 게임 메모리에 저장된 ViewAngle을 수정하여 카메라를 회전시키는 방식이다.  

```text
내 위치 + 적 위치
-> 목표 Pitch와 Yaw 계산
-> ViewAngle 수정
```

이번 문서에서 중점적으로 다룰 방식이다.  

### 2. 화면 좌표를 계산하여 마우스를 이동하는 방식

적의 3D 월드 좌표를 2D 화면 좌표로 변환한 뒤 화면 중앙에서 해당 위치까지 마우스를 이동시키는 방식이다.  

```text
적 월드 좌표
-> WorldToScreen
-> 화면 중앙 기준 이동량 계산
-> SendInput 같은 입력 API로 마우스 이동
```

이 방식은 ViewAngle 메모리를 직접 수정하지 않아도 된다.  

### 3. AI를 이용한 방식

게임 메모리를 읽지 않고 화면 이미지를 분석하여 적을 찾는 방식도 존재한다.  
이는 모델 인식이나 색상 인식에 가까운 방식이며 이 글에서는 다루지 않는다.  

---

## 필요한 정보

에임봇은 플레이어의 시점에서 목표 위치까지 방향을 계산하여 카메라가 바라보는 방향을 변경하는 방식으로 동작한다.  


- Camera Position (Eye Position) : 현재 시점의 위치 
- Target Position : 조준할 대상의 위치 
- View Angle (Pitch, Yaw) : 계산된 각도를 적용할 플레이어의 시점 

위 정보만으로 가장 기본적인 에임봇의 형태를 완성 시킬수 있다.  
하지만 원하는 대상만 조준하기 위해서는 아래와 같은 정보들도 사용하는 경우가 많다.  

팀, 체력, 벽, 타격위치, 거리 등등이 있을걸로 생각된다.  

---

## Euler Angle

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Aimbot/eulerangle.png" alt="Euler Angle" style="display: block; margin: 0 auto;">
  <figcaption>Euler Angle</figcaption>
</figure>

게임에서 바라보는 방향은 보통 각도로 표현된다.  
이때 자주 보이는 개념이 Euler Angle이다.

```text
Pitch -> 위를 볼지 아래를 볼지
Yaw   -> 왼쪽을 볼지 오른쪽을 볼지
Roll  -> 화면을 기울일지
```

FPS 게임에서 조준에 주로 필요한 값은 Pitch와 Yaw이다.  
Roll은 화면 기울기와 관련된 값이라 일반적인 조준 계산에서는 거의 사용하지 않는다.

---

## Degree와 Radian

사람이 각도를 생각할 때는 보통 90도나 180도처럼 Degree 단위를 사용한다.  
하지만 C++의 `atan2` 같은 삼각 함수는 Radian 단위로 값을 반환한다.  

- Degree: 한 바퀴를 360도로 표현 ( 30° , 60°, 90° )
- Radian: 원의 반지름과 호의 길이를 기준으로 표현 ( 1.2 , 3.4, 5.5 )

그래서 `atan2`를 통해 얻은 값을 게임에서 사용하는 각도 값으로 쓰려면 Degree로 변환해야 한다.  

계산 공식은 아래와 같다.  

```cpp
degree = radian * 180.0f / PI;
radian = degree * PI / 180.0f;
```

---

## Yaw / Pitch 계산

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Aimbot/3d.png" alt="3D" style="display: block; margin: 0 auto;">
  <figcaption>3D</figcaption>
</figure>

필요한 데이터와 기본 수학 개념을 알았다면 이제 실제 각도를 계산할 수 있다.  
계산의 시작은 내 눈 위치를 기준점으로 두는 것이다.

```cpp
Vector3 delta;
delta.x = target.x - eye.x;
delta.y = target.y - eye.y;
delta.z = target.z - eye.z;
```

`delta`는 내 위치에서 타겟까지의 방향 벡터이다.  
월드 전체 기준 좌표를 내 위치 기준 좌표처럼 바꿔서 보는 과정이다.

---

### 1. Yaw (좌/우)

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Aimbot/yaw.png" alt="YAW" style="display: block; margin: 0 auto;">
  <figcaption>YAW</figcaption>
</figure>
 
Yaw는 좌우 회전 각도이다.  
위에서 내려다본 2D 평면을 기준으로 내 위치가 `0, 0, 0`에 있다고 가정한다.


위에서 바라본 시점에서는 높이 값인 `z`를 생각하지 않는다.  
바닥 평면에서 타겟이 어느 방향에 있는지만 계산하면 된다.  

```cpp
float yaw = atan2(delta.y, delta.x) * 180.0f / PI;
```

`atan2(delta.y, delta.x)`를 이용해 타겟이 어느 방향에 있는지 각도를 구한다.  
반환값은 Radian이므로 `180 / PI`를 곱해 Degree로 변환한다.

---

### 2. Pitch (상/하)

<figure style="text-align: center;">
  <img src="/assets/img/GameHacking/Aimbot/pitch.png" alt="PITCH" style="display: block; margin: 0 auto;">
  <figcaption>PITCH</figcaption>
</figure>

Pitch는 위 아래 회전 각도이다.  
Pitch를 구하려면 먼저 내 위치에서 타겟까지의 수평 거리가 필요하다.


수평 거리는 `x`와 `y`만 사용해 구한다.

```cpp
float hyp = sqrt(delta.x * delta.x + delta.y * delta.y);
```

`hyp`는 위에서 바라본 평면에서 내 위치와 타겟 사이의 거리이다.  
이 값과 높이 차이인 `delta.z`를 이용하면 위 아래 각도를 구할 수 있다.

```cpp
float pitch = -atan2(delta.z, hyp) * 180.0f / PI;
```

앞의 `-` 부호는 게임에서 사용하는 Pitch 방향에 맞추기 위한 값이다.  
게임마다 위를 볼 때 값이 증가하는지 감소하는지가 다를 수 있으므로 실제 값은 디버깅을 통해 확인해야 한다.

---
### CalcAngle

위 내용을 코드로 정리하면 아래처럼 된다.

```cpp
// src = ViewPos, dst = TargetPos
Vector2 CalcAngle(const Vector3& src, const Vector3& dst)
{
    const Vector3 delta
    {
        dst.x - src.x,
        dst.y - src.y,
        dst.z - src.z
    };

    const float hyp = std::sqrt(delta.x * delta.x + delta.y * delta.y);

    return
    {
        -std::atan2(delta.z, hyp) * 180.0f / PI, std::atan2(delta.y, delta.x) * 180.0f / PI
        // { pitch, yaw }
    };
}
```
이러한 형태의 계산식을 통해 어디를 바라봐야할지 각도를 구할수 있게 된다.  

---

## ViewAngle 적용

목표 각도를 구했다면 현재 ViewAngle을 해당 값으로 바꾸면 된다.

```cpp
Vector2 targetAngle = CalcAngle(localEyePos, targetBonePos);
*(Vector2*)(Client + Offsets::dwViewAngles) = targetAngle;
```

이 방식은 가장 단순하며 타겟이 정해지는 순간 시점이 바로 적을 향해 이동한다.  
아래 영상을 통해 확인을 할수 있다.  

<iframe
        width="100%"
        height="450"
        src="https://www.youtube.com/embed/Hgv_lMx5X0U"
        title="Aimbot(Standard)"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        referrerpolicy="strict-origin-when-cross-origin"
        allowfullscreen>
    </iframe>


---

## Advanced

지금까지 내 위치와 타겟의 위치를 이용해 목표 각도를 계산한 뒤 ViewAngle에 적용하는 가장 기본적인 Aimbot 방식을 알아보았다.  

하지만 이 방식만으로는 대상에 대한 구분없이 항상 조준하게 되므로 여러 문제가 발생하게 된다.  
같은 팀이나 이미 사망한 플레이어를 조준한다던지, 화면 밖이나 벽 너머의 적까지 대상으로 선택할수 있게 된다.  

따라서 실제 구현에서는 조준 대상을 선택하기 전에 다양한 조건을 통해 유효한 타겟만 선별하는 과정이 필요하다.  

일반적으로 체력, 팀, 거리, Fov, 벽 충돌 여부, 타격 부위 등의 조건을 이용해 대상을 필터링 한다.  

---

### FOV

`Fov(Field of View)`는 에임봇이 반응할 범위를 제한하기 위해 사용한다.  
현재 시점에서 가까이 있는 적이 아닌 멀리있는 적을 조준할 필요는 없기 때문이다.    

구하는 방식은 현재 내 시야각과 타겟을 바라보기 위해 계산한 각도의 차이를 이용해 구하면 된다. 

이후 Fov 기능이 켜져 있다면 설정한 범위 밖의 대상은 제외하는 조건을 걸면 된다.   

```cpp
Vector2 delta = GetAngleDelta(MyViewAngle, targetAngle);
float angleDistance = sqrt(delta.x * delta.x + delta.y * delta.y);

if (angleDistance > maxAngleRange) continue;
```

<iframe
        width="100%"
        height="450"
        src="https://www.youtube.com/embed/KMzn3_fLH_8"
        title="Aimbot(Fov)"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        referrerpolicy="strict-origin-when-cross-origin"
        allowfullscreen>
    </iframe>

---

### Smooth

`Smooth` 는 계산된 각도로 한 번에 이동하지 않고 조금씩 이동시키기 위해 사용한다.  
목표 각도로 바로 적용하면 시점이 순간적으로 타겟을 향하기 때문에 자연스러움을 추가 한거라고 생각하면 편하다.  

방법은 현재 내 시야각과 최종적으로 선택된 타겟 각도의 차이를 구한 뒤 그 값을 `smooth` 값으로 나누어 적용하면 된다.  

```cpp
Vector2 delta = GetAngleDelta(MyViewAngle, targetAngle);

FinalAngle.x = MyViewAngle.x + delta.x / smooth;
FinalAngle.y = MyViewAngle.y + delta.y / smooth;
```

<iframe
        width="100%"
        height="450"
        src="https://www.youtube.com/embed/RhOxRNrwhUI"
        title="Smooth"
        frameborder="0"
        allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
        referrerpolicy="strict-origin-when-cross-origin"
        allowfullscreen>
    </iframe>

---

## 마무리

Aimbot의 핵심은 상대를 바라보기 위한 목표 각도를 계산해 View Angle에 적용하는 것이다.  

실제 구현에서는 계산된 각도 뿐만 아니라 Fov, Smooth 등의 조건을 함께 이용해보았다.  

다음 글에서는 화면은 움직이지 않고 발사 각도만 변경하는 `Silent Aimbot`의 동작원리에 대해서 알아보도록 하겠다.  