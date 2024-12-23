# Firmware for M5StickCPlus2 with [web interface](#web-interface-screenshot) 

<img src="./images/preview.png" width="500" alt="Menu demonstration"/>

## Description
This is easily expandable firmware for M5StickCPlus2. You can easily add a new function using the short [guide](#how-to-add-a-new-feature).
## How to build?
You can use the extension **arduino community edition** for build and upload firmware in vs code.

Here is example of configuration files which I'm using:

*.vscode/arduino.json*
```
{
    "sketch": "m5stick_power_firmware.ino",
    "port": "/dev/ttyACM0",
    "board": "m5stack:esp32:m5stack_stickc_plus2",
    "output": "./build",
    "programmer": "esptool",
    "configuration": "PSRAM=enabled,PartitionScheme=default_8MB,CPUFreq=240,FlashMode=qio,FlashFreq=80,FlashSize=8M,UploadSpeed=1500000,LoopCore=1,EventsCore=1,DebugLevel=none,EraseFlash=none"
}
```


*.vscode/settings.json*
```
{
    "C_Cpp.intelliSenseEngine": "Tag Parser",
    "C_Cpp.default.includePath": [
        "${workspaceRoot}",
        "~/Arduino/libraries"
    ],
    "arduino.enableUSBDetection": true,
    "arduino.additionalUrls": ["https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/arduino/package_m5stack_index.json"],
    "arduino.disableIntelliSenseAutoGen": true
}
```


## How to add a new feature?

Add a menu item wherever you want. 
You must specify the name and number of the process that will be launched. You can take the first one available. Check the [process id list](#list-of-process-ids).

*functions/mainMenuLoop.h*
```
MENU mainMenu[] = {
    // other items ...
    {"Hello world!", 7},
};
```
<img src="./images/example-1.png" width="500" alt="Menu demonstration"/>

After that, create a function for the process, let's create a new file for this in the *functions* directory

*functions/helloWorldLoop.h*

```
void helloWorldLoop() {
    if (isSetup()) {
        centeredPrint("Hello world!", SMALL_TEXT);
    }
    checkExit(0);

    // Description of the behavior of the function inside the web interface
    if (isWebDataRequested()) {
        webData = generateWebData("menu", generateFunctionElement("Hello world!", SMALL_TEXT, "center"));
    }
}
```

In this code we used the isSetup, centeredPrint, checkExit and isWebDataRequested functions. These are some of the "utilities" that are used to make writing code easier. They are all located inside files in the utils directory.
There is no full documentation yet, but you can already look at the short [documentation](#description-of-utilities).

Using the isSetup function ensures that the code inside the condition will only be executed once after the function runs.
The checkExit function will perform a process switch when pressing the B button. In this case, we will return to the main menu, since its process is 0.

<br>

Add an include of this function next to the others.

*globals/functions.h*

```
// other includes...
#include "../functions/helloWorldLoop.h"
```

Finally, add the case to the switch inside the loop function in the m5stick_power_firmware.ino file
```
case 7:
    helloWorldLoop();
    break;
```
## ✨ Result:
<img src="./images/example-2.png" width="500" alt="Program demonstration"/>



### List of process IDs

0. mainMenu  
1. clock  
2. battery info  
3. settingsMenu  
4. Wi-Fi AP  
5. brightness  
6. rotation
7. colors

### Description of utilities
*Only a small part of the utilities is described here*

<table>
  <tr>
    <th>Utility name</th>
    <th>Description</th>
    <th>Code example</th>
    <th>Layer</th>
  </tr>
  <tr>
    <td>isBtnBWasPressed<br>isBtnAWasPressed</td>
    <td>Checks physical clicks and clicks from the web interface. </td>
    <td><code>if (isBtnAWasPressed()) {Serial.print("btn A pressed!")}</code></td>
    <td align="center">1</td>
  </tr>
  <tr>
    <td>pressBtnB<br>pressBtnA</td>
    <td>Emulates a button press. You can try it with utility checkBtn..WasPressed)</td>
    <td><code>pressBtnA();<br>if (isBtnAWasPressed()) {Serial.print("btn A pressed!")}</code></td>
    <td align="center">1</td>
  </tr>
  <tr>
    <td>isSetup</td>
    <td>Ensures that code within a condition is executed once when the program starts.</td>
    <td><code>if (isSetup()) {Serial.print("run only ones!")}</code></td>
    <td align="center">2</td>
  </tr>
  <tr>
    <td>checkExit</td>
    <td>If the user exits the program (presses button B) the process will switch to the one passed in the parameters.</td>
    <td><code>checkExit(3)</code></td>
    <td align="center">2</td>
  </tr>
  <tr>
    <td>isWebDataRequested</td>
    <td>Checks whether a new line should be generated describing the current state of the interface. You will most likely need to pass the string "function" as the first argument. The second argument is a string that describes the contents of the interface.</td>
    <td><code>if (isWebDataRequested()) {webData = generateWebData("function", generateFunctionElement("text", SMALL_TEXT, "center"));}</code></td>
    <td align="center">1</td>
  </tr>
  <tr>
    <td>generateFunctionElement</td>
    <td>Creates and returns a string that describes the text parameters to be displayed inside the web interface. The first parameter is a line with text, the second is size, the third is centering.
    The resulting strings can be combined.</td>
    <td><code>generateFunctionElement("text", SMALL_TEXT, "center")<br> // return  "text,2,center;"</code></td>
    <td align="center">1</td>
  </tr>
</table>


### Web interface screenshot:
<img src="./images/web-interface.png" width="350" alt="web interface"/>

## ❤️ Support my project with a star if you liked it. Contributing is welcome.

This is my first open project, I will be glad for any support