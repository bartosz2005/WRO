**We have no cnc or 3D printing model

The program we had use is ardiuno

The camera we used is huskylens

The most detail technologic have will said at the other/document.pdf

The photo have been posted on the photo/20212220_17%.jpg

The team photo should be have two file,one it is funny another one is the normal one.It should be renamed by Funny_team_photo and Team_photo

This is the program of the AI vehicles, please copy it to the Arduino app and up load it into the vehicles.**


#define echoPin1 6 // attach pin 6 Arduino to pin Echo of HC-SR04
#define trigPin1 7 //attach pin 7 Arduino to pin Trig of HC-SR04
#define echoPin2 4 // attach pin 4 Arduino to pin Echo of HC-SR04
#define trigPin2 5 //attach pin 5 Arduino to pin Trig of HC-SR04
#define echoPin3 2 // attach pin 2 Arduino to pin Echo of HC-SR04
#define trigPin3 3 //attach pin 3 Arduino to pin Trig of HC-SR04
#include "HUSKYLENS.h"              //add Huskylens library
#include "SoftwareSerial.h"         //add seecond huskylens library
#include <Servo.h>                  //add servo motor library
#include <Adafruit_MotorShield.h>   //add motorshield library

long duration1, duration2, duration3; // variable for the duration of sound wave travel
int distance1, distance2, distance3; // variable for the distance measurement
int balance=0                        //variable for the forward walking system
HUSKYLENS huskylens;   
void printResult(HUSKYLENSResult result);
String incomingString;

SoftwareSerial mySerial(9, 11); // RX, TX                        //attach pin 9 for RX and 11 for TX,motorshield, dc motor, servo motor
Adafruit_MotorShield AFMS = Adafruit_MotorShield();              
Adafruit_StepperMotor *myStepper = AFMS.getStepper(200, 2);      
Adafruit_DCMotor *myMotor = AFMS.getMotor(1);                    
Servo servo1;

