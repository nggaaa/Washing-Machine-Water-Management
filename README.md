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

byte statusLed    = 12;

byte sensorInterrupt = 0;  // 0 = digital pin 2
byte sensorPin       = 2;

// The hall-effect flow sensor outputs approximately 4.5 pulses per second per
// litre/minute of flow.
float calibrationFactor = 13.75;

volatile byte pulseCount;

float flowRate;
unsigned int flowMilliLitres;
unsigned long totalMilliLitres;

unsigned long oldTime;

int liter;

const int volume = 40; //liter
const int relay = 13;

byte zero[] = {
  B00000,
  B00000,
  B00000,
  B00000,
  B00000,
  B00000,
  B00000,
  B00000
};
byte one[] = {
  B10000,
  B10000,
  B10000,
  B10000,
  B10000,
  B10000,
  B10000,
  B10000
};

byte two[] = {
  B11000,
  B11000,
  B11000,
  B11000,
  B11000,
  B11000,
  B11000,
  B11000
};

byte three[] = {
  B11100,
  B11100,
  B11100,
  B11100,
  B11100,
  B11100,
  B11100,
  B11100
};

byte four[] = {
  B11110,
  B11110,
  B11110,
  B11110,
  B11110,
  B11110,
  B11110,
  B11110
};

byte five[] = {
  B11111,
  B11111,
  B11111,
  B11111,
  B11111,
  B11111,
  B11111,
  B11111
};


void setup()
{

  // Initialize a serial connection for reporting values to the host
  Serial.begin(9600);
  lcd.begin();
  lcd.backlight();

  lcd.createChar(0, zero);
  lcd.createChar(1, one);
  lcd.createChar(2, two);
  lcd.createChar(3, three);
  lcd.createChar(4, four);
  lcd.createChar(5, five);


  // Set up the status LED line as an outputa
  pinMode(statusLed, OUTPUT);
  pinMode(relay, OUTPUT);
  digitalWrite(relay, HIGH);
  digitalWrite(statusLed, HIGH);  // We have an active-low LED attached

  pinMode(sensorPin, INPUT);
  digitalWrite(sensorPin, HIGH);
  lcd.clear();

  pulseCount        = 0;
  flowRate          = 0.0;
  flowMilliLitres   = 0;
  totalMilliLitres  = 0;
  oldTime           = 0;

  // The Hall-effect sensor is connected to pin 2 which uses interrupt 0.
  // Configured to trigger on a FALLING state change (transition from HIGH
  // state to LOW state)
  attachInterrupt(sensorInterrupt, pulseCounter, FALLING);
}

/**
   Main program loop
*/
void loop()
{

  if ((millis() - oldTime) > 1000)   // Only process counters once per second
  {

    // Disable the interrupt while calculating flow rate and sending the value to
    // the host
    detachInterrupt(sensorInterrupt);

    // Because this loop may not complete in exactly 1 second intervals we calculate
    // the number of milliseconds that have passed since the last execution and use
    // that to scale the output. We also apply the calibrationFactor to scale the output
    // based on the number of pulses per second per units of measure (litres/minute in
    // this case) coming from the sensor.
    flowRate = ((1000.0 / (millis() - oldTime)) * pulseCount) / calibrationFactor;

    // Note the time this processing pass was executed. Note that because we've
    // disabled interrupts the millis() function won't actually be incrementing right
    // at this point, but it will still return the value it was set to just before
    // interrupts went away.
    oldTime = millis();

    // Divide the flow rate in litres/minute by 60 to determine how many litres have
    // passed through the sensor in this 1 second interval, then multiply by 1000 to
    // convert to millilitres.
    flowMilliLitres = (flowRate / 60) * 1000;

    // Add the millilitres passed in this second to the cumulative total
    totalMilliLitres += flowMilliLitres;

    unsigned int frac;

    // Print the flow rate for this second in litres / minute
    Serial.print("Flow rate: ");
    Serial.print(int(flowRate));  // Print the integer part of the variable
    Serial.print("L/min");
    Serial.print("\t");       // Print tab space

    // Print the cumulative total of litres flowed since starting
    Serial.print("Output Liquid Quantity: ");
    Serial.print(totalMilliLitres);
    Serial.println("mL");
    Serial.print("\t");       // Print tab space
    Serial.print(totalMilliLitres / 1000);
    Serial.print("L");
    lcd.setCursor(0, 0);
    liter = totalMilliLitres / 1000;
    lcd.print("L: ");
    lcd.print(liter);

    if (liter >= volume) {
      digitalWrite(relay, LOW);
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Proses Selesai");

    } else {
      digitalWrite(relay, HIGH);
      updateProgressBar(liter, volume, 1);
    }


    // Reset the pulse counter so we can start incrementing again
    pulseCount = 0;

    // Enable the interrupt again now that we've finished sending output
    attachInterrupt(sensorInterrupt, pulseCounter, FALLING);
  }
}

/*
  Insterrupt Service Routine
*/
void pulseCounter()
{
  // Increment the pulse counter
  pulseCount++;
}

void updateProgressBar(unsigned long count, unsigned long totalCount, int lineToPrintOn)
{
  double factor = totalCount / 80.0;        //See note above!
  int percent = (count + 1) / factor;
  int number = percent / 5;
  int remainder = percent % 5;
  if (number > 0)
  {
    lcd.setCursor(number - 1, lineToPrintOn);
    lcd.write(5);
  }

  lcd.setCursor(number, lineToPrintOn);
  lcd.write(remainder);
}

```
[Download Source Code](https://gist.github.com/saronggos/6195ba961a87da28dfcac67a86e36c8d/archive/e66fc7dfb78e5edcd0cdec51942d5edea71ff9bb.zip)

For the next, I would to upgrade the features for this project. Wait for the next repository update.
