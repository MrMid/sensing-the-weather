[Previous lesson](../lesson3/README.md)

# Lesson 4: The anemometer

![](../../../images/anemometer.png)

## Introduction

In this lesson students will use the weather station expansion board and the anemometer. Students will firstly learn how the anemometer works, then Python code will be written to interface with it, detect its rotation and calculate the wind speed using a mathematical formula.

## Resources

Students should work in pairs. Each pair will require the following equipment:

- Raspberry Pi
- Weather Station Expansion Board
- Anemometer
- Micro USB power adaptor
- An SD Card with Raspbian already set up through NOOBS
- USB keyboard
- USB mouse
- HDMI cable
- A monitor or TV
- A small phillips screwdriver
- A ruler

## Learning Objectives

- Understand how the anemometer works
- Understand how to write code to interface with it
- Understand how to calculate wind speed from a count of rotations
- Understand what calibration is and why calibration is important

## Starter

Firstly ask everyone to pull the top off their anemometer, it goes back on just as easily. This is done by holding the base in one hand and pulling on the blades/cups with the other hand. It doesn't need much force to come off.

Look at the underside of the blades/cups and you'll see a small metal cylinder on one side. This is a magnet just like the one found on the bucket of the rain gauge. Test it with a paper clip if you like.

![](../../../images/anemometer_magnet.png)

Now use the screw driver to remove the three screws on the bottom of the base. The base should then pop out easily. Slide it down the cable about 10 to 20 cm to get it out of the way. Now if you look inside you'll see our old friend the reed switch again.

![](../../../images/anemometer_reed.png)

So what does this mean? When the blades/cups are in their original position and spinning the magnet will rotate in a tight circle above the reed switch. The magnet most influences the reed switch as it passes over the ends (where the gold wires come out). So for every complete rotation there will be two moments when the switch is closed.

So using a similar programming technique to the rain gauge we can count the number of interrupts and divide it by two to give us the number of complete rotations. We can then calculate the wind speed with some maths using π.

Reassemble the anemometer, put the base back into position and ensure the knot in the cable remains inside. Replace the three screws and push the blades/cups back onto the top. Give it a spin to check it rotates correctly.

## Main Development

### Setting up your Pi

1. Place the SD card into the slot of your Raspberry Pi.
1. Connect the Weather Expansion Board to the GPIO pins.
1. Connect the anemometer to the socket marked *WIND* on the Weather Expansion Board.
1. Next connect the HDMI cable from the monitor or TV.
1. Plug in the USB keyboard and mouse.
1. Plug in the micro USB power supply.
1. When prompted to login type:

    ```bash
    Login: pi
    Password: raspberry
    ```

### Detect the interrupts

1. We're going to carry on with interrupt detection from the previous lesson since this is more efficient than continuous polling. Remember the weather expansion board is *fixed* circuitry that you cannot change. So we need to write our code to accommodate the way it's wired up. The weather expansion board connects the anemometer to GPIO 17 in a *pull up* circuit.
1. Let's start a new program, enter the command below:

  `nano wind_speed.py`

