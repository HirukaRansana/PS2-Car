/******************************************************************
  Made By - HRAirlines (PVT) LTD.
  (HRAirlines - Hiruka Ransana Airlines)

  36012@bandaranayakecollege.com

  Folow me on Arduino Project Hub (hirukaransana)

  Website -

  This is the code of my car
  It is Controlled By the PlayStation2 (PS2) Game controller

  Infomation -
  Body - Designed by Hiruka Ransana
  Model - Vega ETX  (A Sri Lankan Car Company)
  Controller - PlayStation2


  Wireless camera (AV - Audio Video)
  Tranporting Any thing if that's wheigt less than 1.0KG
 ******************************************************************/



#include <PS2X_lib.h>       //for v1.6 PS2 Library

#include <Servo.h>


/******************************************************************
   set pins connected to PS2 controller:
     - 1e column: original
     - 2e colmun: Stef?
   replace pin numbers by the ones you use
 ******************************************************************/


#define PS2_DAT        12
#define PS2_CMD        11
#define PS2_SEL        10
#define PS2_CLK        9

#define leftm1     3
#define leftm2     4
#define rightm1    5
#define rightm2    6
#define left_EN    2
#define right_EN   7

#define redLight   22
#define blueLight  23
#define RBlight    24
#define horn       25
#define L298N      26
#define BreakLed   44


//LEDs for KnightRider Pattern
#define K1         30
#define K2         31
#define K3         32
#define K4         33
#define K5         34
#define K6         35

int Time = 900;           //Delay for light system

//analog values read from PS2 joysticks
int LX = 0;
int LY = 0;
int RX = 0;
int RY = 0;
/******************************************************************
   select modes of PS2 controller:
     - pressures = analog reading of push-butttons
     - rumble    = motor rumbling
   uncomment 1 of the lines for each mode selection
 ******************************************************************/
#define pressures   true
//#define pressures   false
#define rumble      true
//#define rumble      false

PS2X ps2x; // create PS2 Controller Class

Servo cameraServo;  // create servo object to control a servo
Servo cameraServo2;

int error = 0;
byte type = 0;
byte vibrate = 0;


void setup() {

  Serial.begin(9600);

  cameraServo2.attach(45);
  cameraServo.attach(46);

  pinMode(leftm1,  OUTPUT);
  pinMode(leftm2,  OUTPUT);
  pinMode(rightm1, OUTPUT);
  pinMode(rightm2, OUTPUT);
  pinMode(left_EN, OUTPUT);
  pinMode(right_EN, OUTPUT);

  pinMode(redLight, OUTPUT);
  pinMode(blueLight, OUTPUT);
  pinMode(RBlight, OUTPUT);
  pinMode(horn, OUTPUT);
  pinMode(BreakLed, OUTPUT);

  pinMode(K1, OUTPUT);
  pinMode(K2, OUTPUT);
  pinMode(K3, OUTPUT);
  pinMode(K4, OUTPUT);
  pinMode(K5, OUTPUT);
  pinMode(K6, OUTPUT);

  pinMode(L298N, OUTPUT);
  digitalWrite(left_EN, LOW);
  digitalWrite(right_EN, LOW);

  delay(300);  //added delay to give wireless ps2 module some time to startup, before configuring it


  //setup pins and settings: GamePad(clock, command, attention, data, Pressures?, Rumble?) check for error
  error = ps2x.config_gamepad(PS2_CLK, PS2_CMD, PS2_SEL, PS2_DAT, pressures, rumble);

  if (error == 0) {
    Serial.print("Found Controller, configured successful ");
  }
  else if (error == 1)
    Serial.print("No controller found, check wiring, see readme.txt to enable debug. visit www.billporter.info for troubleshooting tips");

  else if (error == 2)
    Serial.print("Controller found but not accepting commands. see readme.txt to enable debug. Visit www.billporter.info for troubleshooting tips");

  else if (error == 3)
    Serial.print("Controller refusing to enter Pressures mode, may not support it. ");

  Serial.print(ps2x.Analog(1), HEX);

  type = ps2x.readType();
  switch (type) {
    case 0:
      Serial.print("Unknown Controller type found ");
      break;
    case 1:
      Serial.print("DualShock Controller found ");
      break;
    case 2:
      Serial.print("GuitarHero Controller found ");
      break;
    case 3:
      Serial.print("Wireless Sony DualShock Controller found ");
      break;
  }
}

void loop() {

  if (ps2x.Button(PSB_START))
  {
    digitalWrite(L298N, HIGH);
    Serial.print("L298N Was Turned on");
    Serial.println("Now You Can Drive Your Car");
  }


  ps2x.read_gamepad(false, vibrate); //read controller and set large motor to spin at 'vibrate' speed

  delay(50);
  if (ps2x.Button(PSB_L1))
  {
    LY = ps2x.Analog(PSS_LY);                     //receive values from p22 joystick
    LX = ps2x.Analog(PSS_LX);
  }
  if (ps2x.Button(PSB_R1))
  {
    RY = ps2x.Analog(PSS_RY);
    RX = ps2x.Analog(PSS_RX);
  }

  if (LY > 200 || RY > 200)                       //check if the joystick pushed up side
  {
    REV();
  }
  if (LY < 100 || RY < 100)
  {
    forward();
  }
  if (LX < 100 || RX < 100)
  {
    left();
  }
  if (LX > 200 || RX > 200)
  {
    right();
  }
  if (LX == 128 && LY == 128 && RX == 128 && RY == 128)
  {
    waithere();
    knightRiderbreak();
  }

  LY = LX = 128;         //return to default vlaues
  RY = RX = 128;         //return to default values
}

