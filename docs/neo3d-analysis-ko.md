# Neo 3D Engine — 전수조사 분석 & 활용 전략 (한국어 정리)

> 이 문서는 Neo 3D Engine 코드베이스 전체(39개 파일 / 약 2,447줄)를 직접 읽고
> 정리한 분석 리포트입니다. 구조 분석부터 설치법, 기술 정체성 판별,
> AI 에이전트 활용 가능성, 웹 포팅 전략, 유튜브 강의 기획, 수익화 전략까지
> 한 문서에 모았습니다.

---

## 📌 저장소 정보

| 항목 | 내용 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/Neo3dEngine |
| **원본 저장소 (upstream)** | https://github.com/IvanSobolev/Neo3dEngine |
| **원작자** | Ivan Sobolev ([@IvanSobolev](https://github.com/IvanSobolev)) |
| **원작 소개 영상** | https://youtu.be/vhYE882B9dE |
| **WIKI** | https://github.com/IvanSobolev/Neo3dEngine/wiki |
| **릴리스** | https://github.com/IvanSobolev/Neo3dEngine/releases/tag/v0.1.1 |
| **라이선스** | GNU GPL-3.0 (copyleft) |
| **버전** | v0.1.1 |
| **분석 기준 커밋** | `91d3edc` (Merge pull request #1) |
| **작성일** | 2026-10-01 |

> ⚠️ 이 저장소는 **포크**입니다. 코드 저작권은 원작자에게 있으며,
> 이 포크의 독자 커밋은 `CLAUDE.md` 추가(`6fa6660`)와 본 문서뿐입니다.
> 외부에 소개할 때는 반드시 원작자 크레딧을 표기해야 합니다.

---

## 1. 한 줄 정체

**터미널(콘솔) 창 안에서 ASCII 문자만으로 3D 장면을 실시간 레이트레이싱하는 C# 게임 엔진.**

- 외부 그래픽 라이브러리 **0개** (OpenGL/DirectX/Vulkan/Unity 전부 미사용)
- GPU 미사용 — 모든 연산을 **CPU에서** 수행 (README에 향후 Vulkan 포팅 계획 언급)
- 밝기를 17단계 문자 그라디언트 `" .:!/r(l1Z4H9W8$@"` 로 변환해 출력
- 목적: **교육 / 데모** (원작자가 유튜브 영상용으로 제작)

| 항목 | 값 |
|---|---|
| 언어 / 런타임 | C# / .NET 8.0 |
| 총 코드량 | 약 2,447줄 (`.cs` 39개) |
| 커밋 수 | 31개 (2026-06-06 시작) |
| 외부 NuGet 패키지 | **0개** |
| 테스트 코드 | **0개** |
| CI 워크플로 | **없음** |

---

## 2. 폴더 구조 전수조사

```
Neo3dEngine/
├── 3dEngine/                          ← 엔진 본체 (클래스 라이브러리 / .dll)
│   ├── Frame.cs                 (64)  ← 게임 루프 심장부
│   ├── Structure/                     ← 수학 기본기 (전부 자체 구현)
│   │   ├── Vector3.cs          (123)  ← 3D 벡터 + 회전행렬 (핵심)
│   │   ├── Vector2.cs           (40)
│   │   ├── Vector2Int.cs        (32)
│   │   ├── Ray.cs               (11)  ← 광선 (시작점 + 방향)
│   │   ├── RenderData.cs        (10)  ← 충돌 결과 (거리/법선/교차점/색)
│   │   └── FacingInfo.cs         (8)  ← .obj face 정보
│   ├── Shape/                         ← 렌더 가능한 도형
│   │   ├── Sphere.cs            (30)  ← 구 (2차방정식 판별식)
│   │   ├── Triangle.cs          (55)  ← Möller–Trumbore 알고리즘
│   │   └── Object3d.cs          (80)  ← 삼각형 묶음 = 3D 모델 + 바운딩 스피어
│   ├── AbstractClass/                 ← 상속용 뼈대
│   │   ├── Scene.cs             (50)  ← Start()/Update() — Unity 패턴
│   │   ├── Screen.cs            (46)  ← 화면 추상화 + Parallel.For
│   │   ├── GameObject.cs         (8)  ← Position / Rotate / Color
│   │   └── Light.cs             (50)  ← 조명 + 그림자(shadow ray)
│   ├── Implementation/
│   │   ├── Camera.cs            (21)  ← UV좌표 → 광선 변환
│   │   ├── ConsoleScreen.cs     (60)  ← 동기 버전 (구버전)
│   │   ├── ConsoleScreenAsync.cs(119) ← 콘솔 출력 최적화 핵심 (RLE)
│   │   ├── DisplaysManager.cs   (38)  ← 동기 버전
│   │   └── DisplayManagerAsync.cs(29) ← 가장 가까운 충돌 탐색
│   ├── Interfaces/                    ← ICamera / IScreen / IDisplays 등
│   ├── Inputs/                        ← OS별 키보드 입력 (Strategy 패턴)
│   │   ├── Input.cs            (103)  ← OS 자동 감지 후 Provider 선택
│   │   ├── Interfaces/IInputProvider.cs (12)
│   │   └── Implementations/
│   │       ├── User32InputProvider.cs  (104) ← Windows (user32.dll)
│   │       ├── LibX11InputProvider.cs  (350) ← Linux (libX11.so.6)
│   │       └── DotNetInputProvider.cs   (54) ← fallback
│   ├── Network/                       ← TCP 멀티플레이
│   │   ├── NetworkManager.cs   (108)  ← 서버/클라이언트 TCP
│   │   ├── PacketManager.cs    (62)   ← 패킷 등록/라우팅/이벤트 구독
│   │   ├── INetworkPacket.cs          ← 직렬화 인터페이스
│   │   └── NetworkUtils.cs     (19)   ← 로컬 IP 조회
│   ├── UI/                            ← 텍스트 UI 오버레이
│   │   ├── UIManager.cs        (14)
│   │   └── UIText.cs            (7)
│   └── StaticClass/
│       ├── GameTime.cs         (23)   ← deltaTime / FPS
│       └── ObjLoader.cs        (89)   ← Blender .obj 파서
│
├── SampleGame/                        ← 실행되는 예제 게임 (.exe)
│   ├── Program.cs              (51)   ← 진입점 ([S]erver / [C]lient 선택)
│   ├── Scenes/
│   │   ├── PreviewScene.cs    (205)   ← 구 + 큐브 + 바닥 + 원숭이 모델
│   │   └── PriviewNetworkScene.cs (282) ← 멀티플레이 로비 + 채팅
│   ├── NetworkPackets/
│   │   ├── TransformPacket.cs  (25)   ← 위치/회전 동기화
│   │   └── ChatPacket.cs       (13)   ← 채팅 메시지
│   └── monkey.obj            (2067)   ← Blender Suzanne 모델
│
├── .github/
│   ├── CONTRIBUTING.md                ← 기여 가이드 (철학: 고성능 + 의존성 0)
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/ (bug-crash-report.yml, feature-request.yml)
├── CLAUDE.md                          ← 페르소나 가이드 (이 포크에서 추가)
├── CHANGELOG.md                       ← v0.1.0 / v0.1.1 변경 이력
├── LICENSE                            ← GNU GPL-3.0 전문
├── readme.md / readmeRU.md            ← 영어 / 러시아어
├── 3dEngine.sln                       ← Visual Studio 솔루션
└── docs/neo3d-analysis-ko.md          ← 📄 이 문서
```

### 조직도 비유

| 폴더/파일 | 역할 비유 | 하는 일 |
|---|---|---|
| `Frame.cs` | 사장님 | "한 프레임 시작! 입력→로직→렌더" 무한 반복 |
| `Structure/` | 경리부 | 벡터 연산, 회전, 거리 계산 |
| `Shape/` | 배우들 | "저한테 광선 쏘셨나요? 2m 앞에서 맞았어요" |
| `Scene.cs` | 감독 | 배우/조명 배치, 매 프레임 지시 |
| `Camera.cs` | 카메라맨 | "이 픽셀은 이 방향으로 광선 쏘세요" |
| `Light.cs` | 조명 기사 | 밝기 계산, 그림자 판정 |
| `ConsoleScreenAsync.cs` | 인쇄소 | 숫자→문자 변환 후 터미널 출력 |
| `Inputs/` | 비서 | 키보드 상태 보고 |
| `Network/` | 외교부 | 다른 PC와 위치/채팅 교환 |

---

## 3. 작동 원리 — 실제 코드 흐름

### 핵심 비유: "손전등으로 어두운 방 더듬기"

캄캄한 방에서 손전등을 한 방향으로 비춰보면 → 뭔가에 닿으면 "2m 앞에 공이 있네",
안 닿으면 "저쪽은 비었구나". **레이트레이싱이 정확히 이것**이며,
다만 손전등을 **픽셀 개수만큼** 쏩니다. (100×30 터미널 = 프레임당 3,000개 광선)

### 1단계 — 게임 루프 (`3dEngine/Frame.cs:35`)

```csharp
while (_isRunning) {
    GameTime.StartFrame();         // 시간 측정 시작
    Input.Update();                // 키보드 상태 폴링
    if (Input.IsGetKey(Escape)) { _isRunning = false; continue; }
    _activeScene.Update();         // 게임 로직 (사용자 코드)
    _screen.RenderFrame(scene);    // 렌더링
    _screen.PrintText("Fps: " + ...);
    GameTime.EndFrame();           // deltaTime / FPS 계산
}
```
→ Unity의 `Update()` 루프와 동일한 구조. 게임 엔진의 뼈대가 한눈에 보입니다.

### 2단계 — 픽셀마다 광선 발사 (`ConsoleScreenAsync.cs:78`)

```csharp
Parallel.For(0, Height, j => {                 // CPU 전 코어 병렬화
    for (int i = 0; i < Width; i++) {
        Vector2 uv = CalculateUV(i, j);        // 화면좌표 → -1~1 정규화
        var pixelData = scene.GetPixelData(uv);
        BrightnessBuffer[j*Width+i] = pixelData.Brightness;
        ColorBuffer[j*Width+i]      = pixelData.Color;
    }
});
```

문자가 세로로 길쭉한 것을 보정하는 종횡비 처리 (`ConsoleScreenAsync.cs:19`):
```csharp
float pixelAspect = 11.0f / 24.0f;             // 문자 가로:세로 비율
_aspectRatio = windowAspect * pixelAspect;
```

### 3단계 — 충돌 판정

**구 (`Shape/Sphere.cs:14`)** — 2차방정식 판별식:
```csharp
float a = ray.RayDirection * ray.RayDirection;  // 연산자 오버로딩 = 내적
float b = 2 * (l * ray.RayDirection);
float c = l * l - R * R;
float d = b * b - 4 * a * c;
if (d < 0) return RenderData.NoRender;          // 판별식 < 0 → 미교차
float intersection = (-b - Math.Sqrt(d)) / (2 * a);   // 더 가까운 해
```

**3D 모델 (`Shape/Object3d.cs:51`)** — 2단계 최적화:
```csharp
// 1차: 바운딩 스피어로 빠른 컷 (성능의 핵심)
Vector3 L = Position - ray.RayStart;
float tca = L * ray.RayDirection;
float d2  = L * L - tca * tca;
if (d2 > _boundingRadius * _boundingRadius) return RenderData.NoRender;

// 2차: 통과한 광선만 삼각형 전수 검사
foreach (var face in _faces) { ... }
```
→ 화면 대부분은 빈 공간이므로 **공 하나 검사로 대다수를 컷**. 이게 없으면 FPS가 나오지 않음.

**삼각형 (`Shape/Triangle.cs`)** — Möller–Trumbore 알고리즘:
백페이스 컬링 → 행렬식(det) 검사 → 중무게좌표 u, v 범위 검사 → 교차 거리 t 산출.

### 4단계 — 조명 + 그림자 (`AbstractClass/Light.cs:33`)

```csharp
float dot = renderData.Normal * lightDir;      // 램버트 확산
if (dot <= 0) return 0;                        // 빛을 등진 면 → 어둠

// ★ 그림자: 충돌점에서 광원으로 광선을 한 번 더 쏜다
const float epsilon = 0.01f;                   // 자기 충돌(self-intersection) 방지
Ray shadowRay = new Ray(point + normal * epsilon, lightDir);
RenderData shadowHit = displaysManager.FindClosestIntersection(shadowRay, sceneObjects);
if (shadowHit.Intersection > -1 && shadowHit.Intersection < distanceToLight)
    return 0;                                  // 가로막힘 → 그림자

float attenuation = LightPower / (distance * distance + 1f);  // 역제곱 감쇠
return (int)(angleFactor * attenuation);
```
→ 광선을 **2번** 쏘는 진짜 레이트레이싱. 픽셀당 비용이 2배로 늘어남.

### 5단계 — 밝기 → 문자 변환

```csharp
private const string Gradient = " .:!/r(l1Z4H9W8$@";   // 17단계
_charBuffer[i] = Gradient[int.Clamp(brightness, 0, Gradient.Length - 1)];
```
문자 순서는 **글자가 차지하는 잉크 밀도** 순서입니다.
공백(0%) → `.` → `:` → ... → `@`(가장 밀도 높음).

```
        .:!!!/rr(ll:.                 ← 위쪽: 약한 광량
     .:!/rl1Z4H99H4Z1l/!:.
   .!/l1Z4H9W8$@@@$8W9H4Z1l!.        ← 중앙: @ 밀집 = 최대 밝기
  .!(1Z4H9W8$@@@@@@@$8W9H4Z1!.
     .:!/rl1Z4H99H4Z1l/!:.
        .:!!!/rr(ll:.                 ← 아래: 그림자
```

### 6단계 — 출력 최적화 (`ConsoleScreenAsync.cs:34`)

`Console.Write`는 느린 시스템 API이므로, 같은 색이 연속되는 구간을 묶어 한 번에 전송
(**Run-Length Encoding**):
```csharp
while (currentIndex + runLength < bufferLength &&
       ColorBuffer[currentIndex + runLength] == currentColor) { runLength++; }
if (Console.ForegroundColor != currentColor) Console.ForegroundColor = currentColor;
Console.Out.Write(_charBuffer, currentIndex, runLength);   // 묶어서 한 번에
```

### 성능 최적화 3대장 요약

| # | 기법 | 효과 | 위치 |
|---|---|---|---|
| ① | `Parallel.For` 병렬화 | 코어 수만큼 배속 | `ConsoleScreenAsync.cs:78` |
| ② | 바운딩 스피어 조기 종료 | 삼각형 검사 대폭 감소 | `Object3d.cs:51` |
| ③ | Run-Length 출력 묶음 | 느린 콘솔 API 호출 최소화 | `ConsoleScreenAsync.cs:34` |

---

## 4. 고급 서브시스템 2개

### 4-1. OS별 키보드 입력 (Strategy 패턴)

**문제:** 터미널 기본 입력(`Console.ReadKey`)은 "타자기" 방식이라
① 동시 입력 불가(W+A 대각선 이동 안 됨) ② 키를 "누르고 있는 상태"를 알 수 없음.

**해결:** OS API를 직접 호출 (P/Invoke).

| 환경 | Provider | 방식 |
|---|---|---|
| Windows | `User32InputProvider` | `GetAsyncKeyState()` + `GetForegroundWindow()` / `IsChild()` / `GetWindow(GW_OWNER)` 로 포커스 판정 (Windows Terminal 탭 지원) |
| Linux (X11) | `LibX11InputProvider` | `libX11.so.6` 의 `XQueryKeymap()`, `XQueryTree()` 로 윈도우 트리 순회, 레이아웃 독립 키 매핑 |
| 그 외 / Wayland / headless | `DotNetInputProvider` | fallback. 90ms 타임아웃으로 KeyUp을 **흉내냄** |

```csharp
// Inputs/Input.cs:19 — OS 자동 감지
if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows)) { /* User32 */ }
else {
    if (sessionType == "wayland" && !string.IsNullOrEmpty(waylandDisplay))
        WarningMessage = "WARNING: Wayland session detected...";   // 보안상 차단 → fallback
    else { /* LibX11 */ }
}
// 강제 종료 시 네이티브 리소스 정리
AppDomain.CurrentDomain.ProcessExit += (s, e) => Dispose();
Console.CancelKeyPress += (s, e) => { Dispose(); };
```

```csharp
// DotNetInputProvider.cs:11 — KeyUp 흉내내기
private readonly TimeSpan _keyReleaseTimeout = TimeSpan.FromMilliseconds(90);
// 90ms 동안 같은 키 입력이 없으면 "손 뗀 것"으로 처리
```

### 4-2. TCP 멀티플레이 + 커스텀 바이너리 직렬화

**패킷 봉투 규격 (헤더 12바이트):**
```
┌───────────┬───────────┬───────────┬─────────────┐
│  TypeId   │ SenderId  │  Length   │   Payload   │
│  4 bytes  │  4 bytes  │  4 bytes  │  (가변)      │
└───────────┴───────────┴───────────┴─────────────┘
```
받는 쪽이 payload를 해석하려면 **먼저 종류를 알아야** 하므로 TypeId가 선행합니다.

**안정적인 패킷 ID — 클래스 이름 해시 (`PacketManager.cs:19`):**
```csharp
int hash = 23;
foreach (char c in str) hash = hash * 31 + c;   // 양쪽 코드가 같으면 ID도 자동 일치
```
→ 수동으로 `1=Chat, 2=Transform` 매핑표를 관리할 필요가 없음.

**스레드 안전 처리 (우체통 패턴):**
```
수신 스레드(백그라운드) → ConcurrentQueue 에 enqueue
                              ↓
메인 스레드 → 매 프레임 ProcessEvents() 로 dequeue 후 핸들러 호출
```
→ 백그라운드 스레드가 게임 상태를 직접 건드리면 경쟁 상태(race)로 터지므로,
큐에만 넣고 **안전한 타이밍에 메인 스레드가 처리**. 게임 네트워킹 정석 패턴.

**이벤트 구독 방식:**
```csharp
PacketManager.RegisterPacket<TransformPacket>();
PacketManager.Subscribe<TransformPacket>(OnTransformReceived);
```

---

## 5. 설치 및 사용법

### 준비물: .NET 8.0 SDK (단 하나)

#### Windows
```powershell
winget install Microsoft.DotNet.SDK.8
dotnet --version                      # 8.0.xxx 확인
git clone https://github.com/bmshin94/Neo3dEngine.git
cd Neo3dEngine\SampleGame
dotnet run --configuration Release    # Release 필수! Debug는 2~3배 느림
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update && sudo apt install -y dotnet-sdk-8.0 libx11-6

echo $XDG_SESSION_TYPE                # "wayland" 면 조치 필요
sudo apt install -y xterm
WAYLAND_DISPLAY= xterm                # X11(XWayland) 모드로 새 터미널

# 열린 xterm 창에서:
cd ~/Neo3dEngine/SampleGame
dotnet run --configuration Release
```
> Wayland 환경에서는 X11 전역 폴링이 OS 보안으로 차단되어
> `DotNetInputProvider` 로 떨어집니다 → 대각선 이동 불가, 입력이 끊김.
> 시작 시 경고 후 2초 대기하고 그대로 실행됩니다 (`Input.cs:70`).

#### macOS
```bash
brew install --cask dotnet-sdk
cd Neo3dEngine/SampleGame && dotnet run -c Release
```
> ⚠️ **macOS 전용 Provider가 없습니다.** `User32`도 `libX11`도 사용 불가 →
> 항상 `DotNetInputProvider` fallback → 조작감 저하.
> 👉 `CoreGraphicsInputProvider` 구현은 좋은 첫 기여(PR) 거리입니다.

### 조작법 (코드에서 확인)

| 키 | 동작 | 소스 |
|---|---|---|
| `W` / `S` | 전진 / 후진 | `PriviewNetworkScene.cs:111` |
| `A` / `D` | 좌 / 우 스트레이프 | `:113` |
| `Space` | 상승 | `:115` |
| `Shift` | 하강 | `:116` |
| `←` `→` | 좌우 회전 (Yaw) | `:100` |
| `↑` `↓` | 상하 시선 (Pitch, ±1.5 클램프) | `:102` |
| `+` / `-` | 조명 밝기 증감 | `:118` |
| `T` | 채팅 모드 토글 | `:81` |
| `Ctrl` | (PreviewScene) 광원 이동 | README |
| `Esc` | 종료 | `Frame.cs:41` |

### 씬 전환 방법

`SampleGame/Program.cs` 마지막 두 줄의 주석을 교체:
```csharp
// 기본값: 멀티플레이 씬 (실행 시 S/C 선택을 물어봄)
new Frame(new PriviewNetworkScene(new DisplayManagerAsync(), isServer, ip, port), new ConsoleScreenAsync()).MainLoop();

// 아래 주석을 풀면: 싱글 씬 (원숭이 + 큐브 + 구 + 바닥) — 처음엔 이걸 권장
//new Frame(new PreviewScene(new DisplayManagerAsync()), new ConsoleScreenAsync()).MainLoop();
```

### 멀티플레이 (같은 LAN 내)

```
[A — 서버]                         [B — 클라이언트]
dotnet run -c Release              dotnet run -c Release
Select Mode: S                     Select Mode: C
Your Local IP: 192.168.0.5         Enter Server IP: 192.168.0.5
Enter Port: 7777                   Enter Server Port: 7777
```
> 인터넷 플레이는 공유기 7777 포트포워딩 필요.
> 다만 아래 "알려진 이슈"의 `_client` 덮어쓰기 문제로 **실질 1:1만** 동작합니다.

### FPS 팁
터미널 창이 작을수록 광선 수가 줄어 FPS가 올라갑니다.
창 크기는 **시작 시 한 번만** 읽으므로(`Console.WindowWidth`, `ConsoleScreenAsync.cs:16`),
실행 중 창 크기를 바꾸면 화면이 깨집니다 → 재시작 필요.

---

## 6. 기술 정체성 — 플러그인? 스킬? MCP?

### 결론: **셋 다 아님.** 순수한 C# 프로그램입니다.

| 분류 | 해당? | 근거 |
|---|---|---|
| Claude Code 플러그인 | ❌ | `.claude-plugin/plugin.json` 없음 |
| Skill | ❌ | `SKILL.md` + frontmatter 없음 |
| MCP 서버 | ❌ | JSON-RPC 서버 구현 없음. 네트워크는 게임용 **raw TCP 소켓** |
| **실제 정체** | ✅ | **.NET 8 클래스 라이브러리(.dll) + 콘솔 앱(.exe)** |

혼동의 원인은 루트의 `CLAUDE.md` 하나뿐입니다.
이 파일은 이 포크에서 추가한 **AI 페르소나 가이드**이며, 엔진 코드와는 무관합니다.

```
Neo3dEngine/
├── 3dEngine/      ← C# 3D 엔진 (AI 무관)
├── SampleGame/    ← C# 예제 게임 (AI 무관)
└── CLAUDE.md      ← 이것만 AI 관련 (이 포크에서 추가)
```

### 확장 아이디어: 실제로 만들 수 있다

**① Skill 로 만들기 (쉬움)**
```
.claude/skills/neo3d/SKILL.md
→ "Neo3D 씬 만들어줘" 로 Scene 클래스 자동 생성
→ Vector3 / Light / Camera API 사용법을 스킬 본문에 수록
```

**② MCP 서버로 만들기 (임팩트 큼)**
```
도구(tools):
  create_scene(objects)          # 씬 구성
  render() -> str                # ASCII 프레임 문자열 반환
  move_camera(dx, dy, dz)
  set_light(x, y, z, power)
  get_frame() -> str

→ LLM이 3D 공간을 "텍스트로 보고" 직접 조작 가능
```

---

## 7. API 토큰 필요 여부

### 결론: **전혀 필요 없음. 토큰 0개.**

| 검사 항목 | 결과 |
|---|---|
| NuGet 외부 패키지 | 0개 (`.csproj` 2개 모두 확인, `ProjectReference` 1개뿐) |
| `HttpClient` 사용 | 없음 |
| 읽는 환경변수 | `XDG_SESSION_TYPE`, `WAYLAND_DISPLAY`, `WINDOWID` (전부 OS 감지용) |
| API 키 / 시크릿 | 0개 |
| 외부 서버 접속 | 없음 (`127.0.0.1` 또는 사용자가 직접 입력한 IP) |
| 계정 가입 / 결제 | 불필요 |

```csharp
// Network/NetworkUtils.cs — 외부로 나가지 않음. 로컬 NIC IP만 조회
var host = Dns.GetHostEntry(Dns.GetHostName());
```

→ 완전 무료, 오프라인 동작, 비행기 모드에서도 실행 가능.

| 작업 | 토큰 필요? |
|---|---|
| Neo3D 빌드 / 실행 | ❌ |
| LAN 멀티플레이 | ❌ (IP만 알면 됨) |
| GitHub push | ✅ GitHub 인증 (SSH 키 또는 PAT) |
| AI 코딩 도구와 함께 개발 | ✅ 해당 서비스의 구독/API 키 |

### ⚠️ 보안 주의

`NetworkManager`에는 인증·검증·암호화가 **전혀 없습니다**:
```csharp
// NetworkManager.cs:63 — 전달된 길이를 그대로 신뢰
int length = reader.ReadInt32();
byte[] data = new byte[length];      // 악의적 거대 값 → OOM 가능
```
- 패킷 길이 상한 검사 없음
- 핸드셰이크 / 인증 없음
- 암호화 없음

→ **신뢰할 수 있는 LAN에서만 사용하고, 인터넷에 포트를 열어두지 마세요.**

---

## 8. AI 에이전트 구축에 도움이 되는가?

### 결론: 직접적으론 ❌ / 간접적으론 ⭐⭐⭐⭐ (의외로 큼)

**직접적 무관:** LLM 호출, 프롬프트, 벡터DB/임베딩, RAG, tool use, 에이전트 루프 — 전부 없음.

### 간접적 활용 4가지

#### ① 에이전트 테스트베드(Gym) — 가장 유망 ⭐⭐⭐⭐⭐

**핵심 통찰: 이 엔진의 출력이 "텍스트"다.**

| | 일반 3D 게임 | Neo3D |
|---|---|---|
| 출력 형식 | 이미지(PNG) | **ASCII 텍스트** |
| LLM이 보려면 | 비전 모델 + 토큰 폭발 | 그냥 읽으면 됨 |
| 토큰 비용 | 높음 | 약 1/50 수준 |
| 정답 검증 | 이미지 비교 (어려움) | `diff` 한 줄 (결정적) |

```
에이전트: get_frame()
결과:    "........................"
         "....  .:!/rl1Zl/:.  ...."
         "... .!(1Z4H9W8$@4Z1!...."
에이전트: "중앙에 밝은 구체 → 접근 시도"
에이전트: move_camera(forward=2)
결과:    (구체가 커짐) → 접근 성공 판정
```

가능한 태스크: "방에서 빨간 공을 찾아가라", "그림자 위치로 광원 방향을 추론하라",
"가려진 물체가 몇 개인지 세라" 등 **공간 추론(spatial reasoning) 벤치마크**.

#### ② 코딩 에이전트 실험장 ⭐⭐⭐⭐⭐

| 조건 | 평가 |
|---|---|
| 코드 크기 2,447줄 | 컨텍스트 윈도우에 전부 들어감 ✅ |
| 의존성 0개 | 환경 셋업 이슈 없음 ✅ |
| 자기완결성 | 외부 서비스 불필요 ✅ |
| 결과 검증 | 시각적 — 즉시 확인 ✅ |
| 개선 여지 | 버그 + 미구현 기능 다수 ✅ |

연습 과제 예: macOS 입력 Provider 구현, 테스트 코드 작성, 알려진 버그 수정, CI 구축.

#### ③ MCP 서버로 감싸 "공간 인지" 도구화 ⭐⭐⭐⭐
위 6절의 MCP 설계안 참조. 도구 설계 + 공간 추론 연구를 동시에 연습 가능.

#### ④ AI 설명 능력 평가용 교재 ⭐⭐⭐
중간 난이도 + 자기완결적 코드 → 요약/설명 품질 평가에 적합.

### 종합 점수

| 목적 | 점수 |
|---|---|
| 에이전트 **프레임워크** 참고 | ⭐ |
| 에이전트 **테스트 환경** | ⭐⭐⭐⭐⭐ |
| 코딩 에이전트 **실습장** | ⭐⭐⭐⭐⭐ |
| **MCP** 설계 연습 대상 | ⭐⭐⭐⭐ |
| **공간 추론 벤치마크** | ⭐⭐⭐⭐ |

---

## 9. React / PHP 포팅 가능성

### React (TypeScript): 가능하며 오히려 더 유리 ⭐⭐⭐⭐⭐

| 항목 | C# 콘솔 | React 웹 |
|---|---|---|
| 설치 장벽 | .NET SDK 필요 | URL만 열면 끝 ✅ |
| 공유 | "SDK 깔고 클론" | 링크 클릭 ✅ |
| 모바일 | ❌ | ✅ |
| 키 입력 구현 | OS API 약 500줄 | `onKeyDown` 몇 줄 ✅ |
| 멀티플레이 | 포트포워딩 필요 | WebSocket (방화벽 무관) ✅ |
| 포트폴리오 | 캡처만 | **직접 조작 가능** ✅ |
| 성능 | 네이티브 (빠름) | JS는 2~3배 느림 / WASM은 비슷 |

#### 포팅 매핑표

| C# | TypeScript / React |
|---|---|
| `struct Vector3` | `class Vector3` 또는 `Float32Array` (성능용) |
| `Parallel.For` | **Web Worker** N개 (코어 수만큼) |
| `ConsoleScreenAsync` | `<pre>` 태그 + `textContent` |
| `" .:!/r...@"` 그라디언트 | 그대로 사용 ✅ |
| `ConsoleColor` | CSS 클래스 / `<span style>` |
| `Console.WindowWidth` | `useRef` + 컨테이너 크기 측정 |
| `Input.IsGetKey()` | `Set<string>` + `keydown`/`keyup` |
| `GameTime.deltaTime` | `requestAnimationFrame(t)` 델타 |
| `TcpClient` | `WebSocket` 또는 WebRTC(P2P) |
| `BinaryWriter` | `DataView` / `ArrayBuffer` |
| `ObjLoader` | `fetch()` + 동일 파서 로직 |
| `while(_isRunning)` | `requestAnimationFrame` 루프 |

#### 추천 아키텍처
```
┌─────────────────────────────────────────┐
│ React (UI)  — 씬 선택, 설정, 채팅창      │
├─────────────────────────────────────────┤
│ 렌더 출력: <pre> 1개                     │
│   (React 리렌더 우회 — ref 직접 조작)    │
├─────────────────────────────────────────┤
│ Web Worker × N  — 레이트레이싱           │
│   화면을 가로줄로 분할 + SharedArrayBuffer│
├─────────────────────────────────────────┤
│ (선택) Rust/C++ → WASM — 약 10배 가속     │
└─────────────────────────────────────────┘
```

#### 성능 함정 2가지
```tsx
// ❌ 프레임마다 setState → 초당 60회 리렌더로 사망
const [frame, setFrame] = useState("");

// ✅ ref로 DOM 직접 조작 (React 우회)
const preRef = useRef<HTMLPreElement>(null);
useEffect(() => {
  const loop = () => {
    preRef.current!.textContent = renderFrame();
    requestAnimationFrame(loop);
  };
  loop();
}, []);
```
2) 글자마다 `<span>` 으로 색을 입히면 DOM 노드가 수만 개 → 멈춤.
   → **Run-Length로 묶어 `<span>` 최소화** (C# 원본과 동일 전략) 또는 단색 + 밝기만 사용.

### PHP: 기술적으로 가능하나 실용성 낮음 ⭐

| 문제 | 설명 |
|---|---|
| 실행 모델 | "요청 → HTML 반환 → 종료" 구조. 무한 게임 루프와 철학이 상반 |
| 속도 | 초당 수백만 부동소수점 연산 필요. C# 대비 10~50배 느림 |
| 병렬 | `Parallel.For` 대응물 없음 (`pcntl_fork` + 공유메모리는 난이도 높음) |
| 키 입력 | `GetAsyncKeyState` 대응물 없음 → 실시간 폴링 사실상 불가 |
| 값 타입 | `struct` 없음. 객체는 힙 + GC → 메모리 압박 |

php-cli로 억지로 구현하면 5~10 FPS, 구 1개 정도. 조작은 거의 불가.
(블로그용 "PHP로 레이트레이싱 해보기" 글감 정도는 가능)

#### PHP의 올바른 자리 — 백엔드
```
┌──────────────────────┐    ┌───────────────────────┐
│ 브라우저 (React+TS)  │◄──►│ PHP (Laravel) 백엔드   │
│ - 레이트레이싱        │    │ - 인증 / 회원관리      │
│ - ASCII 렌더링        │    │ - 씬 저장 / 불러오기   │
│ - 키 입력             │    │ - 랭킹 / 리더보드      │
└──────────────────────┘    │ - 결제 / 구독          │
         ↕ WebSocket         └───────────────────────┘
   (실시간은 Node / Ratchet)          ↕ MySQL
```

### 최종 권장

| 언어 | 적합도 | 용도 |
|---|---|---|
| **TypeScript + React** | ⭐⭐⭐⭐⭐ | 1순위. 접근성/공유/포트폴리오 |
| **Rust → WASM + React** | ⭐⭐⭐⭐⭐ | 성능까지 필요하면 최적 |
| C# (원본) | ⭐⭐⭐⭐ | 학습 최고, 배포 번거로움 |
| PHP (렌더링) | ⭐ | 권장하지 않음 |
| PHP (백엔드) | ⭐⭐⭐⭐ | 비즈니스 로직 담당으로는 적합 |

---

## 10. 유튜브 강의 제작 가능성

### 결론: 매우 적합 ⭐⭐⭐⭐⭐
애초에 원작자가 유튜브 영상용으로 제작한 프로젝트 → 영상 소재로 이미 검증됨.

| 조건 | 평가 |
|---|---|
| 썸네일 임팩트 | ⭐⭐⭐⭐⭐ "터미널에서 3D 게임?" |
| 결과 시각화 | ⭐⭐⭐⭐⭐ 코드 한 줄 → 화면 즉시 변화 |
| 난이도 스펙트럼 | ⭐⭐⭐⭐⭐ 초급(벡터)~고급(P/Invoke) |
| 시청자 설치 장벽 | ⭐⭐⭐⭐ SDK 하나 |
| 한국어 경쟁 | ⭐⭐⭐⭐⭐ 거의 없음 (블루오션) |
| 코드 분량 | ⭐⭐⭐⭐ 2,400줄 → 10~15편 분량 |

### 추천 커리큘럼 (15편)

**시즌 1: 기초 — "터미널에 그림 그리기"**
| # | 제목 | 핵심 | 길이 |
|---|---|---|---|
| 1 | 터미널에 3D가?! 충격 데모 | 완성품 시연 + 로드맵 (훅 영상) | 8분 |
| 2 | ASCII로 명암 만들기 | 그라디언트 원리, 사진→ASCII | 12분 |
| 3 | Vector3 직접 만들기 | 내적/외적/정규화를 그림으로 | 15분 |
| 4 | 게임 루프의 정체 | while + deltaTime + FPS | 12분 |
| 5 | 첫 번째 구 띄우기 | 2차방정식 교차 판정 | 18분 |

**시즌 2: 레이트레이싱 — "빛과 그림자"**
| # | 제목 | 핵심 | 길이 |
|---|---|---|---|
| 6 | 조명과 법선 벡터 | 램버트 확산 | 15분 |
| 7 | 그림자를 만들자 | Shadow ray, epsilon 생략 시 버그 시연 | 15분 |
| 8 | Möller–Trumbore | 논문 알고리즘 → 코드, 중무게좌표 | 20분 |
| 9 | Blender 모델 불러오기 | `.obj` 파서 직접 작성 | 18분 |
| 10 | 최적화 대작전 | 바운딩 스피어 + 병렬 + RLE, FPS Before/After | 20분 |

**시즌 3: 고급 — "진짜 게임으로"**
| # | 제목 | 핵심 | 길이 |
|---|---|---|---|
| 11 | 카메라 1인칭 조작 | 회전행렬, forward/right 벡터 | 15분 |
| 12 | OS API 직접 호출 (P/Invoke) | user32 / libX11, 동시 입력 문제 | 22분 |
| 13 | TCP 멀티플레이 기초 | 패킷 설계 + 바이너리 직렬화 | 20분 |
| 14 | 친구와 함께 플레이 | 위치 동기화 + 채팅, 2대 시연 | 18분 |
| 15 | 웹으로 포팅하기 (React) | TS 변환 + Web Worker | 25분 |

### Shorts / 단편 아이디어
- "터미널에서 3D가 돌아간다" (60초)
- "코드 1줄 지웠는데 그림자가 사라졌다" (epsilon 버그)
- "if 한 줄로 FPS가 10배 됐다" (바운딩 스피어)
- "SSH로 접속해서 서버에 3D 게임 띄우기"
- "GPU 없이 레이트레이싱 하는 방법"
- "AI에게 ASCII 3D를 보여주면 알아볼까?" (MCP 연계)

### 법적 체크리스트 (중요)

| 항목 | 해야 할 일 |
|---|---|
| 원작자 크레딧 | 영상 내 + 설명란에 "원작: Ivan Sobolev / github.com/IvanSobolev/Neo3dEngine" **필수** |
| GPL-3.0 표기 | 라이선스 명시 필수 |
| 수정 코드 배포 | 수정본은 **GPL로 공개**해야 함 → 강의 자료 레포를 공개하면 자동 해결 |
| 영상 광고 수익화 | ✅ 문제 없음 (영상은 별개 저작물) |
| 유료 강의 판매 | ✅ 가능 (영상은 유료 유지 OK, 코드는 GPL 공개) |
| 원본 영상 재업로드 | ❌ 저작권 침해 |

> 더 안전한 방법: 원본을 **참고만** 하고 처음부터 자체 코드로 재작성하면
> GPL 의무 자체가 발생하지 않습니다. "Neo3D에서 영감받아 밑바닥부터 만들기"
> 컨셉이 법적으로도 교육적으로도 최선입니다.

### 제작 실무 팁
- 터미널 폰트를 크게 (18~24pt) — 모바일 시청자 고려
- 어두운 배경 + 고대비 테마
- 창 크기 100×30 전후 (너무 크면 글자 식별 불가)
- Before/After 화면 분할 비교
- 버그 → 디버깅 → 해결 과정을 그대로 노출 (반응 좋음)
- 각 편마다 git 태그(`ep-01`, `ep-02`...) → 시청자 추적 용이
- 수식은 반드시 그림/애니메이션으로 변환

### 현실적 기대치
| 항목 | 예상 |
|---|---|
| 한국어 경쟁 | 거의 없음 |
| 1편(훅) 조회수 | 5,000 ~ 50,000 |
| 시리즈 평균 | 1,000 ~ 5,000 |
| 타겟 | 컴공 학생, 주니어 개발자, 게임 개발 입문자 |
| 광고 수익 | 월 10~50만원 (시리즈 완성 후) |

---

## 11. 수익화 전략

### 11-0. 먼저 — GPL-3.0 제약 완전 정리 (최우선)

GPL의 본질: **"내 코드 자유롭게 써라. 단, 네가 만든 파생물도 자유롭게 풀어라."** (copyleft)

| 하려는 것 | 가능? | 조건 |
|---|---|---|
| 학습 / 개인 사용 | ✅ | 제약 없음 |
| GitHub 포크 / 수정 공개 | ✅ | GPL 유지 |
| **유튜브 영상 수익화** | ✅ | 라이선스 무관 (별개 저작물) |
| **유료 강의 / 전자책** | ✅ | 강의는 유료 OK, 코드는 GPL 공개 |
| 기업 교육 / 워크숍 | ✅ | 코드 공개 시 OK |
| 후원 / 스폰서십 | ✅ | 제약 없음 |
| **SaaS (서버에서만 실행)** | ✅ | GPL-3는 **AGPL이 아님** → 바이너리 미배포 시 소스 공개 의무 없음 |
| 게임 제작 후 Steam 판매 | ⚠️ | 가능하나 **게임 소스도 GPL 공개** 필수 |
| 코드 자체를 유료 판매 | ❌ | 수령자가 재배포 가능 → 비즈니스 성립 불가 |
| 비공개 상용 제품에 탑재 | ❌ | 금지 |
| 라이선스를 MIT로 변경 | ❌ | 원저작권자만 가능 |

> ⚠️ 본 문서는 법률 자문이 아닙니다. 실제 금전이 걸리면 전문가 확인이 필요합니다.
> 특히 "GPL vs AGPL / SaaS 소스 공개 의무" 해석은 전문가 사이에서도 논쟁이 있습니다.

#### GPL을 완전히 벗어나는 유일한 방법
```
Neo3D를 "교재/참고자료"로만 보고,
코드를 한 줄도 복사하지 않고 처음부터 자체 구현
        ↓
저작권 100% 자체 보유 → 라이선스 자유 → 상용 가능
```
알고리즘과 아이디어는 저작권 대상이 아니고, **구체적 코드 표현**만 보호됩니다.
- Möller–Trumbore = 공개 논문 알고리즘
- 램버트 확산 = 물리 법칙
- ASCII 그라디언트 = 아이디어
- 바운딩 스피어 = 표준 기법

단 **clean room 원칙** 준수 필요: 원본을 보며 베끼지 않고, 개념 이해 후 자기 방식으로 작성.
(변수명/구조가 동일하면 의심 소지)

#### 권장 2단계 전략
1. **1단계** — 포크에서 학습 + 유튜브 (GPL 유지, 문제 없음)
2. **2단계** — 배운 내용으로 **TypeScript 버전을 자체 작성** → MIT 라이선스 → 상용 자유

"남의 코드 포팅"보다 "직접 구현"이 포트폴리오 가치도 훨씬 높습니다.

---

### 11-1. 1위 — 유튜브 + 온라인 강의 (가장 현실적)

**3단 깔때기 구조**
```
1단: 유튜브 무료 시리즈 → 광고 수익 + 신뢰/구독자
         ↓
2단: 심화 유료 강의 (인프런 / Udemy / 클래스101) → 메인 수익
         ↓
3단: 멤버십 / 1:1 멘토링 / 부트캠프 → 고단가
```

| 플랫폼 | 가격대 | 수수료 | 예상 수강생 | 월 예상 |
|---|---|---|---|---|
| 유튜브 광고 | 무료 | — | — | 10~50만 |
| 인프런 | 3~8만원 | ~30% | 100~500명 | 30~200만 |
| **Udemy (영문)** | $20~50 | ~50% | 500~3,000명 | 50~500만 |
| 클래스101 | 10~20만원 | ~40% | 50~200명 | 50~300만 |
| 자체 판매 (Gumroad) | 자유 | ~10% | 적음 | 변동 |

> 영문 Udemy 시장이 국내보다 약 10배 큼. 자막은 AI로 처리 가능.

**상품 구성 제안**
```
기본 (39,000원)  — 영상 15편(약 4시간) + 전체 소스 + 단계별 git 태그 + 수학 치트시트
프로 (89,000원)  — 기본 + 웹 포팅편 5편 + MCP 연동편 + 디스코드 Q&A 평생 접근
멘토링 (300,000원 / 4회) — 코드 리뷰 + 커리어 상담
```

**장점:** GPL 제약 0, 초기 비용 ~0, 즉시 시작 가능, 실패해도 포트폴리오로 남음.

---

### 11-2. 2위 — 포트폴리오 → 이직 / 연봉 상승 (금액 최대)

연봉 500만원 상승 → 10년 누적 5,000만원. 사실상 가장 큰 "수익화".

| 증명 역량 | 코드 근거 | 어필 직군 |
|---|---|---|
| 3D 수학 | Vector3, 회전행렬, 내적/외적 | 게임 / 그래픽 / XR |
| 알고리즘 구현력 | Möller–Trumbore | 논문→코드 능력 |
| 성능 최적화 | 바운딩 스피어, RLE, 병렬화 | **수치로 증명 가능** |
| 저수준 시스템 | P/Invoke, user32 / libX11 | 시스템 프로그래밍 |
| 아키텍처 설계 | Strategy 패턴 3 Provider | 설계 역량 |
| 네트워크 | TCP, 바이너리 직렬화 | 서버 개발 |
| 동시성 | Parallel.For, ConcurrentQueue | 멀티스레드 이해 |

**타겟 직군:** 게임 클라이언트(넥슨/엔씨/크래프톤/스마일게이트), 그래픽스 엔지니어,
XR·메타버스, 시스템/백엔드.

**포트폴리오 완성 체크리스트**
```
[ ] README에 GIF 데모 추가            ← 최우선 (3초 안에 승부)
[ ] Before/After FPS 벤치마크 표      ← 면접관 선호도 최상
[ ] 테스트 코드 작성 (현재 0개)
[ ] GitHub Actions CI 추가
[ ] macOS 입력 Provider 구현 → 원본에 PR  ← 오픈소스 기여 이력
[ ] 웹 데모 배포 (GitHub Pages)       ← 면접관이 직접 조작
[ ] 기술 블로그 3편 (그림자 / 최적화 / P/Invoke)
[ ] 아키텍처 다이어그램 (Mermaid)
```

---

### 11-3. 3위 — 웹 SaaS / 서비스 (확장성 최대)

GPL-3는 AGPL이 아니므로, **서버에서만 실행하고 결과만 전송**하면 소스 공개 의무가
발생하지 않는다는 해석이 가능합니다 (브라우저로 JS를 보내면 "배포"에 해당 ⚠️).
가장 안전한 길은 역시 **자체 재작성 → MIT**.

**① ASCII 3D 로고/배너 생성기**
```
타겟: 개발자 (CLI 툴 제작자, GitHub README 꾸미기)
기능: .obj 업로드 → 회전하는 ASCII 애니메이션 → GIF/SVG 다운로드
가격: 무료 3개 / Pro $5·월 (무제한 + 고해상도 + 워터마크 제거)
근거: carbon.now.sh, shields.io 류의 개발자 꾸미기 수요 존재
```

**② 터미널 스크린세이버 / 개발자 장난감**
```
npm install -g ascii3d && ascii3d
무료 오픈소스 + 유료 테마팩 또는 GitHub Sponsors
cmatrix / cbonsai / pipes.sh 처럼 입소문 타면 star 수천 개
```

**③ 3D 수학 인터랙티브 학습 사이트**
```
타겟: 컴공 학생, 게임 개발 입문자
기능: 슬라이더로 벡터/조명 조절 → ASCII 결과 즉시 확인
수익: 구독 $9·월 또는 학교/학원 단체 라이선스 (B2B가 핵심)
```

**④ 코딩 테스트 / 교육 플랫폼 콘텐츠 (B2B, 가장 큰 단가)**
```
타겟: 부트캠프, 대학, 기업 교육팀
상품: "레이트레이싱 구현" 단계별 과제 + 자동 채점
     → ASCII 출력이라 정답 비교가 쉬움(diff) ← 결정적 비즈니스 장점
가격: 기관당 연 500~2,000만원
```

---

### 11-4. 4위 — AI 에이전트 벤치마크 / 데이터셋 (가장 뾰족함)

```
문제: AI 에이전트의 "공간 추론" 능력을 어떻게 측정하는가?
     이미지 기반 벤치마크는 비싸고(비전 모델 + 토큰) 느리다.

해결: ASCII 3D 렌더링으로 측정
     - 토큰 비용 약 1/50
     - 모든 LLM이 텍스트로 바로 처리
     - 정답 검증이 결정적(deterministic)
```

| 수익 방법 | 설명 | 규모 |
|---|---|---|
| 논문 → 커리어 | AI 학회 워크샵 페이퍼 → 연구직 | 간접(큼) |
| 벤치마크 리더보드 운영 | HuggingFace 공개 → 인지도 → 컨설팅 | 중 |
| 기업 평가 서비스 | "우리 에이전트 공간추론 측정" | 건당 수백만 |
| 합성 데이터셋 판매 | ASCII 3D 씬 + 설명 쌍 | 중~대 |
| MCP 서버 오픈소스 | 인지도 → 스폰서 / 채용 제안 | 간접 |

**로드맵**
```
Phase 1  neo3d-mcp 서버 (render / move_camera / get_frame)
Phase 2  태스크 20개 설계 (물체 찾기, 그림자로 광원 추론, 가림 판정)
Phase 3  주요 LLM 비교 평가 → 블로그 공개
Phase 4  논문 또는 리더보드 사이트 ("ASCII-3D-Bench")
```

---

### 11-5. 나머지 아이디어

| # | 아이디어 | 수익 | 난이도 | 비고 |
|---|---|---|---|---|
| 5 | GitHub Sponsors / 후원 | 월 0~30만 | ⭐ | 인지도 확보 후 유효 |
| 6 | 부트캠프 강사 | 시간당 5~15만 | ⭐⭐ | 포트폴리오로 영업 |
| 7 | 기술 블로그 → 제휴/광고 | 월 5~30만 | ⭐⭐ | SEO 유리한 주제 |
| 8 | 기업 사내교육 / 워크숍 | 회당 100~500만 | ⭐⭐⭐ | B2B 고단가 |
| 9 | ASCII 아트 굿즈 | 소액 | ⭐⭐ | 티셔츠/포스터 |
| 10 | 그래픽스 컨설팅 | 건당 수백만 | ⭐⭐⭐⭐ | 실력 증명 후 |
| 11 | 유료 템플릿 / 스타터킷 | 월 10~50만 | ⭐⭐⭐ | ⚠️ GPL이면 재작성 필요 |
| 12 | Steam 게임 출시 | 변동 | ⭐⭐⭐⭐⭐ | ⚠️ GPL 소스 공개 + 엔진 기능 부족 |

---

### 11-6. 실행 로드맵

**1개월차 — 이해 + 기반 (비용 0원)**
```
Week 1  코드 2,447줄 완독
Week 2  직접 수정: 새 씬 제작, 알려진 버그 수정, 테스트 코드 작성
Week 3  포트폴리오화: README GIF, FPS 벤치마크 표, GitHub Actions CI
Week 4  기술 블로그 3편 (그림자 원리 / FPS 10배 최적화 / P/Invoke)
```

**2~3개월차 — 콘텐츠 제작 (비용 ~20만원, 마이크)**
```
[ ] 유튜브 1편(훅 영상) 업로드 → 반응 측정
[ ] 반응 양호 → 시즌1(5편) 제작 / 미미 → 제목·썸네일 교체 후 재시도
[ ] macOS 입력 Provider 구현 → 원본에 PR
[ ] 웹 포팅 시작 (TypeScript, 자체 재작성)
```

**4~6개월차 — 수익화 본격화**
```
[ ] 웹 데모 GitHub Pages 배포
[ ] 인프런 / Udemy 강의 등록 (영문 포함)
[ ] neo3d-mcp 서버 개발 → AI 에이전트 실험
[ ] 이력서 / 포트폴리오 업데이트 → 이직 탐색
```

### 최종 결론 Top 3
```
1위  포트폴리오 → 이직/연봉 상승   : 금액 최대, 확실, GPL 무관. 지금 시작.
2위  유튜브 + 온라인 강의          : 한국어 블루오션, 초기비용 0, 실패해도 자산.
3위  AI 에이전트 벤치마크(MCP)      : 미개척 영역, 트렌디, 논문/화제성.
```

**핵심 요약: "엔진 자체를 판매"하는 길은 GPL로 막혀 있지만,
"이걸로 배운 실력"과 "이걸로 만든 콘텐츠"는 100% 본인 자산이며 그쪽이 금액도 더 큽니다.**

---

## 12. 알려진 이슈 / 개선 기회

코드 전수조사 중 발견한 항목들입니다. 각각이 **기여(PR) 기회**이자 학습 소재입니다.

| # | 위치 | 내용 | 영향 |
|---|---|---|---|
| 1 | `Shape/Sphere.cs:14` | 생성자 파라미터 `localRotate`를 직접 참조 → `GameObject.LocalRotate` 변경이 반영되지 않음 | 런타임 회전 불가 |
| 2 | `Network/NetworkManager.cs:97` | `AcceptClientsAsync()`가 새 접속마다 `_client`/`_stream`을 덮어씀 | **실질 1:1 연결만 가능**, 3인 이상 멀티플레이 불가 |
| 3 | `Network/NetworkManager.cs:63` | `length`를 검증 없이 신뢰해 `new byte[length]` 할당 | 악의적 입력 시 OOM |
| 4 | `Network/NetworkManager.cs` | 인증/핸드셰이크/암호화 전무 | 인터넷 노출 금지 |
| 5 | `AbstractClass/Light.cs:25` | `PointBright(RenderData)` 1인자 오버로드 미사용 | dead code |
| 6 | `SampleGame/Scenes/PriviewNetworkScene.cs` | 클래스명 오타 (`Priview` → `Preview`) | 가독성 |
| 7 | `AbstractClass/Screen.cs:16` | `_aspectRatio` 필드가 선언만 되고 미사용 (파생 클래스가 자체 보유) | dead code |
| 8 | `Implementation/ConsoleScreen.cs`, `DisplaysManager.cs` | Async 버전으로 대체된 구버전이 남아 있음 | 정리 대상 |
| 9 | 전역 | **테스트 코드 0개** | 회귀 방지 불가 → 기여 기회 |
| 10 | 전역 | **CI 워크플로 없음** | 빌드 자동화 기여 기회 |
| 11 | `Inputs/` | **macOS 전용 Provider 없음** | macOS 조작감 저하 → 최적의 첫 PR |
| 12 | `ConsoleScreenAsync.cs:16` | 창 크기를 시작 시 1회만 측정 | 실행 중 리사이즈하면 깨짐 |

> 참고: 이번 분석은 .NET SDK가 설치되지 않은 환경에서 수행되어
> **실제 빌드/실행 검증은 하지 않았습니다.** 코드 정독에 기반한 분석입니다.
> 로컬에서 `dotnet run -c Release` 로 확인하시기를 권합니다.

---

## 13. 최종 요약

| 질문 | 답 |
|---|---|
| 뭐하는 건가? | 터미널에 ASCII 문자로 3D를 실시간 레이트레이싱하는 C# 엔진 |
| GPU 쓰나? | ❌ CPU 전용 (향후 Vulkan 계획 언급) |
| 외부 라이브러리? | ❌ 0개 (.NET 기본 + OS API 직접 호출) |
| 규모? | 2,447줄 — 하루면 완독 가능 |
| 상용 게임 제작 가능? | ❌ 텍스처/머티리얼/물리/사운드/충돌 전부 없음 — **학습용** |
| 플러그인 / 스킬 / MCP? | ❌ 전부 아님. 순수 C# 라이브러리 + 콘솔 앱 |
| API 토큰 필요? | ❌ 0개. 오프라인 동작 |
| AI 에이전트에 도움? | 프레임워크로는 ❌ / **테스트베드로는 ⭐⭐⭐⭐⭐** |
| React 포팅? | ✅ 적합. 오히려 접근성·공유성 우수 |
| PHP 포팅? | 렌더링 ❌ / 백엔드 역할 ✅ |
| 유튜브 강의? | ✅ 매우 적합. 한국어 블루오션 |
| 수익화? | 엔진 판매 ❌ (GPL) / 교육·포트폴리오·SaaS ✅ |
| 가장 멋있는 부분? | 그림자 계산 + OS별 입력 시스템 + Run-Length 출력 최적화 |
| 가장 큰 가치? | 3D 그래픽스 원리 + 실무 패턴 샘플 + 콘텐츠 소재 |

---

## 부록 — 참고 링크

- 이 포크: https://github.com/bmshin94/Neo3dEngine
- 원본: https://github.com/IvanSobolev/Neo3dEngine
- 원본 WIKI: https://github.com/IvanSobolev/Neo3dEngine/wiki
- 원본 Issues: https://github.com/IvanSobolev/Neo3dEngine/issues
- 릴리스 v0.1.1: https://github.com/IvanSobolev/Neo3dEngine/releases/tag/v0.1.1
- 소개 영상: https://youtu.be/vhYE882B9dE
- 기여 가이드: [.github/CONTRIBUTING.md](../.github/CONTRIBUTING.md)
- 변경 이력: [CHANGELOG.md](../CHANGELOG.md)
- 라이선스 (GPL-3.0): [LICENSE](../LICENSE)
- GNU GPL-3.0 원문: https://www.gnu.org/licenses/gpl-3.0.en.html

---

*문서 작성: 2026-10-01 · 분석 기준 커밋 `91d3edc` · 총 39개 소스 파일 / 2,447줄 정독 기반*
