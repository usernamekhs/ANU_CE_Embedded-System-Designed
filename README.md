💡 스마트 하이브리드 무드등 (아두이노 + 블루투스 + CDS)

스마트폰 앱을 통해 색상을 자유롭게 제어하는 "수동 모드"와, CDS(조도) 센서를 이용해 주변이 어두워지면 자동으로 램프가 켜지는 "자동 모드"를 모두 갖춘 스마트 무드등입니다.


✨ 주요 기능

📱 수동 모드: 스마트폰 앱으로 원하는 RGB 색상(빛의 3원색 조합)을 원격 제어합니다.

🌗 자동 모드: CDS 센서가 주변 밝기를 감지하여, 어두워지면 자동으로 켜지고 밝아지면 꺼집니다.

🔄 모드 전환: 블루투스 명령으로 "자동" 모드와 "수동" 모드를 실시간으로 전환할 수 있습니다.

🛠️ 필요 부품

아두이노 Uno (또는 호환 보드)

블루투스 모듈 (HC-05 또는 HC-06)

LED (빨간색) 1개

LED (초록색) 1개

LED (파란색) 1개

CDS 센서 (LDR / 조도 센서) 1개

220Ω 저항 3개 (각 LED 보호용)

10kΩ 저항 1개 (CDS 센서 풀다운 저항용)

브레드보드 & 점퍼선

블루투스 시리얼 터미널 앱이 설치된 스마트폰

🔌 회로 연결

블루투스 (SoftwareSerial 사용):

VCC → 아두이노 5V

GND → 아두이노 GND

TXD → 아두이노 D10 (Software RX)

RXD → 아두이노 D11 (Software TX)

개별 LED (빨강, 초록, 파랑):

[빨간색 LED] (+) 긴 다리 → 220Ω 저항 → 아두이노 D9 (PWM ~)

[빨간색 LED] (-) 짧은 다리 → GND

[초록색 LED] (+) 긴 다리 → 220Ω 저항 → 아두이노 D6 (PWM ~)

[초록색 LED] (-) 짧은 다리 → GND

[파란색 LED] (+) 긴 다리 → 220Ω 저항 → 아두이노 D5 (PWM ~)

[파란색 LED] (-) 짧은 다리 → GND

CDS 센서 (풀다운 저항 방식):

한쪽 다리 → 아두이노 5V

다른 쪽 다리 → 아두이노 A0 (아날로그 입력)

(동시에) A0 핀 → 10kΩ 저항 → 아두이노 GND

🤖 작동 원리

코드는 autoMode라는 하나의 boolean(논리) 변수를 기준으로 작동합니다.

true (자동 모드): A0 핀에서 CDS 센서 값을 계속 읽습니다. 만약 lightLevel 값이 설정한 lightThreshold 값보다 낮아지면(어두워지면) LED가 미리 설정된 색상으로 켜집니다.

false (수동 모드): CDS 센서 값을 무시하고, 오직 블루투스 모듈로 들어오는 색상 명령에만 반응합니다.

🚀 사용 방법

위에 설명된 대로 회로를 조립합니다.

아래의 코드를 아두이노에 업로드합니다.

스마트폰의 블루투스 설정에서 HC-05 또는 HC-06 모듈을 페어링합니다. (비밀번호는 보통 1234 또는 0000입니다.)

블루투스 시리얼 터미널 앱(예: Serial Bluetooth Terminal)을 열어 페어링된 모듈에 연결합니다.

아래의 명령어를 앱의 터미널 창에 입력하여 전송합니다.

📱 블루투스 명령어

스마트폰의 시리얼 터미널 앱에서 아래의 텍스트를 전송하세요:

A — 자동 모드로 전환합니다.

M — 수동 모드로 전환합니다.

R[값] — 빨간색(Red) 밝기를 조절합니다. (예: R255, R100)

G[값] — 초록색(Green) 밝기를 조절합니다. (예: G50)

B[값] — 파란색(Blue) 밝기를 조절합니다. (예: B200)

OFF — 모든 LED를 끕니다. (R,G,B 값을 0으로 설정)

예시 (보라색 만들기):
M 전송 → R255 전송 → G0 전송 → B255 전송

💻 아두이노 전체 코드 (.ino)

마크다운 파일 내에 코드를 삽입할 때는 아래처럼 ```cpp와 ``` 사이에 코드를 넣습니다.

/*
 * --- 스마트 하이브리드 무드등 (아두이노 + 블루투스 + CDS) ---
 * * 버전: 개별 LED (R, G, B) 사용 버전
 * * 기능:
 * 1. 블루투스(HC-05/06)를 통해 스마트폰으로 R, G, B LED의 밝기를 제어합니다. (수동 모드)
 * 2. CDS(조도) 센서로 주변 밝기를 감지하여, 어두워지면 자동으로 LED를 켭니다. (자동 모드)
 * 3. 블루투스 명령('A' 또는 'M')으로 두 모드를 실시간으로 전환합니다.
 * * 회로 연결 (README.md 파일 참조):
 * - Bluetooth (RX: D11, TX: D10)
 * - Red LED   (Signal: D9)
 * - Green LED (Signal: D6)
 * - Blue LED  (Signal: D5)
 * - CdS Sensor (Signal: A0, 10kΩ 풀다운 저항)
 */

