# Braille Viewport — 전자석 래치 점자 디스플레이 / Electromagnetic-Latch Braille Display

> **KO / EN** — 이 문서는 한국어와 영어를 병기합니다. 각 문단·표·목록마다 한국어 다음에 영어가 이어집니다.
> This README is bilingual. Every paragraph, table, and list gives the Korean text first, followed by its English translation.

**팀 카인드랩(Kind Lab)** 4인 팀 프로젝트 **브라힐 뷰포트(Braille Viewport)**의 점자 출력 모듈 펌웨어와 라즈베리파이 인식 엔진(2026-08~09).
시각장애인을 위한 저가형 전자 점자 디스플레이를 목표로, 상용 피에조 셀 대신
ESP32-S3 + DRV8833 + 자작 코일(못 코어 + 네오디뮴 자석)로 점자 핀을 구동한다.

Firmware for the braille output module and the Raspberry Pi recognition engine of **Braille Viewport**, a four-person team project by **Team Kind Lab** (Aug–Sep 2026).
The goal is a low-cost electronic braille display for blind and visually impaired users. Instead of commercial piezo cells, the braille pins are driven by an ESP32-S3, DRV8833 drivers, and hand-wound coils (nail core + neodymium magnet).

핵심 아이디어는 **쌍안정 자기 래치**: 짧은 펄스(기본 20ms)로 핀을 올리면
전원 0 상태로 매달려 유지되고, 반대 극성 펄스로 내린다. 유지 전력이 없어
배터리 구동에 유리하고, 셀당 부품비가 코일+하프브리지 반쪽 수준으로 내려간다.

The core idea is a **bistable magnetic latch**: a short pulse (20 ms by default) raises a pin, which then stays up with zero power, and a pulse of the opposite polarity lowers it. With no holding current the design suits battery operation, and the per-cell part cost drops to one coil plus half of a dual H-bridge driver.

**팀 구성**: 팀 카인드랩 4인이 함께 만들었다. 팀장 윤창훈이 전체 아키텍처·ESP32 펌웨어·
라즈베리파이 인식 엔진(이 저장소)을 맡았고, 나머지 팀원이 하드웨어 조립·문서·AI 파트를
맡았다. 시연 영상: https://youtu.be/Ccwc9DTXhrc

**Team**: Built by the four members of Team Kind Lab. Team lead Changhoon Yoon owned the overall architecture, the ESP32 firmware, and the Raspberry Pi recognition engine (this repository); the other members handled hardware assembly, documentation, and the AI part. Demo video: https://youtu.be/Ccwc9DTXhrc

현재 단계는 3×3 프로토타입 모듈 벤치 검증이며, 상위 제어기(Raspberry Pi)가
시리얼 패턴 프로토콜(`B:XXXXXXXXX`)로 이 모듈을 구동하는 구조다.
아래는 처음 세팅할 때 필요한 전 과정 (보드 인식 → PlatformIO 플래싱).

The project is currently at the bench-validation stage of a 3×3 prototype module. A host controller (Raspberry Pi) drives this module over a serial pattern protocol (`B:XXXXXXXXX`).
Below is the complete first-time setup procedure (board detection → PlatformIO flashing).

## 🙌 코딩 몰라도 됩니다 — 바이브 코딩으로 시작하기 / No coding required — get started with vibe coding

이 저장소는 팀원이 코딩/ESP32를 몰라도 AI에게 시켜서 조작할 수 있게 만들어졌다.
직접 명령어를 칠 필요 없이 AI 코딩 도구에게 한국어로 부탁하면 되고,
**Gemini CLI / Claude Code / Codex 어느 것이든 호환**된다. 하나만 골라 설치하면 된다.

This repository is set up so that team members with no coding or ESP32 experience can operate it by delegating to an AI. Instead of typing commands yourself, ask an AI coding tool in plain language; **Gemini CLI, Claude Code, and Codex are all compatible**. Install just one of them.

### 1) AI 도구 하나 설치 / Install one AI tool

