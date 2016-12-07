# Sensing the Weather - Wind Gust Speed Worksheet

In this lesson you will:

- Understand the definition of a 'gust of wind'
- Translate this definition into a format the computer can understand
- Be able to store wind speed measurements in a list for later analysis
- Be able to manipuate data items in a list

## What hardware will we use?

To measure the wind gust speed we will also use the anemometer, the same instrument we used to measure the wind speed in a previous lesson.


## Getting Started

1. Set up your Raspberry Pi and ensure you are in desktop mode.

1. Launch the terminal

    ![Terminal](images/terminal.png)

1. Move to the `weather station` directory by typing `cd weather_station` and pressing `enter`

1. Make a copy of your wind_final program to a new file called `wind_gust.py` by typing this command into the terminal and pressing enter.

	```bash
	cp wind_final.py wind_gust.py
	```
Alternatively, you could download this [wind speed program](../wind_speed/code/wind_final.py) to your `weather_station` folder.

1. Open the program by typing `sudo idle3 wind_gust.py`. We still need to use all of this code to gather wind speeds, but we need to add to it to be able to store and analyse the wind speeds.
    
## Storing wind speed readings

Since the time period we are interested in is 20 seconds and our existing program records wind speed every 5 seconds (this is the number of seconds stored in the `interval` variable in the program), we need to record the **four** most recent readings to cover the last 20 seconds. To do this we will use a Python data structure called a list. Here is some code which creates an empty list, add this to the top of your code where you have initialised your other variables.

	```python
	store_speeds = []
	```
Now we need to make sure that when we calculate a speed reading inside the function `calc_speed()` that speed reading gets recorded in the list. Your code for this function should look like this at the moment:

	```python
	def calculate_speed(time_sec):
	    global count
	    circumference_cm = (2 * math.pi) * radius_cm
	    rotations = count / 2.0

	    dist_km = (circumference_cm * rotations) / CM_IN_A_KM

	    km_per_sec = dist_km / time_sec
	    km_per_hour = km_per_sec * SECS_IN_AN_HOUR

	    return km_per_hour * ADJUSTMENT
    ```
The final result for the wind speed is the final line where we `return km_per_hour * ADJUSTMENT`. Returning a value immediately ends the function, but we still have some work to do, so let's store this result in a variable so we can return it later.

	```python
	final_speed = km_per_hour * ADJUSTMENT
	```
Now add this speed to our list of wind speed measurements:

	```python
	store_speeds.append(final_speed)
	```

We only want to store the last 4 measurements of wind speed to cover the previous 20 seconds, but if we carry on adding each new measurement to the list, we will soon have a very long list! To prevent this from happening we need to check how long the list is using the `len()` function, and if adding this item takes the list over 4 items, we want to chop off the oldest item, which will be the first item in the list. To do this, we can use the slicing operator:

	```python
	store_speeds = store_speeds[1:]
	```
Each item in a list is called an **element**. This code tells the list to slice off a chunk of the list from element 1 to the end. It might sound like this won't do anything, unless you know that elements in lists are indexed from 0. For example, if I have the list `["a", "b", "c", "d", "e"]` then element 0 is "a", element 1 is "b" etc. 

Add some code to your `calculate_speed()` function to store the reading, here is some pseudo code to help you achieve this:

    > STORE most recent reading IN store_speeds LIST
    > IF store_speeds LIST has MORE THAN 4 elements
    >	store_speeds LIST = SLICE store_speeds LIST from element 1 to end
    

## Checking for gusts

### Question
A gust of wind occurs within a given time period when:
- the highest wind speed measured during that period is above 29.6km/h AND
- the difference between the peak speed and lowest speed in that period is greater than 16.7km/h AND
- the time period is 20 seconds or less

Now that we have the data available to us, how can we ask the computer to check whether a gust has occurred? Take a look at the data below which stores four wind speed readings in km/h, taken 5 seconds apart, and decide for each line of data a) if there was a gust or not, and b) if so, what is the wind gust speed?

	```python
	Data #1 - [1.4, 2.6, 42.9, 4.4]
	Data #2 - [1.3, 17.2, 24.2, 28.8]
	Data #3 - [34.9, 33.1, 35.9, 31.3]
	Data #4 - [2.9, 3.8, 2.1, 6.5]
	```

### Answer
A gust has occurred for Data #1, but not in the other data sets. Let's examine why. We already measured a time period of 20 seconds, but for a gust to have occurred, two further conditions need to be true:

Condition 1 - Highest wind speed above 29.6km/h
Condition 2 - Difference between highest and lowest is above 16.7km/h

So the results for each data set are:

| Data set      | Condition 1   | Condition 2   			| Gust?   |
| ------------- | ------------- | ------------------------- | ------- |
| Data #1      	| True (42.9)	| True (42.9 - 1.4 = 41.5)	| True 	  |
| Data #2      	| False (28.8) 	| True (28.8 - 1.3 = 27.5)	| False   |
| Data #3 		| True (35.9)  	| False (35.9 - 31.3 = 4.6)	| False   |
| Data #4 		| False (6.5)  	| False (6.5 - 2.1 = 4.4)	| False   |

(If you have studied Boolean Logic, you might notice that these examples just happen to form the truth table for the logical operator 'AND' - for the answer to be True, both condition 1 and condition 2 need to be True.)

## Adding a gust detection function

Our final job is to add a function we can call to check whether a gust occurred in the last 20 seconds.

## Summary

- Questions to think about

## What's next

Ideas for things to do next