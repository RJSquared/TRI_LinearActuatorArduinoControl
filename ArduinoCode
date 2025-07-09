/*
Manually move the linear actuator until a desired pressure is reached.
Once a button is pushed to engage MainTainSetpoint control, the program
wll either extend or retract the linear actuator at a slow maintenance speed
to maintain the setpoint.

A0 Strain Meter In
A1 External Setpoint Control (0-5V)
A2 Manual Potentiometer Control (0-5V)
A3
A4
A5

D0
D1  
D2 IN Mode Switch, flip to go into MaintainMode. If INPUT_PULLUP, then flipping switch ON turns output from NORMAL HIGH to LOW
D3
D4
D5 OUT LPWM Retract Speed
D6 OUT RPWM Extend Speed
D7 OUT LPWM Enable
D8 OUT RPWM Enable
D9
D10
D11
D12 IN Emergency Stop Signal
D13 OUT Emergency Stop Signal
*/

//VARIABLE Assignment
  int Speed;
  int strainMeterVal; //calculate Current pressure/force from meter
  int strainMeterSetPoint;//Set once at the beginning of the Mode switch to capture current value and use as set point to compare against current value
  int potValue;//Value from the potentiometer to control speed and direction in manual mode
  int stopRange=50; //full width range of measured ADC bit values from potentiometer, centered on 512,  
                    //which stops the actuator motion in manual control
  bool firstLoopBlock= false; //flips to TRUE after the first MAINTAINFORCE loop. Used to initially record the setpoint value
  float reportingPeriod=1000;//The time period, ms, between printing information on the Serial Monitor
  float previousTime=0;//The previous timestamp from the Serial reporting period above.
  float currentTime=0;//First action of the main Loop function is to "look at the clock" and save it as currentTime
  int min_speed=1;//Minimum speed of actuator during MANUAL MODE

  //MaintainForce Variables
  int setPointWindow= 7;//full width range of setpoint values which are considered Maintained by the MAINTAIN FORCE loop
  int correctionSpeed=30;//The speed of the linear actuator when in MAINTAIN FORCE mode
  int motionPeriod=3;//How long the actuator extends/retracts at the correctionSpeed during Maintain Force Mode
  int stabilizePeriod=10;//how long to wait after motionPeriod for load cell signal to stabilize

//ANALOG Pin Number Assignment 
  int strain_meter_pin=0;
  int external_setpoint_pin=1;
  int pot_pin=2;
  
//DIGITAL Pin Number Assignments
  int e_STOP_OUT_pin = 13;
  int e_STOP_IN_pin = 12;
  int mode_pin = 2;
  int LPWM_pin = 5;  //connect Arduino pin 5 to IBT-2 pin LPWM
  int RPWM_pin = 6;  //connect Arduino pin 6 to IBT-2 pin RPWM
  int L_enable_pin = 7;
  int R_enable_pin = 8 ;


void setup() {
  //Digital Pin Mode Assignments
  pinMode(e_STOP_OUT_pin, OUTPUT);//Normal LOW. If the software needs to throw an Estop, 5V applied to e_STOP_OUT_pin
  digitalWrite(e_STOP_OUT_pin,LOW);//Initialize Estop pin to normal state=off=low
  pinMode(e_STOP_IN_pin, INPUT_PULLUP);//Normal HIGH. If the software needs to abort from external signal, connect to ground
  pinMode(mode_pin, INPUT_PULLUP); //5V, HIGH, is rest state=Manual Mode
  pinMode(LPWM_pin, OUTPUT);
  pinMode(RPWM_pin, OUTPUT);
  pinMode(L_enable_pin, OUTPUT);
  pinMode(R_enable_pin, OUTPUT);
  
  //Start Serial Communication
  Serial.begin(9600);
  Serial.println("Linear_actuator_setpoint_pulse");
  delay(2000);
}

void loop(){ /*We will have two states: One for manual position/force control, and then a second
  for maintaining the force when a switch is thrown.
  */
  
  currentTime=millis();
  
  strainMeterVal = analogRead(strain_meter_pin);//get the current value from the StrainMeter
  
  if(digitalRead(e_STOP_IN_pin)==LOW){  //Do nothing if the external ESTOP switch is engaged, Set Motion to STOP, bypasses the connected else statements below
    
    firstLoopBlock = false;

    potValue=analogRead(pot_pin);//get the current POT value so we know which way it will move when the switch is thrown
    
    if(potValue >= ((512) + (stopRange/2))){  //calculate SPEED variable to print so that we know the starting speed
        Speed = map(potValue, 512 + (stopRange/2), 1023, min_speed, 255);
    }

    else if(potValue <= ((512) - (stopRange/2))){ 
        Speed = map(potValue, 0, 512- (stopRange/2), 255, min_speed); 
    }

    else{ //speed=0
        Speed = 0;
    }


    digitalWrite(LPWM_pin,0);
    digitalWrite(RPWM_pin,0);
    if(currentTime-previousTime>=reportingPeriod){
      Serial.println("ESTOP INPUT Pin is LOW, program is doing nothing, executing first IF statement");
      Serial.println("Current value of Speed is:");
      Serial.println(Speed);
      previousTime=currentTime;
    }
  }
  
  else if(digitalRead(mode_pin)==LOW){ //Normal HIGH. If Mode Switch on DIO2 is connected to ground, then engage MAINTAIN CURRENT FORCE MODE
  
    if(currentTime-previousTime>=reportingPeriod){
      Serial.println("MODE PIN is LOW, program is doing MAINT CURRENT FORCE functon");
      Serial.println("");
    }

    MaintainCurrentForce();
  }
  else if(digitalRead(mode_pin)==HIGH){//Normal LOW, so enter MANUAL POSITION CONTROL. This is the main executed/default Loop action

    if(currentTime-previousTime>=reportingPeriod){
      Serial.println("MODE PIN is HIGH, program is doing ManualPositionControl functon");
      Serial.println("MODE Switch is OFF, MODE_PIN 2 not connected to ground");
      Serial.println("");
      previousTime=currentTime;
    }
    firstLoopBlock=false; //Reset the first loop gate block so that next time the Mode Switch is flipped it will enter the first loop and record the current force
    
    ManualPositionControl();
  }
}


