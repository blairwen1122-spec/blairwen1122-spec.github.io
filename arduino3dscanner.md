
---
layout: default
title: Arduino 3D Scanner
---

# Arduino 3D Scanner — Innovator Journal

## Entry 1: Exploring the Scanning Mechanism

### Project Introduction

For our Tech and Innovation project, we are developing a 3D scanner that can measure the distance between a sensor and surrounding objects. Our long-term goal is to use these measurements to reconstruct the shape of a physical object.

We began by exploring a simpler two-dimensional scanning system before moving toward a more complex design. This allowed us to investigate the relationship between motor movement, distance sensing, and Arduino programming.

Our initial prototype involved three main components:

- **Arduino Uno:** Controls the system and processes sensor measurements.
- **SG90 Servo Motor:** Rotates the sensor through predetermined angles.
- **HC-SR04 Ultrasonic Sensor:** Measures distance using reflected sound waves.

### How the Initial Scanner Works

The basic idea is to rotate a distance sensor through different angles and record the distance measured at each position.

Our Arduino program was designed to move the servo from -90° to +90° in increments of 2°. At each position, the motor pauses briefly while the sensor collects a distance measurement.

The ultrasonic sensor works by emitting a sound pulse and measuring how long it takes for the echo to return.

We can calculate the distance using:

\[
d = \frac{t \times 0.0343}{2}
\]

Where:

- \(d\) represents distance in centimeters.
- \(t\) represents the echo time in microseconds.
- \(0.0343\) represents the approximate speed of sound in centimeters per microsecond.

We divide by two because the sound travels to the object and back.

By recording the angle alongside each distance measurement, we can begin determining where objects are located relative to the scanner.

### Challenges and Limitations

One limitation of this design is that it only collects measurements across a single plane. While this is useful for understanding the basic scanning mechanism, it cannot independently reconstruct the height or complete shape of an object.

Another concern is the timing between motor movement and sensor measurements. Even if the Arduino commands the motor to move to a specific angle, the motor may need additional time to physically reach that position.

This showed me that accurate scanning requires more than simply collecting distance measurements. The position of the sensor must also be reliable.

---

## Entry 2: Stepper Motor Development and Troubleshooting

### Improving the Mechanical System

As our project progressed, we began experimenting with stepper motors to achieve more precise control over the scanner's movement.

Unlike a conventional DC motor, a stepper motor rotates through individual steps. By controlling the number of electrical pulses sent to the motor driver, we can determine how far the motor should rotate.

This is particularly useful for a scanner because each measurement needs to correspond to a predictable position.

However, stepper motors also introduce additional electrical requirements. The Arduino cannot directly supply the current needed to operate the motor, so we used an **A4988-type stepper motor driver**.

The driver receives control signals from the Arduino while using an external power supply to operate the motor.

### Our First Circuit Failure

One of our biggest challenges occurred during our first attempt to power the stepper motor circuit.

We connected the components without first properly configuring the driver's current limit. When we supplied power, the driver began producing smoke, and several wires became extremely hot. Some of the insulation melted, forcing us to disconnect the circuit immediately.

We suspected that excessive current contributed to the failure, although incorrect wiring or another electrical problem could also have been responsible.

This experience revealed an important weakness in our initial approach. We had focused on connecting the components without fully understanding their electrical limitations.

I learned that successfully assembling a circuit does not necessarily mean that the circuit is safe or correctly configured.

### Understanding the A4988 Driver

To understand what went wrong, we examined the driver's different connections.

Our setup used two separate power supplies:

| Connection | Function |
|---|---|
| 12 V external supply | Provides power for the stepper motor |
| Arduino 5 V | Powers the driver's logic circuitry |
| GND | Establishes a shared electrical reference |
| STEP | Controls when the motor advances |
| DIR | Determines the direction of rotation |
| SLEEP | Controls the driver's low-power state |
| RESET | Resets the driver's internal step sequence |

The Arduino provides instructions, while the driver regulates current through the motor's coils.

One important detail is that the 12 V motor supply must remain separate from the Arduino's 5 V logic supply. Connecting these incorrectly could damage the electronics.

### Solving the Motor Control Problem

Even after correcting parts of our circuit, the motor initially refused to move.

We eventually discovered that the **SLEEP and RESET pins** needed to be configured correctly.

On an A4988-type driver, both inputs must be HIGH for normal stepping. In our setup, connecting these pins together helped resolve the problem.

This was an interesting discovery because the motor itself was not necessarily defective. A small control connection was enough to prevent the entire system from functioning.

It made me realize that troubleshooting requires understanding how individual components interact rather than assuming that the most obvious part is responsible.

### Adjusting the Reference Voltage

After replacing the damaged driver, we used a multimeter to adjust its reference voltage, or VREF.

This voltage is used to configure the driver's current limit. We adjusted our test setup to approximately **1.2 V**, with the multimeter displaying a reading close to **1.206 V**.

However, the correct VREF depends on the specifications of the motor and driver. It should not be treated as a universal setting.

We also encountered repeated zero readings while trying to measure the reference voltage.

After further troubleshooting, we discovered that one of the drivers was faulty. Replacing it allowed us to continue testing.

### Our First Successful Motor Test