1. Enter the code shown below:

    ```python
    #!/usr/bin/python
    import RPi.GPIO as GPIO
    
    pin = 17
    count = 0
    
    def spin(channel):
        global count
        count += 1
        print count
    
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)
    GPIO.add_event_detect(pin, GPIO.FALLING, callback=spin, bouncetime=0)
    
    raw_input("Press Enter to exit...")
    ```

  **Code walkthrough:**
  
  | Code | Meaning |
  | --- | --- |
  |`#!/usr/bin/python` | Denotes this file as a Python program.|
  |`import RPi.GPIO as GPIO` |  Imports the `RPi.GPIO` library.|
  |`pin = 17` | A reference variable to store the GPIO pin number connected to the anemometer.|
  |`count = 0` | Defines a variable that will be incremented by one when the anemometer reed switch closes.|
  |`def spin(channel):` | The `def` keyword is used to define your own functions. Here we define a function called `spin`. Lines of code that belong to this function are indented. This will be the call back function that runs when the anemometer reed switch closes. The function takes one parameter, `channel`, which is expected by the `RPi.GPIO` library.|
  |`global count` | This makes the `count` variable declared above available inside the scope of this function. Without this a new copy of the variable would be created locally just for this function and the main `count` variable would never change.|
  |`count += 1` | Incrementing the `count` variable by one. |
  |`print count` | Displays the count.|
  |`GPIO.setmode(GPIO.BCM)` | Sets the pin layout to match the diagrams that are part of this scheme of work.|
  |`GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)` | Enables the internal pull up resistor so that pin 17 always reads HIGH.|
  |`GPIO.add_event_detect(pin, GPIO.FALLING, callback=spin, bouncetime=0)` | This line is calling the `add_event_detect` function in the GPIO library to create the interrupt handler. This function takes four parameters. The GPIO pin number, the type of event (either `RISING`, `FALLING` or `BOTH`), the call back function and a bounce time in milliseconds. We pass in `FALLING` because it's a pull up circuit, when the pin is shorted to ground it goes from HIGH to LOW and therefore we want to detect the voltage `FALLING` from HIGH to LOW. The call back is the code we want to run when the interrupt occurs so here we pass in `spin`. The bounce time zero as opposed to 300 like last time. The anemometer is a rotary device and, unlike the rain gauge, has no parts that can bounce back when force is applied to them. A higher bounce time could cause desired counts to be ignored and we could then calculate the wind speed incorrectly (especially during a gale).|
  |`raw_input("Press Enter to exit...")` | The `raw_input` function is normally used to get text input from the user but here we are using it to hold up the program and prevent it from exiting. Pressing enter will release this function and cause the program to exit. |

1. Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit from nano.
1. Because this is a new file we need to mark it as executable before we run it. Enter the command below:

  `chmod +x wind_speed.py`

1. Run the code and remember to use the `sudo` command:

  `sudo ./wind_speed.py`

1. Hold the base of the anemometer in one hand and slowly rotate the blades/cups with the other. By doing this you can test that there are only ever *two* increments to the count for every complete rotation of the blades/cups. It can be helpful if you choose one cup and use it as a reference point for rotation (so move the one cup all the way round and back to the same position).

  ```
  1
  2
  3
  4
  5
  6
  ```

1. Press `Enter` to exit your program.

### How to calculate wind speed

