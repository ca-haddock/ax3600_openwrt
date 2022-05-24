Installing OpenWrt

To flash this image there is a need for SSH access to the OEM firmware, it isnt't possible for now to flash openwrt from the stock FW with the OpenWRT Factory image.
SSH Access

    Roll back to firmware to 1.0.17 || http://cdn.cnbj1.fds.api.mi-img.com/xiaoqiang/rom/r3600/miwifi_r3600_firmware_5da25_1.0.17.bin
    Setup the router admin password (quick way is using the mobile setup app)
    Login to the router web interface using the password set using the app and get the value of “stok=” from the URL
    Think of a password for SSH logins (8+ chars long, no special chars)

Manual method

You need to use the following URL(s) in order to enable SSH access. Before that please:

    Replace <STOK> with the stok value gained above
    Replace <PASSWORD> with the password generated above


-----


Flashing OpenWrt

After getting SSH access, you will now be able to flash a previously compiled image (by you since for the time being there are no official automated images):

    Copy the OpenWrt generated image (the openwrt-ipq807x-generic-xiaomi_ax3600-squashfs-nand-factory.ubi one) to the /tmp folder over SCP for example
    This is a device with a dual partition scheme layout, so you need to find out which one is running with the command

    nvram get flag_boot_rootfs

    The output should be the partition number where the current system was booted from

    mtd12: 023c0000 00020000 "rootfs"  - is the 0
    mtd13: 023c0000 00020000 "rootfs_1"  - is the 1

    Since you can't flash the current partition because it's locked you can only flash the opposite one by replacing the mtd number in the following command

    ubiformat /dev/mtd12 -f /tmp/openwrt-ipq807x-generic-xiaomi_ax3600-squashfs-nand-factory.ubi -s 2048 -O 2048

    After flashing the image you need to configure the u-boot to boot from the recently flashed image by replacing the 1 with the number of the opposite partition (1 or 0) and running the commands:

    nvram set flag_last_success=0 # if you've flashed mtd12, or "1" if mtd13
    nvram set flag_boot_rootfs=0 # same

    If you don't care about flag_last_success and flag_boot_rootfs variables just set flag_ota_reboot=1! It will boot to the other partition on each reboot (not only the next one!):

    nvram set flag_ota_reboot=1

    save the changes and reboot

    nvram commit
    reboot

    After the router boots it should be running the OpenWrt image, but now you need to flash the other partition to be able to sysupgrade without soft-bricking your router by replacing the mtd number with the one of the opposite partition with the command

    ubiformat /dev/mtd13 -f /tmp/openwrt-ipq807x-generic-xiaomi_ax3600-squashfs-nand-factory.ubi -s 2048 -O 2048

    After this you can flash future OpenWrt images/upgrades as usual

