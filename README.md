# Arduino bootloader

Arduino bootloader which starts in software reset and can be controlled with eeprom flag. For wireless flashing with HC-05 and HC-06 BT modules whitout hw mods.

This is derived from https://github.com/arduino/Arduino/tree/master/hardware/arduino/avr/bootloaders/atmega
I was unable to fork only needed dir and whole arduino project was too large to clone just for fun. Thus created new repo.

Initial commit for .c and Makefile is copy from Arduino project for easy compare. Makefile has flashing commands, but I did find easyer to make flash script for flashing.

I am using atmega328p and this is tested only with 328p, but sould be easy to compile for other chips included in Makefile.

Usage
=====

1. Flash boolader with isp (example AVARISP mkII). Avrdude commands for fuse settings and flashing can be found in flash.
2. Setup bt module with at commands in at mode. I use bananapi 3.3V gpio serial port for sending AT commands. Do not connect 3.3V level module directly to PC serialport. For bt over air flashing only correct baud rate is needed, but name and pin are usefull ;)   
3. Connect atmega/arduino rx / tx to bt module ( use voltage divaider etc. for tx if you have 5V atmega and 3.3V bt module ). Module needs also power and gnd.  
4. I use rfcomm connect rfcomm0 type constant connection. bind type connection that connects when port is used and disconnects when not caused some issus while switching from serial terminal connection to serial flashing.
5. Bootloader is waiting for you first flash so do avrdude -p atmega328p -c arduino -P /dev/rfcomm0 -b 57600 -D  -U flash:w:main.hex:i ( flash something that you can use to softreset the device )
6. Next time bootloader is not waiting for you as your app is running. You need to reset atmega. HW reset is ok, but as we did not connect hw reset to anything we have to do soft reset. Watch dog is able to soft reset atmega so do wdt_enable(WDTO_15MS); while(1); in your app and bootlader will start in about 16MS.
7. Bootlader is waiting 10 sec before continueing to normal mode. Kill serial terminal ( if you are using serial for communicating and flashing ) and starting flashing "avrdude -p atmega328p -c arduino -P /dev/rfcomm0 -b 57600 -D  -U flash:w:main.hex:i" 
8. If you like you can disable bootlader setting last eeprom byte to sometthing else than 0xff. While bootloader is disabled hw reset and watchdog reset are fast.
9. Enable bootloader before next flash by setting last eeprom byte to 0xff. While bootloader is enabled all resets will take about 10sec including hw and watchdog resets.

Main app example simplet
========================

Alway flash something that can used for soft reset and disable/enable for bootloader is good to have too

		// Enable bootlader mode and reboot
		if( command[1] == 'E' )
 		 {
 		 eeprom_update_byte(E2END,0xff);
 		 uart_puts("Bootloader mode enabled. Going to reboot.\n");
 		 wdt_enable(WDTO_15MS); //Watch dog hits in 15ms
 		 while(1);
 		 continue;
 		 }
 
 		// Disable bootloader mode
		if( command[1] == 'e' )
 		 {
 		 eeprom_update_byte(E2END,0);
 		 uart_puts("Bootloader mode disabled.\n");
 		 continue;
 		 }