#include <SoftwareSerial.h> // 아두이노의 기본 시리얼(USB)과 충돌을 피하기 위해 소프트웨어 시리얼 라이브러리 사용

// --- 핀 번호 설정 ---
// 블루투스
const int BT_RX_PIN = 10;
const int BT_TX_PIN = 11;
SoftwareSerial btSerial(BT_RX_PIN, BT_TX_PIN); // (RX, TX)

// 개별 LED (PWM 핀 사용)
const int RED_PIN = 9;   // 빨간색 LED
const int GREEN_PIN = 6; // 초록색 LED
const int BLUE_PIN = 5;  // 파란색 LED

// CDS (조도) 센서
const int CDS_PIN = A0;

// --- 전역 변수 설정 ---
bool autoMode = true;        // 프로그램 시작 시 '자동 모드'로 기본 설정
int lightThreshold = 300; // '어둡다'고 판단하는 기준 값 (0~1023). 
                             // LDR + 10kΩ 풀다운 회로 기준.
                             // 값이 낮을수록 어두운 것입니다. (실제 환경에 맞게 조절 필요)

// ----------------------------------------
//   setup() : 프로그램 시작 시 한 번만 실행
// ----------------------------------------
void setup() {
  // 핀 모드 설정
  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);
  pinMode(CDS_PIN, INPUT); 

  // 통신 시작
  Serial.begin(9600);      // PC의 시리얼 모니터 (디버깅용)
  btSerial.begin(9600);    // 블루투스 모듈 통신 (기본 속도 9600)
  Serial.println("Smart Hybrid Mood Lamp: Initialized (Separate LEDs).");
  Serial.println("Mode: Auto");
}

// ----------------------------------------
//   loop() : setup() 후 계속 무한 반복 실행
// ----------------------------------------
void loop() {
  
  // 1. 블루투스로부터 명령이 수신되었는지 확인
  handleBluetooth();
  
  // 2. 현재 모드가 '자동 모드'인지 확인하고 그에 맞게 작동
  handleAutoMode();
  
  delay(50); // 0.05초마다 반복 (너무 빠른 반복 방지)
}

// ----------------------------------------
//   handleBluetooth() : 블루투스 명령 처리 함수
// ----------------------------------------
void handleBluetooth() {
  if (btSerial.available() > 0) {
    // 블루투스로부터 1바이트(문자 1개)를 읽어옵니다.
    char command = btSerial.read();

    // ----- 모드 전환 -----
    if (command == 'A') {
      autoMode = true;
      Serial.println("Mode switched: Auto");
    } 
    else if (command == 'M') {
      autoMode = false;
      Serial.println("Mode switched: Manual");
    }
    
    // ----- 'OFF' 명령 처리 -----
    else if (command == 'O') {
      // "OFF"라는 3글자 명령어를 확인합니다. (O, F, F)
      if (btSerial.available() >= 2) {
        char f1 = btSerial.read();
        char f2 = btSerial.read();
        if (f1 == 'F' && f2 == 'F') {
          autoMode = false; // 끄기 명령은 수동 모드로 간주
          setLedColor(0, 0, 0); // LED를 끕니다.
          Serial.println("Command: OFF");
        }
      }
    }
    
    // ----- RGB 값 설정 (수동 모드에서만 작동) -----
    else if ((command == 'R' || command == 'G' || command == 'B') && autoMode == false) {
      // 'R', 'G', 'B' 문자 뒤에 오는 숫자 값을 읽어옵니다. (예: "R255")
      int value = btSerial.parseInt();

      if (command == 'R') {
        analogWrite(RED_PIN, value); // 빨간색 LED 밝기 조절
        Serial.print("Set Red: "); Serial.println(value);
      } else if (command == 'G') {
        analogWrite(GREEN_PIN, value); // 초록색 LED 밝기 조절
        Serial.print("Set Green: "); Serial.println(value);
      } else if (command == 'B') {
        analogWrite(BLUE_PIN, value); // 파란색 LED 밝기 조절
        Serial.print("Set Blue: "); Serial.println(value);
      }
    }
  } // if (btSerial.available() > 0) 끝
}

// ----------------------------------------
//   handleAutoMode() : 자동 모드 로직 처리 함수
// ----------------------------------------
void handleAutoMode() {
  // '자동 모드'가 아닐 경우, 이 함수는 즉시 종료
  if (autoMode == false) {
    return;
  }
  
  // '자동 모드'일 경우:
  // 1. CDS 센서로부터 현재 밝기 값을 읽어옵니다. (0 ~ 1023)
  int lightLevel = analogRead(CDS_PIN);
  
  // 2. 현재 밝기 값이 설정된 임계값(lightThreshold)보다 낮은지 (어두운지) 확인
  if (lightLevel < lightThreshold) {
    // 어두울 경우: 미리 설정된 색상 (예: 따뜻한 흰색)으로 LED를 켭니다.
    setLedColor(255, 200, 100); // R, G, B 값 (원하는 색상으로 변경 가능)
  } else {
    // 밝을 경우: LED를 끕니다.
    setLedColor(0, 0, 0);
  }
}

// ----------------------------------------
//   setLedColor() : LED 색상을 한 번에 설정하는 함수
// ----------------------------------------
void setLedColor(int r, int g, int b) {
  analogWrite(RED_PIN, r);
  analogWrite(GREEN_PIN, g);
  analogWrite(BLUE_PIN, b);
}
