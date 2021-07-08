# Info about extractors
The extractors were writtent to parse the log output from pfSense processes. Currently Filterlog and OpenVPN log data is supported.
##Filterlog
The json file includes extractos for both the RFC 3164 and RFC 5424 standard. 
You must be storing the full message data to parse RFC 5424

You can choose which standard to use from pfSense by:  

1. Log into the pfSense WebUI.
2. Go to Status>System Logs.
3. Click on the Settings tab.
4. Select the option you want to use from the Log Message Format drop down menu

***Please note that Syslog RFC 5424 is the recommended option for this guide.***
  
Parses data for the following:
###IPv6
* UDP
* TCP


###IPv4
* TCP
* UDP
* ICMP
* IGMP
* GRE

##OpenVPN
The json file only includes extractos for the RFC 5424 standard.  
  
  The extractors for OpenVPN work by searching for the following line:
  
  ```
  "<29>1 2021-07-07T16:25:15.296751-05:00 source.exampledomainname.com openvpn 45184 - - username/255.255.255.255:7777 MULTI_sva: pool returned IPv4=172.18.34.1, IPv6=(Not enabled)"
  ```
  
  Let's break that down.  
  The line is the full message text obtained from the pfSense input.  
  
  * Timestamp: ```2021-07-07T16:25:15.296751-05:00```  
  * Source data obtained from the syslog format: ```source.exampledomainname.com```  
  * Application: ```openvpn 45184```
  * Username with source IP and port number: ```username/255.255.255.255:7777```
  * The local VPN IP that OpenVPN issues to the user: ```IPv4=172.18.34.1```

  We first use the Regular Expression (regex) to find all log entries that match this line. I use the `MULTI` in `MULTI_sva` as the unique pattern.  
  ```
  openvpn\s\d{1,5}\s-\s-\s+(.*)(MULTI)
  ```
    
  Then we pipe that data into the desired field based on the regex group. In this case, we are finding the username.  
  ```
  openvpn\s\d{1,5}\s-\s-\s+(\w+)
  ```
  In our regex expression, graylog first looks for the word "openvpn", with a space, a number with 1-5 digits, three more spaces, and adds the first word to our group. This matches the field that the username goes into. 
  ```
  openvpn 45184 - - username
  ```
   
   We use similar regex expressions to find the IP's and other useful information. Just remember that Graylog uses the first matcher group for its data. Regex groups are desinated by parentheses.  ( *field data* )
  

##Installation
1. Log in to your Graylog WebUI
2. Go to System>Inputs
3. Click on "Manage extractors" on the input you want to parse data on.
4. Click Actions>Import extractors
5. Paste the entire contents of the JSON file into the Extractors JSON field on the WebUI
6. Click "Add extractors to input".

Be sure to verify that your data is being parsed into the correct fields once you add the extractors.

##Version Info:
###Written and tested on:  
Graylog 4.1.0+4eb2147 on localhost (Private Build 1.8.0_292 on Linux 5.4.0-77-generic)  
pfSense+ 21.02.2-RELEASE.  

This guide comes **ABOSULETLY NO WARRANTY** and is not guaranteed to work on future versions of Graylog or pfSense