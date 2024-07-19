# gardenwrt
openwrt on gardena smart gateway

# disclaimer
```
Your warranty is now void.
I am not responsible for bricked devices, dead SD cards, thermonuclear war,
or you getting fired because your mower stopped mowing the football field.
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

TODO:
- fix dtb
- custom firmware for lemonbeat module to use something sane and open ([like radiohead](https://www.airspayce.com/mikem/arduino/RadioHead/))
- reverse engineer lemonbeat part of the gateway sw (so the insane journey of homeassistant->"ratelimited definetly 24/7 for 20years available cloud"->gateway->mower can be done offline)
- port vendor packages?
- play around with the MFI343S00177 mfi copro and maybe add homekit demo
- radio module uart and swd untested.
- cpu clock seems to be lower than reported - might aswell add overclock
- pcm (no idea, added in dtb and kernel but no audio device available)
- pwm for rgbw stripe support

what works:
- LEDs
- i2c (mfi chip), mcp port expander support in kernel too
- wifi
- ethernet
- squeezelite client -> usb soundcard
  
```
build openwrt from 19005 branch or use prebuilt files from this repo.

tip: press x to get into uboot shell :)

replace stock uboot with the u-boot-with-spl.bin,
it should be compatible with vendor uboot since the only change is being able to boot bigger kernels.
(todo shrink size since 16m is REALLY big)
tftp u-boot-with-spl.bin
mtd erase uboot
mtd write uboot ${fileaddr}

flash openwrt image:
(this should not be needed since it is included in the dtb)
setenv bootargs 'console=ttyS0,115200 ubi.mtd=4 rootfstype=squashfs'
setenv bootcmd "ubi part nand && ubi read 0x81000000 kernel && bootm 0x81000000"
saveenv
mtd erase.dontskipbad nand && tftp 0x80100000 openwrt-ramips-mt76x8-gardena_smart_gateway_mt7688-squashfs-factory.bin && mtd write nand 0x80100000 0 ${filesize}
(if your nand does have bad blocks and causes boot to fail, either create the kernel, rootfs and rootfs_data partitions yourself or just write over them:)
ubi part nand && ubi remove kernel && ubi remove rootfs OR just mtd erase nand
tftp 0x80100000 openwrt-ramips-mt76x8-gardena_smart_gateway_mt7688-squashfs-kernel.bin && ubi part nand && ubi create kernel ${filesize} d 0 && ubi write 0x80100000 kernel ${filesize}
tftp 0x80100000 openwrt-ramips-mt76x8-gardena_smart_gateway_mt7688-squashfs-rootfs.squashfs && ubi part nand && ubi create rootfs ${filesize} d 1 && ubi write 0x80100000 rootfs ${filesize}
ubi create rootfs_data - d 2
reset
enjoy :)

```
## luci and wifi
no idea whats wrong, currently don't care.
```
ip addr add 10.42.0.2/16 dev eth0 && ip link set eth0 up
/etc/init.d/firewall stop
vi /etc/config/wireless (configure wireless manually)
/etc/init.d/network restart
chmod 0644 /usr/share/rpcd/ucode/luci
service rpcd restart
```
