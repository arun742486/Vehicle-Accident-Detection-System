#include <LiquidCrystal.h>
 #include <Wire.h>
 #include <TinyGPS.h>
 #include <SoftwareSerial.h>
 #define xPin A1
 #define yPin A2
 //Initialize LCD
 LiquidCrystal lcd(7, 6, 5, 4, 13, 
12);
 // Take multiple samples to reduce 
noise
 const int samples = 10;
 unsigned long fix_age;
 SoftwareSerial GSM(2,3);
SoftwareSerial GPS(8,9);
 TinyGPS gps;
 void gpsdump(TinyGPS &gps);
 bool feedgps();
 void getGPS();
 long lat, lon;
 float LAT, LON;
 char inchar; // Will hold the 
incoming character from the GSM 
shield
 const int switch_Pin = 10;//Switch 
pin to stop message
 int in1 = A0;
int stop = 0;
 bool switch_status = LOW;
 int timer = 0u;
 char *phone_no[] = {
 "+917068022681", 
"+917017340926", 
};
 void setup(){
 pinMode(in1, INPUT);
 pinMode(switch_Pin, INPUT);
 Wire.begin();
 lcd.begin(16, 2);
 lcd.clear();  
lcd.print("ACCIDENT ");
 lcd.setCursor(0,1);
lcd.print("DETECTION SYSTEM");
 delay(500);
 GSM.begin(9600);
 Serial.begin(9600);
 Serial.println("Initializing....");
 initModule("AT","OK",1000);
 initModule("ATE1","OK",1000);
 initModule("AT+CPIN?","READY",
 1000);  
initModule("AT+CMGF=1","OK",
 1000);     
initModule("AT+CNMI=2,2,0,0,0","OK",
 1000);  
Serial.println("Initialized 
Successfully");
GSM.print("AT+CMGS=\"");
 GSM.print(phone_no[1]);
 GSM.println("\"\r\n");
 delay(1000);
 GSM.println("Welcome to Vehicle 
Accident Detection System");
 delay(300);
 GSM.write(byte(26));
 delay(3000); 
getGPS(); 
}
 void loop(){
 //Read raw values
 int xRaw=0,yRaw=0;
 for(int i=0;i<samples;i++)
 {
 xRaw+=analogRead(xPin);
 yRaw+=analogRead(yPin);
}
 xRaw/=samples;
 yRaw/=samples;
 //-------------------------------------------------------------
Serial.print(xRaw);
 Serial.print("\t");
 Serial.print(yRaw);
 Serial.println();
 //-------------------------------------------------------------
if((digitalRead(in1) == 
LOW)||(xRaw<276)||(xRaw>376)||(yRaw<
 276)||(yRaw>376))
 {
 lcd.clear();
lcd.print("Accident Detected");
 lcd.setCursor(0, 1);
 lcd.print("Wait for switch 
input");
 delay(500);
 lcd.clear();
 lcd.print("Timer Start");
 unsigned long startTime = 
millis();
 while (millis() - startTime < 
10000) {
 lcd.setCursor(0, 2);
 lcd.print((millis() - 
startTime) / 1000);
 }
 switch_status = 
digitalRead(switch_Pin);
 delay(100);
if (switch_status == LOW) {
 lcd.clear();
 lcd.print("Sending Message");
 delay(500);
 sms();
 } else {
 lcd.clear();
 lcd.print("Driver OK");
 lcd.setCursor(0, 1);
 lcd.print("Stop message");
 delay(500);
 }
 }
 if(GSM.available() >0){inchar=GSM.
 read();
 if(inchar=='R'){inchar=GSM.read(); 
if(inchar=='I'){inchar=GSM.read();
 if(inchar=='N'){inchar=GSM.read();
if(inchar=='G'){  
GSM.print("ATH\r");
 delay(1000);
 getGPS();     
GSM.print("AT+CMGS=\"");GSM.
 print(phone_no[1]);GSM.
 println("\"\r\n");
 delay(1000);
 GSM.println("RING Reply");
 GSM.print("http://maps.google.com/?
 q=loc:");
 GSM.print(LAT/1000000,7);
 GSM.print(",");
 GSM.println(LON/1000000,7);
 delay(300);
 GSM.write(byte(26));
 delay(5000);
 }
 }
}
 }
 }
 long lat, lon;
 unsigned long fix_age, time, 
date, speed, course;
 unsigned long chars;
 unsigned short sentences, 
failed_checksum;
 // retrieves +/- lat/long in 
1000000ths of a degree
 gps.get_position(&lat, &lon, 
&fix_age);
 }
void sms(){
 getGPS();     
GSM.print("AT+CMGS=\"");GSM.
 print(phone_no[1]);GSM.
 println("\"\r\n");
 delay(1000);
 GSM.println("Accident Detected..");
 GSM.println("Open the link given 
below to get the location:-");
 GSM.print("http://maps.google.com/?
 q=loc:");
 GSM.print(LAT/1000000,7);
 GSM.print(",");
 GSM.println(LON/1000000,7);
 delay(300);
 GSM.write(byte(26));
 delay(5000);  
}
void getGPS(){
 bool newdata = false;
 unsigned long start = millis();
 // Every 1 seconds we print an 
update
 while (millis() - start < 1000){
 if (feedgps ()){
 newdata = true;
 }
 }
 if (newdata){
 gpsdump(gps);
 }
 }
 bool feedgps(){
 while (GPS.available()){
 if (gps.encode(GPS.read()))
 return true;
 }
}
 return 0;
 void gpsdump(TinyGPS &gps){
 //byte month, day, hour, minute, 
second, hundredths;
 gps.get_position(&lat, &lon);
 LAT = lat;
 LON = lon;
 {
 feedgps(); // If we don't feed 
the gps during this long routine, 
we may drop characters and get 
checksum errors
 }
 }
 void initModule(String cmd, char 
*res, int t){
 while(1){
    Serial.println(cmd);
    GSM.println(cmd);
    delay(100);
    while(GSM.available()>0){
       if(GSM.find(res)){
        Serial.println(res);
        delay(t);
        return;
       }else{Serial.
 println("Error");}}
    delay(t);
  }
 }