| 도구 / Tool | 설치 (PowerShell에 붙여넣기) / Install (paste into PowerShell) | 실행 / Run | 계정 / Account |
|---|---|---|---|
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | `npm install -g @google/gemini-cli` | `gemini` | 구글 계정 (**무료**)<br>Google account (**free**) |
| [Claude Code](https://claude.com/claude-code) | `irm https://claude.ai/install.ps1 \| iex` | `claude` | Claude 유료 구독<br>Paid Claude subscription |
| [Codex](https://github.com/openai/codex) | `npm install -g @openai/codex` | `codex` | ChatGPT 유료 구독<br>Paid ChatGPT subscription |

- Gemini CLI/Codex는 [Node.js LTS](https://nodejs.org)를 먼저 설치해야 한다 (계속 "다음"으로 설치)<br>Gemini CLI and Codex need [Node.js LTS](https://nodejs.org) installed first (just keep clicking "Next")
- 뭘 쓸지 모르겠으면 **무료인 Gemini CLI** 추천<br>Not sure which to pick? The **free Gemini CLI** is recommended

### 2) 실행 후 아래 문장을 그대로 붙여넣기 / Launch the tool and paste this sentence as-is

> https://github.com/changhoon-yoon/braille-drv8833 클론해서 README대로 개발 환경을 세팅해줘.
> 드라이버나 프로그램 설치가 필요하면 알려주면서 진행해줘.

English version of the prompt:

> Clone https://github.com/changhoon-yoon/braille-drv8833 and set up the development environment as described in the README.
> If any drivers or programs need to be installed, tell me and go ahead.

### 3) 보드 연결 후 — USB로 보드를 꽂고 이렇게 시키면 끝 / After connecting the board over USB, just ask this

> 보드에 펌웨어 업로드하고, 전 코일 왕복 스윕 테스트(a_r) 돌려서 결과 알려줘

> Upload the firmware to the board, run the all-coil round-trip sweep test (a_r), and report the result.

이후에도 전부 말로 하면 된다. 예시:
- "점자 패턴 `B:101000000` 보내줘"
- "펄스 세기를 5% 올려서 다시 테스트해줘"
- "업로드가 안 되는데 원인 찾아줘"

From here on, everything can be done in plain language. Examples:
- "Send braille pattern `B:101000000`"
- "Raise the pulse power by 5% and test again"
- "The upload fails, find the cause"

아래부터는 수동으로 세팅하고 싶은 사람(또는 AI가 참고할)을 위한 상세 가이드다.

The rest of this document is a detailed guide for anyone who wants to set things up manually (and for the AI to refer to).

## 하드웨어 / Hardware

| 부품 / Part | 내용 / Details |
|---|---|
| MCU | ESP32-S3 DevKitC (N16R8, 16MB Flash + 8MB PSRAM) |
| 드라이버 / Driver | DRV8833 듀얼 H브리지 모듈 ×5 (모듈당 코일 2개)<br>DRV8833 dual H-bridge modules ×5 (2 coils per module) |
| 코일 / Coil | 자작 — 못 코어에 에나멜선, 위에 네오디뮴 자석 핀<br>Hand-wound: enameled wire on a nail core, neodymium-magnet pin on top |
| 전원 / Power | 모터전원(VM) 5~6V — 벤치 서플라이 권장 (전류 제한 기능이 고장을 잡아줌)<br>Motor supply (VM) 5–6 V; a bench supply is recommended (its current limit catches faults) |
| USB-시리얼 / USB-serial | 보드 온보드 CH343 (COM 라벨 포트)<br>On-board CH343 (the port labeled COM) |

### 배선 (핀맵) / Wiring (pin map)

DRV8833 모듈 기준. 각 모듈 공통: VM=5~6V, GND=ESP32와 공통, **EN/SLP=3V3 (빼먹으면 전부 침묵)**.
VM 가까이에 470uF 전해캡 (긴 다리 = +).

Per DRV8833 module. Common to every module: VM = 5–6 V, GND shared with the ESP32, **EN/SLP = 3V3 (forget this and nothing moves)**.
Place a 470 µF electrolytic capacitor close to VM (long leg = +).

| 모듈 / Module | IN1(AIN1) | IN2(AIN2) | A출력 / Out A | IN3(BIN1) | IN4(BIN2) | B출력 / Out B |
|---|---|---|---|---|---|---|
| #1 | GPIO4 | GPIO5 | coil0 | GPIO6 | GPIO7 | coil1 |
| #2 | GPIO8 | GPIO9 | coil2 | GPIO10 | GPIO11 | coil3 |
| #3 | GPIO12 | GPIO13 | coil4 | GPIO14 | GPIO15 | coil5 |
| #4 | GPIO16 | GPIO17 | coil6 | GPIO18 | GPIO21 | coil7 |
| #5 | GPIO38 | GPIO39 | coil8 | — | — | 미사용 / unused |

> ⚠️ 코일 0·4·5·6은 감은 방향이 반대라 펌웨어 `COILS[]` 표에서 두 핀을 맞바꿔 극성을 보정해 둠 (배선은 위 표 그대로). 새 코일을 달았는데 `p`(밀기)에서 핀이 내려가면 그 코일의 두 GPIO를 표에서 맞바꾸면 된다.
>
> ⚠️ Coils 0, 4, 5 and 6 are wound in the opposite direction, so the firmware `COILS[]` table swaps their two pins to correct the polarity (the wiring stays exactly as in the table above). If you install a new coil and the pin goes *down* on `p` (push), swap that coil's two GPIOs in the table.

코일 배치는 행 우선: coil0=왼위 … coil8=오른아래. 코일은 OUT1/2(A) 또는 OUT3/4(B)에 연결.
극성이 반대면 밀기/당기기가 뒤바뀌므로 코일 두 선만 맞바꾸면 된다.

Coils are laid out row-major: coil0 = top-left … coil8 = bottom-right. Each coil connects to OUT1/2 (A) or OUT3/4 (B).
If the polarity is reversed, push and pull are swapped; just swap the coil's two wires.

> ESP32-S3 핀 주의: GPIO26~37(플래시/PSRAM), 0/19/20/43~46(부트/USB), 48(온보드 LED)은 사용 금지.
>
> ESP32-S3 pin caution: do not use GPIO26–37 (flash/PSRAM), 0/19/20/43–46 (boot/USB), or 48 (on-board LED).

## 1. ESP32 인식시키기 (Windows) / Getting the ESP32 recognized (Windows)

1. USB 케이블로 보드의 **COM(UART) 라벨 포트**를 PC에 연결 (데이터 지원 케이블일 것 — 충전전용 케이블이 의외로 흔한 함정)<br>Connect the board's **port labeled COM (UART)** to the PC with a USB cable (it must be a data cable; charge-only cables are a surprisingly common trap)
2. 장치 관리자 → 포트(COM & LPT)에 **`USB-Enhanced-SERIAL CH343 (COMx)`** 가 보이면 인식 성공<br>In Device Manager → Ports (COM & LPT), if **`USB-Enhanced-SERIAL CH343 (COMx)`** appears, the board is recognized
3. 안 보이면 CH343 드라이버 설치: WCH 공식 [CH343SER 드라이버](https://www.wch.cn/downloads/CH343SER_EXE.html) 설치 후 재연결<br>If it does not appear, install WCH's official [CH343SER driver](https://www.wch.cn/downloads/CH343SER_EXE.html) and reconnect
4. PowerShell로도 확인 가능 / You can also check from PowerShell:
   ```powershell
   [System.IO.Ports.SerialPort]::GetPortNames()
   ```

**COM 번호는 USB 꽂는 자리마다 바뀔 수 있다** (이 프로젝트에서도 COM5→COM8 이력 있음).
그래서 `platformio.ini`에 포트를 고정하지 않고 자동 탐지에 맡긴다.

**The COM number can change depending on which USB port you use** (this project has already gone from COM5 to COM8).
That is why `platformio.ini` does not pin a port and leaves it to auto-detection.

## 2. PlatformIO 설치 / Installing PlatformIO

VSCode 없이 CLI만으로 충분하다 / The CLI alone is enough, no VS Code needed:

```powershell
pip install platformio
```

설치 확인 / Verify the install:

```powershell
pio --version
```

(`pio`가 PATH에 없으면 `C:\Users\<이름>\AppData\Local\Programs\Python\Python3xx\Scripts\pio` 전체 경로로 실행)

(If `pio` is not on PATH, run it by its full path: `C:\Users\<name>\AppData\Local\Programs\Python\Python3xx\Scripts\pio`)

## 3. 빌드 & 플래싱 / Build & flash

```powershell
git clone https://github.com/changhoon-yoon/braille-drv8833.git
cd braille-drv8833

pio run              # 빌드만 (첫 빌드는 툴체인 다운로드로 수 분 소요) / build only (first build takes minutes to download the toolchain)
pio run -t upload    # 빌드 + 플래싱 (포트 자동 탐지) / build + flash (port auto-detected)
```

특정 포트를 강제하려면 / To force a specific port:

```powershell
pio run -t upload --upload-port COM8
```

### 플래싱이 안 될 때 / When flashing fails

| 오류 / Error | 원인 / Cause | 해결 / Fix |
|---|---|---|
| `Could not open COMx ... PermissionError(13)` | 다른 프로그램이 포트 점유 (Docklight, 시리얼 모니터 등)<br>Another program holds the port (Docklight, serial monitor, etc.) | 해당 프로그램에서 연결 해제 (Docklight는 F6) 후 재시도<br>Disconnect in that program (F6 in Docklight) and retry |
| `Could not open COMx ... FileNotFoundError(2)` | 그 COM 번호에 보드가 없음 (USB 자리 바뀜/빠짐)<br>No board on that COM number (USB port changed / unplugged) | `GetPortNames()`로 현재 번호 확인, 케이블 재연결<br>Check the current number with `GetPortNames()` and reconnect the cable |
| 아예 포트가 안 잡힘<br>No port shows up at all | 드라이버 미설치 or 충전전용 케이블<br>Driver not installed, or a charge-only cable | CH343 드라이버 설치, 케이블 교체<br>Install the CH343 driver, swap the cable |

## 4. 시리얼 모니터 / Serial monitor

```powershell
pio device monitor    # 115200bps, Ctrl+C로 종료 / 115200 bps, Ctrl+C to quit
```

또는 Docklight/기타 터미널로 115200bps 연결. **플래싱 전엔 반드시 연결을 끊을 것.**

Or connect with Docklight or any other terminal at 115200 bps. **Always disconnect before flashing.**

## 5. 시리얼 명령 / Serial commands

### 패턴 프로토콜 (상위 제어기 → 보드) / Pattern protocol (host controller → board)

```
B:XXXXXXXXX\n     9자리 0/1, 행 우선 (왼위→오른아래)         / nine 0/1 digits, row-major (top-left → bottom-right)
B:101000000\n     coil0, coil2 올림 — 0 자리는 무조건 당김 펄스로 내림 / raise coil0 and coil2; every '0' gets an unconditional pull pulse
B:000000000\n     전부 내림                                   / lower all
```

'0' 자리는 상태 추적과 무관하게 매번 반대 극성 펄스를 발사하므로,
크로스토크로 어긋난 핀도 패턴 적용 때마다 물리적으로 재정렬된다.

Because every '0' position fires a reverse-polarity pulse regardless of the tracked state, pins knocked out of place by crosstalk are physically re-aligned each time a pattern is applied.

발사는 항상 순차(동시 발사 금지, `,`/`.`로 간격 조절)이며 순서는 **0,1,2,3,5,6,7,8,4 — 중앙 4번이 마지막**.
중앙은 이웃 8개의 자기장 영향을 전부 받으므로, 이웃이 자리 잡은 뒤 마지막에 처리한다 (`FIRE_ORDER[]`).

Firing is always sequential (never simultaneous; adjust the gap with `,`/`.`), in the order **0,1,2,3,5,6,7,8,4 — center coil 4 last**.
The center is affected by the magnetic fields of all eight neighbors, so it is handled last, after the neighbors have settled (`FIRE_ORDER[]`).

### 벤치 튜닝 키 / Bench tuning keys

| 키 / Key | 동작 / Action |
|---|---|
| `0`~`8` | 대상 코일 선택<br>Select the target coil |
| `u` / `d` | 선택 코일 올리기 / 전부 내리기+모드 정지<br>Raise the selected coil / lower all and stop the current mode |
| `p` / `q` | 밀기 펄스 / 당기기 펄스<br>Push pulse / pull pulse |
| `a_h` / `a_l` | 전 코일 순차 올리기(올린 채 유지) / 전 코일 순차 내리기 — 3글자 이어서 입력<br>Sequentially raise all coils (and hold) / sequentially lower all coils; type the 3 characters in a row |
| `a_r` | 왕복 스윕 (`a_h` → 대기 → `a_l`, 무전원 래치 확인용)<br>Round-trip sweep (`a_h` → wait → `a_l`), to verify the zero-power latch |
| `t` / `r` | 자동 반복 펄스 / 연속 ON-OFF 교대 (60초 자동정지)<br>Auto-repeat pulse / continuous ON-OFF alternation (auto-stops after 60 s) |
| `o` | 100% 연속 ON (전류계 측정용, 발열 주의)<br>100% continuous ON (for ammeter measurement; watch for heat) |
| `+` `-` | 펄스폭 ±10ms (기본 20ms)<br>Pulse width ±10 ms (default 20 ms) |
| `(` `)` | 펄스 파워 ∓5% (기본 70%)<br>Pulse power ∓5% (default 70%) |
| `<` `>` | 반복 간격 ∓250ms (기본 3000ms)<br>Repeat interval ∓250 ms (default 3000 ms) |
| `,` `.` | 순차 발사 간격 ∓10ms (기본 20ms). 9개 동시 발사는 ~8A로 전원이 못 버텨 항상 순차<br>Sequential firing gap ∓10 ms (default 20 ms). Firing all 9 at once would draw ~8 A, more than the supply can handle, so firing is always sequential |
| `h` | 래치/홀드 모드 전환 (기본 래치 = 유지전류 0)<br>Toggle latch/hold mode (default latch = zero holding current) |
| `s` / `?` | 상태 / 도움말<br>Status / help |

## 6. 라즈베리파이 연동 (무한 캔버스 리더) / Raspberry Pi integration (infinite-canvas reader)

파이 4가 카메라로 ArUco 마커를 읽어 절대좌표를 구하고, 가상 페이지의 3×3 창을
`D:XXXXXXXXX` 패턴으로 이 보드에 보낸다. 코드는 `pi/` 폴더, 파이 쪽 경로는 `~/braille/`.

A Raspberry Pi 4 reads ArUco markers with its camera to obtain absolute coordinates, then sends the 3×3 window of a virtual page to this board as a `D:XXXXXXXXX` pattern (`D:` is a diff update that fires only the dots that changed; `B:` is a full resync). The code lives in the `pi/` folder; on the Pi it goes under `~/braille/`.

| 역할 / Role | 파일 / File |
|---|---|
| 카메라 → 좌표 → 패턴 전송 (메인 루프)<br>Camera → coordinates → pattern transmit (main loop) | `pi/reader.py` (`--camera --view --yellow ...`) |
| 보드와 시리얼 (부팅 대기 → `t`로 자동테스트 정지 → `D:`/`B:` 전송)<br>Serial link to the board (wait for boot → stop the auto-test with `t` → send `D:`/`B:`) | `pi/braille_link.py` |
| 마커 보드 생성 / 좌표 계산 / 가상 페이지 / 라이브 뷰어<br>Marker board generation / coordinate math / virtual page / live viewer | `marker_board.py` `locator.py` `virtual_page.py` `viewer.py` |
| 상시 실행 서비스<br>Always-on service | `braille-reader.service` (systemd, 죽으면 3초 뒤 재시작 / restarts 3 s after a crash) — 뷰어 / viewer: `http://<파이IP>:8501` |

파이 준비: `sudo apt install python3-opencv python3-serial esptool`, 보드는 파이 USB → `/dev/ttyACM0`.

Pi setup: `sudo apt install python3-opencv python3-serial esptool`; the board plugs into the Pi's USB port and shows up as `/dev/ttyACM0`.

**보드 펌웨어를 파이에서 굽기** — PC에서 `pio run` 후 한 줄:

**Flashing the board from the Pi** — after `pio run` on the PC, a single line:
```bash
bash pi/deploy_fw.sh                        # .local 안 풀리면: bash pi/deploy_fw.sh chyoon_pi4b8g@<파이IP>
                                            # if .local does not resolve: bash pi/deploy_fw.sh chyoon_pi4b8g@<Pi IP>
```
서비스 정지 → 산출물 4개 복사 → `esptool --no-stub`(데비안 패키지엔 S3 스텁이 없음) → 서비스 재시작까지 자동.
수동으로 파이에서: `bash ~/braille/flash_fw.sh`.

It automatically stops the service → copies the 4 build artifacts → runs `esptool --no-stub` (the Debian package has no S3 stub) → restarts the service.
To do it manually on the Pi: `bash ~/braille/flash_fw.sh`.

## 개발 워크플로우 / Development workflow

펌웨어 개발은 [Claude Code](https://claude.com/claude-code)를 활용한 바이브 코딩으로 진행했다
(요청 → 코드 수정 → 빌드 → 플래싱 → 커밋). 기능 단위로 커밋하며 커밋 메시지에 변경 이유를 남긴다.

Firmware development was done by vibe coding with [Claude Code](https://claude.com/claude-code) (request → code change → build → flash → commit). Commits are made per feature, and every commit message records why the change was made.

### 로드맵 / Roadmap

- [x] 1코일 벤치: 래치 물리 검증, 최소 펄스폭 탐색 (20ms)<br>Single-coil bench: physical latch validation, minimum pulse-width search (20 ms)
- [x] 펄스 래치 구동 전환 (유지전류 0) + 펄스 파워 % 조절<br>Switch to pulse-latch drive (zero holding current) + pulse power % control
- [x] 9코일 패턴 프로토콜 (`B:XXXXXXXXX`) + 전 코일 스윕 테스트<br>9-coil pattern protocol (`B:XXXXXXXXX`) + all-coil sweep test
- [x] 3×3 모듈 전 셀 조립 및 스윕 검증 (8/30 전 코일 스윕으로 코일 0/4/5/6 극성 보정)<br>Full 3×3 module assembly and sweep validation (Aug 30: all-coil sweep, polarity correction for coils 0/4/5/6)
- [ ] 셀 간 자기 간섭(크로스토크) 대책 — 상단 철 와셔 래치 / 철판 격벽<br>Inter-cell magnetic interference (crosstalk) countermeasures: iron-washer latch on top / steel-plate partitions
- [x] 상위 제어기(Pi) 연동 — 카메라 ArUco 좌표 인식 → 창 패턴 출력 (8/25 리더 v1, `pi/reader.py`)<br>Host controller (Pi) integration: camera ArUco localization → window pattern output (Aug 25: reader v1, `pi/reader.py`)
- [ ] 9핀 래칭 재현성 개선 (핀별 기계 편차, 유지력)<br>Improve 9-pin latching repeatability (per-pin mechanical variance, holding force)
- [ ] 카메라 문자 인식(OCR) → 점자 변환 연동<br>Camera text recognition (OCR) → braille conversion
- [ ] 당사자(시각장애 학생·특수교사) 촉각 판독성 평가<br>Tactile readability evaluation with end users (blind students, special-education teachers)
