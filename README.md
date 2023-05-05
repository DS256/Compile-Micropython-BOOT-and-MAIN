# Compiling (mpy_cross) Micropython BOOT and MAIN Programs on Raspberry PI PICO W

## INTRODUCTION

I like to have my Python programs compiled, when they are in the field, and the unit they are running in can potentially disappear. At the moment, you cannot cross compiled the Micropython BOOT and MAIN programs from PY to MPY files. I worked up an easy solution which allows for the bulk of the code to be compiled and also allows GLOBAL variables to still be shared between BOOT and MAIN

## APPROACH

Micropython programs can be compiled using [MPY_CROSS](https://pypi.org/project/mpy-cross/). This is straightforward and I will not go into detail

Modules can be IMPORTed and called from BOOT and MAIN. These modules can be compiled through even though BOOT and MAIN are not. This is straighforward but managing shared GLOBAL variables between BOOT and MAIN when using IMPORTED modules presented a bit of a challenge. For those who do not know, GLOBAL variables are shared between BOOT and MAIN in Micropython which allows data to be passed from one to the other. An example is IP address if BOOT makes the WiFi connection. The challenge is that GLOBAL variables are not accessble to IMPORTed modules. This means they need to be passed and returned in the calls.

## EXAMPLE

The following is code that should work on a PICO W running Micropython.

When RESET, BOOT.PY is run. This declares the GLOBAL variable BOOT_OK and calls the compiled program BOOT_DISPLAY().
```
# boot.py - to call compiled boot_display.mpy
# This is specific to Micropython

from boot_display import boot_display

# In MP global varibles are shared between BOOT.PY and MAIN.PY
# Global variables are not shared with IMPORTed modules
# Need to get value through return on call

global ip_addr 

ip_addr=boot_display() 
```
Note that the value for the GLOBAL variable IP_ADDR must come back as a returned value.

BOOT_DISPLAY.PY looks like this. It would be cross compiled to BOOT_DISPLAY.MPY for deployment
```
# boot_display - Main Code of Boot that can be compiled to .MPY
# This is specific to Micropython

def boot_display():
   
    # Connect WIFI
    
    ip_addr_assigned='192.168.1.42'
 
    return ip_addr_assigned # Return to get registered as global variable in main
```
After BOOT is run, you should be able to see the following in the SHELL window
```
>>> ip_addr
'192.168.1.42'
```
Next MAIN.PY is automatically run. Note, we have to pass BOOT_OK to MAIN_DISPLAY()
```
# main.py - to call compiled main_display.mpy
# This is specific to Micropythons

from main_display import main_display

global ip_addr          # Set in BOOT.PY+BOOT_DISPLAY.PY

main_display(ip_addr)   # Need to pass as variable
```
Finally, MAIN_DISPLAY.PY looks like this. It would be cross compiled to MAIN_DISPLAY.MPY for deployment.
```
# main_display.py - Main program that can be compiled to .MPY
# This is specific to Micropython

def main_display(from_ip_addr):

    # Do some work like weather readings
    
    # Send Results
    # publish(from_ip_addr, readings)
                
    return
```
## SUMMARY
I present this as giving back to all the examples I've used from others. In particular, I didn't know GLOBAL variables were not shared with IMPORTED modules. Hope you find some use for this.
