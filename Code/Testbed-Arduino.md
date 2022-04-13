```
// Motor Setting
byte M_LED = 2;
byte M1_dirPin = 11;
byte M2_dirPin = 10;
byte M3_dirPin = 9;
byte M1_stepPin = 6;
byte M2_stepPin = 5;
byte M3_stepPin = 3;
byte Motor_ENB = 13;
// Switch Sensor Setting
int S1 = A0;
int S2 = A1;
int S3 = A2;
int S4 = A3;
int S5 = A4;
int S6 = A5;
int S[6] = {0, 0, 0, 0, 0, 0};
float s[6] = {0, 0, 0, 0, 0, 0};
// Action Requirement
float A[4] = {0.0, 0.0, 0.0, 0.0};
// Position
int P[3] = {0, 0, 0};
int Pu[3] = {0, 0, 0};
// Distance Counting
float Step_Rotation = 10000.0;
int M1_Steps = 0;
int M2_Steps = 0;
int M3_Steps = 0;
// Overshoot Flag
int Flag = 0;

void SensorValue(int S[])
{
  S[0] = analogRead(S1);
  S[1] = analogRead(S2);
  S[2] = analogRead(S3);
  S[3] = analogRead(S4);
  S[4] = analogRead(S5);
  S[5] = analogRead(S6);
}

void sensorvalue(float s[])
{
  SensorValue(S);
  float a = 0.3;
  s[0] = a*s[0] + (1-a)*S[0];
  s[1] = a*s[1] + (1-a)*S[1];
  s[2] = a*s[2] + (1-a)*S[2];
  s[3] = a*s[3] + (1-a)*S[3];
  s[4] = a*s[4] + (1-a)*S[4];
  s[5] = a*s[5] + (1-a)*S[5];
  }

void CurrentPosition(int P[])
{
  P[0] = M1_Steps;
  P[1] = M2_Steps;
  P[2] = M3_Steps;
}

void ActionValue(float A[])
{
  while(!Serial.available()) {}
  String readString,str[4];
  int count=0;
  while (Serial.available())
  {
    if (Serial.available() >0)
    { 
      char c = Serial.read();
      readString += c;
      str[count] +=c;
      if(c==','){count+=1;}
      if(c=='\n'){}
    }
  }
  A[0] = str[0].toFloat();
  A[1] = str[1].toFloat();
  A[2] = str[2].toFloat();
  A[3] = str[3].toFloat();
}

void UpdatePosition(int Pu[],int Flag)
{
  CurrentPosition(P);
  if(A[1]>300.0 or A[2]>480.0 or A[3]>80.0){Flag=1;} else {Flag=0;}
  int m1_step = int(A[1]/110.0*Step_Rotation);
  int m2_step = int(A[2]/110.0*Step_Rotation);
  int m3_step = int(A[3]/110.0*Step_Rotation);
  Pu[0] = m1_step-P[0];
  Pu[1] = m2_step-P[1];
  Pu[2] = m3_step-P[2];
}

void GotoPosition()
{
  UpdatePosition(Pu,Flag);
  if(Flag==0){
    int M1_sign,M2_sign,M3_sign;
    if(Pu[0]<0){digitalWrite(M1_dirPin, HIGH);M1_sign =-1;} else {digitalWrite(M1_dirPin, LOW);M1_sign=1;}
    if(Pu[1]<0){digitalWrite(M2_dirPin, LOW);M2_sign=-1;}   else {digitalWrite(M2_dirPin, HIGH);M2_sign=1;}
    if(Pu[2]<0){digitalWrite(M3_dirPin, HIGH);M3_sign=-1;}  else {digitalWrite(M3_dirPin, LOW);M3_sign=1;}
    int P_max = max(max(abs(Pu[0]),abs(Pu[1])),abs(Pu[2]));
    digitalWrite(M_LED, HIGH);
    for(int n=0;n<(P_max);n++)
    { 
      SensorValue(S);
      if(n<abs(Pu[0])){digitalWrite(M1_stepPin, HIGH);digitalWrite(M1_stepPin, LOW);M1_Steps += M1_sign; delay(0.000002);}
      if(n<abs(Pu[1])){digitalWrite(M2_stepPin, HIGH);digitalWrite(M2_stepPin, LOW);M2_Steps += M2_sign; delay(0.000002);}
      if(n<abs(Pu[2])){digitalWrite(M3_stepPin, HIGH);digitalWrite(M3_stepPin, LOW);M3_Steps += M3_sign; delay(0.000002);}
    }
    digitalWrite(M_LED, LOW);
  }
}

void SendPosition()
{
  float x_p,y_p,z_p;
  CurrentPosition(P);
  x_p = P[0]*110.0/Step_Rotation;
  y_p = P[1]*110.0/Step_Rotation;
  z_p = P[2]*110.0/Step_Rotation;
  Serial.print(3.0);
  Serial.print(" ");
  Serial.print(x_p);
  Serial.print(" ");
  Serial.print(y_p);
  Serial.print(" ");
  Serial.print(z_p);
  Serial.println(" ");
}


void action_in_arduino()
{
  ActionValue(A);
  if (A[0]==0.0){initialization();}
  if (A[0]==1.0){GotoPosition();}
  if (A[0]==2.0){SendPosition();}
}

void initialization()
{ 
  digitalWrite(M1_dirPin, HIGH);
  digitalWrite(M2_dirPin, LOW);
  digitalWrite(M3_dirPin, HIGH);
  digitalWrite(M_LED,HIGH);
  while(1)
  {
    sensorvalue(s);
    if(s[0]<400){digitalWrite(M1_stepPin, HIGH);digitalWrite(M1_stepPin, LOW);}
    if(s[2]<400){digitalWrite(M2_stepPin, HIGH);digitalWrite(M2_stepPin, LOW);}
    if(s[4]<400){digitalWrite(M3_stepPin, HIGH);digitalWrite(M3_stepPin, LOW);}
    if(s[0]>400 && s[2]>400 && s[4]>400){break;}
    delay(0.00001);
  }
  digitalWrite(M_LED,LOW);
  delay(1);
  
  digitalWrite(M1_dirPin, LOW);
  digitalWrite(M2_dirPin, HIGH);
  digitalWrite(M3_dirPin, LOW);
  digitalWrite(M_LED,HIGH);
  while(1)
  {
    sensorvalue(s);
    if(s[0]>400){digitalWrite(M1_stepPin, HIGH);digitalWrite(M1_stepPin, LOW);}
    if(s[2]>400){digitalWrite(M2_stepPin, HIGH);digitalWrite(M2_stepPin, LOW);}
    if(s[4]>400){digitalWrite(M3_stepPin, HIGH);digitalWrite(M3_stepPin, LOW);}
    if(s[0]<400 && s[2]<400 && s[4]<400){break;}
    delay(0.00001);
  }
  digitalWrite(M_LED,LOW);
  delay(1);

  digitalWrite(M1_dirPin, HIGH);
  digitalWrite(M2_dirPin, LOW);
  digitalWrite(M3_dirPin, HIGH);
  digitalWrite(M_LED,HIGH);
  while(1)
  { sensorvalue(s);
    if(s[0]<400){digitalWrite(M1_stepPin, HIGH);digitalWrite(M1_stepPin, LOW);}
    if(s[2]<400){digitalWrite(M2_stepPin, HIGH);digitalWrite(M2_stepPin, LOW);}
    if(s[4]<400){digitalWrite(M3_stepPin, HIGH);digitalWrite(M3_stepPin, LOW);}
    if(s[0]>400 && s[2]>400 && s[4]>400){break;}
    delay(0.000001);
  }
  digitalWrite(M_LED,LOW);
  M1_Steps = 0;
  M2_Steps = 0;
  M3_Steps = 0;
  delay(1);
}

void setup() {
  Serial.begin(115200);
  
  pinMode(M_LED,OUTPUT);
  pinMode(M1_dirPin, OUTPUT);
  pinMode(M2_dirPin, OUTPUT);
  pinMode(M3_dirPin, OUTPUT);
  pinMode(M1_stepPin, OUTPUT);
  pinMode(M2_stepPin, OUTPUT);
  pinMode(M3_stepPin, OUTPUT);
  pinMode(Motor_ENB,OUTPUT);
  
  digitalWrite(Motor_ENB, LOW);
  delay(10);
}

void loop() {
  action_in_arduino();
}
```
