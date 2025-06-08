### attiny85와 USB 통신을 이용하여 mouse jiggler를 만드는 방법

- 저전력 소형 마이크로컨트롤러인 ATtiny85를 사용
- V-USB 라이브러리 또는 DigiUSB 기반으로 USB HID 마우스 기능 구현 가능
- USB 신호 라인(D+, D−)을 저항과 지그재그 회로를 통해 직접 연결
- 마우스 포인터를 주기적으로 작게 움직여 컴퓨터 절전 모드 방지
- DigiMouse 사용이 불가능한 경우, LED로 포인터 이동을 시각화하는 방식도 대안으로 활용

### 프로그램의 작동 원리와 USB 통신 방식

- 주기적으로 랜덤 시간(1~59초)에 마우스 움직임 시뮬레이션 실행
- 이동 거리(1~3픽셀), 방향(상/하/좌/우) 모두 랜덤
- 움직인 후 다시 원래 위치로 복귀해 **커서가 최종적으로 변하지 않음**
- USB HID(Human Interface Device) 프로토콜을 따라, 컴퓨터는 ATtiny85를 마우스로 인식
- USB 통신 시 low-speed 방식 (1.5Mbps), USB descriptor를 통해 장치 정보 전달

### attiny85와 USB 통신 이해

- ATtiny85는 자체적으로 USB 포트를 갖고 있지 않음
- 소프트웨어 USB 구현(V-USB)을 통해 일반 GPIO 핀으로 USB 신호 전송 가능
- PB3 (D−), PB4 (D+) 등의 핀을 통해 USB 통신 처리
- 정확한 클럭 주파수(12MHz 이상) 설정 필요 – 타이밍 맞추기 위함
- 제한된 리소스 환경에서도 HID 장치로 인식 가능하게 설계

### attiny85 마이크로컨트롤러와 USB 인터페이스의 기본 원리

- ATtiny85는 AVR 기반 8비트 마이크로컨트롤러
- 최대 8KB 플래시, 512B SRAM, I/O 핀 5개 제공
- USB는 기본적으로 차분 신호(D+/D−)로 데이터 전송
- HID 프로토콜을 사용하면 드라이버 없이도 작동 가능
- USB 인터페이스는 디스크립터 설정, 엔드포인트 통신, 인터럽트 처리가 기본 구조

### attiny85와 USB 인터페이스에 대한 설명

- USB 인터페이스는 ATtiny85의 일반 디지털 핀을 활용해 소프트웨어 방식(소프트USB)으로 구성
- HID 클래스는 키보드, 마우스 등 입력 장치를 표현하는 데 사용됨
- Mouse jiggler는 이 HID 기능을 응용하여 작은 움직임을 반복 생성
- LED 기반 구현에서는 실제 포인터는 움직이지 않지만, 디버깅 및 시각화용으로 사용 가능
- 최소한의 회로와 펌웨어로 USB HID 장치를 구현할 수 있는 효율적 솔루션

### 프로그램 소스코드

```bash
// #include <DigiMouse.h> // DigiMouse 사용 불가 환경

// 픽셀 움직임을 시각화하는 LED 9개 (0~8 핀 사용 가정)
const int ledPins[9] = {1, 2, 3, 4, 5, 6, 7, 8, 9};

// 중앙값: idx 4가 시작 위치
const int startPos = 4;

void setup() 
{
  for (int i = 0; i < 9; i++) {
    pinMode(ledPins[i], OUTPUT);
    digitalWrite(ledPins[i], LOW);
  }

  // 무작위성 확보
  randomSeed(analogRead(0)); // millis()보다 더 다양한 값 생성 가능
}

void loop() 
{
  // 마우스가 움직일 시점 - 1초 ~ 59초 사이 랜덤
  int waitTime = random(1000, 59000);
  delay(waitTime);

  // 이동 거리: 1 ~ 3 픽셀
  int distance = random(1, 4);

  // 방향: 0=오, 1=왼, 2=위, 3=아래
  int direction = random(0, 4);

  // 이동할 위치 계산 (중앙에서 이동)
  int movePos = getNewPosition(startPos, distance, direction);

  // LED 시각화: 중앙 -> 이동 -> 중앙
  blinkLED(ledPins[startPos], 100);   // 시작 LED
  blinkLED(ledPins[movePos], 300);    // 이동 LED
  blinkLED(ledPins[startPos], 100);   // 원위치 복귀 LED
}

// 위치 계산 함수 (3x3 그리드 내 제한)
int getNewPosition(int currentPos, int distance, int direction)
{
  int row = currentPos / 3;
  int col = currentPos % 3;

  switch (direction) {
    case 0: col = min(2, col + distance); break; // 오른쪽
    case 1: col = max(0, col - distance); break; // 왼쪽
    case 2: row = max(0, row - distance); break; // 위
    case 3: row = min(2, row + distance); break; // 아래
  }

  return row * 3 + col;
}

// LED 깜빡이기 함수
void blinkLED(int index, int duration) {
  digitalWrite(ledPins[index], HIGH);
  delay(duration);
  digitalWrite(ledPins[index], LOW);
}
```

### DIGITAL (PWM~)

```
0   1   2   3~  4   5~  6~  7   8   9~  10~ 11~ 12  13
```

- 디지털 입출력 핀(I/O) 으로 사용할 수 있음.
- `digitalWrite()`, `digitalRead()` 명령으로 제어할 수 있음
- HIGH/LOW, 즉 켜기/끄기, 1/0 값을 전송하거나 읽는 데 사용
- `3~`, `5~`, `6~`, `9~`, `10~`, `11~` 등 숫자 옆에 `~` 표시
    - 이 핀들이 PWM (Pulse Width Modulation) 을 지원한다는 뜻
    - 디지털 핀인데 아날로그처럼 동작할 수 있다는 뜻: `analogWrite()`
        
        → LED 밝기를 조절: `value` 값은 0~255 사이