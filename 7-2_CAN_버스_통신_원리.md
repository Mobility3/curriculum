### CAN 버스 통신 프로토콜의 기본 작동 원리와 구조

- 분산형 아키텍처: 중앙 제어기 없이 각 ECU가 독립적으로 통신 (Multi-Master).
- 메시지 기반 통신: 주소가 아닌 메시지 ID 기준. 브로드캐스트 방식.
- 비동기 직렬 통신: 두 개의 와이어(CAN_H, CAN_L)를 통한 차동 전송 → 노이즈 내성 우수.
- 전송 단위: 프레임
    - 주요 필드: SOF, ID, DLC, Data, CRC, ACK, EOF

### CAN 프로토콜이 자동차 네트워크에서 주로 사용되는 이유

- 실시간성 보장: 우선순위 기반 중재(Arbitration).
- 유연한 네트워크 확장성: 노드 추가/제거 시 재구성 불필요.
- 통신 안정성: 하드웨어 수준의 오류 검출 및 자동 복구.
- 높은 호환성: 대부분의 자동차 제조사 및 산업기기에서 채택.

### CAN 버스의 데이터 전송 방식과 오류 검출 기능

- 프레임 기반 전송
    - ID: 메시지 식별자 및 우선순위
    - Data: 최대 8바이트 (Classic), 최대 64바이트 (CAN FD)
- 에러 검출 메커니즘 (총 5가지)
    1. Bit Error
    2. Stuff Error
    3. CRC Error
    4. Form Error
    5. ACK Error
- 에러 발생 시 자동 재전송 및 Error Frame 전송

### CAN 버스에서 데이터가 충돌 없이 전송되는 방법

- 중재(Arbitration) 방식
    - 모든 노드가 동시에 전송 가능 (Multi-Master)
    - 비트 단위 비교: ‘0’이 ‘1’을 덮어씀 → ID가 작은 메시지가 승자
    - 충돌 감지 시 패자는 전송 중단 → 물리적 충돌 없음
- | ECU_A(ID: 0x120) | ECU_B(ID: 0x100) | → ECU_B가 전송 지속 (ID 우선순위 높음)

### 임베디드 리눅스 시스템에서 CAN 버스 통신 구현 방법

- 필수 요소
    - CAN 하드웨어 (USB2CAN, SPI-CAN, 내장 CAN 등)
    - 커널 모듈: can, can_raw, can_dev, slcan, vcan
    - 유틸리티: can-utils (cansend, candump 등)
- CAN 인터페이스 구성 명령 예시
    - 가상 인터페이스:
        
        ```bash
        sudo modprobe vcan
        sudo ip link add dev vcan0 type vcan
        sudo ip link set up vcan0
        ```
        
    - 실제 인터페이스 (예: USB2CAN):
        
        ```bash
        sudo ip link set can0 type can bitrate 500000
        sudo ip link set up can0
        ```
        

### 리눅스 기반 장치에서 CAN 버스 구현 절차

1. 하드웨어 연결
    - USB2CAN 모듈 등으로 PC와 차량 CAN 버스 연결
2. 드라이버 로딩
    - modprobe can, modprobe slcan, modprobe can_raw 등
3. 인터페이스 설정
    - ip link set can0 type can bitrate 500000
    - ip link set up can0
4. 통신 테스트
    - 송신: cansend can0 123#11223344
    - 수신: candump can0
5. 통신 분석/로깅 도구
    - canplayer, cangen 등 활용

### 임베디드 리눅스에서 USB2CAN을 활용한 CAN 버스 통신 실습

### 1. 실습 환경 준비

- 하드웨어
    - USB2CAN 모듈 (예: PEAK, Kvaser, Seeed Studio 등)
    - CAN 통신 테스트용 장비 (또는 동일 USB2CAN 2개 연결)
    - 리눅스 PC 또는 임베디드 리눅스 보드 (라즈베리파이, Jetson, etc.)
- 소프트웨어 설치

