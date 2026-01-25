# Project-시계 & 온습도 디스플레이 : Linux Device Driver

## 1. 프로젝트 개요
* 본 프로젝트는 Raspberry Pi 4 환경에서 리눅스 커널 디바이스 드라이버를 직접 설계하고 구현하여 하드웨어를 제어하는 임베디드 시스템
* RTC(Real Time Clock) 모듈과 온습도 센서(DHT11), OLED 디스플레이, 로터리 엔코더를 연동하여 현재 시간 및 실내 환경 정보를 표시하고, 하드웨어 인터럽트를 통해 시간을 보정하는 기능

## 2. 프로젝트 목표 
* 단순한 라이브러리 활용을 넘어 커널 레벨에서 GPIO, I2C, SPI, Interrupt를 직접 제어함으로써 로우 레벨 시스템의 동작 원리를 이해

## 3. 개발 기간
* 25.12.24 - 25.12.29

## 4. 하드웨어 구성

<div align="center">
<img width="801" height="607" alt="image" src="https://github.com/user-attachments/assets/3d292a40-5e7a-4668-bdb9-29e65c159264" />
</div>

| Type | Model | Spec | Interface |
|:---:|:---:|:---:|:---:|
| **MCU** | Raspberry Pi 4B | 4GB RAM | - |
| **Display** | DOL128640 | 0.96" OLED (128x64) | I2C |
| **RTC** | DS1302 | Real Time Clock Module | 3-Wire (GPIO) |
| **Sensor** | DHT11 | Temp & Humidity Sensor | 1-Wire |
| **Input** | EC11 | Rotary Encoder & Switch | GPIO |

## 5. 주요 기능 및 동작 모드
* Mode 0 (기본 화면): 현재 날짜, 시간, 온도, 습도를 OLED에 실시간 표시 
* Mode 1 (시 조정): 로터리 회전으로 '시(Hour)' 변경 
* Mode 2 (분 조정): 로터리 회전으로 '분(Minute)' 변경 
* Mode 3 (초 조정): 로터리 회전으로 '초(Second)' 변경

<div align="center">
<img width="200" height="410" alt="image" src="https://github.com/user-attachments/assets/7346cf57-af4b-4e5c-a6bc-47c409120e99" />
</div>

## 6. 시스템 아키텍처

### 1) Software Control Flow (FSM)

<div align="center">
<img width="822" height="460" alt="image" src="https://github.com/user-attachments/assets/88ebbf94-f48e-4b68-b2aa-10d6fd158e1a" />
</div>

* **Interrupt Handling (S0 → S2)**
    * **GPIO_IRQ**: 로터리 엔코더 회전 또는 스위치 입력 시 하드웨어 인터럽트 발생 (`Request_irq`, `IRQF_TRIGGER`)
    * **Handlers**: `rot_a_handler`와 `sw_handler`가 호출되어 신호의 에지(Edge)를 감지하고 소프트웨어 디바운싱을 수행

* **Kernel State Update (S3)**
    * 인터럽트 결과에 따라 커널 내부의 시간 값이나 설정 모드(`mode 0~3`) 상태를 갱신

* **User Space Interaction (S4 → S7)**
    * **Read Operation (`my_read`)**: 유저 애플리케이션의 요청이 오면 `copy_to_user`를 통해 커널의 포맷팅된 문자열 데이터(시간, 온습도)를 유저 공간으로 전달
    * **Write Operation (`my_write`)**: 유저가 시간을 설정하면 `my_write`가 호출되어 RTC 모듈(DS1302)에 물리적인 시간 정보를 기록

### 2) 통신 프로토콜
| Protocol | Target Device | Description |
|:---:|:---:|---|
| **I2C** | OLED Display | Start/Stop 비트 및 ACK 체크를 통한 패킷 전송, 화면 데이터 업데이트 |
| **1-Wire** | DHT11 Sensor | 단일 버스 통신, Start Signal 후 40bit 데이터(습도/온도/체크섬) 수신 및 검증 |
| **3-Wire** | DS1302 RTC | SPI와 유사한 방식(CE, I/O, SCLK)으로 Command Byte 전송 후 데이터 Read/Write |


## 7. 프로젝트 결과
<div align="center">
<img width="695" height="385" alt="image" src="https://github.com/user-attachments/assets/573d1f56-c468-4772-b8ab-3df927ee90a6" />
</div>

