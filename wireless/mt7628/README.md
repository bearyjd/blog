## Prepare

Make sure that you have got the wireless driver of MT7628 from MediaTek or your distributor.   

You can find the source code in [](./src), please create a fold named wlan_assoclist in you buildroot's package path, and copy all files into it.  

## Compile

Assume that you have known how to build a firmware, and you have built a firmware for MT300N-V2. We can execute this command to build our demonstration code.

```
make package/wlan_assoclist/{clean,compile} V=s
```

## Further development  

It is only contains a demonstration code to get wireless assoclist. You can integrate the function to your projet.
