# Major-Project
#define BLYNK_PRINT Serial
#define BLYNK_TEMPLATE_ID "TMPL3GPT0YUC8"
#define BLYNK_TEMPLATE_NAME "SmartLock"
#define BLYNK_AUTH_TOKEN "yvlVqRlVf4KEodUSxAZC3FsiecRNj8cA"
#include <ESP8266_Lib.h>
#include <BlynkSimpleShieldEsp8266.h>
char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "siri";
char pass[] = "sirisha7";
#include <SoftwareSerial.h>
SoftwareSerial EspSerial(12, 11); // RX, TX
#define ESP8266_BAUD 9600
ESP8266 wifi(&EspSerial);
#include <LiquidCrystal.h>
#include <Keypad.h>
LiquidCrystal lcd (7, 6, 5, 4, 3, 2);
long randNumber;
String pass_key = "", auth_key = "";
char customKey;
const byte ROWS = 4;
const byte COLS = 3;
// Define the Keymap
char keys[ROWS][COLS] = {
  {'1', '2', '3'},
  {'4', '5', '6'},
  {'7', '8', '9'},
  {'*', '0', '#'}
};
byte rowPins[ROWS] = { A0,A1,A2,A3};
byte colPins[COLS] = { 8, 9, 10 };
// Create the Keypad
Keypad keypad = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );
int GLED = A4;
int buz = 13;
int relay = A5;
int attempts = 0;
void setup() {
  Serial.begin(9600);
  EspSerial.begin(ESP8266_BAUD);
  Blynk.begin(auth, wifi, ssid, pass);
  pinMode(GLED, OUTPUT);
  pinMode(buz, OUTPUT);
  pinMode(relay, OUTPUT);
  digitalWrite(GLED, HIGH);
  digitalWrite(buz, LOW);
  digitalWrite(relay, LOW);
  lcd.begin (16, 2);
  lcd.clear ();
  lcd.setCursor (4, 0);
  lcd.print (F("WELCOME "));
  lcd.setCursor (0, 1);
  lcd.print (F(" .............. "));
  delay(1000);
  lcd.clear();
  lcd.setCursor (0, 0);
  lcd.print (F(" OTP BASED DOOR "));
  lcd.setCursor (0, 1);
  lcd.print (F("  LOCK SYSTEM   "));
  delay (1000);
  lcd.clear();
}

void Send_OTP()
{
  randomSeed(analogRead(0));
  randNumber = random(1000, 9999);
  auth_key = String(randNumber);
  Blynk.virtualWrite(V0, auth_key);
  Serial.println(F(" "));
  Serial.print(F("OTP: "));
  Serial.println(auth_key);
  delay(1000);
  OTP_Validation();
}
void OTP_Validation()
{
  int k = 0;
  pass_key = "";
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Key: ");

  lcd.setCursor(5, 0);
  while (k < 4)
  {
    customKey = keypad.getKey();
    if (customKey) {
      lcd.write(customKey);
      delay(500);
      pass_key += customKey;
      k++;
      // lcd.clear();
    }
  }
  password_check();
}

void password_check()
{
  if (pass_key == auth_key)
  {
    lcd.clear();
    lcd.print("Authorized OTP  ");
    Blynk.virtualWrite(V1, 1);
    Blynk.virtualWrite(V2, 0);
    delay(2000);
    Blynk.virtualWrite(V1, 0);
    Blynk.virtualWrite(V2, 0);
    attempts = 0;
    digitalWrite(GLED, LOW); // LED ON
    digitalWrite(relay, HIGH); // Lock Open
    delay(2000);
    digitalWrite(GLED, HIGH); // LED OFF
    digitalWrite(relay, LOW); // Lock Close
    lcd.clear();
  }
  else
  {
    lcd.clear();
    lcd.print("Unauthorized OTP");
    Blynk.virtualWrite(V2, 1);
    Blynk.virtualWrite(V1, 0);
    delay(2000);
    Blynk.virtualWrite(V2, 0);
    Blynk.virtualWrite(V1, 0);
    digitalWrite(buz, HIGH);
    delay(2000);
    digitalWrite(buz, LOW);
    attempts++;
      lcd.clear();
      lcd.print("Trail: ");
      lcd.print(attempts);
      delay(1000);
      
      if (attempts >= 3)
      {
        lcd.clear();
        lcd.print("No Trails");
        delay(1000);
       attempts=0;
     //      while(1); // use this if you want to reset
     Send_OTP();
    }
    else
    {
      OTP_Validation();
    }
    
  }
}

void loop() {
  Blynk.virtualWrite(V1, 0);
  Blynk.virtualWrite(V2, 0);
  lcd.print("press * for OTP");
  delay(2000);
  lcd.clear();
  digitalWrite(GLED, HIGH);
  digitalWrite(buz, LOW);
  digitalWrite(relay, LOW);
  customKey = keypad.getKey();
  if (customKey == '*') {
    Send_OTP();

  }
  Blynk.run();
}