//Functions

/*
----------------------------------------
---------------MANUALMODE---------------
----------------------------------------
*/

void ManualPositionControl(){//Read the input of a potentiometer, and slowly move 
                             //the actuator. Motion is stopped when Potentiometer
                             // reading is 512+/-stopRange 
  
      potValue = analogRead(pot_pin);       //read user input from the potentiometer
    
      if(potValue >= ((512) + (stopRange/2))){  //extension if PotPin reading is above theshold
        Speed = map(potValue, 512 + (stopRange/2), 1023, min_speed, 255);
        analogWrite(RPWM_pin, Speed);
        analogWrite(LPWM_pin, 0);
      }
    
      else if(potValue <= ((512) - (stopRange/2))){     //retraction if PotPin reading is below threshold
        Speed = map(potValue, 0, 512- (stopRange/2), 255, min_speed);
        analogWrite(RPWM_pin, 0);
        analogWrite(LPWM_pin, Speed);
      }
      
      else{ //No motion if in the "Dead Zone" of 512+/-stopRange
        Speed = 0;
        analogWrite(RPWM_pin, Speed);
        analogWrite(LPWM_pin, Speed);
      }
    }
  
/*
----------------------------------------
---------------MAINTAIN FORCE-----------
----------------------------------------
*/

void MaintainCurrentForce(){ //On the fist loop, use the current value of the StrainMeter as the setpoint while the Mode Switch is flipped
//Initially Read the value of the strain meter to lock in set point,
// and then move the actuator if the current reading is different than the captured set point.

//First LOOP:
//----------------------------------------------------------------------------------------------
//----------------------------------------------------------------------------------------------
  if(firstLoopBlock == false){//If this is the first time through the looop, Set the force setpoint
                           //after the Mode Switch is thrown
    
    strainMeterSetPoint = strainMeterVal;//Save the current pressure reading and make it the target set point
    
    firstLoopBlock = true;//set the gate blocker so this block doesn't execute on subsequent loops while the mode switch is engaged. 
    
    Serial.println("First loop of MAINTAIN FORCE has executed");
    Serial.println("");
    Serial.print("Strain Meter Set Point:");
    Serial.println(strainMeterSetPoint);
    Serial.println("");
    Serial.print("First Loop Block Value:");
    Serial.println(firstLoopBlock);
    Serial.println("");
    //delay(5000);
  }
//----------------------------------------------------------------------------------------------
//---------------------------------------------------------------------------------------------- 

  if (strainMeterVal >= strainMeterSetPoint-setPointWindow/2 && strainMeterVal<=strainMeterSetPoint+setPointWindow/2){
    //if the measured Strain Meter value is within the setPointWindow, don't move the actuator
    analogWrite(LPWM_pin, 0);//stop any reverse motion
    analogWrite(RPWM_pin, 0);//stp any forward motion

    if(currentTime-previousTime>=reportingPeriod){//reporting function
      Serial.println(currentTime);
      Serial.println(previousTime);
      Serial.println("");
      //delay(1000);
      Serial.print("Actuator Motion:");
      Serial.println("NONE");
      Serial.print("Strain Meter Set Point:");
      Serial.println(strainMeterSetPoint);
      Serial.print("Strain Meter Value:");
      Serial.println(strainMeterVal);
      previousTime=currentTime;
    }
  }
  
  else if(strainMeterVal<strainMeterSetPoint-setPointWindow/2){//Increase Pressure If Current Value is too small compared to set point
    analogWrite(RPWM_pin,correctionSpeed);
    analogWrite(LPWM_pin,0);
    delay(motionPeriod);
    analogWrite(RPWM_pin,0);
    delay(stabilizePeriod);

    if(currentTime-previousTime>=reportingPeriod){//reporting function
      Serial.print("Actuator Motion:");
      Serial.println("EXTEND");
      Serial.print("Strain Meter Set Point:");
      Serial.println(strainMeterSetPoint);
      Serial.print("Strain Meter Value:");
      Serial.println(strainMeterVal);
      previousTime=currentTime;
    }
  }
  
  else if(strainMeterVal>strainMeterSetPoint+setPointWindow/2){//Decrease Pressure If Current Value is too big compared to set point
    analogWrite(LPWM_pin,correctionSpeed);
    analogWrite(RPWM_pin,0);
    delay(motionPeriod);
    analogWrite(LPWM_pin, 0);
    delay(stabilizePeriod);

    if(currentTime-previousTime>=reportingPeriod){//reporting function
      Serial.print("Actuator Motion:");
      Serial.println("EXTRACT");
      Serial.print("Strain Meter Set Point:");
      Serial.println(strainMeterSetPoint);
      Serial.print("Strain Meter Value:");
      Serial.println(strainMeterVal);
      previousTime=currentTime;
    }
  }
  
}
