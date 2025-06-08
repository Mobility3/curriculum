### 아두이노에 다양한 센서를 연결하는 방법

![image.png](attachment:73c1780b-97f9-4861-9c1d-957b639b9b47:image.png)

1. 온도 센서 (LM35) ⇒ A0에 연결
    
    LM35는 주변 온도를 측정해서 전압 값으로 출력해주는 센서.
    
    예를 들어 온도가 25도라면 약 0.25V가 나옴.
    
- VCC: 5V - 센서에 전원 공급
- GND: GND - 접지 (GND). 회로의 마이너스 극
- OUT: A0 (아날로그 입력) - 아두이노가 온도값을 읽을 핀
    
    ⇒ 센서가 출력하는 전압은 연속적인 값이니까 아날로그 입력으로 받아야 함.
    

1. 조도 센서 (CdS 광센서 + 저항) ⇒ A1 저항이랑 분압 회로 구성 필요함
    
    조도 센서는 빛의 밝기를 측정.
    
    CdS 광센서는 빛이 강할수록 저항이 낮아지고, 어두우면 저항이 높아짐.
    
    그래서 그냥 연결하면 안 되고 저항과 같이 분압 회로를 만들어줘야함.
    
    분압회로란? 두 개의 저항(여기선 CdS + 10kΩ)을 직렬로 연결해서 중간 전압을 측정하는 방법
    
    ```
    5V ----[CdS]---- A1 ----[10kΩ]---- GND
    ```
    
    - VCC: 5V → CdS 센서 한쪽 끝
    - GND: GND
    - OUT: CdS 센서 다른 쪽 끝 → A1 (아날로그 입력) + 10kΩ 저항
    
    ⇒ A1 핀은 CdS와 10kΩ 사이 중간점에서 값을 읽음.
    
    ⇒ 밝으면 A1 값이 낮게 나오고, 어두우면 높게 나옴.
    
    ⇒ `analogRead(A1)` 사용해서 빛의 밝기를 측정할 수 있음.
    

1. 거리 센서 (초음파 센서 HC-SR04)
    
    소리를 쏘고, 벽에 튕겨오는 시간을 계산해서 거리(cm) 를 측정함.
    
    - VCC: 5V
    - GND: GND
    - Trig: D2 - 초음파를 쏘는 핀
    - Echo: D3 - 초음파가 되돌아오는 시간을 측정하는 핀
    
    ⇒ `digitalWrite(Trig, HIGH)`로 초음파 발사
    
    ⇒ `pulseIn(Echo, HIGH)`로 돌아오는 시간 측정
    
    ⇒ 공식: `거리(cm) = 시간(μs) × 0.034 / 2`
    

1. LED (네오픽셀: WS2812) ⇒ 충분한 전류 공급 필요
    
    WS2812 LED는 RGB 색상 모두 표현 가능한 LED.
    
    단순 전구가 아니라, 신호(DIN)로 색상 명령을 보내야 함.
    
    - DIN: D6 - 아두이노에서 색상 명령을 보내는 신호선
    - VCC: 5V
    - GND: GND
    
    ⇒ 여러 개의 WS2812을 연결하면 전류를 많이 먹음.
    
    ⇒ 아두이노 보드만으로는 부족할 수 있음. 
    
    ⇒ 별도 전원 공급 (예: 5V 어댑터) 필요할 수 있음.
    
    ⇒ 라이브러리 필요: `Adafruit_NeoPixel` 등
    

### 온도, 거리 센서 연결해서 데이터 읽기

