#include "Wire.h"
#define I2C_DEV_ADDR 0x5F

uint32_t i = 0;
unsigned int send_value = 2000;

void write_channel(byte device_address, unsigned int channel, unsigned int value) {
  unsigned int tft_value = 0;

  Wire.beginTransmission(device_address);
  Wire.write(1 + (channel * 2)); // Pointer Register
  tft_value = value >> 8;
  Wire.write(tft_value); //  Register [1] Config
  tft_value = value & 0xFF;
  Wire.write(tft_value); //  Register [1] Config
  Wire.endTransmission();
}

void config_dac(byte device_address, unsigned int c1, unsigned int c2, unsigned int c3, unsigned int c4) {
  unsigned int tft_value = 0;
  Wire.beginTransmission(device_address);
  Wire.write(0x01); // Pointer Register
  tft_value = c4 << 3;
  tft_value = tft_value | (c3 << 2);
  tft_value = tft_value | (c2 << 1);
  tft_value = tft_value | c1;
  Serial.print("BINARY PRINT  ");
  Serial.println(tft_value, BIN);
  Wire.write(tft_value); //  Register [1] Config
  Wire.endTransmission();
}

void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Wire.begin(16, 17);
  delay(200);
  SCAN();                 // Scan and Print I2C Address of connected devices
  delay(1000);

  config_dac(I2C_DEV_ADDR,0, 0, 1, 1); // Configure DAC as First two channels 0 - 10V and last two channels 4-20mA
  send_value = 2000;      // Set the initial ADC output value as 2000 / Output is 12bit / Maximum value is 4095
}

void loop() {
  config_dac(I2C_DEV_ADDR, 0, 0, 1, 1); // Configure DAC as First two channels 0 - 10V and last two channels 4-20mA
  for (int i = 1; i < 5; i++) {
    write_channel(I2C_DEV_ADDR, i, send_value); // Write channel sends the ADC value to the channel
    Serial.println(i);
    Serial.println(send_value);
    delay(100);
  }
  send_value = send_value + 100; // Increase output value by 100 each cycle.
  if (send_value > 4095)
    send_value = 0; // If the value exceeds 4095 , reset the value to zero

  delay(5000);
}

void SCAN() {
  byte error, address;
  int nDevices;
  Serial.println("Scanning...");
  nDevices = 0;
  for (address = 1; address < 127; address++) {
    Wire.beginTransmission(address);
    error = Wire.endTransmission();

    if (error == 0) {
      Serial.print("I2C device found at address 0x");
      if (address < 16)
        Serial.print("0");
      Serial.println(address, HEX);
      nDevices++;
    } else if (error == 4) {
      Serial.print("Unknown error at address 0x");
      if (address < 16)
        Serial.print("0");
      Serial.println(address, HEX);
    }
  }
  if (nDevices == 0)
    Serial.println("No I2C devices found");
  else
    Serial.println("done");
}
