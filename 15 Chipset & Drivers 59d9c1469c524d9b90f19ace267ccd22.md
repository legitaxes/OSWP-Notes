# 15. Chipset & Drivers

1. Can use airmon-ng to display the name and chipset 
    1. if it doesnt show, check the loaded modules or drivers
    2. If its usb, use `lsmon` 
    3. Might need  to reboot as module may stay loaded after adapter is plugged in and then unplugged.
2. If no modules loaded, can look at `dmesg` before and after plugging in device. May indicate what chipset is and if there are any errors. 
    1. run `lsusb -vv` for description of USB id
    
    ![Untitled](Untitled%2013.png)
    
    - important to note is `idVendor` and `idProduct`
    - both of these are from a text file `[/var/lib/usbutils/usb.ids]`
    - similar file exists for PCI/PCIe devices, pci.ids, in `/usr/share/misc/`
3. for most internal devices, run `lspci`or `lspci -n` 

1. Alternatively, can look for driver file names which often contains the name of the chipset or the driver to use. 
    1. If the driver is packed in executable (.msi, .exe) need to unpack first
    2. Need to do several times as is the case when it bundled with wifi manager
    
2. Once chipset is determined, determining the driver is fairly straightforward. 
    1. Search on Linux-Wireless wiki to help in figuring out which driver to use or use `Google` or `Wikidevi`
    2. Don’t need to worry if find conflicting results. There are several drivers on Linux with same wireless card
3. Once again, can just run airmon-ng to check the chipset
4. On Windows, can just check the device manager and the ID would be found in the Details pane under “Hardware ID”