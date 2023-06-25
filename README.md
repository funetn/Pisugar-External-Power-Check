# PiSugar External Power Check

I'm a hack coder that uses the internet heavily to help me solve my issues and come to a solution (THANK YOU TO all of the internet resources that I've referenced - too many to note and I've bounced around to way to many sites to keep track of which ones I've pulled methods/statements from).

Summary of the issue: I'm running VOLUMIO (media player) in my vehicle with a PISUGAR 3.  I wanted to keep the RPi0W running while I stop at stores/gas stations/etc without reboot due to power fluxuation from turn off my vehicle and restarting.  This will allow for immediate resuming of music.  I also wanted to do this without direct connecting to the vehicle's battery - thus needed to incorporate a UPS (to be purchased - didn't feel like DIY'ing one).

My first thought was to hard wire the RPi0W to the vehicle's battery then have a GPIO pin get pulled HIGH from the voltage from a secondary AUX USB plug (using an optical relay) and based on this issue a shutdown command to the RPi0W after x minutes (i.e 30 mins).  The RPi0W would still have a UPS connected to it to maintain power and bridge the fluxuation of power when restarting the vehicle.  Fortuntately in my timeframe of thinking how to resolve the issue PISUGAR came out with the PISUGAR 3 (or I just ran across it!).  The PISUGAR 3 for the RPi0 has a flag/bit that can be read which lets the user know if the UPS has external power connected to it or not.  Based on this "feature" I purcahsed the PISUGAR 3 (not the pro version since I'm using for a RPi0W).

PiSugar 3 Github page - https://github.com/PiSugar/PiSugar/wiki/PiSugar-3-Series

Solution - Purchased the PISUGAR 3 and started to figure out how to implement my x minutes delay for shutting down the RPi0W via the "connected power" flag.  This is the code that I came up with that works.

This is not the most elegant means to do this but with my limited programming skills seems to be functional and might provide a basis for other's projects and to improve upon.

## Manual Configurations:
1) Crontab entry:

*/30 * * * *   /usr/bin/python /volumio/pisugar_power_check.py

@reboot       /usr/bin/python /volumio/pisugar_power_check.py

2) Manually create a file called 'pisugar_power_check.power.txt' and put in the folder that you have the code referencing (in my case /volumio folder).  I tossed the Python script in the /volumio folder also since VOLUMIO is installed as a full image to the SD card so I didn't think it would matter where I placed it since it the system (SD card) needs to be rebuild the entire SD card will be overwritten.

## Logging
There are messages which are written to a "pisugar_power_check.log" that provide information on the status of the UPS's power connect and some other informative messages.

## Possible improvements:
    * Use the PISUGAR RTC to run shutdown after calculating current time + shut off delay (currently scraching head on this option - need to store the current time + shut delay in a file and read this and determine if it's time to shutdown??) vs. using the SLEEP function (which keeps the current script in a suspended state and another instance will be executed). This would allow the Python script to execute without SLEEPING, yet still check if it's time to shutdown.  Not sure if really that important or not.
    * Use of flock vs. the power/nopower file flipping (via the "mv" command to rename)
    * Use a 0 or 1 in a file to check as well but found the os.path.isfile check to work for me (at time of testing) without me opening/writing to a file.
    * Write a byte to an open I2C register and read this to know if I'm currently waiting to shutdown down or not (vs. using a file/flock)
    * Error checking


I was unable to find any code that provided the solution to my issue so hope this provides someone with some insight to resolve a similiar issue.

The code could be streamlined alot probably, such as using variables for the power/nopower filename, subroutines, etc.. I do realize the value of good programming practices (of which I utilized almost none here!) such as using variables for such things to keep things easy to modify without updating code in multiple places or subroutines vs. duplicate code.

Some steps might have been missed - documenting a few weeks after the fact and much was adhoc coding.  You might have to do things such as sudo apt get/update xyz packages.

## Possible future changes:
    1) make subroutines for some of the logic
    2) use Python Scheduler module to schedule shutdown based on current time + 30 mins and cancel the scheduled task if external power is applied before shutdown

Feedback is appreciated (though it might be over my head! :) ).  Hope this helps others get started with their PISUGAR 3 automation.

## Disclaimer
This is a hobby project with no warranty provided or implied whatsoever. This setup works for my purposes - your results might vary.
