// CODE #1
// this code is for testing the motors and the encoder
// the car must (move forward => stop for a second => move backward =>stop for a second) in a loop


// pins => motor 1
const int encoderPinC1 = 18;   // channel A for motor 1
const int encoderPinC2 = 19;   // channel B for motor 1
const int enablePin = 14;      // enable pin for motor 1 (esp pin must generate PWM signal)
const int input1Pin = 26;      // input 1 for motor 1
const int input2Pin = 27;      // input 2 for motor 1

// pins => motor 2
const int encoder2PinC1 = 23;  // channel A for motor 2
const int encoder2PinC2 = 22;  // channel B for motor 2
const int enable2Pin = 25;     // enable pin for motor 2 (esp pin must generate PWM signal)
const int input3Pin = 33;      // input 1 for motor 2
const int input4Pin = 32;      // input 2 for motor 2

// variables => tracking encoders
volatile long encoderCount1 = 0; // motor 1
volatile long encoderCount2 = 0; // motor 2
int lastEncoded1 = 0;            // motor 1
int lastEncoded2 = 0;            // motor 2

// handle encoder interrupts => motor 1
void updateEncoder1() {
  int MSB = digitalRead(encoderPinC1); // most significant bit
  int LSB = digitalRead(encoderPinC2); // least significant bit

  int encoded = (MSB << 1) | LSB;      // current value: combine the two pin states (from channel A and channel B)
  int sum = (lastEncoded1 << 2) | encoded; // sum: combine current value with previous value

  if (sum == 0b1101 || sum == 0b0100 || sum == 0b0010 || sum == 0b1011) {
    encoderCount1++;
  }
  if (sum == 0b1110 || sum == 0b0111 || sum == 0b0001 || sum == 0b1000) {
    encoderCount1--;
  }

  lastEncoded1 = encoded; // store the current state (track)
}

// handle encoder interrupts for motor 2
void updateEncoder2() {
  int MSB = digitalRead(encoder2PinC1); // most significant bit
  int LSB = digitalRead(encoder2PinC2); // least significant bit

  int encoded = (MSB << 1) | LSB;       // current value: combine the two pin states (from channel A and channel B)
  int sum = (lastEncoded2 << 2) | encoded; // sum: combine current value with previous value

  if (sum == 0b1101 || sum == 0b0100 || sum == 0b0010 || sum == 0b1011) {
    encoderCount2++;
  }
  if (sum == 0b1110 || sum == 0b0111 || sum == 0b0001 || sum == 0b1000) {
    encoderCount2--;
  }

  lastEncoded2 = encoded; // store the current state (track)
}

void setup() {
  
  Serial.begin(115200);

  // pin modes => motor 1
  pinMode(encoderPinC1, INPUT);
  pinMode(encoderPinC2, INPUT);
  pinMode(enablePin, OUTPUT);
  pinMode(input1Pin, OUTPUT);
  pinMode(input2Pin, OUTPUT);

  // pin modes => motor 2
  pinMode(encoder2PinC1, INPUT);
  pinMode(encoder2PinC2, INPUT);
  pinMode(enable2Pin, OUTPUT);
  pinMode(input3Pin, OUTPUT);
  pinMode(input4Pin, OUTPUT);

  // interrupts for encoders (attach)
  attachInterrupt(digitalPinToInterrupt(encoderPinC1), updateEncoder1, CHANGE);
  attachInterrupt(digitalPinToInterrupt(encoderPinC2), updateEncoder1, CHANGE);
  attachInterrupt(digitalPinToInterrupt(encoder2PinC1), updateEncoder2, CHANGE);
  attachInterrupt(digitalPinToInterrupt(encoder2PinC2), updateEncoder2, CHANGE);

  // enable both motors
  digitalWrite(enablePin, HIGH);
  digitalWrite(enable2Pin, HIGH);
}

void loop() {
  // first move both motors => forward
  digitalWrite(input1Pin, HIGH);
  digitalWrite(input2Pin, LOW);
  digitalWrite(input3Pin, LOW);
  digitalWrite(input4Pin, HIGH);
  Serial.println("moving forward...");
  delay(20000);

  // stop both motors
  digitalWrite(input1Pin, LOW);
  digitalWrite(input2Pin, LOW);
  digitalWrite(input3Pin, LOW);
  digitalWrite(input4Pin, LOW);
  Serial.println("Stopped.");
  delay(1000); 

  // second move both motors => backward
  digitalWrite(input1Pin, LOW);
  digitalWrite(input2Pin, HIGH);
  digitalWrite(input3Pin, HIGH);
  digitalWrite(input4Pin, LOW);
  Serial.println("Moving backward...");
  delay(20000);

  // stop both motors
  digitalWrite(input1Pin, LOW);
  digitalWrite(input2Pin, LOW);
  digitalWrite(input3Pin, LOW);
  digitalWrite(input4Pin, LOW);
  Serial.println("Stopped.");
  delay(1000); 

  // serial print => encoder counts
  Serial.print("Encoder 1 Count: ");
  Serial.println(encoderCount1);
  Serial.print("Encoder 2 Count: ");
  Serial.println(encoderCount2);
}