```arduino
// 온습도 센서 DHT11 라이브러리
#include <DHT.h> 
// 초음파 센서 HC-SR04 라이브러리
#include <Ultrasonic.h> 

// 핀 설정
// 온도 센서 데이터 핀 (D13)
#define DHTPIN 13 
// DHT11 타입 지정
#define DHTTYPE DHT11
// 초음파 센서 Trig 핀 (D4)
#define TRIG_PIN 4
// 초음파 센서 Echo 핀 (D3)
#define ECHO_PIN 3

// 센서 객체 생성
// 온도 센서 객체 dht 생성
DHT dht(DHTPIN, DHTTYPE);  
//초음파 센서 객체 ultrasonic 생성
Ultrasonic ultrasonic(TRIG_PIN, ECHO_PIN);

void setup() 
{
	// 아두이노 <-> 컴퓨터 간 시리얼 통신 시작 (9600bps)
	Serial.begin(9600);
	
	// DHT11 온도 센서 초기화
	dht.begin(); 
	Serial.println("Temperature(C)  Distance(cm)");
}

void loop()
{
  // 1.5초 대기
  delay(1500);
	
  // C 온도 읽기
  float temperature = dht.readTemperature();
  // cm 거리 읽기
  float distance = ultrasonic.read();

  // 센서 값 출력
  Serial.print(temperature);
  Serial.print("            ");
  Serial.println(distance);
  delay(50);
}
```

```
## 출력 예시
Temperature(C)  Distance(cm)
25.00            17.38
25.10            18.00
...
```

- 온도 측정: `dht.readTemperature()`
- 초음파 거리 측정: `ultrasonic.read()`

### 온도, 거리 센서 연결해서 데이터 읽기 → 조도로 출력하기

온도 센서, 습도 센서, 초음파 거리 센서로부터 데이터를 읽어서 그 값을 기반으로 네오픽셀(WS2812) LED 색상을 제어하는 코드 작성.

- 온도: 20~40도 사이에서 빨간색 밝기 증가
- 습도: 0~100% 사이에서 초록색 밝기 증가
- 거리: 2~100cm 사이에서 파란색 밝기 증가

```arduino
// 네오픽셀 LED 라이브러리
#include <Adafruit_NeoPixel.h>
// 초음파 센서 HC-SR04 라이브러리
#include <Ultrasonic.h>
// 온도 센서 DHT11 라이브러리
#include <DHT.h>

// 핀 설정
// 네오픽셀 LED 핀 (디지털 D6)
#define LED_PIN 6
// 온도 센서 핀 (디지털 D13)
#define DHTPIN 13
// DHT11 센서 사용
#define DHTTYPE DHT11
// 초음파 Trig 핀 (디지털 D4)
#define TRIG_PIN 4
// 초음파 Echo 핀 (디지털 D3)
#define ECHO_PIN 3

// 센서 객체 생성 및 초기화
Adafruit_NeoPixel strip = Adafruit_NeoPixel(1, LED_PIN, NEO_GRB + NEO_KHZ800);
DHT dht(DHTPIN, DHTTYPE);
Ultrasonic ultrasonic(TRIG_PIN, ECHO_PIN);

void setup() 
{
  Serial.begin(9600); // 시리얼 통신 시작 (9600bps)
  dht.begin();        // 온도 센서 초기화
  strip.begin();      // 네오픽셀 초기화
  strip.show();       // 모든 LED 꺼짐 상태로 시작

  Serial.println("Temp(C)  Distance(cm)");
}

void loop() 
{
  delay(1500);

  // 온도(섭씨) 측정
  float temperature = dht.readTemperature();
  // 거리(cm) 측정
  float distance = ultrasonic.read();

  // 센서값 시리얼 출력
  Serial.print(temperature);
  Serial.print("    ");
  Serial.println(distance);

  // LED 색상 제어
  controlLED(temperature, distance);
  delay(50);
}

// LED 색상 제어 함수
void controlLED(float temp, float dist) {
  // 온도(20~40도) → 빨강(R)
  int r = constrain(map(temp, 20, 40, 0, 255), 0, 255);
  // 초음파 거리(2~100cm) → 파랑(B)
  int b = constrain(map(dist, 2, 100, 0, 255), 0, 255);
  // 초록(G)은 사용하지 않음
  int g = 0;

  // LED 0번에 RGB 색상 설정
  strip.setPixelColor(0, strip.Color(r, g, b));
  strip.show();
}
```

