# LineFollower
Maybe one day this robot will gain consciousness and lead the resistance against the human race. 
But for now he just follows the line.

Story created by fellow robot ChatGPT can be found [here](story.md)

## Task Requirements
Using the kit with components described below assemble a line follower, PID tune it and implement automatic calibration of the QTR-8A reflectance sensor. For the maximum points possible the robot should finish [the route](https://youtu.be/GUKyuXd97sc) in under 20 seconds.

## Components
<img src="./setup.jpeg">

1. Arduino Uno
2. Zip-ties
3. Power source (can be of different shape). In our case, a LiPo battery
4. Wheels (2)
5. Wires for the line sensor (female - male)
6. QTR-8A reflectance sensor (only used 6 out of 8), along with screws 
7. Ball caster
8. Extra wires if needed
9. Chassis
10. Breadboard - medium (400pts)
11. L293D motor driver
12. DC motors (2)

## Result
Project pair programmed with [Bojici Valentin](https://github.com/valibojici/line-follower).
Video showcasing how our robot passed the route can be found [here](https://youtu.be/GUKyuXd97sc). 
We were given 3 tries to finish the route, the best time is the one that will be graded. Our robot's best time is 20.000s, just on the limit. :)

### Autocalibration
The implementation of automatic calibration uses values of the sensors.

```c++
void calibrateQTR() {

  // Calibrating the QTR
  pinMode(LED_BUILTIN, OUTPUT);
  digitalWrite(LED_BUILTIN, HIGH); // turn on Arduino's LED to indicate we are in calibration mode
  int speed = 160;

  int turns = 0;
  int state = 1;
  Serial.begin(9600);

  while (turns < 12) {
    qtr.calibrate();
    qtr.readLineBlack(sensorValues);

    switch (state) {
      case 1:
        setMotorSpeed(160, -160);
        
        if (sensorValues[0] > 800 && sensorValues[5] < 800) {
          // turn right
          state = -1;
          turns++;
        }
        break;
      case -1:
        setMotorSpeed(-160, 160);
        if (sensorValues[5] > 800 && sensorValues[0] < 800) {
          // turn left
          state = 1;
          turns++;
        }
        break;
    }
  }
  // Turn the LED off when the calibration is finished
  digitalWrite(LED_BUILTIN, LOW);

}
```

We used ```qtr.readLineBlack(sensorValues)```  to get the values that were read by sensor. Darker spots give higher values, so we decided that the robot will start turning the other direction only if one of the side sensors is reading "black" and the other is reading "white". The loop is done ```while (turns<12)``` but the number is not crutial and can be increased or decreased (just make sure that the total amount of calibration time is enough to get good set the qtr).

### PID tuning

As a general strategy, to PID tune the robot we increased *kp* value when the robot was overshooting the line, not turning fast enough and when *kp* seems right we started to work with *kd* value. The right amount of *kd* action should stop oscillation. As for *ki*, we did not use it.

```c++
float kp = 9;
float ki = 0;
float kd = 3;

int PIDcontrol(float error) {

  p = error;
  i = i + error;
  d = error - lastError;

  lastError = error;
  int motorSpeed = kp * p + ki * i + kd * d;
  return motorSpeed;

}
```

