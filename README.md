# Washing Machine Water Management
I made this project for my teacher who has a problem like this. So I made this code and it crossed my mind to publish it to github. 

### Component
- Arduino (Uno, Mega, Pro mini, Nano)
- LCD (Im using I2C)
- Relay (For turn on solenoid)
- Power Supply (I'm using 5V)
- Buzzer Active
- Flow Sensor
- Box (Optional)
- Button (Optional if you using box) for reset button

Connection
```
- VCC => 5V
- GND => GND

Relay
- IN => 13

Flow Sensor
- DATA => 2

LCD I2C (I'm using Arduino Uno)
- SDA => A4
- SCL => A5

Button
# You can connect to RST pin to GND pin
```

And this is the shape of this tool
<img src="https://drive.google.com/uc?export=view&id=1bgqnSFk2PjINKFgASrTntyFMVphg4JgA" alt="" witdh=50><br><br>
#### Source Code
```cpp

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x26, 16, 2);
byte sensorInterrupt = 0;
byte sensorPin       = 2;
float calibrationFactor = 4.7;
volatile byte pulseCount;
float flowRate;
unsigned int flowMilliLitres;
unsigned long totalMilliLitres;
unsigned long oldTime;
int liter;
int statusalarm=0;
const int relay = 13;
int buzzer = 8;
void setup()
{
  Serial.begin(9600);
  lcd.begin();
  lcd.backlight();
  noTone(buzzer);
  pinMode(relay, OUTPUT);
  pinMode(buzzer, OUTPUT);
  digitalWrite(relay, HIGH);
  digitalWrite(buzzer, LOW);
  pinMode(sensorPin, INPUT);
  digitalWrite(sensorPin, HIGH);
  pulseCount        = 0;
  flowRate          = 0.0;
  flowMilliLitres   = 0;
  totalMilliLitres  = 0;
  oldTime           = 0;
  attachInterrupt(sensorInterrupt, pulseCounter, FALLING);
}

void loop()
{

  if ((millis() - oldTime) > 1000)   // Only process counters once per second
  {
    lcd.clear();
    detachInterrupt(sensorInterrupt);
    flowRate = ((1000.0 / (millis() - oldTime)) * pulseCount) / calibrationFactor;
    oldTime = millis();
    flowMilliLitres = (flowRate / 60) * 1000;
    totalMilliLitres += flowMilliLitres;
    unsigned int frac;
    Serial.print("Flow rate: ");
    Serial.print(int(flowRate));  
    Serial.print("L/min");
    Serial.print("\t");
    Serial.print("Output Liquid Quantity: ");
    Serial.print(totalMilliLitres);
    Serial.println("mL");
    Serial.print("\t");       // Print tab space
    Serial.print(totalMilliLitres / 1000);
    Serial.print("L");
    liter = totalMilliLitres / 1000;
    lcd.print("L: ");
    lcd.print(liter);
    if (liter >= 10) {
      digitalWrite(relay, HIGH);
      if(statusalarm >=60){
          digitalWrite(buzzer, HIGH);
      }else{
          digitalWrite(buzzer, LOW);
      }
      statusalarm++;
    } else {
      digitalWrite(relay, LOW);
    }
    pulseCount = 0;
    attachInterrupt(sensorInterrupt, pulseCounter, FALLING);
  }
}
void pulseCounter()
{
  pulseCount++;
}

```
[Download Source Code](https://gist.github.com/saronggos/6195ba961a87da28dfcac67a86e36c8d/archive/e66fc7dfb78e5edcd0cdec51942d5edea71ff9bb.zip)

For the next, I would to upgrade the features for this project. Wait for the next repository update.
