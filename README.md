rise# TRONXY - Add Auto Leveling !
> Written by Ghassan Yusuf

![](./images/01.jpg)

## My Story
My story with this 3D printer, back in 2017 I bought two of these 3D printers in a hope to make business with them, and provide 3D printing services to my customers who where hobbyist in robotics and university engineering projects. how ever the business went south since my printing quality was bad and several prints didn't go well because of the bed leveling issue. and the amount of time it takes me to level the 3D printer before every print is 5-10 minutes, imagine every print I need to do this. later on they where laying down in my work shop not knowing what to do with them and I didn't want to throw them away because they cost me some money and I just refuse to throw them or give them away. So I called the supplier if there was a possibility of upgrading this 3D printer !? the supplier said he stopped 3D printing business long time ago, so I looked around the internet to find solutions sadly I couldn't find any person who did it and willing to share the configuration file nor made a repo about it. so I decided I will do it the hard way its either I succeed or die trying.

## Who is this for ?
This repository is created for people who bought this 3D printer and got stuck with it, so this repository is all about upgrading the functionality of the TRONXY 3D printer and add an auto leveling feature to it and more.

#### Note :
This doesn't require you to change the board, you are going to deal with every thing that you have, you need to just add a probe and make a small circuit and 3D print one part that will hold the probe in place. then do the right changes in [Configuration.h](./Marlin-1.1.x/Marlin/Configuration.h) file in marlin firmware.

