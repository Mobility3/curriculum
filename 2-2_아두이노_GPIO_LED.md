### 필요한 재료

- Arduino Uno
- 브레드보드
- LED
- 저항(220Ω)
- 점퍼선
    
    ![image.png](attachment:8b2676dc-97b3-4669-be1b-d7439eb004c6:image.png)
    

### 회로 구성

- 전류는 + (전원) → - (GND) 방향으로 흐른다.
- LED는 전류가 한 방향으로만 흐를 때만 빛이 난다.
- 저항은 LED가 타지 않게 전류를 줄여주는 장치다.
    
    ![https://m.blog.naver.com/codingbird/222299358577](attachment:6c494236-7691-4e49-93a5-a057475c7b49:image.png)
    
    https://m.blog.naver.com/codingbird/222299358577
    

### 테스트 시뮬레이션

![image.png](attachment:8fcddaa6-de2c-4314-a8d5-2bf3ed2ff35b:image.png)

![image.png](attachment:788f2105-66b8-41f5-a085-22f4c31fadce:image.png)

```arduino
void setup() {
  pinMode(13, OUTPUT);  // D13 핀을 출력으로 설정
}

void loop() {
  digitalWrite(13, HIGH);  // LED ON
  delay(1000);             // 1초 대기
  digitalWrite(13, LOW);   // LED OFF
  delay(1000);             // 1초 대기
}
```

- LED 테스트
- 참고자료: [https://egeasy.tistory.com/entry/아두이노-LED-제어하기](https://egeasy.tistory.com/entry/%EC%95%84%EB%91%90%EC%9D%B4%EB%85%B8-LED-%EC%A0%9C%EC%96%B4%ED%95%98%EA%B8%B0)

### 모스부호 SOS 알고리즘

- **S = ...** (3개의 짧은 점) - LED 짧게 (0.2초)
- **O = ---** (3개의 긴 선) - LED 길게 (0.6초)
- **S = ...** (3개의 짧은 점) - LED 짧게 (0.2초)
- 각 신호 사이: 0.2초 OFF
- 문자 간 간격: 0.6초
- 메시지 간 간격: 1.4초

```arduino
void dot() {
  digitalWrite(13, HIGH);
  delay(200);  // 점: 0.2초
  digitalWrite(13, LOW);
  delay(200);
}

void dash() {
  digitalWrite(13, HIGH);
  delay(600);  // 선: 0.6초
  digitalWrite(13, LOW);
  delay(200);
}

void sos() {
  // S
  dot(); dot(); dot();
  delay(600);  // 문자 간격
  // O
  dash(); dash(); dash();
  delay(600);  // 문자 간격
  // S
  dot(); dot(); dot();
  delay(1400);  // 메시지 간격
}

void setup() {
  pinMode(13, OUTPUT);
}

void loop() {
  sos();  // 무한 반복
}
```

https://www.tinkercad.com/things/bIJDVg3vfJs-fantastic-bombul-hango/editel?returnTo=https%3A%2F%2Fwww.tinkercad.com%2Fdashboard

- `pinMode(13, OUTPUT)`: D13 핀을 출력모드로 설정
- `digitalWrite(pin, HIGH/LOW)`: 전압을 인가하거나 끊어 LED ON/OFF
- `delay(ms)`: 주어진 시간만큼 대기 (ms 단위)
- `dot()`, `dash()` 함수: LED 점등 시간 조절을 통해 모스부호 구현
- `loop()`: S.O.S 신호를 무한 반복

### GPIO 핀 제어 기초 명령어

| 명령어 | 설명 |
| --- | --- |
| `pinMode(pin, mode)` | 핀의 입출력 모드 설정 (`INPUT`, `OUTPUT`) |
| `digitalWrite(pin, value)` | 디지털 신호 출력 (`HIGH` = 5V, `LOW` = 0V) |
| `delay(ms)` | ms 단위로 코드 실행 일시 중지 |

### 소비 전력 계산

- LED 1개: 약 20mA
- LED 5개 사용 시 총 전류 ≈ 100mA
- USB 전원 공급 한계: 약 500mA
- 그 이상 사용할 경우 별도의 5V 1A 어댑터 사용 권장