```bash
sudo apt update
sudo apt install can-utils
```

### 2. CAN 인터페이스 설정

- USB2CAN 모듈 인식 확인 → can0 또는 slcan0 등의 인터페이스가 나타나야 함

```bash
dmesg | grep can
ifconfig -a      # 또는: ip link
```

- can0 설정 (bitrate 설정 및 활성화)

```bash
# 일반적으로 500kbps 사용
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```

- slcan 기반인 경우

```bash
sudo modprobe can
sudo modprobe can_raw
sudo modprobe slcan

# 연결: ttyUSB0 기준
sudo slcand -o -c -s6 /dev/ttyUSB0 can0
sudo ip link set up can0
```

### 3. 기본 통신 테스트

- 두 대의 보드나 USB2CAN 2개 연결 시, 한 쪽에서 송신 → 한 쪽에서 수신 확인 가능
- 송신 예시

```bash
cansend can0 123#1122334455667788
```

- 수신 예시

```bash
candump can0
```

### 4. 성능 및 안정성 테스트

- 랜덤 트래픽 생성 (g 10 : 10ms 간격, n 1000 : 1000개 메시지 전송)

```bash
cangen can0 -g 10 -n 1000
```

- 로깅 및 분석

```bash
candump -l can0
# 로그파일은 /tmp/candump-*.log 생성됨
```

- 재생

```bash
canplayer -I /tmp/candump-*.log
```

### 5. 송수신 자동 테스트 스크립트

- 송수신 자동화 스크립트 (test_can.sh)

```bash
#!/bin/bash

# 인터페이스 설정
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0

# 수신 백그라운드
candump can0 > canlog.txt &
DUMP_PID=$!

# 송신 테스트
echo "Sending test CAN frames..."
cansend can0 100#DEADBEEF
cansend can0 101#CAFEBABE
cansend can0 102#11223344

sleep 1
kill $DUMP_PID

echo "Logged CAN data:"
cat canlog.txt

```

- 저장 후 실행:

```bash
chmod +x test_can.sh
./test_can.sh
```

### 6. C 프로그램으로 수신

- can_recv.c (CAN 수신 코드)

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <net/if.h>
#include <sys/socket.h>
#include <linux/can.h>
#include <linux/can/raw.h>
#include <sys/ioctl.h>

int main() {
    int sock;
    struct sockaddr_can addr;
    struct ifreq ifr;
    struct can_frame frame;

    sock = socket(PF_CAN, SOCK_RAW, CAN_RAW);
    strcpy(ifr.ifr_name, "can0");
    ioctl(sock, SIOCGIFINDEX, &ifr);

    addr.can_family = AF_CAN;
    addr.can_ifindex = ifr.ifr_ifindex;
    bind(sock, (struct sockaddr *)&addr, sizeof(addr));

    printf("Listening on CAN bus (can0)...\n");
    while (1) {
        int nbytes = read(sock, &frame, sizeof(struct can_frame));
        if (nbytes > 0) {
            printf("ID=0x%X DLC=%d Data=", frame.can_id, frame.can_dlc);
            for (int i = 0; i < frame.can_dlc; i++)
                printf("%02X ", frame.data[i]);
            printf("\n");
        }
    }

    close(sock);
    return 0;
}
```

- 빌드 및 실행

```bash
gcc -o can_recv can_recv.c
sudo ./can_recv
```

### 7. 성능 및 오류 분석

| 테스트 항목 | 도구 | 설명 |
| --- | --- | --- |
| 전송 속도 | cangen, candump | -g 값 변경하며 최대 처리 속도 측정 |
| 에러율 분석 | candump + 로그 확인 | 데이터 누락/에러 수 수동 체크 |
| 안정성 | 반복 송수신 테스트 | 자동화 스크립트로 반복 확인 |
| 오류 상황 | ID 중복, 무응답 시뮬레이션 | 응답 없음, 충돌 시 에러 프레임 확인 |