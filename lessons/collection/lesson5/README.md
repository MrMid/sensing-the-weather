[Previous lesson](../lesson4/README.md)

# Lesson 5: The wind vane

![](../../../images/wind_vane.png)

## Introduction

In this lesson students will use the weather station expansion board and the wind vane. Students will firstly learn how the wind vane works and understand the differences between analogue and digital signals. Python code will be written to convert the analogue signal from the wind vane to a digital number using an ADC (Analogue to Digital Converter). This number will then be used to determine the direction the wind vane is pointing.

## Resources

Students should work in pairs. Each pair will require the following equipment:

- Raspberry Pi
- Weather Station Expansion Board
- Wind vane
- Micro USB power adaptor
- An SD Card with Raspbian already set up through NOOBS
- USB keyboard
- USB mouse
- HDMI cable
- A monitor or TV
- A small phillips screwdriver (magnetic if possible)

## Learning Objectives

- Understand how the wind vane works
- Be able to differentiate between analogue and digital signals
- Understand the purpose of an Analogue to Digital Converter
- Understand digital communication using the I²C protocol
- Determine the direction of the wind vane

## Starter

A wind vane shows the direction *from which* the wind is coming, not where it's going (this can be confusing because TV weather maps show the opposite). It works by the wind exerting force on a vertical blade which rotates to find the position of least resistance, this position is then aligned with the direction of the oncoming wind.

Also known as a weather vane these are often found as decorative adornments at the highest point on buildings or churches. Typically taking the shape of a cockerel, horse or ship.

### How it works

The wind vane is the most complex of the sensors studied so far. It does use reed switches and magnets but it works in a completely different way. Ask the students to pull the top off their wind vane, it should come off without much force. On the underside you'll again find the metal cylinder which is the magnet.

![](../../../images/wind_vane_magnet.png)

*Note: Disassembling the next part is only recommended for students with good motor skills. Younger students may have difficulty reassembling it.*

Next take the screwdriver and remove the three screws in the base, slide the base panel down the cable a bit to get it out of the way. If you look inside now you'll see there are eight reed switches arranged like the spokes of a wheel. Remove the three remaining screws allowing the circuit board to come free. Do not lose these screws.

![](../../../images/wind_vane_reed.png)

Examine the green side of the circuit board now, this is the side that faces upward inside the wind vane. The magnet points down onto it. North is at the top in the picture above, where the two black clips for the anemometer socket are.

Look closely and you'll see there is a ring of metal that goes all the way around the edge. There is also a smaller ring in the centre. Each reed switch connects to the outer ring to the inner ring through a resistor. You'll see that `SW1` (switch 1) has `R1` near it (resistor 1), similarly `SW2` has `R2` and so on up to 8.

