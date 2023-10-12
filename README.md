// Arduino pin assignment
#define PIN_LED  9
#define PIN_TRIG 12   // sonar sensor TRIGGER
#define PIN_ECHO 13   // sonar sensor ECHO

// configurable parameters
#define SND_VEL 346.0     // sound velocity at 24 celsius degree (unit: m/sec)
#define INTERVAL 25      // sampling interval (unit: msec) - 변경된 샘플링 주기
#define PULSE_DURATION 10 // ultra-sound Pulse Duration (unit: usec)
#define _DIST_MIN 100.0   // minimum distance to be measured (unit: mm)
#define _DIST_MAX 300.0   // maximum distance to be measured (unit: mm)

#define TIMEOUT ((INTERVAL / 2) * 1000.0) // maximum echo waiting time (unit: usec)
#define SCALE (0.001 * 0.5 * SND_VEL) // coefficent to convert duration to distance

unsigned long last_sampling_time = 0; 

float USS_measure(int TRIG, int ECHO) {
  digitalWrite(TRIG, HIGH);
  delayMicroseconds(PULSE_DURATION);
  digitalWrite(TRIG, LOW);
  
  return pulseIn(ECHO, HIGH, TIMEOUT) * SCALE; 
}

void setup() {
  // initialize GPIO pins
  pinMode(PIN_LED, OUTPUT);
  pinMode(PIN_TRIG, OUTPUT);  // sonar TRIGGER
  pinMode(PIN_ECHO, INPUT);   // sonar ECHO
  digitalWrite(PIN_TRIG, LOW);  // turn-off Sonar 

  // initialize serial port
  Serial.begin(57600);
}

void loop() {
  unsigned long current_time = millis();
  float distance;


  if (current_time - last_sampling_time >= INTERVAL) {

    distance = USS_measure(PIN_TRIG, PIN_ECHO);

    last_sampling_time = current_time;

 
    int ledBrightness = map(distance, _DIST_MIN, _DIST_MAX, 255, 0); // 거리에 따라 LED 밝기 계산
    ledBrightness = constrain(ledBrightness, 0, 255); // 밝기 값을 0과 255 사이로 제한
    analogWrite(PIN_LED, ledBrightness); // LED 밝기 제어
  }


  Serial.print("Min:"); Serial.print(_DIST_MIN);
  Serial.print(",distance:"); Serial.print(distance);
  Serial.print(",Max:"); Serial.print(_DIST_MAX);
  Serial.println("");
}