After approximately two classes of troubleshooting, we successfully operated one stepper motor using an Arduino test program.

Although this did not mean that our scanner was complete, it was an important milestone.

We had moved from a circuit that was unsafe and nonfunctional to one capable of producing controlled movement.

### Reflection

The most important lesson from this stage was learning to troubleshoot systematically.

Initially, it was tempting to repeatedly change the wiring or code whenever the motor failed to respond. However, our experience showed that problems could originate from the power supply, driver configuration, faulty hardware, or control signals.

I realized that engineering is not simply about finding a solution that works. It is also about understanding why that solution works and identifying the cause when something fails.

---

## Entry 3: Investigating Infrared Distance Measurement

### Introducing the Sharp Infrared Sensor

Alongside our work with the stepper motor, we began investigating a **Sharp GP2Y0A21 infrared distance sensor**.

Unlike the ultrasonic sensor used in our original prototype, this sensor measures distance using infrared light and optical triangulation.

The sensor emits infrared light toward an object. When the light reflects back, its position on an internal detector changes depending on the object's distance.

The sensor converts this information into an analog voltage that can be read by the Arduino.

This introduces a new challenge because the voltage does not change linearly with distance.

### Connecting the Sensor to the Arduino

The infrared sensor has three primary wires:

| Sensor Wire | Arduino Connection |
|---|---|
| Red | 5 V |
| Black | GND |
| White | Analog input |

To prepare the connections, we carefully removed insulation from the ends of the wires and connected them to jumper wires.

We used electrical tape to secure the temporary connections.

Although this was sufficient for preparing our prototype, a more permanent design will require stronger connections. Loose wiring could interrupt the sensor signal, especially when the scanning mechanism begins moving.

### Understanding the Sensor's Measurements

The Sharp GP2Y0A21YK0F sensor has an intended measuring range of approximately **10–80 cm**.

Within this range, the output voltage generally decreases as the distance increases.

However, this relationship is nonlinear.

This means we cannot simply multiply the voltage by a constant to calculate distance accurately.

Instead, we will need to calibrate the sensor by recording voltage readings at known distances.

One possible approach is to use a power regression model to approximate the relationship between voltage and distance.

We can then compare the calculated distances with actual ruler measurements to evaluate the accuracy of our model.

### Why Calibration Matters

One thing I found particularly interesting is that receiving a sensor reading does not automatically mean that the measurement is accurate.

The Arduino might consistently read the same voltage, but our conversion into centimeters could still be incorrect.

This distinction is important because our final scanner will depend on the accuracy of thousands of individual measurements.

Even small errors could affect the shape of the reconstructed object.

### Sensor Orientation and Surface Properties

Another factor we need to consider is how different materials reflect infrared light.

Dark and light surfaces may reflect different amounts of infrared light, potentially influencing the sensor's measurements.

The sensor's orientation relative to an object's surface can also affect the results.

For this reason, we need to consider how the sensor will be mounted and how objects will move through its field of view.

A mechanically stable design will be important for maintaining consistent measurements.

---

## Entry 4: Reflection and Future Development

### Current Progress

At this stage, our project has made progress in two important areas: controlled mechanical movement and distance sensing.

We successfully operated one stepper motor after resolving several electrical problems, and we prepared the infrared sensor connections while investigating how its measurements could be interpreted.

However, these components still need to function together before we can produce a complete scanning system.

### Next Steps

Our next priorities include:

1. Testing the infrared sensor at known distances.
2. Developing a reliable voltage-to-distance conversion.
3. Confirming that the stepper motor moves consistently.
4. Integrating motor movement with distance measurements.
5. Experimenting with multiple motors to scan across different planes.
6. Designing stable sensor and motor mounts using laser cutting or 3D printing.

Once these components operate reliably together, we can begin collecting the spatial data needed for three-dimensional reconstruction.

### Overall Reflection

What has stood out to me most throughout this project is how closely mechanical design, electronics, and programming depend on one another.

Before beginning, it was easy to imagine a 3D scanner as a sensor that simply measures the shape of an object. In practice, developing even a basic prototype requires several systems to work together accurately.

Our difficulties with the stepper motor were frustrating, particularly because we spent so much time troubleshooting before achieving a successful result.

However, these challenges also helped us develop a deeper understanding of the technology.

I have come to appreciate that engineering progress is not always measured by how quickly a finished product takes shape. Sometimes, understanding why a component failed is just as valuable as getting it to work.

Moving forward, I want to focus on integrating our individual components into a coordinated system. The real challenge will be ensuring that every distance measurement corresponds to a known physical position.

Ultimately, this project has shown me that successful engineering requires patience, careful experimentation, and the ability to learn from unexpected results.

---

## Technical References

- [Pololu — A4988 Stepper Motor Driver](https://www.pololu.com/product/1182)
- [Allegro MicroSystems — A4988 Documentation](https://www.allegromicro.com/)
- [Sharp — Infrared Distance Sensor Documentation](https://global.sharp/)

---

## AI Use

I used ChatGPT to help organize my journal, explain technical concepts, and improve the clarity of my writing. The project progress and troubleshooting experiences are based on our group's documented work. AI assistance was used for writing and explanation, not for conducting physical experiments.
