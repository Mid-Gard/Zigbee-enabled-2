# Zigbee-enabled-2
Livestock, Weather, Soil

## Weather Statiom
Micro:Bit python code to send paramters from the micro:climate kit.

```
def on_temperature_higher_than():
    serial.write_line("[W1]Warning: High temperature detected! (>40 Deg Celsius)")
BME280.temperature_higher_than(40, on_temperature_higher_than)

def on_gesture_eight_g():
    serial.write_line("[W1]Warning: Heavy impact detected!")
input.on_gesture(Gesture.EIGHT_G, on_gesture_eight_g)

def on_button_pressed_a():
    serial.write_line("[W1]Switched to USB")
    serial.redirect_to_usb()
    serial.set_baud_rate(BaudRate.BAUD_RATE9600)
    serial.write_line("[W1]Switched to USB")
input.on_button_pressed(Button.A, on_button_pressed_a)

def on_gesture_free_fall():
    serial.write_line("[W1]Warning: Free fall detected!")
input.on_gesture(Gesture.FREE_FALL, on_gesture_free_fall)

def on_gesture_tilt_left():
    serial.write_line("[W1]Warning:Tilt detected!")
input.on_gesture(Gesture.TILT_LEFT, on_gesture_tilt_left)

def on_gesture_six_g():
    serial.write_line("[W1]Warning: Mid impact detected!")
input.on_gesture(Gesture.SIX_G, on_gesture_six_g)

def on_button_pressed_b():
    serial.write_line("[W1]Switched to UART")
    serial.redirect(SerialPin.P15, SerialPin.P14, BaudRate.BAUD_RATE9600)
    serial.write_line("[W1]Switched to UART")
input.on_button_pressed(Button.B, on_button_pressed_b)

def on_gesture_shake():
    serial.write_line("[W1]Warning: Rapid convulsions detected!")
input.on_gesture(Gesture.SHAKE, on_gesture_shake)

def on_gesture_three_g():
    serial.write_line("[W1]Warning: Low impact detected!")
input.on_gesture(Gesture.THREE_G, on_gesture_three_g)

weatherbit.start_weather_monitoring()
weatherbit.start_rain_monitoring()
weatherbit.start_wind_monitoring()
serial.redirect(SerialPin.P15, SerialPin.P14, BaudRate.BAUD_RATE9600)
BME280.address(BME280_I2C_ADDRESS.ADDR_0X76)
# basic.pause(1000)
# serial.write_line("W1=" + str(input.running_time() / 1000) + " " + str(weatherbit.temperature() / 100) + " " + str(weatherbit.humidity() / 1024) + " " + str(weatherbit.pressure() / 256)  + " " + str(weatherbit.altitude()))
# basic.pause(1000)
# serial.write_line("W2=" + str(weatherbit.soil_moisture()) + " " + str(weatherbit.soil_temperature() / 100))
# basic.pause(1000)
# serial.write_line("W3=" + str(weatherbit.wind_speed()) + " " + weatherbit.wind_direction() + " " + str(weatherbit.rain()))
# basic.pause(1000)
# serial.write_line("W4=" + str(input.acceleration(Dimension.X)) + " " + str(input.acceleration(Dimension.Y)) + " " + str(input.acceleration(Dimension.Z)) + " " + str(input.acceleration(Dimension.STRENGTH)))
# basic.pause(1000)
# serial.write_line("W5=" + " " + str(input.magnetic_force(Dimension.X)) + " " + str(input.magnetic_force(Dimension.Y)) + " " + str(input.magnetic_force(Dimension.Z)) + " " + str(input.magnetic_force(Dimension.STRENGTH)))
# serial.write_line(str(input.light_level()))
# str(input.light_level()))

def on_forever():
    serial.write_line("" + "\n" + "W1=" + ("" + str(input.running_time())) + " " + ("" + str(weatherbit.temperature() / 100)) + " " + ("" + str(BME280.humidity())) + " " + ("" + str(BME280.pressure(BME280_P.H_PA))) + " " + ("" + str(weatherbit.wind_speed())) + " " + weatherbit.wind_direction() + " " + ("" + str(weatherbit.rain())))
    serial.write_line("" + "\n" + "W2=" + ("" + str(input.acceleration(Dimension.X))) + " " + ("" + str(input.acceleration(Dimension.Y))) + " " + ("" + str(input.acceleration(Dimension.Z))) + " " + ("" + str(input.acceleration(Dimension.STRENGTH))) + " " + ("" + str(input.magnetic_force(Dimension.X))) + " " + ("" + str(input.magnetic_force(Dimension.Y))) + " " + ("" + str(input.magnetic_force(Dimension.Z))) + " " + ("" + str(input.magnetic_force(Dimension.STRENGTH))))
    basic.pause(1000)
basic.forever(on_forever)
```

## Soil Moisture Sensor and Water Pump Actuator

