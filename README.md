# Arduino / atmega328 bootloader for over air flashing (BT)

This bootloader starts by default after software or hw reset and thus enableds flashing mode after softeare watchdog reset. Blootloader can be disabled and enabled by eeprom flag. I use this for wireless flashing with HC-05 and HC-06 BT modules whitout sny hw mods to BT modules. This sould work with any serial bridge that is tx/rx capaple. I use this also in some cases when aruino bootloader needs to boot faster as boot is fast when bootloader is disabled. 

This repo is copy from https://github.com/arduino/Arduino/tree/master/hardware/arduino/avr/bootloaders/atmega
I was unable to fork only needed dir and whole arduino project was too large to clone just for fun. Thus created new repo. Initial commit for .c and Makefile is copy from Arduino project for easy compare. Makefile has flashing commands, but I did find easyer to make flash script for flashing.

I am using atmega328p and this is tested only with 328p, but sould be easy to compile for other chips included in Makefile.

This is not true autoreset flashing as flashing softweare is unable to hw reset the chip and reset must be done via main program, but this enables you to reset/flash hard to reach bords remotely over the air. 

Usage
=====

1. Flash boolader with isp (example AVARISP mkII). Avrdude commands for fuse settings and flashing can be found in file "flash".
2. Setup bt module with at commands in at mode. I use bananapi 3.3V gpio serial port for sending AT commands. Do not connect 3.3V level module directly to PC serialport. For bt over air flashing only correct baud rate is needed, but name and pin code are usefull ;)   
3. Connect atmega/arduino rx / tx to bt module ( use voltage divaider etc. for tx if you have 5V atmega and 3.3V bt module ). Module needs also power and gnd.  
4. I use "rfcomm connect rfcomm0" for constant connection. "rfcomm bind" type connection that connects when port is used and disconnects when not caused some issus while switching from serial terminal connection to serial flashing.
5. Bootloader is waiting for you first flash so do avrdude -p atmega328p -c arduino -P /dev/rfcomm0 -b 57600 -D  -U flash:w:main.hex:i ( flash something that you can used to softreset the device [watchdog reset] )
6. Next time bootloader is not waiting for you as your sketch is running. To flash you need to reset atmega. HW reset is ok, but normally not available in plain rx/tx connection. Watchdog is able to softreset atmega so do wdt_enable(WDTO_15MS); while(1); in yoursketch and bootlader will start in about 16MS.
7. Bootlader is waiting 10 sec before continuing to your sketch. Kill serial terminal ( if you are using same serial for communicating and flashing like I am ) and start flashing "avrdude -p atmega328p -c arduino -P /dev/rfcomm0 -b 57600 -D  -U flash:w:main.hex:i" 
8. Bootloader can be disabled by setting last eeprom byte to something else than 0xff. While bootloader is disabled hw reset and watchdog reset are fast and go after few if's to sketch.
9. Enable bootloader before next flash by setting last eeprom byte to 0xff. While bootloader is enabled all resets will take about 10sec as bootloader is waiting for flashing to start.

Main app example simplet
========================

Always flash something that can be used for softreset (watchdog). I have automatic reset after bootloader is enabled.

		// Enable bootlader mode and reboot
		if( command[1] == 'E' )
 		 {
 		 eeprom_update_byte(E2END,0xff); //Enable bootloader 
 		 uart_puts("Bootloader mode enabled. Going to reboot.\n");
 		 wdt_enable(WDTO_15MS); //Watch dog hits and reboots chip to bootloader
 		 while(1); 
 		 continue;
 		 }
 
 		// Disable bootloader mode. Fast reset as bootloader is not waiting at all. 
		if( command[1] == 'e' )
 		 {
 		 eeprom_update_byte(E2END,0);
 		 uart_puts("Bootloader mode disabled.\n");
 		 continue;
 		 }
