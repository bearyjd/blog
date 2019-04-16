Remote Command Execution Vulnerability  
======================================  

We found a potential security vulnerability in v2.264 or v2.27 firmware. We recommend all of our customers can upgrade to v3.x firmware. But if someone doesn't like to update, we provide the patch to protect the router.  

Patch Tree  
----------  

You will see there are two folders. The purpose of them is to: 

**mips_24kc** - CGI binary files for AR150, MIFI, AR300M16, AR300M, AR750  

**mipsel_24kc** - CGI binary files for MT300A, MT300N, MT300N-V2  

How to Use  
----------  

#### Apply to Router  

You can use WinSCP drap and drop those CGI files to router's ***/www/cgi-bin*** folder. If you don't know how to use WinSCP, please refer to this: <https://docs.gl-inet.com/en/3/app/ssh/#winscp>  

**Import: Used this method, those files will disappear when you reset the router.**  

#### Built-in Patch to Firmware via Buildroot  

Assume you know how to use OpenWRT's Buildroot, and you had built a firmware by yourself. You can create a folder named **files/www/cgi-bin**, and copy those CGI files into it. The buildsystem will copy the **files** folder to **rootfs** automatically. 

Please refer to this: <https://openwrt.org/docs/guide-developer/build-system/use-buildsystem#custom_files>  

#### Built-in Patch to Firmware via Imagebuilder  

The step of built-in patch to firmware via imagebuilder is the same as buildroot, which we mentioned above. The only one difference is that we should pass a **files** variable explicitly in imagebuilder.

Please refer to this: <https://openwrt.org/docs/guide-user/additional-software/imagebuilder#files_variable>  

Regards,
Kyson Lok