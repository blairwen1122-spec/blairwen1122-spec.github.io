
# Video game control project

## Project Overview

For this project, I expanded on my original Metro Runner game that was created using HTML, CSS, and JavaScript. The game was inspired by Subway Surfers, where the player runs along three lanes while avoiding obstacles (barriers, and overhead signs). The player has the choice to  move left or right, jump, or slide. The game becomes progressively faster as the score increases. HTML Canvas codes for the graphics, and JavaScript is used to control the player's movement, detecting collisions, generating obstacles,  and keeping track of scoring.

Since I had already worked with Arduino in class, I used an Arduino Uno and a joystick. My goal was to make the joystick control the same movements as the arrow keys without having to rebuild the entire game.

**Connecting the Joystick to the Arduino**

I asked ChatGPT which components I needed and how to connect them through wiring.

| **Joystick Pin** | **Arduino Pin** | <br> |
| ---------------- | --------------- | --------------------------- |
| GND | GND | Ground connection |
| +5V | 5V | Power supply |
| VRx | A0 | Reads left and right |
| VRy | A1 | Reads up and down |
| SW | D2 | Detects the joystick button |

Initially, I was confused about where D2 was located on the Arduino Uno, so I sent ChatGPT a picture of my board and asked it to identify the pin. It told me that D2 refers to digital pin 2, not analog pin A2, so then connected the joystick's SW pin to D2 to  use the built-in button to pause the game.

## Arduino Code

After wiring, ChatGPT helped me write an Arduino program that would translate the joystick's movements into directions the computer could recognize.

Code:

```cpp
const int xPin = A0;

const int yPin = A1;

const int buttonPin = 2;



const int LOW_LIMIT = 350;

const int HIGH_LIMIT = 700;



String lastDirection = "CENTER";

bool lastButtonPressed = false;

unsigned long lastButtonTime = 0;



void setup() {

  Serial.begin(9600);

  pinMode(buttonPin, INPUT_PULLUP);

}



void loop() {

  int x = analogRead(xPin);

  int y = analogRead(yPin);

  bool buttonPressed = digitalRead(buttonPin) == LOW;



  String direction = "CENTER";



  if (x < LOW_LIMIT) {

    direction = "LEFT";

  } else if (x > HIGH_LIMIT) {

    direction = "RIGHT";

  } else if (y < LOW_LIMIT) {

    direction = "UP";

  } else if (y > HIGH_LIMIT) {

    direction = "DOWN";

  }



  if (direction != lastDirection) {

    if (direction != "CENTER") {

      Serial.println(direction);

    }

    lastDirection = direction;

  }



  if (buttonPressed && !lastButtonPressed &&

      millis() - lastButtonTime > 200) {

    Serial.println("PAUSE");

    lastButtonTime = millis();

  }



  lastButtonPressed = buttonPressed;

  delay(15);

}
```

The code uses analogRead() to measure the joystick's position. Values below 350 or above 700 trigger a direction, while readings between those values are treated as neutral. lastDirection prevents repeated commands when the joystick is held in one position, and Serial.println() sends the direction to the computer at 9600 baud.

## Connecting the Joystick to the HTML Game

Something that made me frustrated was that the Arduino Uno could not directly control my browser game through its normal USB serial connection. When I asked ChatGPT about this, it told me to use Python to translate Arduino messages into keyboard presses.

I installed two Python libraries through Terminal:

```bash
python3 -m pip install pyserial pynput
```

`pyserial` receives data from the Arduino, while `pynput` simulates keyboard inputs.

ChatGPT then gave me a Python program, which I pasted into Sublime Text and saved on my Desktop as `joystick_bridge.py`.

```python
import time

import serial

from serial.tools import list_ports

from pynput.keyboard import Controller, Key



keyboard = Controller()



key_map = {

    "LEFT": Key.left,

    "RIGHT": Key.right,

    "UP": Key.up,

    "DOWN": Key.down,

    "PAUSE": "p"

}



ports = list(list_ports.comports())



for i, port in enumerate(ports):

    print(f"{i}: {port.device} - {port.description}")



choice = int(input("Enter Arduino port number: "))



with serial.Serial(ports[choice].device, 9600, timeout=0.1) as arduino:

    time.sleep(2)

    arduino.reset_input_buffer()



    while True:

        raw = arduino.readline().decode(

            "utf-8", errors="ignore"

        ).strip()



        if raw in key_map:

            key = key_map[raw]

            keyboard.press(key)

            keyboard.release(key)
```

The `key_map` dictionary connects the Arduino's messages to Metro Runner's keyboard controls. For example, when the joystick sends `LEFT`, Python simulates the left arrow key, and the character to move one lane left.

To run the program, I used:

```bash
cd ~/Desktop

python3 joystick_bridge.py
```

I then needed to select the Arduino's serial port, keep the Python program running, and open the original HTML game in Chrome. The Arduino IDE Serial Monitor had to be closed so Python could access the USB connection.

**Outcome and Challenge**

I ended up setting up the Arduino joystick circuit, writing the Arduino sketch, installed the Python libraries, and created the bridge program for my HTML metro runner game. The last step was confirming that all joystick movements worked in Chrome. The most frustrating part was connecting the hardware to the browser. At the beginning I expected the Arduino to directly connect with Metro Runner, but the game could only recognize keyboard inputs. Using Python solved this problem without needing changes to the original JavaScript.
