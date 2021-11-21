# klipper-secondmcu-creality
Setup for the 8-bit creality boards as a secondary MCU on Klipper on a Raspberry Pi

ARM-Linux seems to have a kernel bug that prevents _some_ FDTI serial connection from being stable. Specifically, this hit me when I was trying to setup a Creality V1.1.5 board as a second MCU for my 3D printer, so I could run more steppers that my main SKR E3 Mini had available (for a triple Z setup)

Here are details of the bug: https://github.com/raspberrypi/linux/issues/2406

After some testing I couldn't find any way to connect the creatily board to my Klipper-running Raspberry Pi 4B via USB without itfailing, mid-print, with the following error in dmesg:

`h341-uart ttyUSB0: usb_serial_generic_read_bulk_callback - urb stopped: -32`

So I decided to switch to using UART directly, however the creality board doesn't expose UART0 headers but the ATMega does have a second UARD on pins PD2(RX) PD3(TX). Here is how I set this up:

* 1. Connect the creality board to the Raspberry Pi via USB
* 2. Note the new USB ID /dev/serial/by-id/
* 3. `cd klipper`
* 4. `make menuconfig`
* 5. Set up the config as specified in the image and exit saving the config

![Image showing compilation options](creality_second_MCU_compile.png?raw=true)

* 6. `make flash FLASH_DEVICE=/dev/serial/by-id/[THE FILE NAME YOU SAW APPEAR IN #2]`
* 7. Edit the /boot/config.txt file and add the line `dtoverlay=uart3`
* 8. Shut down the Raspberry Pi and disconnect the power
* 9. Connect 3 jumpers from the Raspberry Pi GPIO to the Creality board EXT header as in the image

NOTE, this is for a Raspberry Pi 4B that has more UARTs than other Reaspberry Pi's, setup for those _will_ be different

![Image showing connection pinouts](creality_second_MCU_pins.png?raw=true)

Refer to this: https://raspberrypi.stackexchange.com/questions/45570/how-do-i-make-serial-work-on-the-raspberry-pi3-pizerow-pi4-or-later-models/107780#107780 for information on RPi 4B UARTS

* 10. Start the Raspberry Pi and wait for klipper to start
* 12. SSH into the Raspberry pi and vrify that /dev/ttyAMA1 now exists
* 11. Add the following sections to your klipper printer.cfg (You may need to change this dection depending on your probe/endstop/stepper/leadscrew configuration)

```
[mcu creality]
serial: /dev/ttyAMA1

[stepper_z]
# Marked X on board
step_pin: creality:PD7
dir_pin: creality:PC5
enable_pin: !creality:PD6
microsteps: 16
rotation_distance: 4
endstop_pin: probe:z_virtual_endstop
position_max: 330
position_min: -4
homing_speed: 10

[stepper_z1]
# Marked Y on board
step_pin: creality:PC6
dir_pin: creality:PC7
enable_pin: !creality:PD6
microsteps: 16
rotation_distance: 4

[stepper_z2]
# Marked Z on board
step_pin: creality:PB3
dir_pin: creality:PB2
enable_pin: !creality:PA5
microsteps: 16
rotation_distance: 4
```
