# gardenwrt
openwrt on gardena smart gateway

# disclaimer
```
Your warranty is now void.
I am not responsible for bricked devices, dead SD cards, thermonuclear war,
or you getting fired because your mower stopped moving the football field.
Please do some research if you have any concerns about features included in this ROM before flashing it!
YOU are choosing to make these modifications, and if you point the finger at me for messing up your device, I will laugh at you.
```

# warning
DO NOT plug in a power supply with >5V!

Due to a hardware design oversight (maybe put the Diode closer to the PTC fuse so it trips faster or use a ***smaller*** fuse, HW dev.) the board will start cooking the area near the diode and ***might*** cause a fire.  

# Art. No. 19000 (the one with the antenna sma connectors)
get the all-in-one pack (with squeezelite) from the archive and follow these steps or build the image yourself from my 19000 openwrt branch.
```
0. Get sam-ba tool from microchip
1. Solder uart header and sam-ba usb header
2. Connect the sam-ba usb (the unpopulated 4 pin header near the leds) via a cable to your pc
3. Short nand data lines with tweezers
4. Plug device into power
5. A usb cdc serial device should appear (remove short now), if not try again. 
6. run ./sam-ba.exe -p serial -b sam9xx5-ek -a lowlevel
7. run ./sam-ba.exe -p serial -b sam9xx5-ek -a extram
8. run ./sam-ba.exe -p serial -b sam9xx5-ek -a nandflash
9. run ./sam-ba.exe -p serial -b sam9xx5-ek -a nandflash -c read:dump.bin 
(this is your full flash in case you want to revert back.)
10. run ./sam-ba.exe -p serial -b sam9xx5-ek -a nandflash -c erase
11. run ./sam-ba.exe -p usb -b sam9xx5-ek -a nandflash -c writeboot:at91bootstrap.bin (from openwrt downloads section, nand version)
12. run ./sam-ba.exe -p usb -b sam9xx5-ek -a nandflash -c write:u-boot.bin:0x40000 (from openwrt downloads section, nand version)
13. run ./sam-ba.exe -p usb -b sam9xx5-ek -a nandflash -c write:openwrt-at91-sam9x-gardena_smart_gateway_19000-fit-zImage.itb:0x200000
14. run ./sam-ba.exe -p usb -b sam9xx5-ek -a nandflash -c write:openwrt-at91-sam9x-gardena_smart_gateway_19000-ubifs-root.ubi:0x800000
15. turn device off and on
16. fix uboot env - connect uart and do:
setenv bootcmd 'nand read 0x22000000 0x200000 0x600000; bootm 0x22000000'
setenv bootargs 'console=ttyS0,115200 rootfstype=ubifs ubi.mtd=6 root=ubi0:rootfs rw'
saveenv
boot
17. ???
18. have fun.
```

# Art. No. 19005

TODO, got initramfs to boot.
