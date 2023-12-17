# 433RF-Duo-Arduino
Code for a 433MHz RF Wireless Transmitter and Receiver module.
I had a lot of issues trying to find help when I was first learning about these devices. 
This code specifically controls the position of a partial rotational servo by measuring 
the value outputted by a potentiometer.

Here are the RF units I used:
https://www.amazon.com/Wireless-Transmitter-Receiver-Arduino-Raspberry/dp/B08TMKKZ74/ref=asc_df_B08TMKKZ74/?tag=hyprod-20&linkCode=df0&hvadid=647314916867&hvpos=&hvnetw=g&hvrand=1227580353072077204&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9012405&hvtargid=pla-1968245714851&psc=1&mcid=7bbc66bb4d2e33b7b5518e571e90d424

and for the library to learn more please read: 
https://www.arduino.cc/reference/en/libraries/rc-switch/

# Code Explanation
### Transmitter code:
```ino
#include <RCSwitch.h>
```
Here is where we include the RCSwitch library. A library in the case of INO and CPP and many other coding languages, stores a series of predefined functions. In this case, the library is used for controlling RF devices.
```ino
RCSwitch mySwitch = RCSwitch();
```
This is where we declare an object that is named ```mySwitch``` under the class ```RCSwitch```, if you want to use an object in your code you must declare it first. This object will be used to interact with our RF transmitter.
```ino
void setup() {
  Serial.begin(9600);
  mySwitch.enableTransmit(10);
}
```
The ```setup()``` function is a standard Arduino function that is executed once when the microcontroller is powered on or reset.
```Serial.begin(9600)``` initializes serial communication at a baud rate of 9600. This is useful for debugging and printing messages to the serial monitor in the Arduino IDE, technically this is a redundant line of code, but it is extremely useful if you're wondering why nothing is happening.
```mySwitch.enableTransmit(10)``` configures the ```mySwitch``` object to use pin 10 as the transmitter pin. The RF transmitter is connected to this pin. ```.enableTransmit(...)``` is another ```RCSwitch``` function that is imported with the library.
```ino
void loop() {
  int value = analogRead(A1);
  value = map(value, 0, 1024, 0, 180);
  mySwitch.send(value, 30);
}
```
```loop()``` function is another standard Arduino function that runs continuously after the ```setup()``` function.

```int value = analogRead(A1);``` reads the analog voltage from pin A1. In this case a potentiometer, but could be anything like another sensor producing an analog signal.
```value = map(value, 0, 1024, 0, 180);``` maps the analog input value (ranging from 0 to 1023) to a new value ranging from 0 to 180. This is often done to convert a sensor reading to a range suitable for a specific application. As in this application, we are using a partial rotational servo. Which only goes 0 degrees to 180 degrees.
```mySwitch.send(value, 30);``` sends the mapped value (between 0 and 180) using the RCSwitch library. The second parameter, 30, represents the number of bits to send. This may need adjustment based on the requirements of the RF receiver. For my purposes, 30 worked well but each person's mileage may vary.

### Receiver code:
```ino
#include <Servo.h>
#include <RCSwitch.h>
```
Here are the necessary libraries. The ```Servo.h``` library is used for controlling servo motors, and ```RCSwitch.h``` is used for working with RF communication.
```ino
Servo servo;
RCSwitch mySwitch = RCSwitch();
```
These lines declare two objects: ```servo``` of the Servo class, which will be used to control the servo motor, and ```mySwitch``` of the RCSwitch class, which will be used to receive RF signals.
```ino
void setup() {
  Serial.begin(9600);
  mySwitch.enableReceive(0);
  servo.attach(3);
}
```
```Serial.begin(9600);``` initializes serial communication at a baud rate of 9600. This is used for debugging and sending messages to the serial monitor.
```mySwitch.enableReceive(0);``` configures the mySwitch object to enable RF signal reception on pin 0. The RF receiver is connected to this pin.
```servo.attach(3);``` attaches the servo motor to pin 3. The servo control signal will be sent to this pin.
```ino
void loop() {
  if (mySwitch.available()) {
    Serial.println(mySwitch.getReceivedValue());
    servo.write(mySwitch.getReceivedValue());
    mySwitch.resetAvailable();
  }
}
```
```if (mySwitch.available()) {``` checks if there is a valid RF signal available.
```Serial.println(mySwitch.getReceivedValue());``` prints the received value to the serial monitor. This is useful for debugging and understanding the values received from the RF transmitter.
```servo.write(mySwitch.getReceivedValue());``` sets the position of the servo motor based on the received value. The ```getReceivedValue()``` function retrieves the value received by the RF receiver, and it is assumed to represent the desired position of the servo motor.
```mySwitch.resetAvailable();``` resets the availability of the RF receiver, indicating that the signal has been processed.