```
/* XBee Initialisation */
#include <SoftwareSerial.h> 
SoftwareSerial XBee(2, 3); // (RX, TX) 
/* DHT11 Initialisation */
#include <DHT11.h>
DHT11 dht11(5);

/* Water Pump Initialisation */
const int relayPump = 7; // Relay "In" --> Digital Pin 7
unsigned long startTime = 0; // Start time value
const int pumpTimeout = 10000; // Pump runtime (milliseconds)
const int offDuration = 3000; // Pump off-time (milliseconds)
int pumpState = 0; // Pump State (ON = 1, OFF = 0)
String req_char = 'M'; // Required character

/* Soil Moisture Initialisation */
int soilVal = 0; // soil moisture value stored here
int soilPin = A5;
int soilPower = 6;

bool motorNeedsToggle = false; // Flag for motor control

int humidity = 0;
int temperature = 0;
unsigned long lastTransmissionTime = 0;

void setup() {

  pinMode(relayPump, OUTPUT);
  digitalWrite(relayPump, HIGH);  


  pinMode(soilPower, OUTPUT);
  digitalWrite(soilPower, LOW);

  XBee.begin(9600);
  Serial.begin(9600);
}

// void controlMotor(char command)
// {
//   if (command == 'M') 
//   {
//     digitalWrite(relayPump, LOW);
//     Serial.println("[PUMP1] Motor Started");
//     startTime = millis();
//     pumpState = 1;
//   } 
//   else if (command == 'm') 
//   {
//     digitalWrite(relayPump, HIGH);
//     Serial.println("[PUMP1] Motor Stopped");
//     pumpState = 0;
//   } 
  //else {
  //Serial.println("[PUMP1] Received unknown character: " + String(command));
  //}
//}


void loop() {
// ######## MOTOR CONTROL ##########

  char incomingChar;

  // Read characters efficiently until 'm' is received
  while (true) 
  {
    incomingChar = Serial.read();
//    Serial.println(incomingChar);

    // Check for 'm' and handle motor control
    if (incomingChar == 'm') 
    {
      digitalWrite(relayPump, !digitalRead(relayPump));

      if (digitalRead(relayPump) == LOW) 
      {
        Serial.println("[MOTOR] Started");
        startTime = millis();
        pumpState = 1;
      } 
      else 
      {
        Serial.println("[MOTOR] Stopped");
        pumpState = 0;
      }
      break; // Exit loop after handling 'm'
    }
  }



  // while (true) 
  // {
  //   char incomingChar = Serial.read();
  //   Serial.println(incomingChar);
  //   // Check for 'M' or 'm' immediately upon reception
  //   if (incomingChar == 'M') {
  //     controlMotor('M');  // Motor ON
  //     break;  // Exit the loop after handling 'M'
  //   } else if (incomingChar == 'm') {
  //     controlMotor('m');  // Motor OFF
  //     break;  // Exit the loop after handling 'm'
  //   }



  //while (XBee.available()) {  // Read all available bytes
  //while (true) {

  // Serial.println("[SOILDEV1]");  // Print the received line for debugging
  // Serial.println(command);

  //String command = Serial.readStringUntil('\n');  // Read the entire line, name it "command"
//   String command = Serial.readStringUntil('\n');  // Read the entire line, name it "command"

//   if (command.startsWith("$"))  // Check for the starting pattern, S0=, for backend commands
//   { 
//     String action = command.substring(1);  // Exclude the first three characters of "command" string
// //      
// // PUMP START
// //
//     if (action == "1") 
//     {
//       digitalWrite(relayPump, LOW);  // Start the pump
//       Serial.println("[PUMP1] Pump Started");
//       startTime = millis();
//       pumpState = 1;
//     }
// //
// // PUMP STOP
// // 
//     else if (action == "0") 
//     {
//     digitalWrite(relayPump, HIGH);   // Stop the pump
//     Serial.println("[PUMP1] Pump Stopped"); 
//     pumpState = 0;
//     }
// //
// // ERROR CONDITION
// //
//     else 
//     {
//       Serial.print("[PUMP1] Error: Unidentified command: ");
//       Serial.println(command);  // Print the received line for debugging
//     }
//   }

// ######## SENDING DHT11, MQ2 AND SOIL MOISTURE DATA ########

  // Data transmission with 1-second interval


  
  while (millis() - lastTransmissionTime >= 1000) 
  {
    lastTransmissionTime = millis();
    // Read temperature and humidity
    int result = dht11.readTemperatureHumidity(temperature, humidity);
    // Send sensor data
    Serial.print("\nS1=");
    Serial.print(temperature);
    Serial.print(" ");
    Serial.print(humidity);
    Serial.print(" ");
    Serial.println(readSoil());
  }
  // SEE FUNCTION FOR SOIL MOISTURE POWER AND DATA AT THE END

  // DHT11
//   int humidity = 0;
//   int temperature = 0;
//   int result = dht11.readTemperatureHumidity(temperature, humidity);

// //  SENDING ALL DATA
//   Serial.print("\nS1=");
//   Serial.print(temperature); // Temperature
//   Serial.print(" ");
//   Serial.print(humidity); // Humidity
//   Serial.print(" "); 
//   Serial.println(readSoil()); // Soil Moisture value (End line)
//   delay(1000);

//  ########     TIMEOUT CODE BELOW   #########

//  (CurrentTime - StartTime) returns the current time value for the pump's cycle
  if (pumpState == 1) 
  {
    if (millis() - startTime >= pumpTimeout) //+ offDuration) 
    {
      digitalWrite(relayPump, HIGH); // Pump turns off (loops after 'Off Duration')
      Serial.println("[PUMP1] Pump Stopped (Timeout)");
      pumpState = 0;
    }
  }
}

//SOIL MOISTURE DATA
int readSoil() 
{
  digitalWrite(soilPower, HIGH);//turn D6 "On"
  //delay(10);//wait 1000 milliseconds 
  soilVal = analogRead(soilPin);//Read the SIG value form sensor 
  digitalWrite(soilPower, LOW);//turn D6 "Off"
  return soilVal;//send current moisture value
}




```