So what is going on here? Firstly students need to understand what a [resistor](http://en.wikipedia.org/wiki/Resistor) is. These are small components that resist/reduce the flow of electrical current but do not stop it, at the same time they also reduce the voltage moving through the circuit. Resistors can have different values, a low resistance value would let almost all voltage/current through but a high resistance value would let very little through.

The wind vane is working like a big variable resistor, think of a volume knob. Look at the schematic diagram below (a zigzag line is the symbol for a resistor). The idea is that voltage comes in on the outer ring and can take a path through any of the switches to the inner ring which is connected directly to ground. As the magnet rotates different reed switches will open and close and thus switch their corresponding resistor in and out of the circuit.

![](../../../images/wind_vane_schematic.png)

Each of the 8 resistors have different values which you'll see printed in white text next to them, this then allows the wind vane to have 16 possible combinations of resistance since the magnet is able to close two reed switches when half way between them.

Reassemble the wind vane now. Firstly locate the letter N on the side of the base, insert the circuit board with the green side facing away from you so that the anemometer socket aligns with North. Replace the three smaller screws (this step can be tricky and a magnetic screwdriver helps a lot). Next replace the base ensuring the knot in the cable remains inside. Finally replace the three larger screws.

### How can we measure it?

So we now understand that the wind vane is essentially a variable resistor similar to a volume knob (but with only 16 positions). Resistance is something that we can't measure directly because it's a passive property of the wind vane. What we need to do is measure something that changes as a *consequence* of the resistance. Namely the *voltage* going through the wind vane. The voltage level passing through it will go up and down as different resistors are switched on and off by the magnet. That, we *can* measure.

This is going to be entirely different to what we have done before. With the rain gauge and the anemometer we were working with voltage levels changing between 0 volts meaning *LOW* and 3.3 volts meaning *HIGH*. Our code could only tell us if a GPIO pin was HIGH or LOW but not somewhere in between. This is what is known as a *digital* signal, all or nothing, 1 or 0, HIGH or LOW. For the wind vane we need to accommodate a range between HIGH and LOW, this is known as an *analogue* signal.

### Analogue vs Digital

It is important for us to understand the general concept of analogue and digital. Think of a gaming control pad like the one below. The circle is highlighting the thumb joystick and the directional-pad. Ask the class which one is analogue and which one is digital?

![](../../../images/xbone_pad.png)

**Answers:**

- Thumb joystick: Analogue
  
  The thumb joystick is analogue because it provides a full range of motion between each direction. In a driving game you have the option to steer gently around a long sweeping corner or hard around a hairpin for example.

- Directional-pad: Digital

  The directional-pad is digital because the each direction button has only two states, on and off. Just like HIGH and LOW. In a driving game it would be like steering a car using the indicator stick, you would have full left and full right only. It would be very tricky to control!

Analogue and digital both have their place and often one works better for a particular task than the other. For a game like a flight simulator you would want analogue control to aim the plane, whereas for something simple like a jump, run and shoot platform game digital control is better.

### Analogue to Digital Conversion

So to recap then. The wind vane has a voltage going through it and this will vary according to which resistors are switched in and out by the reed switches and magnet. The challenge we face is being able to observe this analogue signal changing on a computer which is basically a digital machine.

![](../../../images/adc_msop10.png)

To do this we're going to use a clever microchip called an [Analogue to Digital Converter](http://en.wikipedia.org/wiki/Analog-to-digital_converter) or ADC for short. The weather expansion board has one of these built in (as do most games consoles). An ADC chip, like the one above, has a number of input pins. One of them is connected to the voltage going through the wind vane. We don't need to worry about the internal workings of the chip we just need to understand that it can convert from a continuous analogue voltage to a number (in code) that represents the voltage *magnitude*. More voltage will give a higher number, less voltage a lower one.

The *resolution* of an ADC chip describes the number of unique readings that it can produce to represent this magnitude. The resolution value is given in *bits* (a bit is a binary 1 or 0). A ten bit ADC, for example, can produce values between 0 and 1023 (`1111111111` in binary = `1023` in decimal).

The ADC we're using has 15 bits of resolution. So this can produce values between 0 and 32767 in our code. That's quite a lot of resolution! However the wind vane has only 16 possible combinations of resistance that each represent a compass direction. This gives us 16 possible voltage levels to convert with the ADC. In turn this gives us only 16 possible readings from the ADC that lie somewhere between 0 and 32767. 

We therefore need to find out what those 16 numbers are and translate them into a compass direction in our code. A sensible option is to use degrees clockwise from North. This a better way to represent a compass angle on a computer (as opposed to the traditional North, South, East, West) because it allows for easy numeric comparison and or averaging.

### How do we talk to the ADC chip?

There is one more layer that we need to reveal.

Generally speaking most computer chips communicate by sending and receiving binary data. In order to have successful two way communication there needs to be set of rules (like a language) that both chips obey. In computing terms this is known as a *protocol*. The ADC chip on the weather expansion board uses the [I²C protocol](http://en.wikipedia.org/wiki/I%C2%B2C) for this (pronounced I-squared-C) which stands for Inter-Integrated Circuit. Several other chips on the weather expansion board also use it, such as the pressure and humidity sensors (covered in separate lessons). We're not going to go into the finer details of I²C but it is important that we understand the general principle so that the code we write makes sense later on.

![](../../../images/i2c-diagram.png)

With I²C there is usually one *master* device and several *slaves* that are all connected through a pair of wires known as the I²C *bus* (as above). One wire is for transferring data and the other is used for a timing signal. Often referred to as `DATA` *(SDA)* and `CLOCK` *(SCL)* respectively.

In our case the master is the Raspberry Pi itself and the slave devices are things like this ADC chip and various other sensor chips. The master is essentially in charge of all the slave devices and is responsible for starting and stopping all communications. The master can send data to a slave (write) or receive data back from one (read) but the slaves never communicate with each other directly.

Each device on the I²C bus has a unique *address* which is used by the master to differentiate slave devices during communication. We will need to put the I²C bus address of the ADC into our Python code later.

In practise the Pi (the master) will send a binary sequence to a slave device with the desired address (the ADC), the binary usually instructs the slave to do something (such as measure the analogue voltage from the wind vane). The Pi then waits for a *tiny* amount of time for the ADC to do its work whereupon it sends a read command. The ADC then starts sending binary data back to the master and this will represent the value of the measurement it just took.

Our task is to understand what binary sequence we need to send (from the master Pi to the slave ADC) in order to make the ADC take a reading and *then* to understand what to do with the binary data that we get back as the answer. Only when we've got that far we can then start looking at converting the reading into a compass direction for the wind vane.

There is a lot to take in here but do not be discouraged. A lot of these techniques will be reused for other sensors on the weather expansion board in future lessons.

## Main Development


### Setting up your Pi

1. Place the SD card into the slot of your Raspberry Pi.
1. Connect the Weather Expansion Board to the GPIO pins.
1. Connect the wind vane to the socket marked *WIND* on the Weather Expansion Board.
1. Next connect the HDMI cable from the monitor or TV.
1. Plug in the USB keyboard and mouse.
1. Plug in the micro USB power supply.
1. When prompted to login type:

    ```bash
    Login: pi
    Password: raspberry
    ```

## Plenary

[Next lesson](../lesson6/README.md)
