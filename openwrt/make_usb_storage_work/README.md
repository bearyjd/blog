# Support USB on LEDE 

### Step 1: Ensure dts file has enabled ehci and ohci

```
&ehci {
    status = "okay";
};

&ohci {
    status = "okay";
};
```

### Step 2: Enable usb support on linux kernel  

`$ make kernel_menuconfig`

Select in turn:  
```
Device Drivers  --->
	[*] USB support  --->		
```

### Step 3: Select usb kernel module and dependency libraries  

`$ make menuconfig`  

Select in turn:  
```
Kernel modules  --->
	USB Support  --->  
		-*- kmod-usb-core  
		<*> kmod-usb-ledtrig-usbport  
		<*> kmod-usb-ohci 
		<*> kmod-usb-storage 
		<*> kmod-usb2
	Filesystems  --->
		<*> kmod-fs-exfat  
		<*> kmod-fs-ext4  
		<*> kmod-fs-vfat  
Utilities  --->  
	Filesystem  --->  
		-*- ntfs-3g  
		<*> ntfs-3g-utils
Libraries  --->  
	<*> libblkid  
	-*- libuuid	
```  

### Step 4: Support automatic mount and led indicator 

Related files on the toplevel directory of the buildroot, as follow:  
```
files/
├── etc
│   ├── config
│   │   └── system
│   └── hotplug.d
│       └── block
│           └── 10-mount
└── usr
    └── sbin
        └── blkid

```

### Step 5: Rebuild firmware  

`$ make V=s`  

# Support SD Card on LEDE 

### Step 1: Ensure dts file has enabled sdhci  

```
&sdhci {
    status = "okay";
};
```  

### Step 2: Select SD card kernel module  

```
Kernel modules  --->
	Other modules  ---> 
		-*- kmod-mmc  
 		-*- kmod-sdhci  
		<*> kmod-sdhci-mt7620  		
```  

Besides, it also needs to add filesystem module and dependency libraries and so on which has done on supported USB of *Step 3* and *Step 4*. If you have done supported USB, then you should rebuild the firmware directly, otherwise, please repeat the above *Step 3* and *Step 4*.
