# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services
First, run `netdiscover -r 192.168.1.255/16`. This shows all of the machines on the network:

![netdiscover](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/netdiscover_picture_redteam.jpg)

After reconnaisance, http://192.168.1.110 is found to be a wordpress site. This is our target.

![Wordpress](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/wordpress_website_image_redteam.jpg)

Nmap scan `sudo nmap 192.168.1.110` results reveal the below services and OS details:

![nmap](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/nmap_scan_redteam.jpg)

This scan identifies the services below as potential points of entry:
- Target 1
  - SSH, port 22
  - HTTP, port 80
  - RPC Bind, port 111
  - Netbios SSN, port 139
  - Microsoft-DS, port 445

Next, a vulnerability scan reveals many exploit options with the open ports:

`nmap -sV --script vul 192.168.1.110`
ADD SCREENSHOTS HERE AFTER GETTING THEM OFF OF REMOTE DESKTOP

_TODO: Fill out the list below. Include severity, and CVE numbers, if possible._

FILL THIS BASED ON THE SCAN! The following vulnerabilities were identified on each target:
- Target 1
  - List of
  - Critical
  - Vulnerabilities

_TODO: Include vulnerability scan results to prove the identified vulnerabilities._

Using wpscan against the target-- `wpscan --url http://192.168.1.110/wordpress --enumerate u`, we find two distinct users: 'michael' and 'steven'.

![wpscan1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/wpscan_redteam_1.jpg)
![wpscan2](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/wpscan_redteam_2.jpg)


### Exploitation
Now that ssh is open and we have enumerated users, we can try to access the target by brute force.  This can be done using an application like hydra, but fortunately, we guessed the password on the third attempt. Running `ssh michael@192.168.1.110` and the password "michael" granted access to the target.

![ssh](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/SSH_as_Michael_redteam.jpg)


The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
- Target 1
  - `flag1.txt`: _TODO: Insert `flag1.txt` hash value_
    - **Exploit Used**
      - _TODO: Identify the exploit used_
      - _TODO: Include the command run_
  - `flag2.txt`: _TODO: Insert `flag2.txt` hash value_
    - **Exploit Used**
      - _TODO: Identify the exploit used_
      - _TODO: Include the command run_
