# Arduino 3D Scanner

## Project Overview / Goal

For this project I worked with Matthew Ma and Steven Shi, and the goal was to build a simple 3D scanning system using an arduino uno, ultrasonic distance sensor, and servo motor. Instead of measuring distance from only one direction, we put the sensor on the servo so it could scan across different angles. The main idea was to connect distance with position. Each measurement records both the angle of the sensor and how far away the object is. By collecting many of these measurements, the scanner can then begin to map the shape and location of objects around it.

## Engineering / Development Process

We tested the motor movement and distance sensor separately before combining them into one system. In our final prototype, the ultrasonic sensor sits on top of the servo and sweeps back and forth across the scanning area. At each angle, the sensor sends out an ultrasonic pulse. This pulse reflects off the object being detected and returns to the sensor. The arduino measures the time it takes for this and uses the speed of sound to calculate the distance. The scanner then records the angle and distance together. An example that can explain myself better is (130, 40), which means that the sensor detected an object about 40 cm away while pointing at 130 degrees. Thus, repetition of this at different angles creates a basic map of the surrounding space and an idea to work with. 

We also learned that engineering affects the results significantly. If the basic wiring is incorrect then the code doesn’t run, and if the sensor is loose or still moving when it measures, the readings can become inaccurate and imprecise. Therefore, aligning the sensors, keeping track of motor movement and timing were all important parts of getting the scanner to work consistently. 

## Final Prototype Demonstration

Here is the 3D scanner working:

<video width="700" controls>
  <source src="img-5423_xHzhEUZ8 (1).mp4" type="video/mp4">
</video>

## My Initial Code

```cpp
#include <Servo.h> 
 
Servo s; 
int trig = 7, echo = 6; 
 
void setup() { 
  Serial.begin(115200); 
  s.attach(9); 
  pinMode(trig, OUTPUT); 
  pinMode(echo, INPUT); 
} 
 
void loop() { 
  for (int a = -90; a <= 90; a += 2) { 
    s.write(a + 90); 
    delay(50); 
 
    digitalWrite(trig, LOW); 
    delayMicroseconds(2); 
    digitalWrite(trig, HIGH); 
    delayMicroseconds(10); 
    digitalWrite(trig, LOW); 
 
    long t = pulseIn(echo, HIGH); 
    float d = t * 0.0343 / 2; 
 
    Serial.print(a); 
    Serial.print(",");
    Serial.println(d); 
  } 
}
```


## Code Explanation

The code controls the servo and ultrasonic sensor by making them work together. It moves the sensor through different angles, waits for it to settle quickly, measures the distance, and sends the angle and distance to the computer. This process repeats across the scanning range and then reverses, resulting in a fast and continuous back-and-forth scan.

## Reflection / Next Steps

This project helped me more clearly understand that scanning is both about distance and position. The servo position with the sensor reading together is what makes the data useful for mapping an object. If I continue the project, I will also add a second direction of movement so that the scanner can collect multiple layers instead of one horizontal scan. Those layers could eventually be combined into a 3D model of an object. It might even pick up texture or detect crevices to improve accuracy. 
