# Arduino bootloader

Arduino bootloader which starts in software reset and can be controlled with eeprom flag. For wireless flashing with HC-05 and HC-06 BT modules whitout hw mods.

This is derived from https://github.com/arduino/Arduino/tree/master/hardware/arduino/avr/bootloaders/atmega
I was unable to fork only needed dir and whole arduino project was too large to clone just for fun. Thus created new repo.

Initial commit for .c and Makefile is copy from Arduino project for easy compare. Makefile has flashing commands, but I did find easyer to make flash script for flashing.

I am using atmega328p and this is tested only with 328p, but sould be easy to compile for other chips included in Makefile.