void forward()                             //Driving Forward
{
  Serial.print("forward");
  digitalWrite(left_EN, HIGH);
  digitalWrite(right_EN, HIGH);

  digitalWrite(leftm1, HIGH);
  digitalWrite(leftm2, LOW);
  digitalWrite(rightm1, HIGH);
  digitalWrite(rightm2, LOW);

}
void REV()                                 //Reversing
{
  Serial.print("rev");
  digitalWrite(left_EN, HIGH);
  digitalWrite(right_EN, HIGH);

  digitalWrite(leftm1, LOW);
  digitalWrite(leftm2, HIGH);
  digitalWrite(rightm1, LOW);
  digitalWrite(rightm2, HIGH);
}
void left()                                //Halfly rotating left
{
  Serial.print("left");
  digitalWrite(left_EN, LOW);
  digitalWrite(right_EN, HIGH);

  digitalWrite(leftm1, LOW);
  digitalWrite(leftm2, LOW);
  digitalWrite(rightm1, HIGH);
  digitalWrite(rightm2, LOW);

}
void halfLeftfd()                          //Halfly turning left with going FORWARD
{
  digitalWrite(left_EN, 155);
  digitalWrite(right_EN, HIGH);

  digitalWrite(leftm1, HIGH);
  digitalWrite(leftm2, LOW);
  digitalWrite(rightm1, HIGH);
  digitalWrite(rightm2, LOW);

}
void halfLeftrev()                         //Halfly turning left with going REVERSE
{
  digitalWrite(left_EN, 155);
  digitalWrite(right_EN, HIGH);

  digitalWrite(rightm1, HIGH);
  digitalWrite(rightm2, LOW);
  digitalWrite(leftm1, HIGH);
  digitalWrite(leftm2, LOW);
}
void right()                               //Halfly rotating right
{
  Serial.print("right");
  digitalWrite(left_EN, HIGH);
  digitalWrite(right_EN, LOW);

  digitalWrite(rightm1, LOW);
  digitalWrite(rightm2, LOW);
  digitalWrite(leftm1, HIGH);
  digitalWrite(leftm2, LOW);
}
void halfRightfd()                         //Halfly turning right with going FORWARD
{
  digitalWrite(left_EN, HIGH);
  digitalWrite(right_EN, 155);

  digitalWrite(rightm1, HIGH);
  digitalWrite(rightm2, LOW);
  digitalWrite(leftm1, HIGH);
  digitalWrite(leftm2, LOW);
}
void halfRightrev()                        //Halfly turning right with going REVERSE
{
  digitalWrite(left_EN, HIGH);
  digitalWrite(right_EN, 155);

  digitalWrite(rightm1, LOW);
  digitalWrite(rightm2, HIGH);
  digitalWrite(leftm1, LOW);
  digitalWrite(leftm2, HIGH);
}
void waithere()                            //Stop
{
  Serial.print("stop");
  digitalWrite(left_EN, LOW);
  digitalWrite(right_EN, LOW);

  digitalWrite(rightm1, LOW);
  digitalWrite(rightm2, LOW);
  digitalWrite(leftm1, LOW);
  digitalWrite(leftm2, LOW);
}
void Horn1()
{
  Serial.print("Beep");
  digitalWrite(horn, HIGH);
  delay(500);
  digitalWrite(horn, LOW);
  delay(200);
  digitalWrite(horn, HIGH);
  delay(1500);
  digitalWrite(horn, LOW);
}
void Horn2()
{
  digitalWrite(horn, HIGH);
  delay(500);
  digitalWrite(horn, LOW);
  delay(500);
  digitalWrite(horn, 190);
  delay(500);
  digitalWrite(horn, LOW);
  delay(500);
}
void emergency()
{
  Serial.print("!!Emergency!!");
  digitalWrite(RBlight, HIGH);
  digitalWrite(redLight, HIGH);
  delay(100);
  digitalWrite(redLight, LOW);
  digitalWrite(redLight, HIGH);
  delay(400);
  digitalWrite(redLight, LOW);
  digitalWrite(blueLight, HIGH);
  delay(100);
  digitalWrite(blueLight, LOW);
  digitalWrite(blueLight, HIGH);
  delay(400);
  digitalWrite(blueLight, LOW);
}
void knightRiderbreak()
{
  digitalWrite(K1, HIGH);
  digitalWrite(K6, HIGH);
  delay(Time);
  digitalWrite(K1, LOW);
  digitalWrite(K6, LOW);

  digitalWrite(K2, HIGH);
  digitalWrite(K5, HIGH);
  delay(Time);
  digitalWrite(K2, LOW);
  digitalWrite(K5, LOW);

  digitalWrite(K3, HIGH);
  digitalWrite(K4, HIGH);
  delay(Time);
  digitalWrite(K3, LOW);
  digitalWrite(K4, LOW);
}


/******************************************************************


 ******************************************************************/
