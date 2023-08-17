# OSSEC Introduction

OSSEC is easy to use and provides a high level of system surveillance for a small amount of effort. OSSEC is a Host-based Intrusion Detection System (HIDS). Using a HIDS allows you to have real time visibility into what security events are taking place on a server.



**OSSEC provides several functions**

* Real-time log monitoring.
* File integrity checking - detects changes to files and system paths.
* Changes to the system/running services (netstat) / disk space/password file changes.
* Real-time blocking of detected attacks through firewall rule modification.



# Before moving forward, you must have installed Postfix on your server to receive alert emails in your Gmail.



**Postfix installation**



**Step 1:** Install Postfix
In the first step install Postfix on your Ubuntu system. You can do this by running the following command −



```
sudo apt install postfix
```


During installation, you will be prompted to select mail server configuration and there options. You should choose "Internet Site" and fill in your server's domain name when asked.



![Setect mail server configuation](https://github.com/khanadm/OSSEC-SETUP/assets/106643382/af14f124-7efa-4e62-bc8c-721048d8cffa "Setect mail server configuation")



**Step 2:** Configure Postfix
Once Postfix is installed, you need to configure it to use Gmail as a relay for all emails. Open the main Postfix configuration file by using the following command −



```
sudo nano /etc/postfix/main.cf
```


Add the following lines to the end of the file 



```
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
```

Save and exit the file.


**Step 3:** Create a Gmail App Password
To use Gmail as a relay server, you need to create an App Password in Gmail configuration. This password is used to authenticate Gmail's servers for relaying emails. To create an App Password, you can follow these steps −

* Log in to your Gmail account.

* Go to your Google Account settings page.

* Click on "Security".

* Under "Signing in to Google", click on "App Passwords".

* Select "Mail" as the app and "Other (Custom Name)" as the device.

* Enter a name for the password and click on "Generate".

* Make a note of the password that is generated.


**Step 4:** Add Gmail Credentials in Postfix
Now that you have created an App Password, you need to add it to Postfix. Create a new file called "sasl_passwd" in the /etc/postfix directory by running the following command −

```
sudo nano /etc/postfix/sasl_passwd
```


Add the following line to the file, replacing "your-email@gmail.com" with your Gmail address and "your-password" with the App Password that you generated −

 [smtp.gmail.com]:587 your-email@gmail.com:your-password
Save and exit the file.


Now, use following command to hash the sasl_passwd file −

```
sudo postmap /etc/postfix/sasl_passwd
```


**Step 5:** Restart Postfix
Now restart Postfix service to apply the changes by using following command −



```
sudo systemctl restart postfix
```



Step 6: Test the Configuration
To test the configuration, send an email using the "mail" command, and replace "recipient@email.com" with the email address you want to send the email to −

```
echo "Test email" | mail -s "Test subject" recipient@email.com
```



If everything is configured correctly, the email should be sent and received by the recipient.

**Now the Postfix mail server is ready to work.** 



# Installing OSSEC

First we  need to install some libraries (libevent, libpcre2, zlib, openssl) 



 ```
 sudo apt install libevent-dev
 ```


```
sudo apt install libpcre2-dev
```



```
sudo apt install zlib1g-dev
```


```
sudo apt install libssl-dev
```

Now you have clone source code zip from from this repo 


https://github.com/ossec/ossec-hids/releases


After clone run this command 

```
tar zxvf ossec-hids-3.7.0.tar.gz
```



```
cd ossec-hids-3.7.0
```



```
sudo ./install.sh
```



```
1. What kind of installation do you want (server, agent, local or help)?  local

* If you are doing a basic install to a single server select 'local'.
This creates a single install to monitor only the server you are
installing on. See the documentation on the site for details on
setting up multiple agents on a number of servers that all report back
to a server.

2- Setting up the installation environment.

 - Choose where to install the OSSEC HIDS [/var/ossec]:

   - Installation will be made at  /var/ossec .

3- Configuring the OSSEC HIDS.

 3.1- Do you want e-mail notification? (y/n) [y]:
  - What's your e-mail address?   -- enter your email address here

 - We found your SMTP server as: example.test.com.
  - Do you want to use it? (y/n) [y]: n

  - What's your SMTP server ip/host? enter your preffered smtp server here

 3.2- Do you want to run the integrity check daemon? (y/n) [y]:
   (this is for file integrity checking, alerts you to changes to
files on your system)

  - Running syscheck (integrity check daemon).

 3.3- Do you want to run the rootkit detection engine? (y/n) [y]:
  (this checks for rootkits on a regular basis)

  - Running rootcheck (rootkit detection).

 3.4- Active response allows you to execute a specific
      command based on the events received. For example,
      you can block an IP address or disable access for
      a specific user.
      More information at:
      https://ossec-docs.readthedocs.io/en/latest/manual/ar/

  - Do you want to enable active response? (y/n) [y]:
(this can block attacks that meet certain rules)
```

**Note** If you select [y] yes for Active response, you are adding Intrusion Prevention capability. This is good, but you need to white list your own IP's as you don't want Active response to trigger against your IP and auto block your access. This could happen if you failed multiple ssh logins, or if you were to run a vulnerability scan against your IP as OSSEC would detect this as an attack. Your IP would get blocked, and you would be unable to ssh to your server for example to manage it!




**After compiling is complete you will be presented with final instructions:**

```
- System is Debian (Ubuntu or derivative).
 - Init script modified to start OSSEC HIDS during boot.

 - Configuration finished properly.

 - To start OSSEC HIDS:
               /var/ossec/bin/ossec-control start

 - To stop OSSEC HIDS:
               /var/ossec/bin/ossec-control stop

 - The configuration can be viewed or modified at /var/ossec/etc/ossec.conf


   Thanks for using the OSSEC HIDS.
   If you have any question, suggestion or if you find any bug,
   contact us at contact@ossec.net or using our public maillist at
   ossec-list@googlegroups.com
   ( https://www.ossec.net/about.html#support-options ).

   More information can be found at https://www.ossec.net

   ---  Press ENTER to finish (maybe more information below). ---
   ```


That's it your done. Now start the server with:



 ```
 sudo /var/ossec/bin/ossec-control start
 ```

we have installed ossec, Now want to block brute force attacks for some period. OSSEC blocks these attacks by default for 600 secs using
iptables and hosts.deny linux mechanisms. 



If you want to receive emails from OSSEC, you need to configure it in the ossec.conf file.



```
sudo vim  /var/ossec/etc/ossec.conf
```

```
<ossec_config>
  <global>
    <email_notification>yes</email_notification>
    <email_to>YOUR_EMAIL_ID</email_to>
    <smtp_server>SMTP_SERVER_IP</smtp_server>
    <email_from>ossecm@ip-172-31-21-96</email_from>
  </global>
  ```



If you want to Decrease or Increase the timing for blocking period you can change here:


 ```
 sudo vim  /var/ossec/etc/ossec.conf
 ```


```    
    <command>host-deny</command>
    <location>local</location>
    <level>6</level>
    <timeout>YOUR_TIMING</timeout>
  </active-response>


  <active-response>
    <!-- Firewall Drop response. Block the IP for
       - 600 seconds on the firewall (iptables,
       - ipfilter, etc).
      -->
    <command>firewall-drop</command>
    <location>local</location>
    <level>6</level>
    <timeout>YOUR_TIMING</timeout>
  </active-response>
```



![OSSEC BLOCK FOR 600 secs](https://github.com/khanadm/OSSEC-SETUP/assets/106643382/9b1be38b-0bb4-4493-99a8-04f153e24539 "OSSEC BLOCK FOR 600 secs")





Thankyou !!!