Ask the class to think back to their [geometry](http://www.bbc.co.uk/schools/gcsebitesize/maths/geometry/circlesrev1.shtml) maths lessons.

Our overall goal is to calculate the wind speed, or rather the speed at which the anemometer cups are spinning. Speed is a measurement of distance over time, for example: 20 kilometres *per hour*. So to calculate wind speed we first need to measure how far the cups have travelled *in* a given block of time.

The time we measure for is essentially arbitrary. We could measure for 5, 10 or 60 seconds during which we keep counting the interrupts coming from GPIO 17. So *we* decide the time to measure for, let the interrupts occur so that the `count` variable increments via the `spin` function and then divide `count` by two to get the number of rotations (remember there are two interrupts per rotation). We then need to know the *distance* the cups have travelled and divide *that* by the time we chose. How can we get the distance from the number of rotations?

Since the cups rotate in circle we can use the anemometer circumference multiplied by the number of rotations to give us this distance. It can be helpful to imagine wrapping a tape measure around your waist, if you then hold the tape out straight that is the circumference of your waist as a distance. Now think of the anemometer like a wheel going along the ground, every rotation moves it forwards and increases the distance by the circumference. How can we calculate the circumference?

![](../../../images/pi_diagram.png)

The number π is a mathematical constant that is the ratio of any circles circumference to its diameter, π is approximately 3.14159. Using π we can calculate the circumference of a circle if we know its diameter or radius. Take a ruler and measure the radius of the anemometer now. Measure from the small depression in the top to the very edge of one of the cups. You should find it's about 9 cm.

The formula to calculate circumference from radius is: **2πr**

Once we know the circumference we can multiply it by the number of rotations (given by count ÷ 2) to get the *total* distance.

Speed is then distance ÷ time.

So our maths to get the wind speed will be as follows:

- Count anemometer interrupts for time length **t**.
- Calculate anemometer circumference **c** with 2πr, radius **r** is 9 cm (c = 2 × π × 9).
- Calculate rotations **rot** by dividing count by two (rot = count ÷ 2)
- Calculate total distance **d** by multiplying the circumference **c** by the rotations **rot** (d = c × rot)
- Calculate speed as total distance divided by time (speed = d ÷ t)

### Units of measurement

When doing these kinds of calculations we must pay attention to the units of measurement used. The above example uses the radius in *cm* to calculate a circumference. So **c** will also be in *cm* and therefore so will the total distance **d**. Assuming the time **t** was in seconds our speed would then be in *cm per second*, not *km per hour*.

To be able to give the speed in km per hour we need to do two things:

- Convert either the radius or the total distance into kilometres before dividing by time. 

  There are 100 centimetres in a meter, and 1000 meters in a kilometre. So if you take a value in cm and divide it by 100,000 (100 × 1000) the answer is the distance in kilometres (or as a fraction of a kilometre).
- Convert speed as distance per second into distance per hour.

  There are 60 seconds in a minute and 60 minutes in an hour, so if we multiply the distance per second by 3600 (60 × 60) we can convert to distance per hour.

### Program the calculation

1. Let's continue editing our program, enter the command below:

  `nano wind_speed.py`

1. Remember to add the new `import` line at the top. The `time` and `math` libraries are now needed. Change your code to match the code below:

    ```python
    #!/usr/bin/python
    import RPi.GPIO as GPIO
    import time, math
    
    pin = 17
    count = 0
    
    def calculate_speed(r_cm, time_sec):
        global count
        circ_cm = (2 * math.pi) * r_cm
        rot = count / 2.0
        dist_km = (circ_cm * rot) / 100000.0 # convert to kilometres
        km_per_sec = dist_km / time_sec
        km_per_hour = km_per_sec * 3600 # convert to distance per hour
        return km_per_hour
    
    def spin(channel):
        global count
        count += 1
        print count
    
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)
    GPIO.add_event_detect(pin, GPIO.FALLING, callback=spin, bouncetime=0)
    
    interval = 5
    
    while True:
        count = 0
        time.sleep(interval)
        print calculate_speed(9.0, interval), "kph"
    ```

  **Code walkthrough:**
  
  | Code | Meaning |
  | --- | --- |
  |`#!/usr/bin/python` | Denotes this file as a Python program.|
  |`import RPi.GPIO as GPIO` |  Imports the `RPi.GPIO` library.|
  |`import time, math` | Imports both the `time` and `math` libraries. Multiple libraries can be imported on the same line with a comma to separate them.|
  |`pin = 17` | A reference variable to store the GPIO pin number connected to the anemometer.|
  |`count = 0` | Defines a variable that will be incremented by one when the anemometer reed switch closes.|
  |`def calculate_speed(r_cm, time_sec):` | Defining a function called `calculate_speed` to perform the wind speed calculation and return the answer. It will take two parameters: the radius, `r_cm`, and the time in seconds, `time_sec`, that we have been counting for.|
  |`global count` | This makes the `count` variable declared above available inside the scope of this function.|
  |`circ_cm = (2 * math.pi) * r_cm` | Calculates the circumference using the *2πr* calculation. When you see brackets around part of a calculation like this it is instructing the computer to perform that part *first*. So this is saying multiply π by two, get the answer and multiply *that* by `r_cm`.|
  |`rot = count / 2.0` | Calculates the number of rotations. Simply divide the `count` of interrupts by decimal two.|
  |`dist_km = (circ_cm * rot) / 100000.0` | Calculates the total distance. Note the use of brackets again. Multiply the circumference by the number of full rotations and then divide by 100,000 to get the answer in kilometres.|
  |`km_per_sec = dist_km / time_sec` | Calculate the speed per second. Divide the distance in kilometres by the time in seconds.|
  |`km_per_hour = km_per_sec * 3600` | Convert distance per second into distance per hour by multiplying by 3600.|
  |`return km_per_hour` | Returns the `km_per_hour` variable as the result of the function.|
  |`def spin(channel):` | This will be the call back function that runs when the anemometer reed switch closes.|
  |`global count` | This makes the `count` variable declared above available inside the scope of this function.|
  |`count += 1` | Incrementing the `count` variable by one. |
  |`print count` | Displays the count.|
  |`GPIO.setmode(GPIO.BCM)` | Sets the pin layout to match the diagrams that are part of this scheme of work.|
  |`GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)` | Enables internal pull up resistor so that pin 17 always reads HIGH.|
  |`GPIO.add_event_detect(pin, GPIO.FALLING, callback=spin, bouncetime=0)` | Calling the `add_event_detect` function in the GPIO library to create the interrupt handler.|
  |`interval = 5` | This will be the time interval in seconds to count interrupts for before we attempt to calculate the speed.|
  |`while True:` | An infinite loop that must be manually aborted by the user.|
  |`count = 0` | On each iteration of this loop we should reset the interrupt count to zero, we want to see if the speed has gone up or down since last time so we need to discard the interrupt counts from the previous iteration.|
  |`time.sleep(interval)` | Pauses the execution of the code, in this loop only, for the number of seconds in the `interval` variable. Meanwhile the interrupt counting will continue in the background and the `count` variable will increase in value.|
  |`print calculate_speed(9.0, interval), "kph"` | Calls the `calculate_speed` function passing in the value 9.0 for the radius and `interval` for the time. The return value from `calculate_speed` is passed to the `print` command which shows it on screen along with the text `kph`.|

1. Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit from nano.

1. Run the code and remember to use the `sudo` command:

  `sudo ./wind_speed.py`

1. Start spinning the anemometer and every five seconds you should see a wind speed measurement display on the screen.

  ```
  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  11
  12
  13
  14
  15
  16
  17
  18
  19
  20
  21
  4.07150407905 kph
  ```
  
1. This is usually where students like to compete to see who has the best pair of lungs. It's very easy to make yourself feel faint by doing this so make sure everyone is warned to not be standing up while blowing on the anemometer.

1. Press `Ctrl - C` to exit your program.

### Calibration

Calibration is the practise of adjusting the value reported by a sensor to match a scientifically known value. In order to do this for the anemometer you would need a wind tunnel in your school where you could control the wind speed to an exact value. Since most of us don't have this available we'll need to make do.

Generally speaking anemometers tend to report the wind speed slightly lower than it actually is, a problem which gets progressively worse as the wind speed increases.

To compensate for this a calibration curve or lookup table can be used. This is typically information which is provided in the product [datasheet](https://www.argentdata.com/files/80422_datasheet.pdf) however the information given is very limited.

It only says: *A wind speed of 1.492 MPH (2.4 km/h) causes the switch to close once per second*.

In our code we have the line `interval = 5` which is used to define the length of time we count interrupts for before doing the speed calculation.  So, according to the datasheet, if we count 5 interrupts in 5 seconds the wind speed should come out as 2.4 kph. Let's investigate this.

1. Run the code and remember to use the `sudo` command:

  `sudo ./wind_speed.py`

1. You don't need to worry about getting the interrupts to occur at exactly one second intervals here, just spin the anemometer and stop it when the number reaches 5.

  ```
  1
  2
  3
  4
  5
  1.01787601976 kph
  ```
  Okay so this is wrong by about a half. What could be going on?
  The fault must be with one of the following:
  
  - The speed calculation is wrong
  - The datasheet doesn't match our anemometer

1. Since the maths is pretty basic my view is that the datasheet, or rather our interpretation of it, is wrong. It could be that the anemometer the datasheet refers to produces only one interrupt per complete rotation. Ours produces 2!

  This would mean that 10 interrupts would happen for 5 full rotations, so repeat the experiment again and this time stop spinning the anemometer when the number reaches 10.
  
  ```
  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  2.03575203953 kph
  ```
  
  That's a bit more like it. 
  
1. Press `Ctrl - C` to exit your program.
  
  We're now in a position to work out how to calibrate the result of the speed calculation. If we divide 2.4 (the number we expect) by the number we calculated the result will be the ratio of one to the other (in the same way that π is a ratio of the diameter of a circle to its circumference). We can then multiply future calculations by this ratio to calibrate them.
  
  For example, taking the kph value of 2.03575203953 from the above test we would do 2.4 ÷ 2.03575203953 = 1.178925504.
  
  1.178925504 is the ratio. We can round it up to *1.18*. 
  
1. So does 2.03575203953 × 1.18 = 2.4? Let's test it. First we need to change our code, enter the command below:

    `nano wind_speed.py`
  
    Find the `calculate_speed` function and modify the last line so that it reads `return km_per_hour * 1.18`.
    
    For example:
  
    ```python
    def calculate_speed(r_cm, time_sec):
        global count
        circ_cm = (2 * math.pi) * r_cm
        rot = count / 2.0
        dist_km = (circ_cm * rot) / 100000.0 # convert to kilometres
        km_per_sec = dist_km / time_sec
        km_per_hour = km_per_sec * 3600 # convert to distance per hour
        return km_per_hour * 1.18
    ```
    
1. Press `Ctrl - O` then `Enter` to save, followed by `Ctrl - X` to quit from nano.

1. Run the code and remember to use the `sudo` command:

  `sudo ./wind_speed.py`
  
1. Repeat the experiment again and stop spinning the anemometer when the number reaches 10, so again 5 complete rotations over 5 seconds.
  
  ```
  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  2.40218740664 kph
  ```
  
  There it is. The answer is approximately 2.4 but with a few extra decimal places of percision that we don't need to worry about. This is likely to be as correct as we're able to make it.

1. Spin the anemometer as fast as you like to see other calibrated values.
1. You can also experiment with measuring for different interval periods. Try changing the line `interval = 5` to another number, perhaps 10. If you keep the rotation speed of the anemometer constant the calculated speed should not change regardless of how long you measure for.
1. The anemometer is designed to be connected to the weather expansion board via the wind vane (wind direction sensor). You'll notice that the anemometer has quite a short cable whereas the wind vane has a long one. On the base of the wind vane there is a socket into which the anemometer can be connected. This then allows both wind vane and anemometer to be deployed at a distance together.
  - Unplug anemometer from the weather expansion board.
  - Plug the anemometer into the base of the wind vane.
  - Plug the wind vane into the socket marked *WIND* on the weather expansion board.
  - Test that your code still works with the anemometer connected in this way.
1. Press `Ctrl - C` to exit your program.

## Plenary

Ask the class the following questions.

1. Why we could not use a pull down circuit to detect the anemometer spinning?
1. Why is the bounce time zero milliseconds in our code?
1. What might happen if the bounce time was higher?
1. Why is calibration important?
1. Have we done enough to calibrate the anemometer?

**Answers:**

1. The weather expansion board has fixed circuitry that we cannot change. The rain gauge has two wires; one is hard wired to GPIO 17 and the other is hard wired to ground. Which means we can only short GPIO 17 to ground. If we used a pull down on GPIO 17 we would be shorting ground to ground and this would not produce a detectable change in the `HIGH` or `LOW` state of GPIO 17 when the anemometer spins. It would only ever read `LOW`.
1. The anemometer is a rotary device and, unlike the rain gauge, has no parts that can bounce back when force is applied to them. Its design makes it rotate in only one direction too, so there is no need for any de-bouncing.
1. At high wind speeds some interrupts could occur during the bounce time causing them to be ignored. This could produce an incorrect wind speed calculation.
1. Because we want to have confidence that our measurements are correct (or are at least within an acceptable tolerance).
1. We know that the higher the wind speed the further from correct the anemometer becomes. In order to compensate for this we would need different calibration ratios for different speeds. With the information provided by the datasheet we have done as much as we can.

[Next lesson](../lesson5/README.md)
