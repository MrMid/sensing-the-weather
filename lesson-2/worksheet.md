# Weather Station Basic I/O - Capturing input signals

In this lesson you will:

1. Create and execute a program to get the current state of pin 4 and display it on-screen
2. Use a `while` loop to repeat this **polling** of the pin and output the result
3. Add a delay to slow down the rate of polling
4. Adapt the program to use a pull down circuit
4. Explore the timings of the loop to get the ideal sensitivity

## Connecting a button
Before we start working with the weather station kit, we are going to ensure we can capture a simple input signal. To do this we need to connect a button to one of the GPIO pins.

1. Follow the [button guide](http://raspberrypi.org/guides/gpio/button) to connect a single button to **Pin 4**.

![Pull up wires](images/pull_up_wire.png)

## Sensing the input

2. Setup your Raspberry Pi and ensure you are in Desktop mode.

3. Launch the LXterminal window

![LX Terminal](images/lxterminal.png)

4. Create a directory by typing `mkdir weather_station` follow by `enter`

5. Create an empty program file called pullup.py by typing `sudo idle3 weather_station/pullup.py`

6. Enter the following code:

```python
#!/usr/bin/python3
import RPi.GPIO as GPIO
import time

pin = 4

GPIO.setmode(GPIO.BCM)
GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP

pin_value = GPIO.input(pin)
if pin_value == True:
  print ("HIGH")
else:
  print("LOW")
```

7. To run you code press the **F5** key on your keyboard, you will be asked to save, then an extra window will pop up and should print either `HIGH` or `LOW` depending on whether the button is pressed or not.

### Code explained:

  ```python
  #!/usr/bin/python3
  ```

This line denotes the file as a Python program so that the computer knows *how* to run the code.

  ```python
  import RPi.GPIO as GPIO
  import time
  ```

Imports the `RPi.GPIO` library that allows you to control the GPIO pins, and the `time` library to measure time or make the program sleep.

  ```python
  pin = 4
  GPIO.setmode(GPIO.BCM)
  GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)
  ```

These three lines set up a variable call pin to store the pin number. We then set the scheme for referring to our pins as the BCM scheme. The important bit is the third line, where we set up the pin as an input using a pull up resistor.

   ```python
   pin_value = GPIO.input(pin)
   ```

This line reads the state of the pin and stores the result in a variable called `pin_value`. This will either be *True* or *False*.

    ```python
    if pin_value == True:
      print ("HIGH")
    else:
      print("LOW")
    ```

These lines check the value of the `pin_value` variable, and will print out `HIGH` if the value is **True** and `LOW` if the value is **False**.

So now we can check the state of the button, but only **once**

## Repeatedly Polling
Now that we can check the state of the button we ought to check continously and report it's state. We can do this by **Polling** the pin every 0.5 seconds and constantly outputing the state.

For this we need to add an infinite loop to our code, in python we use a **while True:** statement and indent all the code that will be looped.

1. Change your program to put the last 4 lines inside a while loop and add a 0.5 second delay like this:

```python
while True:
  if pin_value == True:
    print ("HIGH")
  else:
    print("LOW")
  time.sleep(0.5)
```
2. Check your code against this [solution](code/pullup.py) to ensure it is correct.
3. Run you code again by pressing **F5**, to exit the program press `Ctrl + C` on your keyboard.
4. You should see `HIGH` when the button isn't pressed and `LOW` when it is pressed.

## Getting the timing right

So now you've managed to check the status of the input pin over and over, does it always work though?

1. Try running your program again and press the button continuously, can you press it fast enough that it fails to detect some presses.
2. The fact that the program only checks every 0.5 seconds means that some presses aren't detected.
3. Adjust the timing of the program by changing the number in the line
`time.sleep(0.5)`

Think carefully about whether you want to increase this number or decrease it, adjust it until it's sensitive enough.

## Using a pull down circuit

In our program we have used what's called a pull up resistor, which ensures the pin reads as `HIGH` until the button is pressed at which points the voltage drops and the pin reads `LOW` (For a more detailed explanation see the [Pull up / Pull down Guide](http://raspberrypi.org/guides/gpio/pull_up_down))

Let's see what the difference is when we use a pull down resistor.
1. From your pullup.py program in IDLE click the **file** menu and select **save as**, replace the file name with `pulldown.py`

2. Change the wiring of your button to a **pull down** circuit as shown here.
![Pull up wires](images/pull_down_wire.png)

3. Update the code to use a pull down circuit rather than pull up, this is a single change on the line that reads
`GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)`
which should become
`GPIO.setup(pin, GPIO.IN, GPIO.PUD_DOWN)`

your code should now look like [this](code/pulldown.py)

4. Run your pull_down code by pressing **F5** and see it working, what difference does a pull down circuit make?




Here we are going to use the internal pull up resistor to make GPIO 4 always read `HIGH`, then we will short it to ground through the wires so that it will read `LOW` when we touch the wires together.

*Note* : Pin numbers may be a little confusing at first, as there are a couple of numbering schemes which can be used. We will be using the **BCM scheme**. It may be worth having a pin reference sheet on the classroom wall or directing students to a guide such as [this](http://pi.gadgetoid.com/pinout). Some cases also have pin references on them.

Students should follow the worksheet to build their circuits and programs. There are a few points where you might wish to get the class to discuss what is happening.

The [worksheet](worksheet.md) follows these steps:

1. Create and execute a program to get the current state of pin 4 and display it on screen.
2. Use a `while` loop to repeat this **polling** of the pin and output the result.
3. Add a delay to slow down the rate of polling.
4. Adapt the program to use a pull down circuit.
4. Explore the timings of the loop to get the ideal sensitivity.
*****






Let's give it a try in practice.

## Reading an input pin

First we need to be able to make sure we can read the input from pin 4. We will start by building a **pull up** circuit.

Connect your wires / buttons as shown:

![Pull up wires](images/pull_up_wire.png)

1. Open LXTerminal from the menu bar:

  ![](images/lxterminal.png)

1. In LXTerminal, enter the command `nano pullup.py` and press Enter. 'nano' is a text editor program.
1. Enter the code below:

  ```python
  #!/usr/bin/python3
  import RPi.GPIO as GPIO
  import time

  pin = 4

  GPIO.setmode(GPIO.BCM)
  GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)

  pin_value = GPIO.input(pin)
  if pin_value == True:
    print ("HIGH")
  else:
    print()"LOW")
  ```



1. Press `Ctrl - O` then Enter to save, followed by `Ctrl - X` to quit nano.
1. Next, mark the file as executable with the following command:
`chmod +x pullup.py`

1. Type `sudo ./pullup.py` to test your code, ensuring that when the button is pressed you get the output `LOW` and `HIGH` if it is not pressed.

## Repeatedly polling the input pin

Currently, our code checks the pin status once and displays an appropriate output. Next, we are going to add a slight adaptation to make it check repeatedly or **poll** the pin.

1. Edit your code again. In LXTerminal enter the command `nano pullup.py` and press Enter.

1. You want to make the code simply check the pin over and over again until we stop the program. To do this we will wrap the main five lines of our program in a `while` loop. We will also add a pause to the program so that it doesn't check too often.

1. Adapt your code so that the last section looks like this (be careful to get the indentation correct):

  ```python
  GPIO.setup(pin, GPIO.IN, GPIO.PUD_UP)

    while True:           #This line tells the program to loop the following indented section
        pin_value = GPIO.input(pin)
        if pin_value == True:
          print ("HIGH")
        else:
          print("LOW")
        time.sleep(0.5)   #This line adds a 0.5 second pause between polls.
  ```

1. Press `Ctrl - O` then Enter to save, followed by `Ctrl - X` to quit nano.
1. Enter `sudo ./pullup.py` to run your code.
1. The text `HIGH` should begin scrolling up the screen. When you hold the wires together (close the switch) for a few seconds, you'll see the text `LOW` because you're shorting the pin to ground. Release the wires (open the switch) and it will return to `HIGH` because of the internal pull *up* resistor.

  ```
  HIGH
  HIGH
  HIGH
  HIGH
  LOW
  LOW
  LOW
  LOW
  HIGH
  HIGH
  HIGH
  HIGH
  ```

1. Press `Ctrl - C` to exit your program.

## Adjusting polling time

Now that our code constantly polls the input pin for its state, we need to think about timing.

1. Can you press the button fast enough that the program misses it?

1. Experiment with the line `time.sleep(0.5)`. Change the time so that it always detects your input. Try different times and find the biggest pause that still detects the input.

## Pull down circuit

1. Remove the jumper cables from the Raspberry Pi GPIO pins and reattach them as shown in the diagram below. Take care to select the correct pins.

  ![](images/pull_down_wire.png)

1. The code required to test the pull down circuit is almost identical to that for the pull up, so to save time we will just make a copy of your file and change one thing. Enter the command below (this takes a copy of `pullup.py` and saves it as `pulldown.py`):

  `cp pullup.py pulldown.py`

1. Enter the command below to edit the new file:

  `nano pulldown.py`

1. There is one line in the program that needs updating to reflect the change from a **pull up** to a **pull down**. Which line is it?

1. Update the program and press `Ctrl - O` then Enter to save, followed by `Ctrl - X` to quit nano.

1. The file doesn't need to be marked as executable with `chmod` since this property was copied from the original file. You can go ahead and run your code now, but remember to use `sudo`:

  `sudo ./pulldown.py`

1. The text `LOW` should begin scrolling up the screen. When you hold the wires together (close the switch) for a few seconds, you'll see the text `HIGH` because you're shorting the pin to 3.3 volts. Release the wires (open the switch) and it will return to `LOW` because of the internal pull *down* resistor.

  ```
  LOW
  LOW
  LOW
  LOW
  HIGH
  HIGH
  HIGH
  HIGH
  LOW
  LOW
  LOW
  LOW
  ```
1. Press `Ctrl - C` to exit your program.

## What's next?

In this lesson we have made our program respond to a single button press using a **pull up** and **pull down** setup.

- Are either of these circuits better than the other? Does it make a difference which one we use?
- In our code we used a button to print a simple statement. What else could you make it do?
- Could you connect multiple buttons to your Raspberry Pi and detect the states of each? Could you count the number of button presses?
