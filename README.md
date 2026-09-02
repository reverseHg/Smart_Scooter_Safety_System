# Smart Scooter Safety System

AI 영상인식과 센서를 이용해 전동킥보드의 **음주 가능성, 헬멧 착용 여부, 다인 탑승 여부**를 확인하고,
조건을 만족하지 않으면 시동을 차단하는 안전 보조 시스템입니다.

## 주요 기능

- MQ-3 센서로 알코올 반응값 측정
- USB 카메라 + YOLO로 헬멧 착용 여부 인식
- 얼굴 검출로 MQ-3 측정 위치 접근 여부 확인
- FSR 4개로 탑승 및 다인 탑승 판단
- Arduino Uno에서 센서값 수집, 16x2 LCD/LED/Relay 제어
- Raspberry Pi 5에서 전체 상태 판단, YOLO, 음성 안내
- Raspberry Pi ↔ Arduino USB Serial 통신

> MQ-3 값은 시제품의 알코올 반응 감지용이며 법적 혈중알코올농도 측정값으로 사용하지 않습니다.

## 하드웨어 구성

```text
USB Camera
    |
Raspberry Pi 5
  - YOLO
  - 얼굴 검출
  - 음성 안내
  - 상태 머신
    |
    | USB Serial
    |
Arduino Uno
  - MQ-3 : A0
  - FSR1 : A1
  - FSR2 : A2
  - FSR3 : A3
  - FSR4 : A4
  - 16x2 LCD
  - Relay
  - Green / Red LED
```

## Arduino 핀

| 장치 | 핀 |
|---|---|
| MQ-3 AO | A0 |
| FSR1 | A1 |
| FSR2 | A2 |
| FSR3 | A3 |
| FSR4 | A4 |
| LCD RS | D2 |
| LCD E | D3 |
| LCD D4 | D4 |
| LCD D5 | D5 |
| LCD D6 | D6 |
| Relay S | D8 |
| LCD D7 | D9 |
| Green LED | D10 |
| Red LED | D11 |

## 16x2 LCD 연결

| LCD 핀 | 연결 |
|---|---|
| 1 VSS | GND |
| 2 VDD | 5V |
| 3 VO | 10k 가변저항 가운데 |
| 4 RS | D2 |
| 5 RW | GND |
| 6 E | D3 |
| 7~10 D0~D3 | 미사용 |
| 11 D4 | D4 |
| 12 D5 | D5 |
| 13 D6 | D6 |
| 14 D7 | D9 |
| 15 A | 5V |
| 16 K | GND |

## FSR 회로

각 FSR은 독립적인 분압회로로 연결합니다.

```text
5V
 |
FSR
 |
 +------ Arduino Analog Pin
 |
고정저항
 |
GND
```

고정저항 값은 실제 발판에서 0명/1명/2명 측정 결과가 1023에 포화되지 않도록 실험해 결정합니다.

## 폴더 구조

```text
Smart-Scooter-Safety-System/
├─ README.md
├─ requirements.txt
├─ .gitignore
├─ arduino/
│  ├─ smart_scooter_controller/
│  │  └─ smart_scooter_controller.ino
│  └─ sensor_test/
│     └─ sensor_test.ino
├─ raspberry_pi/
│  ├─ main.py
│  └─ config.py
└─ model/
   └─ README.md
```

## Raspberry Pi 설치

```bash
pip install -r requirements.txt
sudo apt update
sudo apt install espeak-ng alsa-utils
```

NS8002 스피커 모듈로 음성을 출력하려면 Raspberry Pi 5의 USB 오디오 장치/USB 사운드카드 등에서
아날로그 오디오 신호를 NS8002 입력으로 전달합니다.

## YOLO 모델

학습한 가중치를 아래 위치에 둡니다.

```text
model/best.pt
```

## 실행

1. Arduino에 `smart_scooter_controller.ino` 업로드
2. Arduino와 Raspberry Pi를 USB로 연결
3. USB 카메라 연결
4. `raspberry_pi/config.py`의 임계값 수정
5. 실행

```bash
python3 raspberry_pi/main.py
```

## 동작 순서

```text
음주 측정 안내
    ↓
카메라 얼굴 위치 확인
    ↓
MQ-3 통과
    ↓
YOLO 헬멧 확인
    ↓
FSR 1인 탑승 확인
    ↓
START_ON
    ↓
Relay ON / Green LED / LCD START ENABLE
```

하나라도 조건을 충족하지 못하면 `START_OFF`로 시동을 차단합니다.

## 실제 실험 후 반드시 수정

`raspberry_pi/config.py`

```python
ALCOHOL_THRESHOLD = 400
RIDER_PRESENT_THRESHOLD = 200
MULTI_RIDER_THRESHOLD = 1200
MQ3_ZONE = (170, 180, 470, 470)
```

위 값들은 예시값입니다. 특히 FSR은 제작한 발판에서 **0명 / 1명 / 2명 데이터를 반복 측정한 뒤** 최종 기준값을 결정하세요.

## GitHub 업로드 전 체크

- [ ] `best.pt` 업로드 방식 결정 (Git LFS 또는 별도 다운로드 링크)
- [ ] 실제 FSR/MQ-3 임계값 입력
- [ ] 카메라의 MQ-3 영역 좌표 설정
- [ ] README에 팀명/작품명 추가
- [ ] GitHub 저장소를 심사위원이 접근할 수 있게 설정
- [ ] 개발완료보고서에 GitHub 링크 삽입
