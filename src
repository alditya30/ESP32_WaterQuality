#define BLYNK_TEMPLATE_ID "TMPL6hz3aoqa7"
#define BLYNK_TEMPLATE_NAME "Monitoring"
#define BLYNK_AUTH_TOKEN "aKMRK9y78WZ32Q6pm-_y1WI6NzzLX38w"
#define BLYNK_PRINT Serial

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <HTTPClient.h>
#include <time.h>
#include <Arduino.h>
#include <LiquidCrystal_I2C.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <ESP32Servo.h>

const char* wifiSsid = "Wokwi-GUEST";
const char* wifiPassword = "";

const char* ntpServer = "pool.ntp.org";
const long  gmtOffset_sec = 25200;
const int   daylightOffset_sec = 0;

String GOOGLE_SCRIPT_ID = "AKfycbz2CsB3e6eblrClcVsanMFPev7iFulNsSDWlsA8V5aWXGPLGnQEPcqqI94D5A0xMX0u-Q";

const int DHTpin=13;
const int salinePin=32;
const int DOpin=33;

#define tempVpin V0
#define phVpin V1
#define salineVpin V2
#define doVpin V3

LiquidCrystal_I2C lcd(0x27,20,4);
DHT dhtSensor(DHTpin,DHT22);
Servo servoAddWater;
Servo servoRedWater;
Servo servoSaltWater; //to add salinity
Servo servoFreshWater; //to reduce salinity
const int servoAddWaterPin=18;
const int servoRedWaterPin=19;
const int servoSaltWaterPin=17;
const int servoFreshWaterPin=16;

float tempC;
float pH;
float saline;
float DO;
float analogSaline;
float analogDO;

float tempUpp=32;
float tempLow=27;
float saltUpp=21;
float saltLow=15;
int angle0;
int angle1;
int angle2;
int angle3;

const int sensorReadingInterval = 500;
unsigned long lastSensorReadingTime = 0;

float floatMap(float x, float in_min, float in_max, float out_min, float out_max) {
  return (x - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
}

// String getTime(){
  
// }

// void logData(temp, ph, saline, DO){
  
// }

void setup() {
  lcd.init();
  lcd.backlight();
  Serial.begin(115200);
  dhtSensor.begin();
  servoAddWater.attach(servoAddWaterPin);
  servoRedWater.attach(servoRedWaterPin);
  servoFreshWater.attach(servoFreshWaterPin);
  servoSaltWater.attach(servoSaltWaterPin);
  servoAddWater.write(0);
  servoRedWater.write(0);
  servoFreshWater.write(0);
  servoSaltWater.write(0);

  WiFi.begin(wifiSsid, wifiPassword);

  Blynk.begin(BLYNK_AUTH_TOKEN, wifiSsid, wifiPassword);

  // Init and get the time
  configTime(gmtOffset_sec, daylightOffset_sec, ntpServer);
}

void loop() {
  Blynk.run();
  lcd.clear();

  unsigned long currentTime = millis();

  bool shouldMeasure = (currentTime - lastSensorReadingTime) > sensorReadingInterval;

  if (shouldMeasure){
    lastSensorReadingTime = currentTime;
    analogSaline = analogRead(salinePin);
    analogDO = analogRead(DOpin);

    saline = floatMap(analogSaline, 0, 4095, 0, 50);
    DO = DO = floatMap (analogDO, 0, 4095, 0, 40);

    lcd.setCursor(0,0);
    int tpry = dhtSensor.readHumidity();
    pH = tpry % 14;
    tempC = dhtSensor.readTemperature();

    lcd.print("T:");
    lcd.print(tempC,2);
    lcd.print(" \xDF");
    lcd.print("C");

    lcd.setCursor(0,1);
    lcd.print("pH:");
    lcd.print(pH,2);

    lcd.setCursor(0,2);
    lcd.print("Salt.:");
    lcd.print(saline,2);

    lcd.setCursor(0,3);
    lcd.print("DO:");
    lcd.print(DO,2);
    lcd.print("mg/L");

    Blynk.virtualWrite(tempVpin, tempC);
    Blynk.virtualWrite(phVpin, pH);
    Blynk.virtualWrite(salineVpin, saline);
    Blynk.virtualWrite(doVpin, DO);

    if (tempC>tempUpp){
      angle0=90;
      angle1=0;
    } else if (tempC<tempLow){
      angle0=0;
      angle1=90;
    } else {
      angle0=0;
      angle1=0;
    }

    if (saline>saltUpp){
      angle2=90;
      angle3=0;
    } else if (saline<saltLow){
      angle2=0;
      angle3=90;
    } else {
      angle2=0;
      angle3=0;
    }
    
    servoAddWater.write(angle0);
    servoRedWater.write(angle1);
    servoFreshWater.write(angle2);
    servoSaltWater.write(angle3);

    if (WiFi.status() == WL_CONNECTED) {
    static bool flag = false;
    struct tm timeinfo;
    if (!getLocalTime(&timeinfo)) {
      Serial.println("Failed to obtain time");
      return;
    }
    char timeStringBuff[50]; //50 chars should be enough
    strftime(timeStringBuff, sizeof(timeStringBuff), "%A, %B %d %Y %H:%M:%S", &timeinfo);
    String asString(timeStringBuff);
    asString.replace(" ", "-");
    Serial.print("Time:");
    Serial.println(asString);

    String urlFinal = "https://script.google.com/macros/s/"+GOOGLE_SCRIPT_ID+"/exec?"+"time=" + asString + "&temp=" + String(tempC) + "&ph=" + String(pH) + "&saline=" + String(saline) + "&DO=" + String(DO);
    Serial.print("POST data to spreadsheet:");
    Serial.println(urlFinal);
    HTTPClient http;
    http.begin(urlFinal.c_str());
    http.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS);
    int httpCode = http.GET(); 
    Serial.print("HTTP Status Code: ");
    Serial.println(httpCode);
    //---------------------------------------------------------------------
    //getting response from google sheet
    String payload;
    if (httpCode > 0) {
        payload = http.getString();
        Serial.println("Payload: "+payload);    
    }
    //---------------------------------------------------------------------
    http.end();
    }
  }

}

