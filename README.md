# 영남향우회 AI체스로봇

## Doosan Robotics Rokey Boot Camp 6기 협동2 A-2조 프로젝트 복원자료입니다.

# AI Chess Robot – ROS2 Source

이 디렉토리는 AI 체스 로봇 시스템의 ROS2 소스 코드가 포함된 `src/` 디렉토리이다.  
Vision, AI 판단, Cloud 연동, 로봇 제어를 하나의 시퀀스로 통합하는 것을 목표로 한다.

---

## 1. Overview

본 시스템은 체스판을 인식하여 현재 보드 상태를 판단하고,
AI(Stockfish)를 통해 최적의 수를 계산한 뒤,
사용자 검증을 거쳐 로봇이 실제 체스 말을 이동시키는 ROS2 기반 시스템이다.

Vision 결과, 시스템 상태, 사용자 입력은 Firebase Realtime Database를 통해
비동기적으로 공유된다.

---

## 2. Directory Structure

src/
└─ cobot2/
   ├─ .env                    # Firebase 및 환경 변수 설정
   ├─ cobot2/
   │  ├─ STT.py               # 음성 인식 (시작/명령 트리거)
   │  ├─ UI.html              # Web UI (체스판, 로그, 사용자 입력)
   │  ├─ chess_grid.json      # 체스 좌표 매핑 정보
   │  ├─ data.json            # 시스템 상태 / 테스트용 데이터
   │  ├─ miccheck.py          # 마이크 입력 테스트
   │  ├─ onrobot.py           # 그리퍼 및 로봇 I/O 제어
   │  ├─ robot_action.py      # 체스 말 이동 Action 서버
   │  ├─ stockfish.py         # 체스 AI (Stockfish 연동)
   │  ├─ vision_db.py         # Vision 결과 → Firebase DB 전송
   │  └─ venv_voice/          # 음성 인식용 Python 가상환경
   └─ package.xml             # ROS2 패키지 설정

## 3. Core Modules Description

### Vision & Data

- `vision_db.py`
  - 체스판 인식 결과를 Firebase Realtime Database의 `board_state` 경로로 전송
  - Vision 노드와 메인 제어부(Main Logic) 사이의 데이터 브리지 역할 수행

---

### AI

- `stockfish.py`
  - FEN 문자열을 입력으로 받아 최선의 수 계산
  - 계산된 결과를 메인 제어 흐름으로 전달하여 다음 단계 결정에 사용

---

### Robot Control

- `robot_action.py`
  - 체스 말 이동을 수행하는 ROS2 Action 서버
  - 체스 좌표 기반 pick & place 동작 수행

- `onrobot.py`
  - 로봇 그리퍼 및 디지털 I/O 제어
  - 체스 말 파지 및 해제를 담당

---

### Voice & UI

- `STT.py`
  - 음성 명령을 통한 시스템 시작 및 제어 트리거
  - 음성 입력을 이벤트 형태로 시스템에 전달

- `UI.html`
  - 체스판 상태 시각화
  - AI 추천 수 표시
  - 사용자 승인 / 거절 입력 인터페이스 제공

---

## 4. Communication Flow

- **Vision → Firebase DB**
  - 체스판 인식 결과 공유

- **Main Logic ↔ Stockfish**
  - FEN 기반 AI 수 계산 요청 및 결과 수신

- **User ↔ UI**
  - AI 추천 수에 대한 사용자 승인 / 거절 처리

- **Robot Action**
  - 승인된 수에 대해 실제 체스 말 이동 수행

Firebase Realtime Database는  
시스템 간 비동기 통신을 위한 **공유 메모리(Shared Memory)** 역할을 수행한다.

---
## 5. Development Environment

- **OS**: Ubuntu 22.04 LTS (Jammy Jellyfish)
- **Middleware**: ROS 2 Humble
- **Language**: Python 3.10.12
- **Key Libraries**:
  - **ROS2**
    - rclpy

  - **Robot Control**
    - DSR_ROBOT2
    - DR_init
    - onrobot (custom wrapper)

  - **Vision / AI**
    - ultralytics (YOLO)
    - opencv-python
    - numpy

  - **Chess AI**
    - stockfish

  - **Cloud / UI**
    - firebase-admin
    - json

  - **Voice Control**
    - openwakeword
    - sounddevice / pyaudio


  ## Hardware Setup
  - doosan m0609
  - logitech webcam c270
  
  ---

## 6. Dependencies / Requirements

본 패키지(`cobot2`)는 아래 의존성을 필요로 합니다.

### OS / ROS
- Ubuntu 22.04
- ROS 2 Humble
- colcon build environment

---

  ## 1 System Packages (APT)

  ```bash
  sudo apt update

  # Stockfish 엔진 (stockfish.py에서 /usr/games/stockfish 사용)
  sudo apt install -y stockfish

  # PyAudio / SoundDevice 계열(마이크 입력) 빌드 및 런타임 의존성
  sudo apt install -y portaudio19-dev python3-pyaudio

  # (선택) OpenCV를 pip 대신 apt로 쓰고 싶으면
  # sudo apt install -y python3-opencv


  python3 -m pip install --upgrade pip

  # STT.py
  python3 -m pip install openai sounddevice scipy numpy python-dotenv

  # vision_db.py (Vision + DB)
  python3 -m pip install opencv-python ultralytics pillow firebase-admin

  # onrobot.py (Modbus TCP gripper control)
  python3 -m pip install pymodbus

  # stockfish.py (python wrapper)
  python3 -m pip install stockfish

  # CPU 버전 예시 (환경에 맞게 설치)
  python3 -m pip install torch torchvision
  ```


## 6. Build

```bash
colcon build --packages-select cobot2 cobot2_interface doosan-robot2
source install/setup.bash
```
---

## 7. Run
```bash
# 로봇 연결
ros2 launch  dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=real host:=192.168.1.100 port:=12345 model:=m0609

# ros2 실행
- ros2 run cobot2 main
- ros2 run cobot2 robot_action
- ros2 run cobot2 stockfish
- ros2 run cobot2 run_voice.sh
- ros2 run cobot2 vision_db ( 또는 python3 vision_db.py)
```