## What do you get with TRONXY DIY 3D printer kit
![](./images/09.jpg)
This 3D printer comes equipped with the following
* [Melzi v2 Controller Board](https://reprap.org/wiki/Melzi)
* [ZONESTAR](./images/12.jpg) 20x4 Lines LCD Display With 4 Analog Push Buttons
* [Single Extruder & Hot End](./images/15.jpg)
* Heated Bed
* Three End Stop Switches For X, Y, Z
* Five Stepper Motors
* ATX Power Supply 12V 15Amps
* Cables Connector For All (Steppers, End Stops, Heaters, Fans, Thermistors)

## Follow the steps to add auto leveling feature
1. Software Pre request
  * You must have **Arduino IDE** installed or [download](https://www.arduino.cc/download_handler.php?f=/arduino-1.8.12-windows.exe) it and install it
  * You must have **VSCode** installed or [download](https://code.visualstudio.com/download) it and install it
  * You must install [Platform IO](https://youtu.be/CB8qC_-RHQI) **on VSCode**
2. Build a voltage level translator circuit for the inductive auto leveling probe
  Bill Of materials (BOM) - Hardware Materials
  * Inductive NPN normally open probe type [SN04](./documents/sn04-datasheet.pdf)
  * Circuit [Strip Board](./images/19.jpg) or what ever you prefer
  * Optocoupler 4Pin Chip Type [PC817](./documents/pc817.pdf)
  * 10k Ohm Resistor
  * [3 Pin JST Connector](./images/20.jpg) "you can cut the z end stop connector and use it but you will be missing one pin"
  * wild the circuit to the auto leveling probe wires and the connector for the board
  * heat shrink to cover the circuit board
3. Install the probe by connecting it to the board
4. Burning a bootloader in to [Melzi v2](https://reprap.org/wiki/Melzi) board with [Anet A8 (opti boot)](https://github.com/SkyNet3D/anet-board)
5. Configuring [Marlin 1.1.9](https://github.com/MarlinFirmware/Marlin/archive/1.1.x.zip)
6. Uploading Successfully Compiled Marlin Firmware To Your 3D printer Controller Board
  * Home the X, Y, Z axis Using G28 command.
  * Finally - Adjust Z Offset
  * Run Homing Command G28, Then Run Bilinear Auto Leveling Command G29 For Test
7. Tips For 3D Printing From Now On
8. Thing To Print To Make Your 3D Printer Look better

## 1. Software Pre Request
This is the beginning of our walkthrough we need several tools to be able to achieve many goals one step at a time. these software are going to help us do the following

1. **Arduino IDE** - Will allow us to bur a bootloader on to our board
2. **VSCode And Platform IO** - Will allow us to compile and burn the marlin firmware on to the board

> now lets start with the hardware after installing the previous programs and making them ready for use.

## 2. Build a voltage level translator circuit
When you purchase a SN04 probe, you have to know that it works on a different voltage level than your micro controller that runs your 3D printer, so connecting it directly will trigger the magic smoke and fry your board, that's why we need to build a voltage level translate between the probe output voltage to our micro controller voltage. there are many ways to do it but I built it as simple as possible and it works perfectly. you might get it with a 3 pin JST connector

![](./images/20.jpg)

If it doesn't come with a 3 pin JST connector you have to find one do the wiring then proceed or If not then proceed to the following
1. change the wires order in the connector use a sharp tool and press on the pin to push them out then change their order
  * brown because the board will provide you with 12V DC from that pin
  * black because the board will have GND connection Over There
  * blue because that's where the GPIO of the trigger signal
2. Move few centimeters away and cut the cord and strip the wires and make them ready for soldering
3. Build the below circuit in the diagram

![](./images/33.png)

the PRB_ prefix means that's the wires from the probe side, the CON_ prefix means the wires that are in their current order on the 3 pin JST connector. and that's how it looks like

![](./images/23.jpg)

, then take some heat shrink tubing cover the circuit and heat it up to shrink the tube, and that's how it looks

![](./images/24.jpg)

now we need to move to the next step to test the probe behavior

## 3. Install the probe by connecting it to the board
Turn the 3D printer power on, Install the probe in the Z end stop connector,  like the following picture ![](./images/32.jpg)

Then to test the probe do the following
1. Keep away from any metallic object, the probe light should be turned off.
2. Put the probe very close to a metallic object the probe light will turn on.

see the result in the following picture it should be what you get exactly
![](./images/34.png)

Now, Its time to test the result with **GCODE commands**

1. Keep away from any metallic object, the probe light should be turned off, and in the command line terminal **M119** then press enter, on the terminal the printer replies with the end stop status (**OPEN**) or (**H**) which means High
2. Put the probe very close to a metallic object the probe light will turn on, and in the command line terminal **M119** then press enter, on the terminal the printer replies with the end stop status (**CLOSE**) or (**L**) which means Low

  * if that's the case, Then you are read to move on to the next step, preparing to burn Marlin firmware on to your board.
  * if that's not the case, Then please check you circuit connections or your 3 pin JST connector if the wires are in the correct order.

## 4. Burning a bootloader in to Melzi v2 board with Anet A8 (opti boot)

#### Step 1 - Prepare An Arduino Board To Work As an ISP Programmer
Now, In this step we need to grab an **Arduino UNO** to make it work as an ISP programmer, in order to burn **Anet A8 (Obti boot)** on the board before burning the marlin firmware.

Head To ( File -> Examples -> Arduino ISP )

![](./images/35.png)

Head To ( Tools -> Boards -> Arduino/Genuino Uno)

![](./images/36.png)

Head To ( Tools -> Port -> Select Arduino Available Port )

![](./images/37.png)

Then Below The Menu Bar Click Tick Sign To Compile, If Successfully Compiled With no errors, Then click the arrow sign next to the tick sign and wait for Arduino to report Done Uploading Sketch.

#### Step 2 - Download The Necessary Files Of The Boot Loader & Install It On Arduino IDE
The board that you have originally don't have a bootloader, so it cant talk to your compiler to burn the firmware via USB like the Arduino does, so in order to make it accept new firmware through USB. we have to download some files to make Arduino environment to recognize the board as (Anet A8) [Click Here To Download The Files Needed](https://github.com/SkyNet3D/anet-board)

![](./images/38.png)

Then unzip head to the hardware folder within copy the anet folder

![](./images/39.png)

Head to this location on your hard drive ( C:\Program Files (x86)\Arduino\hardware ) and past the anet folder there

![](./images/40.png)

Restart Arduino IDE, so the changes will take place, Head to ( Boards -> Anet V1.0 (Optiboot))

![](./images/41.png)

#### Step 3 - Connect ISP Wires From Arduino Uno To Your Board

Now connect the SPI wires between **Arduino Uno** and your board as the following

* Arduino 5V -> MelziV2 ICSP 5V
* Arduino Pin 13 (SCK) -> MelziV2 ICSP (SCK)
* Arduino Pin 12 (MISO) -> MelziV2 ICSP (MISO)
* Arduino Pin 11 (MOSI) -> MelziV2 ICSP (MOSI)
* Arduino Pin 10 (SS) -> MelziV2 ICSP (RESET)
* Arduino GND -> MelziV2 ICSP GND

Arduino SPI Pinout
![](./images/45.png)

Melzi V2 ICSP Connector
![](./images/44.png "Melzi V2.0 ICSP Connector")

#### Step 4 - Select The Anet A8 Board & Select The Programmer (Arduino as ISP)

Head to ( Boards -> Programmer -> Arduino as ISP )

![](./images/42.png)

make sure that you selected every thing like (Step1) Then click the Burn bootloader (Step2) to burn optiboot to your board - done

![](./images/43.png)

at this point after burning the bootloader to your board you no longer need the **Arduino UNO** connected to your board, disconnect **Arduino UNO** and connect the USB cable directly to your board.

## 5. Configuring Marlin

### Preparing Visual Studio Code Environment
Start Visual Studio code, and make sure you installed Platform I/O previously, If you haven't installed platform I/O yet please click here to see how to install Platform I/O then continue with this tutorial

![](./images/46.png)

![](./images/47.png)

1. File -> Open Project Folder

![](./images/48.png)

Head to the folder you downloaded and navigate to marlin-1.1.x

then click select folder

2. Head to file platform.ini

![](./images/49.png)

change the content to be same as number 2, to make the compiler recognize the board micro controller

3. Head to [Configuration.h](./Marlin-1.1.x/Marlin/Configuration.h) File

![](./images/50.png)

if you have bought the very same 3D printer with same T2 belts and same pully, here are some value you need to put in [Configuration.h](./Marlin-1.1.x/Marlin/Configuration.h) file in marlin folder to get the right movement distances.

### Parameters To Change
These are some fixed parameters based on TRONXY 3D Printer build

#### Serial Port - Line 115
```
#define SERIAL_PORT 0
```
#### Baudrate - Line 126
```
#define BAUDRATE 115200
```
#### MotherBoard - Line 133 to 135
```
#ifndef MOTHERBOARD
  #define MOTHERBOARD BOARD_MELZI
#endif
```
#### Extruder - Line 149
```
#define EXTRUDERS 1
```
#### Filament Diameter - Line 152
```
#define DEFAULT_NOMINAL_FILAMENT_DIA 1.75
```
#### Thermal Settings - Line 313 to 319
```
#define TEMP_SENSOR_0 1
#define TEMP_SENSOR_1 0
#define TEMP_SENSOR_2 0
#define TEMP_SENSOR_3 0
#define TEMP_SENSOR_4 0
#define TEMP_SENSOR_BED 1
#define TEMP_SENSOR_CHAMBER 0
```
#### Mechanical End Stops - Line 530 to 537
```
#define X_MIN_ENDSTOP_INVERTING true // set to true to invert the logic of the endstop.
#define Y_MIN_ENDSTOP_INVERTING true // set to true to invert the logic of the endstop.
#define Z_MIN_ENDSTOP_INVERTING true // set to true to invert the logic of the endstop.
#define X_MAX_ENDSTOP_INVERTING false // set to true to invert the logic of the endstop.
#define Y_MAX_ENDSTOP_INVERTING false // set to true to invert the logic of the endstop.
#define Z_MAX_ENDSTOP_INVERTING false // set to true to invert the logic of the endstop.
#define Z_MIN_PROBE_ENDSTOP_INVERTING true // set to true to invert the logic of the probe.
```
#### Stepper Drivers Line - 553 to 563
```
#define X_DRIVER_TYPE  A4988
#define Y_DRIVER_TYPE  A4988
#define Z_DRIVER_TYPE  A4988
//#define X2_DRIVER_TYPE A4988
//#define Y2_DRIVER_TYPE A4988
//#define Z2_DRIVER_TYPE A4988
#define E0_DRIVER_TYPE A4988
//#define E1_DRIVER_TYPE A4988
//#define E2_DRIVER_TYPE A4988
//#define E3_DRIVER_TYPE A4988
//#define E4_DRIVER_TYPE A4988
```
#### Steps Per Millimeters - Line 611
in case you have the same setup just copy these values to the [Configuration.h](./Marlin-1.1.x/Marlin/Configuration.h)
```
#define DEFAULT_AXIS_STEPS_PER_UNIT   { 100, 100, 400, 95 }

```
in case you use different pullies and belts and micro stepping, then you need to calculate the values
#### DEFAULT MAX FEEDRATE - Line 618
```
#define DEFAULT_MAX_FEEDRATE          { 8000, 8000, 3000, 10000 }
```

#### DEFAULT MAX ACCELERATION - Line 626
```
#define DEFAULT_MAX_ACCELERATION      { 8000, 8000, 3000, 10000 }
```

#### Use Z MIN PIN for Probe - Line 677
```
#define Z_MIN_PROBE_USES_Z_MIN_ENDSTOP_PIN
```

#### Fixed Probe (Inductive Probe) - Line 719
```
#define FIX_MOUNTED_PROBE
```

#### Probe Offset Parameters - Line 776 to 795
> here you have to install the probe and measure the distance between the center point of the probe to the center point of the nozzle, and set X_PROBE_OFFSET_FROM_EXTRUDER value and the MIN_PROBE_EDGE is the width or diameter of your probe and set the speed of MULTIPLE_PROBING to 2 because it will give you accurate readings.

```
#define X_PROBE_OFFSET_FROM_EXTRUDER -39  // X offset: -left  +right  [of the nozzle]
#define Y_PROBE_OFFSET_FROM_EXTRUDER 0  // Y offset: -front +behind [the nozzle]
#define Z_PROBE_OFFSET_FROM_EXTRUDER 0   // Z offset: -below +above  [the nozzle]

// Certain types of probes need to stay away from edges
#define MIN_PROBE_EDGE 18

// X and Y axis travel speed (mm/m) between probes
#define XY_PROBE_SPEED 3000

// Feedrate (mm/m) for the first approach when double-probing (MULTIPLE_PROBING == 2)
#define Z_PROBE_SPEED_FAST HOMING_FEEDRATE_Z

// Feedrate (mm/m) for the "accurate" probe of each point
#define Z_PROBE_SPEED_SLOW (Z_PROBE_SPEED_FAST / 2)

// The number of probes to perform at each point.
//   Set to 2 for a fast/slow probe, using the second probe result.
//   Set to 3 or more for slow probes, averaging the results.
#define MULTIPLE_PROBING 2
```

#### Z Clearance And Offset - Line 811 - 820
```
#define Z_CLEARANCE_DEPLOY_PROBE    2 // Z Clearance for Deploy/Stow
#define Z_CLEARANCE_BETWEEN_PROBES  2 // Z Clearance between probe points
#define Z_CLEARANCE_MULTI_PROBE     1 // Z Clearance between multiple probes
#define Z_AFTER_PROBING             1   // Z position after probing is done

#define Z_PROBE_LOW_POINT          -3 // Farthest distance below the trigger-point to go before stopping

// For M851 give a range for adjusting the Z probe offset
#define Z_PROBE_OFFSET_RANGE_MIN -20
#define Z_PROBE_OFFSET_RANGE_MAX 20
```

#### Invert Stepper Enable Signal - Line 827 to 830
```
#define X_ENABLE_ON 0
#define Y_ENABLE_ON 0
#define Z_ENABLE_ON 0
#define E_ENABLE_ON 0 // For all extruders
```

#### Disables axis stepper immediately when it's not being used - Line 834 to 843
```
#define DISABLE_X false
#define DISABLE_Y false
#define DISABLE_Z false
// Warn on display about possibly reduced accuracy
//#define DISABLE_REDUCED_ACCURACY_WARNING

// @section extruder

#define DISABLE_E false // For all extruders
#define DISABLE_INACTIVE_EXTRUDER false // Keep only the active extruder enabled.
```

#### Invert stepper directions - Line 848 to 859
```
#define INVERT_X_DIR false
#define INVERT_Y_DIR false
#define INVERT_Z_DIR true

// @section extruder

// For direct drive extruder v9 set to true, for geared extruder set to false.
#define INVERT_E0_DIR false
#define INVERT_E1_DIR false
#define INVERT_E2_DIR false
#define INVERT_E3_DIR false
#define INVERT_E4_DIR false
```

#### Homing And Travel Limits - Line 872 to 889
```
#define X_HOME_DIR -1
#define Y_HOME_DIR -1
#define Z_HOME_DIR -1

// @section machine

// The size of the print bed
#define X_BED_SIZE 220
#define Y_BED_SIZE 220
#define Z_DEP_SIZE 200

// Travel limits (mm) after homing, corresponding to endstop positions.
#define X_MIN_POS -2.5
#define Y_MIN_POS -2.5
#define Z_MIN_POS 0
#define X_MAX_POS X_BED_SIZE
#define Y_MAX_POS Y_BED_SIZE
#define Z_MAX_POS Z_DEP_SIZE
```

#### Bed Leveling - Line 974 to 978
```
//#define AUTO_BED_LEVELING_3POINT
//#define AUTO_BED_LEVELING_LINEAR
#define AUTO_BED_LEVELING_BILINEAR
//#define AUTO_BED_LEVELING_UBL
//#define MESH_BED_LEVELING
```

#### AUTO BED LEVELING LINEAR Settings - Line 1020 to 1028
> if you want to probe for mare points on the bed adjust GRID_MAX_POINTS_X and GRID_MAX_POINTS_Y to your desired values or else leave it as it is

```
// Set the number of grid points per dimension.
#define GRID_MAX_POINTS_X 3
#define GRID_MAX_POINTS_Y GRID_MAX_POINTS_X

// Set the boundaries for probing (where the probe can reach).
#define LEFT_PROBE_BED_POSITION MIN_PROBE_EDGE
#define RIGHT_PROBE_BED_POSITION (X_BED_SIZE + X_PROBE_OFFSET_FROM_EXTRUDER)
#define FRONT_PROBE_BED_POSITION MIN_PROBE_EDGE
#define BACK_PROBE_BED_POSITION (Y_BED_SIZE - MIN_PROBE_EDGE)
```

#### Z SAFE HOMING - Line 1143 to 1152
```
#define Z_SAFE_HOMING

#if ENABLED(Z_SAFE_HOMING)
  #define Z_SAFE_HOMING_X_POINT ((X_BED_SIZE) / 2)  // X point for Z homing when homing all axes (G28).
  #define Z_SAFE_HOMING_Y_POINT ((Y_BED_SIZE) / 2)  // Y point for Z homing when homing all axes (G28).
#endif

// Homing speeds (mm/m)
#define HOMING_FEEDRATE_XY (50*60)
#define HOMING_FEEDRATE_Z  (4*60)
```

#### EEPROM Settings - Line 1225
```
#define EEPROM_SETTINGS // Enable for M500 and M501 commands
```

#### Preheat Constants - Line 1256 to 1263
```
#define PREHEAT_1_TEMP_HOTEND 210
#define PREHEAT_1_TEMP_BED     60
#define PREHEAT_1_FAN_SPEED     0 // Value from 0 to 255

#define PREHEAT_2_TEMP_HOTEND 240
#define PREHEAT_2_TEMP_BED    110
#define PREHEAT_2_FAN_SPEED     0 // Value from 0 to 255
```

#### SDCARD Support - Line 1428
```
#define SDSUPPORT
```

#### LCD Menu Items - Line 1454
```
#define SLIM_LCD_MENUS
```

#### LCD Controller Selection - Line
> In case you have the LCD Screen Like Me With 5 Buttons & 20x4 Lines Display

```
#define ZONESTAR_LCD
```

#### End Of Configuration Process
Save the file, and move on to the next step, uploading the firmware to your board

## 6. Uploading Compiled Marlin Firmware

1. Compile the Code
2. Check If The File Compiled Successfully
3. On this step assuming you got it compiled successfully -> make sure that the USB cable is connected between your computer and your 3D printer board -> Click The Arrow To Start Uploading The Firmware.

![](./images/51.png)

Once Finished You can connect to your 3D printer software before you start printing preferably using the following software (click the names so you can download them if you don't have them and install them)

* [Pronterface - Pronterface](http://kliment.kapsi.fi/printrun/Printrun-win-18Nov2017.zip) "Don't require Installation Just Unzip And Run The Red Exe File"
* [Repetier-Host](http://download.repetier.com/files/host/win/setupRepetierHost_2_1_6.exe) "Installation Required"

Connect to your 3D printer by selecting the baud rate you have specified in the [Configuration.h](./Marlin-1.1.x/Marlin/Configuration.h) on line# 126 and then select the USB port. Click Connect. Here You are connected And done with this step now its time to set the Z Offset.

### Adjusting Z offset

  1. Preheat Your HOTEND and Heated BED
  2. Using Commands Text Box Do The following -> Type G28 and press SEND Button "The Printer Will Home X, Y Axis And Finally will move to the center of the bed and try to Home Z"
  3. At this point the sensor will detect your bead but your nozzle will not be touching the bed.
  4. Do the following steps
  
```
  G1 X110 Y110 Z5 F3000 ; move to the center of your bed
  M211          ; gives the status
  M211 S0       ; disable software limits off -> now you can only use your Panel to Lower the Nozzle to the bed.
```
 
 5. Take a small peace of paper put it between the nozel and the bed, then using the printer panel select -> prepare -> move axis ->  Z axis -> move 0.1 mm -> rotate the knobe counter clock wise untill the nozel touches the paper and have a slite grip on it. then move on with the following.
 
```
  M114          ; this gives you your status where are you at (x, y, z position)
  M851 Z-0.4    ; TO SET THE OFFSET VALUE For Example -0.4 Is Our Offset
  M500          ; TO SAVE TO EEPROM
  M503          ; MAKE SURE THE OFFSET IS STORED CORRECT
```

### Auto tune PID for the heater

1. Make sure you hotend it cold.
2. Using the command line on prontor face type -> M303 S240

this will command the auto tune algorithm to heat up the nozel 8 time from room temperature up to 240 degrees celisious, then it will generate 3 perameters to se for you printer KP, KI, KD

```
#define DEFAULT_Kp 16.19
#define DEFAULT_Ki 1.33
#define DEFAULT_Kd 49.38
```
3. Now using the comand line insert the values like the following -> M301 P16.90 I1.33 D49.38 -> M500 to store the settings

  Now you are done and ready for 3D printing congratulation.

### Calibrating Extruder Steps/mm
To begin, send the command **M503** to your printer. This will return a string of values to your monitor. Find the line that starts with echo: **M92**, then find the E-value (which is usually at the end of this line). This is the current steps/mm value.
Now for the physical steps/mm value. First, we need to know how much filament was actually extruded. We can find this by measuring the distance from the extruder to the mark on the filament, then subtracting that value from 120:
    **120 – [length from extruder to mark] = [actual length extruded]**
Next, we need to know how many steps the extruder took to extrude that much filament. We can determine this value by multiplying the steps/mm value by the length we should have extruded, in this case 100 mm:
    **[steps/mm value] x 100 = [steps taken]**
Using this, we can obtain the physical, correct steps/mm value by dividing by the length extruded:
    **[steps taken] / [actual length extruded] = [accurate steps/mm value]**
Now all we have to do is set this as the printer’s steps/mm value, and we should be good to go!
(Reference)[https://all3dp.com/2/extruder-calibration-6-easy-steps-2/]

## 7. Tips & Good Practice For 3D Printing From Now On
since you are going to use the auto leveling sensor, you have to know that metals expand when Heated up, so if you do auto leveling before heating the bed it might give you wrong results that may cause you the loss of your print. so take the following steps in every print. You need to customize your starting GCODE to do the following before any prints

1. Preheat The BED and HOTEND corresponding to the material you are going to print with
2. Run Homing Routine G28
3. Run Auto Leveling Routine G29
4. Start Your Object Printing

These steps can be done, by Starting GCODE.

### Starting GCODE
```
; Pefix G-Code for most 200mm x 200mm 3D printers by www.DIY3DTech.com to clean nozzle
; Place as start G-Code in Slicer
; Use of this code is at your own risk (no warranties made or implied)

M117 Clean                ; Indicate nozzle clean in progress on LCD
M107                      ; Turn layer fan off
G21                       ; Set to metric [change to G20 if you want Imperial]
G90                       ; Force coordinates to be absolute relative to the origin
G28                       ; Home X/Y/Z axis
G29                       ; Auto Leveling The Bed
G0 X1 Y1 Z0.15 F2500      ; Move in 1mm from edge and up [z] 0.15mm
G92 E0                    ; Set extruder to [0] zero
G1 X190 E50 F500          ; Extrude 100mm Filament along X axis 190mm long to prime and clean the nozzle
G92 E0                    ; Reset extruder to [0] zero end of cleaning run
G1 E-3 F500               ; Retract Filament by 3 mm to reduce string effect
G1 X190 Y1 Z15 F2500      ; Move over and rise to safe Z height

; Recommend turning off SKIRT in the slicer to avoid strings pulled into first layer
; Begin printing with sliced GCODE after here
```

### Ending GCODE
```
M117 Finished             ; Indicate nozzle clean in progress on LCD
M104 S0
M140 S0
G92 E1                    ; Retract the filament
G1 E-1 F300               ; Retract -1 mm Feed rate 300 mm/s
G0 X0 Y210 F2500          ; Move The Plate To The Front For Easier Part Removal
M84                       ; Turn Off Steppers
```

## 8. Thing To Print To Make Your 3D Printer Look better
1. [Geeetech Prusa i3 Electronics Cover](https://www.thingiverse.com/thing:2194218)
![](https://cdn.thingiverse.com/renders/c1/54/d4/74/eb/6b62c0eae72f31f23106261cbc709f32_preview_featured.jpg "Geeetech Prusa i3 Electronics Cover")

### References
* [Developer Of Melzi](https://github.com/reprappro/melzi) - Who created the board
* [RepRap WiKi - Melzi](https://reprap.org/wiki/Melzi) - Documentation about the board
* [Geeetech WiKi - Melzi](http://www.geeetech.com/wiki/index.php/Melzi_V2.0) - Documentation about the board from Geeetech
* [Wiring Bl Touch - Melzi](https://www.antclabs.com/wiring3) - Walk through, Connecting a BLTouch to this board
