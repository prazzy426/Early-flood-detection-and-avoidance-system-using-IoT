# Early-flood-detection-and-avoidance-system-using-IoT
To detect the early flood and avoid the overflow of water from the dam.
#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
BlynkTimer timer;
#include <Servo.h>
#include "DHT.h"
#define DHTPIN D3
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
char auth[] = "xNtOJ8HCq8iRFbRw1alI1UYfYQ5sR_hq";
char ssid[] = "OnePlus";
char pass[] = "op123456";
int trig = D1;
int echo = D2;
int rain = A0;
int rainval;
double duration;
int distance;
Servo gate;
WidgetLED green(V1);
WidgetLED yell(V2);
WidgetLED red(V3);

void setup()
{
  Serial.begin(9600);
  Blynk.begin(auth, ssid, pass);
  timer.setInterval(1000,dam);
  timer.setInterval(3000,hum);
  dht.begin();
  pinMode(echo,INPUT);
  pinMode(trig,OUTPUT);
  pinMode(rain,INPUT); 
  gate.attach(D5);
}

void loop()
{
  Blynk.run();
  timer.run();
}
void dam()
{
  rainval = analogRead(rain);
  Blynk.virtualWrite(V4,rainval);
  
  digitalWrite(trig,LOW);
  delayMicroseconds(2);
  digitalWrite(trig,HIGH);
  delayMicroseconds(10);
  digitalWrite(trig,LOW);
  duration = pulseIn(echo,HIGH);
  distance = (0.034*duration)/2;
  
  if(rainval>0 && rainval<700)
  { 
    Blynk.virtualWrite(V0,"Heavy Rain");
    if(distance>0 && distance<5)
    { 
      red.on();
      green.off();
      yell.off();
      gate.write(90);
      Blynk.virtualWrite(V5,"Dam level is Full");
      
    }
    else if(distance>5 && distance<10)
    {
      red.off();
      green.off();
      yell.on();
      gate.write(0);
      Blynk.virtualWrite(V5,"Dam level is medium");
    }
    else if(distance>10)
    {
      red.off();
      green.on();
      yell.off();
      gate.write(0);
      Blynk.virtualWrite(V5,"Dam level is low");
    }
    else
    {
      red.off();
      green.on();
      yell.off();
      gate.write(0);
      Blynk.virtualWrite(V5,"Dam level is empty");
    }  
  }
  else if(rainval>700)
  { 
    Blynk.virtualWrite(V0,"Low Rain");
    if(distance>0 && distance<5)
    {
      red.on();
      green.off();
      yell.off();
      gate.write(90);
      Blynk.virtualWrite(V5,"Dam level is full");
    }
    else if(distance>5 && distance<10)
    {
      red.off();
      green.off();
      yell.on();
      gate.write(0);
      Blynk.virtualWrite(V5,"Dam level is medium");
    }
    else if(distance>10)
    {
      red.off();
      green.on();
      yell.off();
      gate.write(0);
      Blynk.virtualWrite(V5,"Dam level is low");
    }
    else
    {
      red.off();
      green.on();
      yell.off();
      gate.write(0);
      Blynk.virtualWrite(V5,"Dam level is empty");
    }
  }
  else
  {
    red.off();
    green.on();
    yell.off();
    gate.write(0);
    Blynk.virtualWrite(V5,"Dam level is empty");
  }
}  
void hum()
{
   float h = dht.readHumidity();
   float t = dht.readTemperature();
    
   if (isnan(h) || isnan(t))
   {
     Serial.println(F("Failed to read from DHT sensor!"));
     return;
   }
   Blynk.virtualWrite(V6,t);
   Blynk.virtualWrite(V7,h);
   delay(2000);
}
