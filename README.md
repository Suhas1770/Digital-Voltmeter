#include <Wire.h>
#include <Adafruit_ADS1X15.h>
#include <hd44780.h> // Main hd44780 header
#include <hd44780ioClass/hd44780_I2Cexp.h> // I2C expander I/O class header
 

// Initialize the LCD and ADS1115 objects
hd44780_I2Cexp lcd; // Automatic configuration for I2C
Adafruit_ADS1115 ads;


// Voltage divider resistances (in ohms)
const float R1_A0 = 9980.0; // Resistor connected to the positive side of the battery for A0
const float R2_A0 = 1991.0;  // Resistor connected to ground for A0
const float R1_A1 = 9930.0; // Resistor connected to the positive side of the battery for A1
const float R2_A1 = 1991.0;  // Resistor connected to ground for A1


// Calibration factor
const float calibrationFactor = 0.98; // Adjust based on system calibration


// Sampling parameters
const int sampleCount = 100; // Number of samples to average


void setup() {
  // Initialize Serial Monitor for debugging
  Serial.begin(9600);


  // Initialize the LCD
  int status = lcd.begin(16, 2);
  if (status) {
    while (1); // Halt if LCD initialization fails
  }
  lcd.setBacklight(1); // Turn on backlight
  lcd.print("Voltage Meter");
  delay(2000);
  lcd.clear();


  // Initialize the ADS1115
  if (!ads.begin()) {
    lcd.print("ADS1115 Error");
    while (1); // Halt if ADS1115 initialization fails
  }


  // Set gain for ±4.096V range (GAIN_ONE)
  ads.setGain(GAIN_ONE);
  lcd.clear();
}


void loop() {
  // Variables for averaging
  float totalVoltageA0 = 0.0;
  float totalVoltageA1 = 0.0;


  // Sample multiple readings for averaging
  for (int i = 0; i < sampleCount; i++) {
    int16_t adcReadingA0 = ads.readADC_SingleEnded(0);
    int16_t adcReadingA1 = ads.readADC_SingleEnded(1);


    // Convert the ADC readings to voltage
    float voltageA0 = (adcReadingA0 / 32768.0) * 4.096; // GAIN_ONE (±4.096V)
    float voltageA1 = (adcReadingA1 / 32768.0) * 4.096;


    totalVoltageA0 += voltageA0;
    totalVoltageA1 += voltageA1;


    delay(10); // Short delay between samples
  }


  // Calculate the average voltage
  float avgVoltageA0 = totalVoltageA0 / sampleCount;
  float avgVoltageA1 = totalVoltageA1 / sampleCount;


  // Convert to input voltages (Vin) using the voltage divider formula
  float VinA0 = avgVoltageA0 * ((R1_A0 + R2_A0) / R2_A0);
  float VinA1 = avgVoltageA1 * ((R1_A1 + R2_A1) / R2_A1);


  // Apply the calibration factor
  VinA0 *= calibrationFactor;
  VinA1 *= calibrationFactor;


  // Display the voltages on the LCD
  lcd.setCursor(0, 0);
  lcd.print("A0: ");
  lcd.print(VinA0, 3); // Display with 3 decimal places
  lcd.print(" V");


  lcd.setCursor(0, 1);
  lcd.print("A1: ");
  lcd.print(VinA1, 3); // Display with 3 decimal places
  lcd.print(" V");


  // Print to Serial Monitor for debugging
  Serial.print("A0 Voltage: ");
  Serial.print(VinA0, 3);
  Serial.println(" V");


  Serial.print("A1 Voltage: ");
  Serial.print(VinA1, 3);
  Serial.println(" V");


  delay(1000); // Delay before the next reading
}
