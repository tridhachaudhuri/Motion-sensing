# Motion-sensing
This include the individual code for sensing motion using a PIR sensor.
/*
To check motion sensing using  a PIR sensor and sending a message to the person required stating that someone has come to the house
*/

#include<SoftwareSerial.h>
SoftwareSerial mySerial(9,10);

// Variable Setup
int calibrationTime = 10;        
long unsigned int lowIn;         //the time when the sensor outputs a low impulse
long unsigned int pause = 5000;  	//the amount of milliseconds the sensor has to be low
					//before we assume all motion has stopped
boolean lockLow = true;
boolean takeLowTime;  

int pirPin = 7;    //the digital pin connected to the PIR sensor's output
int ledPin = 8;	

/////////////////////////////
//SETUP
void setup(){
// Start Serial for debugging on the Serial Monitor
  mySerial.begin(9600);
  Serial.begin(9600);

  pinMode(pirPin, INPUT);
  pinMode(ledPin, OUTPUT);
  digitalWrite(pirPin, LOW);

  //give the sensor some time to calibrate
  Serial.print("calibrating sensor ");
    for(int i = 0; i < calibrationTime; i++){
      Serial.print(".");
      delay(1000);
      }
    Serial.println(" done");
    Serial.println("SENSOR ACTIVE");
    delay(50);
  }

////////////////////////////
//LOOP
void loop(){

     if(digitalRead(pirPin) == HIGH){
       digitalWrite(ledPin, HIGH);   //the led shows the sensors output pin state

       if(lockLow){  
         //makes sure we wait for a transition to LOW before any further output is made:
         lockLow = false;            
         Serial.println("---");
         Serial.print("motion detected at ");
         Serial.print(millis()/1000);
         Serial.println(" sec"); 
         Serial.print("Sending Message now.....");
         SendMessage();
         delay(50);
         }         
         takeLowTime = true;
       }

     if(digitalRead(pirPin) == LOW){       
       digitalWrite(ledPin, LOW);  //the led visualizes the sensors output pin state

       if(takeLowTime){
        lowIn = millis();          //save the time of the transition from high to LOW
        takeLowTime = false;       //make sure this is only done at the start of a LOW phase
        }
       //if the sensor is low for more than the given pause, 
       //we assume that no more motion is going to happen
       if(!lockLow && millis() - lowIn > pause){  
           //makes sure this block of code is only executed again after 
           //a new motion sequence has been detected
           lockLow = true;                        
           Serial.print("motion ended at ");      //output
           Serial.print((millis() - pause)/1000);
           Serial.println(" sec");
           delay(50);
           }
       }
       
  }
  void SendMessage()
  {
    if(Serial.available()>0)
      {
    Serial.println("ready...");
    mySerial.println("AT+CMGF=1");
  delay(1000);
  Serial.print("0");
  mySerial.println("AT+CMGS=\"+91xxxxxxxxxx\"\r");
  delay(1000);
  Serial.print("1");
  mySerial.println("Someone has come to your house."); //this following message is sent to the 
							//person’s phone.
  Serial.print("2");
  delay(100);
  Serial.print("3");
  mySerial.println((char)26);
  delay(1000);
   }
 }