void setup() {
  pinMode(trigPin1, OUTPUT); // Sets the trigPin as an OUTPUT
  pinMode(echoPin1, INPUT); // Sets the echoPin as an INPUT
  pinMode(trigPin2, OUTPUT); // Sets the trigPin as an OUTPUT
  pinMode(echoPin2, INPUT); // Sets the echoPin as an INPUT
  pinMode(trigPin3, OUTPUT); // Sets the trigPin as an OUTPUT
  pinMode(echoPin3, INPUT); // Sets the echoPin as an INPUT
  servo1.attach(10);
  mySerial.begin(9600);
  while (!huskylens.begin(mySerial)) 
  {
    Serial.println(F("Begin failed!"));                                  //debug of huskeylens
    Serial.println(F("1.Please recheck the \"Protocol Type\" in HUSKYLENS (General Settings>>Protocol Type>>Serial 9600)"));
    Serial.println(F("2.Please recheck the connection."));
    delay(100);
  }
  Serial.begin(9600); // // Serial Communication is starting with 9600 of baudrate speed
}
void loop() { 

  digitalWrite(trigPin2, LOW);                  //take the data of ultrasonic senser 2
  delayMicroseconds(120);

  digitalWrite(trigPin2, HIGH);
  delayMicroseconds(1000);

  digitalWrite(trigPin2, LOW);

  duration2 = pulseIn(echoPin2, HIGH);

  distance2 = duration2 * 0.034 / 2; // Speed of sound wave divided by 2 (go and back)
  Serial.print("Distance from sensor 2: ");
  Serial.print(distance2);
  Serial.println(" cm");

  digitalWrite(trigPin1, LOW);                    //take the data of ultrasonic senser 3
  delayMicroseconds(120);

  digitalWrite(trigPin1, HIGH);
  delayMicroseconds(1000);

  digitalWrite(trigPin1, LOW);

  duration1 = pulseIn(echoPin1, HIGH);

  distance1 = duration1 * 0.034 / 2; // Speed of sound wave divided by 2 (go and back)
  Serial.print("Distance from sensor 1: ");
  Serial.print(distance1);
  Serial.println(" cm");
  digitalWrite(trigPin3, LOW);
  delayMicroseconds(120);
  digitalWrite(trigPin3, LOW);

  duration3 = pulseIn(echoPin3, HIGH);

  distance3 = duration3 * 0.034 / 2; // Speed of sound wave divided by 2 (go and back)
  Serial.print("Distance from sensor 3: ");
  Serial.print(distance3);
  Serial.println(" cm");
  digitalWrite(trigPin3, HIGH);
  delayMicroseconds(1000);

  
if(((distance3-distance2)=((distance2-distance3)))){
  servo1.write(90);
}

else if((distance3-distance2)>0){
  servo1.write(0);
}
else if((distance2-distance3)>0){
  servo1.write(180);
}


  if ((distance1 >= 0) and (distance1 < 35) and (abs(distance3-distance2) >= 35)) {
    if (!huskylens.request()) {
      incomingString = (F("Fail to request data from HUSKYLENS, recheck the connection!"));
      if (incomingString.length() > 2) {
        String messStr = "AT+SEND=0,";              // messStr(AT COMMAND) is to be sent to the LoRa module to send the relavant data
        messStr += (incomingString.length());
        messStr += ",";
        messStr += incomingString;
        Serial.println(messStr);
      }
    }
    else if (!huskylens.isLearned()) {
      incomingString = (F("Nothing learned, press learn button on HUSKYLENS to learn one!"));
      if (incomingString.length() > 2) {
        String messStr = "AT+SEND=0,";              // messStr(AT COMMAND) is to be sent to the LoRa module to send the relavant data
        messStr += (incomingString.length());
        messStr += ",";
        messStr += incomingString;
        Serial.println(messStr);
      }
    }
    else if (!huskylens.available()) {
      incomingString = "No block or arrow appears on the screen!";
      if (incomingString.length() > 2) {
        String messStr = "AT+SEND=0,";              // messStr(AT COMMAND) is to be sent to the LoRa module to send the relavant data
        messStr += (incomingString.length());
        messStr += ",";
        messStr += incomingString;
        Serial.println(messStr);
      }
    }
    else
    {
      while (huskylens.available())
      {
        HUSKYLENSResult result = huskylens.read();
        printResult(result);
      }
    }
    delay(500);
  }
}

void printResult(HUSKYLENSResult result) {
  if (result.command == COMMAND_RETURN_BLOCK) {
    incomingString = (String() + F("\nBlock:xCenter=") + result.xCenter + F("\nyCenter=") + result.yCenter + F("\nwidth=") + result.width + F("\nheight=") + result.height + F("\nID=") + result.ID);
    if ((result.ID='1') and (result.xCenter>=235)) {
      servo1.attach(10);
      delay(50);
      servo1.write(180);
      Serial.println("Right");
      servo1.detach();
    }
    if ((result.ID='1') and (result.xCenter<80)) {
      servo1.attach(10);
      delay(50);
      servo1.write(0);
      Serial.println("Left");
      servo1.detach();
    }
    if ((result.ID='1') and (result.xCenter>=80) and (result.xCenter<235)){
      servo1.attach(10);
      delay(50);
      servo1.write(90);
      Serial.println("Center");
      servo1.detach();
    }
  }
  else if (result.command == COMMAND_RETURN_ARROW) {
    incomingString = (String() + F("Arrow:xOrigin=") + result.xOrigin + F(",yOrigin=") + result.yOrigin + F(",xTarget=") + result.xTarget + F(",yTarget=") + result.yTarget + F(",ID=") + result.ID);
  }
  else {
    incomingString = ("Object unknown!");
  }
  if (incomingString.length() > 2) {
    String messStr = "AT+SEND=0,";              // messStr(AT COMMAND) is to be sent to the LoRa module to send the relavant data
    messStr += (incomingString.length());
    messStr += ",";
    messStr += incomingString;
    //Serial.println(messStr);
  }
}
