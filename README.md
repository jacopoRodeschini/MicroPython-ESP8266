# ESP8266 & MICROPYTHON
> Simple - cheaper - Wifi - Fast - 1Mb flash size.

[MicroPython](https://docs.micropython.org/en/latest/index.html) is a software implementation of a programming language largely compatible with Python 3, written in C, that is optimized to run on a microcontroller. MicroPython is a full Python compiler and runtime that runs on the micro-controller hardware. In addition to implementing a selection of core Python libraries, MicroPython includes modules such as “machine” for accessing low-level hardware

First part of this guide are centered to flash micropython on ESP8266 based board, the next one show how to run python on the device. 

### :notebook: Dictionary

* In computing, [**firmware**](https://en.wikipedia.org/wiki/Firmware) is a specific class of computer software that provides the low-level control for a device's specific hardware. Firmware is held in non-volatile memory devices such as ROM, EPROM, EEPROM, and Flash memory. 

* A **bootloader** is a computer program that is responsible for booting a computer.

* A **driver** provides a software interface to hardware devices, enabling operating systems and other computer programs to access hardware functions without needing to know precise details about the hardware being used.


### :rocket: Setting up

This guide are joined with Ubuntu 18.04

```
>> echo "hi, I'm:"
>> echo "my brain is:"
>> uname -v

hi, I'm:
my brain is:
#34~18.04.1-Ubuntu
```

The first step for the initial configuration is to set user permission. The default user can’t use tty* (serial interface, COM) of the machine.



```
# Use this command to find your permission
>> groups
```
if tty is not present

```
>> sudo usermod -a -G tty $USER
```

Now plug and unplug your device from the USB port and use **dsmeg** command to see what happened.

You see that ESP8266 use a CH340g driver. Different ESP8266-based boards can use a different converter. The following procedure doesn’t change with to respect the converter. The only minor changes concern the right driver installations for the specific converter. In the next section, we perform this task.

### Connections

You need to connect with machines where ESP are attached and control if you have installed the driver for esp8266 (ch340) (installing the driver is the most critical operation, for more info follow this [documentation](https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all#linux))

```
>> ssh pi@<ip> ;; (view the configuration ./ssh/config)
```

for the Raspbian OS, the CH340g driver is already installed. Use this line of code to upgrade the driver

```
>> sudo apt update
>> sudo apt upgrate
```

Now attach ESP8266 with the CH340 driver and type again **dmesg** command to see if the driver is installed correctly

```
>> dmesg         :: view where ESP8266 is attached
>> lsusb	   :: view usb devices 
>> ls /dev/tty*  :: look for /dev/ttyUSB0
```

Now the ESP8266 is correctly connected to your machine.

Usually, the best practice is to experiment with drivers and install third-party software on virtual machines (VMs). In this case, if you are working on a VM (hos on your pc), you must enable the access of the VM to the serial ports (the **lsusb** command doesn't show the device). 

Go to VM setting and add new serial port [documentation](https://www.linux-kvm.org/page/USB_Host_Device_Assigned_to_Guest). At the end, you need to add qemu-KVM to the **dialout** group. Follow this [documentation](https://askubuntu.com/questions/112568/how-do-i-allow-a-non-default-user-to-use-serial-device-ttyusb0). 

```
# 1) Uninstal brltty driver

>> sudo apt remove brltty 

# 2) Add new hardware (virt-viewr)

# 3) Check the connection 
>> dmesg         :: view where esp are attached
>> ls /dev/tty*  :: look for /dev/ttyUSB0
```

### :rocket: ESPTOOl (Esptool.py)

For the next step, it’s necessary to install esptool.py (esp tool). This python script communicates with ESP8266 ROM and performs some tasks such as erasing and updating the firmware. To install esptool, open the terminal and use the python packets manager **pip**.

```
# Python and pip version 
>> python -V & pip -V

# Install esptool script
>> pip install esptool.py

# Esptool options help
>> esptool.py -help
```
For example, to see the chip specifications use the following line of code:

```
# Chip specifications
>> esptool.py chip_id --port /dev/ttyUSB0

# Command output
Detecting chip type... ESP8266
Chip is ESP8266EX
Features: WiFi
Crystal is 26MHz
MAC: 5c:cf:7f:3d:52:dd
Uploading stub...
Running stub...
Stub running...
Chip ID: 0x003d52dd

```
Another useful command is the **–flash_id** with which it's possible to check the memory of the ESP8266 device. The flash memory size information is mandatory to install the right firmware.
```
# ESP8266 flash information
>> esptool.py flash_id

# Command output 
...
Manufacturer: c8
Device: 4016
Detected flash size: 4MB
...

```

### :mega: Upload Firmware

In this step, we erase the last firmware boot on ESP and upload the new firmware for the micropython interpreter (in other words we upload the interpreter). The main download page is available from this list [download](http://micropython.org/download). Download the latest firmware based on your flash memory size. In my case flash memory size is 1Mb.

```
# Download the firmware (replace filename with your version)
>> cd Download
>> wget https://micropython.org/download/esp8266-1m/[filename]
```

```
# Erase the flash 
>> esptool.py --port /dev/ttyUSB0 erase_flash  

# Upload the new firmware
>> esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash 
 --flash_size=detect 0 esp8266-20180511-v1.9.4.bin --flash_mode=dout 
```
For more info on uploading the firmware step and fixing the bugs follow the official [documentation](https://docs.micropython.org/en/latest/esp8266/tutorial/intro.html) of micropython firmware upload.

Note that on the line to load the new firmware there are options ** - flash_mode = dout **, in my case without this option they cause a failure of the upload.


### :sunglasses: RELP Terminal

The RELP ia a inline interpreter of micropython, [documentation](https://docs.micropython.org/en/latest/esp8266/tutorial/repl.html)). You can open a session with the ESP8266 board and type python code that will evaluate on the board. The REPL has history, tab completion, auto-indent and paste mode for a great user experience. To open a new session you can use the **picocom** command.

```
# Open serial communication. Use [C-a C-x] to close session 
>> picocom /dev/ttyUSB0 -b115200

# When the terminal is ready
...
>>>  // python interpreter
>>> 3 + 1 
4
>>> print('Hello Word!!!')
Hello Word!!!
...


# Now, write python code in the RELP interpreter and enjoy your ESP8266 ;). 
# In the following examples we turn on and off a simple pin named led
...
>>> from machine import Pin
>>> led = Pin(12,Pin.OUT)
>>> led.on()
>>> led.off()
>>> led.on()
```

The REPL interpreter is always available on the UART0 serial peripheral, which is connected to the pins GPIO1 for TX and GPIO3 for RX. The baudrate of the REPL is 115200

### :v: Run Script

An import task is to run a python script (.py) that starts running when the board are powered. For this task is necessary to install the **ampy** tool.

```
# Install the ampy tool
>> sudo apt install ampy upgrade
```

Then use the following line of code to run the main.py script on the ESP8266 device. The output was printed in the command line.

```
# Run main.py script in the ESP8266 device
>> ampy --port /dev/ttyUSB0 run main.py
```

When you use an infinite loop add **–no-output** options in the command and open a RELP terminal (with picocom) to see the execution of the program.

```
>> ampy --port /dev/ttyUSB0 run --no-output main.py
```
To run the main.py script in the ESP8266 every time when the board is powered, it's necessary to load the main.py script in the ESP memory. To this step use the *put* command of ampy tool.

```
# Load the main.py script in the ESP8266 memory
>> ampy --port /dev/ttyUSB0 put myfile.py /main.py
```
For more details on the ampy tool or more options detail follow the official ampy[documentation](https://www.digikey.com/en/maker/projects/micropython-basics-load-files-run-code/fb1fcedaf11e4547943abfdd8ad825ce).


### Visual Studio Code (vs)

Now, it's useful setting up VS Code to develop faster. To install and download the micropython extension see the [documentation](https://marketplace.visualstudio.com/items?itemName=dphans.micropython-ide-vscode). It's very, very easy!

--
### Download libraries

Micropython provides a very simple but very useful package manager. The new libraries are saved under '/lib' folder. 


```
# Open RELP session 
>> picocom /dev/ttyUSB0 -b115200
...


# Import package manager
>>> import upip

# Install micropython library 
>>> upip.install('package_name');
...

# See the filesystem
>>> os.listdir()
['boot.py', 'lib.py', 'data.txt']
...
```

There are two important files: boot.py & main.py.
- **boot.py** script is executed when the pyboard boots up.

- **main.py** is the main script that will contain your Python program. It is executed after boot.py.


```
# Reboot the device
>>> import machine
>>> machine.reset() // hard reboot 
```


This is enough 
Good programming time ;)
