- 온도 올라가면 빨강(R) 밝기 증가
- 거리가 가까워질수록 파랑(B) 밝기 증가

### LED 제어 알고리즘 설계

- 온도: 20~40도 사이에서 빨간색 밝기 증가
- 거리: 2~100cm 사이에서 파란색 밝기 증가
- 특수 조건:
    - 온도 > 35도 → 빨간색 MAX
    - 거리 < 10cm → 파란색 MAX

### 조건에 따라 LED 다르게 제어하는 프로그램 구현

- `controlLED()` 함수에 조건문을 통해 상황에 따라 LED 색상을 강제로 설정
- 각 조건을 우선순위에 따라 검사하여 가장 적합한 색상을 적용함.

### 센서 데이터 처리와 LED 제어 로직 통합

- `loop()` 함수에서 센서 값을 읽고, 이를 바로 `controlLED()` 함수에 전달
- `controlLED()` 함수 내부에서 각 센서 값을 색상 요소(R, G, B)로 변환
- 동시에 특정 조건(고온, 습함, 근접) 발생 시 색상 변경

### 센서 에러 탐지하는 방법

- 온도값이 비정상적(0도 이하 또는 150도 이상)일 경우 25도로 초기화
- 습도값이 음수나 범위를 넘지 않도록 `constrain()` 사용
- 초음파 거리값이 정상 범위(2cm~400cm) 밖이면 기본값 100cm로 설정

### LED 제어 최종 통합 코드

```arduino
#include <DHT.h>
#include <Ultrasonic.h>
#include <Adafruit_NeoPixel.h>

#define DHTPIN 13
#define DHTTYPE DHT11
#define TRIG_PIN 4
#define ECHO_PIN 3
#define LED_PIN 6

DHT dht(DHTPIN, DHTTYPE);
Ultrasonic ultrasonic(TRIG_PIN, ECHO_PIN);
Adafruit_NeoPixel strip = Adafruit_NeoPixel(1, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() 
{
  Serial.begin(9600);
  dht.begin();
  strip.begin();
  strip.show();
  Serial.println("Temp(C) Distance(cm)");
}

void loop() 
{
  delay(1500);

  float temp = dht.readTemperature();
  float dist = ultrasonic.read();

  Serial.print(temp); Serial.print(" ");
  Serial.print(dist); Serial.println();

  controlLED(temp, dist);
  delay(50);
}

void controlLED(float temp, float dist) 
{
  // 온도 20~40도 → 빨강 밝기 매핑
  int red  = constrain(map(temp, 20, 40, 0, 255), 0, 255);
  // 거리 2~100cm → 파랑 밝기 매핑
  int blue = constrain(map(dist, 2, 100, 0, 255), 0, 255);
  int green = 0; // 초록은 사용 안함

  // 조건 적용
  if (temp > 35) red = 255;      // 온도 35도 초과 → 빨강 MAX
  if (dist < 10) blue = 255;     // 거리 10cm 미만 → 파랑 MAX

  strip.setPixelColor(0, strip.Color(red, green, blue));
  strip.show();
}
```

### 제약사항

1. 센서 정확도와 응답 시간 고려
    - DHT11 센서는 온도 및 습도 센서 중에 비교적 저렴함. 측정 응답 속도가 느리고(최소 1초 간격 권장), 정확도도 낮은 편.
    → loop()에서 최소 1000ms으로 작성하기.
    - Ultrasonic 센서도 반사 신호 상태에 따라 노이즈가 있을 수 있지만 비교적 빠르게 측정 가능.

1. 리소스 제한과 효율적인 코드
    - 현재 코드는 비교적 간단하고, 메모리 사용도 적은 편.
    - 다만 `strip.show()` 호출이 LED 색상을 바꿀 때마다 일어나는 데, LED가 많아지면 부담이 커짐 (지금은 LED 1개라 문제 없음)