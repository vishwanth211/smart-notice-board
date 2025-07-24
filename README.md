#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "BluetoothSerial.h"

BluetoothSerial SerialBT;
LiquidCrystal_I2C lcd(0x27, 16, 2);  // Adjust I2C address if needed

String message = "";
int scrollIndex = 0;
unsigned long lastScroll = 0;
const int scrollDelay = 400; // ms

void setup() {
  Serial.begin(115200);
  SerialBT.begin("SmartNoticeboard"); // Bluetooth device name
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("BT Ready!");
  delay(2000);
  lcd.clear();
}

void loop() {
  // Check for incoming BT message
  if (SerialBT.available()) {
    message = SerialBT.readStringUntil('\n');  // Read until newline
    message.trim();
    scrollIndex = 0; // Reset scroll
    lcd.clear();
  }

  // Scroll the message if too long
  if (message.length() <= 16) {
    lcd.setCursor(0, 0);
    lcd.print(message);
  } else {
    if (millis() - lastScroll > scrollDelay) {
      lcd.setCursor(0, 0);
      lcd.print(message.substring(scrollIndex, scrollIndex + 16));
      scrollIndex++;
      if (scrollIndex > message.length() - 16) scrollIndex = 0;
      lastScroll = millis();
    }
  }
}
