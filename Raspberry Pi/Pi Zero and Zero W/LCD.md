# Waveshare 2.4" LCD module with Ili9341 controller

1. Once the OS is flashed and the pi has booted, use:
   `sudo raspi-config` and go to the interface and ensure SPI is enabled, once it is enabled, reboot the pi.
2. Once the pi has rebooted, remove power and use these connection to connect the screen to the pi:
   + VCC →Pin 1 (3.3V)
	+ GND →Pin 9 (GND)
   + DIN (MOSI) →Pin 19 (GPIO 10 / MOSI)
	+ CLK (SCLK) →Pin 23 (GPIO 11 / SCLK)
   + CS →Pin 24 (GPIO 8 / CE0) 
   + DC →Pin 18 (GPIO 24 - or your choice)
   + RST →Pin 22 (GPIO 25 - or your choice)
   + BL (Backlight) →Pin 17 (3.3V, or a PWM pin if you want software brightness control)

	Once all pins are connected, boot the pi and the screen should be on and fully white
4. Next, open the boot config file with this command: `sudo nano /boot/firmware/config.txt`. Once that file is open, navigate to the section labelled `[all]` and paste this: `	dtoverlay=fbtft,spi0-0,ili9341,speed=16000000,dc_pin=24,reset_pin=25,framebuffer_width=320,framebuffer_height=240`. Use `Ctrl+O` to save the current file and `Ctrl+X` to exit the file
5. Next, open the cmdline config file with this command `sudo nano /boot/firmware/cmdline.txt`. Once in the file, move to the end of the line, add a space, and paste this: `	fbcon=map:10 fbcon=font:VGA8x8 fbcon=rotate:3
`, use the VGA8x8 to change the size of the font, use the rotate to edit the rotation of the screen.
6. Once both of the previous steps have been completed reboot the pi using `sudo reboot now` and it should boot onto the screen eventually

