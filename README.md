# ESP8266 - MICROPYTHON

**Why esp8266?**

Simple - cheaper - Wifi - Fast - 1Mb flash size.

**Why micropython?**

MicroPython is a software implementation of a programming language largely compatible with Python 3, written in C, that is optimized to run on a microcontroller.

MicroPython is a full Python compiler and runtime that runs on the micro-controller hardware. In addition to implementing a selection of core Python libraries, MicroPython includes modules such as “machine” for accessing low-level hardware

Forst part of guide are centered to load micropython on NodeMcu v3 board, next show hoe to build custom PCB with ESP8266.

# Dictionary

**Firmware**:

E’ un programma, ovvero una sequenza di istruzioni, integrato direttamente in un componente elettronico programmato (es. BIOS su ROM). I dispositivi più recenti consentono l’aggiornamento del firmware, in una scheda elettronica esso generalmente trova posto all’interno di una memoria ROM o flash.Il suo scopo è quello di avviare il componente stesso e consentirgli di interagire con altri componenti hardware tramite l’implementazione di protocolli di comunicazione o interfacce di programmazione. Rappresenta di fatto il punto di incontro fra le componenti logiche e fisiche di un dispositivo elettronico, ossia tra software e hardware.

**Bootloader**:

Spesso esiste un altro componente software più semplice e di livello più basso, che si occupa delle funzioni minimali necessarie a gestire la memoria non volatile e a caricare il firmware, denominato bootloader.

**Driver**:

Esso permette al sistema operativo di utilizzare l’hardware senza sapere come esso funzioni, ma dialogandoci attraverso un’interfaccia standard. Ne consegue che un driver è specifico sia dal punto di vista dell’hardware che pilota, sia dal punto di vista del sistema operativo per cui è scritto. Non è possibile utilizzare driver scritti per un sistema operativo su uno differente, perché l’interfaccia è generalmente diversa.


# Setting up

This guide are joined with Ubuntu 18.04

```
echo "hi, I'm:"
uname -s
echo "my brain is:"
uname -v
hi, I'm:
Linux
my brain is:
#34~18.04.1-Ubuntu SMP Fri Feb 28 13:42:26 UTC 2020
```

First step for configuration system is set user permission. Default user can’t use tty* (serial interface, COM) of machine. For this topic see below:

use this command to find your permission

```
groups
```

if not see tty type of permission

```
sudo usermod -a -G tty yourname
or
sudo usermod -a -G tty $USER
```
Now plug and unplug your CH340 device from the USB port and use **dsmeg** command to see what appened.

Tou see that esp8266 use ch340g driver (usb-uart converter) [new version or different version of NodeMcu usp cp12… converter. The procedure don’t change, for driver installations see next sections].

# Connections

Connctiong with machines were ESP are attached and controll if you have installad driver for esp8266 (ch340) (install driver are the most critical operation, for more info follow [link](https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all#linux)

```
ssh pi@<ip> ;; (view ./ssh/config)
```

for rasbian, ch340 driver is already installed. Use this to upgrade driver

```
sudo apt update
sudo apt upgrate
```
Attach esp8266 with (ch340) and type **dmesg** and look you have success install the driver

```
dmesg         :: view where esp are attached
ls /dev/tty*  :: look for /dev/ttyUSB0
```

Now NodeMcu are correctly connect with our machine (Ubuntu desktop).


# ESP TOOl (Esptool.py)

For next step, it’s necessary install esptool.py (esp tool), this python script comunicate with NodeMcu rom and erase/update firmware. To install it, open terminal and use python packets maneger **pip** (here latest python version 3.7).

```
python -v 
pip install esptoo.py
esptool.py -help
```
for example, for see the chip specifications use:

```
esptool.py chip_id
```
another useful optios are –flash_id
```
esptool.py flash_id
```
with flash_id controll the memory of esp chip, with this information intall the true firmaware, to see the full command options add –help in the command (for example specify the port, bit rate and other).


usually to experiment with drivers, and install third party software and work on virtual machines. In this case, if you are working on a virtual machine (VM) on your pc, you must enable the access of the VM to the serial ports. 

Go to VM setting and add new port [link][https://www.linux-kvm.org/page/USB_Host_Device_Assigned_to_Guest]. At the end you need to add qemu-KVM to the **dialout** group [link][https://askubuntu.com/questions/112568/how-do-i-allow-a-non-default-user-to-use-serial-device-ttyusb0]. 

```
gropus
users
sudo adduser second_user dialout
```

# Upload Firmware

in this step we cancell last firmaware and upload the micropython interpreter. Download form this [download][http://micropython.org/download#esp8266] the latest firmware (check your flash memory size), in my case real flash size are 1Mb (Att: the flash_id return 4Mb, for nodeMcu this are divide by 4), then upload [fn:2].

```
esptool.py --port /dev/ttyUSB0 chip_id // chip information

esptool.py --port /dev/ttyUSB0 erase_flash  // cancell previos firmware

esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash 
 --flash_size=detect 0 esp8266-20180511-v1.9.4.bin --flash_mode=dout 

```
for more info of this step and fix bug follow firmware upload (official [documentation][https://docs.micropython.org/en/latest/esp8266/tutorial/intro.html] of micropython firmware upload).

Note that line to upload new firmware there are **–flash_mode=dout** options, missing it cause failure of upload.


# RELP Terminal

RELP (inline interpreter micropython) is a more powerfull features of micropython (for me). You can open inline sesson with board (NodeMcu) and type upython code that will evalutate on board.The REPL has history, tab completion, auto-indent and paste mode for a great user experience. For open a new session type:

```
picocom /dev/ttyUSB0 -b115200
```

Terminal ready

```
>>>  // python interpreter
>>> 3 + 1 
4
>>> print('Hello Word!!!')
Hello Word!!!

[C-a C-x] close session

now, put inside a python code e walla :). 
in the following examples we turn on and off a simple pin named led
>>> from machine import Pin
>>> led = Pin(12,Pin.OUT)
>>> led.on()
>>> led.off()
>>> led.on()
```
Now with program ours nodemcu in micrpython. For micropython [documentation][http://docs.micropython.org/en/latest/] follow “Reference for ESP8266”.

# Run Script

An import task, is run a upython script (.py) that start run when board are powered. For this install ampy

```
sudo apt install ampy upgrade
```
then use this to run main.py script. The output was printed in command line.

```
ampy --port /dev/ttyUSB0 run main.py
```
when you use a infinite loop add –no-output options and open a repl terminal to see execution of program (picocom)

```
ampy --port /dev/ttyUSB0 run --no-output main.py
```
if you put our main file in memory, this run every times when board is powerd, use:

```
ampy --port /dev/ttyUSB0 put myfile.py /main.py
```
for more details on ampy tool or more oprions follow ampy [documentation][https://www.digikey.com/en/maker/projects/micropython-basics-load-files-run-code/fb1fcedaf11e4547943abfdd8ad825ce]

















