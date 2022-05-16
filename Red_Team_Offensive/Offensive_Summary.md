# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services
First, run `netdiscover -r 192.168.1.255/16`. This shows all of the machines on the network:

![netdiscover](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/netdiscover_picture_redteam.jpg)

After reconnaissance, http://192.168.1.110 is found to be a wordpress site. This is our target.

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

`nmap -sV --script vuln 192.168.1.110`

![nmapvulnvid](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/nmapvulnscan1.jpg)

A video of the entire output from the vulnerability scan can be seen here: https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/nmap_vulnerability_scan_redteam.mp4

This nmap scan revealed many more vulnerabilities associated with the wordpress site, however only a few exploits were necessary to find the flags:

| Vulnerability | CVE | Port Used |   Description   | Severity|Link for more information|
|---|---|---|---|---|---|
| Open SSH | CVE-2015-5600  |  SSH  | Unrestricted access allows brute force attacks and high CPU usage (Denial of Service)  |CVSS v2 Score 8.5 "High" but awaiting reevaluation  | https://nvd.nist.gov/vuln/detail/CVE-2015-5600  |
| Python Privileges |   | SSH  | User steven had privileges to run python. This is exploitable using a python bash shell (PTY Shell) to escalate steven's privileges to root | High  | https://docs.python.org/2/library/pty.html  |
| Weak Passwords and Unsalted Password Hashes |   | SSH and HTTP | Using weak passwords that are unsalted and breakable with software like John the Ripper gives easy access to external parties. | High  |  https://github.com/openwall/john |
| Username Enumeration Not Disabled | | SSH | WPScan was able to determine usernames through a brute force scan |High| |
|MySQL Access| | SSH | Access to MYSQL allowed the attackers to view the database and directory content. SQL Injections have potential to execute SQL commands that create, retrieve, update, and delete data in the database. |High| |


Using wpscan against the target-- `wpscan --url http://192.168.1.110/wordpress --enumerate u`, we find two distinct users: 'michael' and 'steven'.

![wpscan1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/wpscan_redteam_1.jpg)
![wpscan2](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/wpscan_redteam_2.jpg)


### Exploitation
Now that ssh is open and we have enumerated users, we can try to access the target by brute force.  This can be done using a brute force application like WPScan (Example: `wpscan --url http://test.local/ --passwords passwords.txt`), but fortunately, we guessed the password on the third attempt. Running `ssh michael@192.168.1.110` and the password "michael" granted access to the target.

![ssh](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/SSH_as_Michael_redteam.jpg)

Directed to the config file for wordpress, we located and viewed wp-config.php file at /var/www/html.  

![configfind](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/Finding_wpconfig_redteam.jpg)

On the way to that file, the second flag was found:

![flag2](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/Flag2_redteam.jpg)

and using `grep -R ./ -e flag*` found the first flag:

![flag1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/flag1_Redteam.jpg)

Running `nano wp-config.php` revealed:

![wpconfig1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/WPConfig1_redteam.jpg)

Here we see the credentials for the user "root" with the password of "R@v3nSecurity"

Next, run `mysql -u root -p` and type in R@v3nSecurity as the password when prompted. This opens a MySQL session.

![sql1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/mysql_1_redteam.jpg)

Then checking what databases are available, run `show databases;`

![sql2](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/mysql_2_redteam.jpg)

Then selecting wordpress as the database to use, run `use wordpress;`

Run `show tables;` to see data we can inspect to try to find the flags.

![sql3](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/mysql_3_redteam.jpg)

Here, we can inspect the data inside these tables using `select * from <table name>;`. By doing this and inspecting several tables another 2 flags (flags 3 and 4) were found:

`select * wp_posts;`:

![flag3](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/flag3_redteam_insidemysqlselectasterixfromwp_posts.jpg)

Inspecting other tables revealed hashed credentials for the users michael and steven:

`select * from wp_users;`

![userhashes](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/mysql_4_hashes_redteam.jpg)

Using the hashed credentials saved to a .txt file and running that file through John the Ripper revealed the password for the user, "steven".

![johntheripper](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/JohnCrackedHash_and_Ssh_as_root_redteam.jpg)

Then sign in as steven using ssh and the password cracked by running John:
`ssh steven@192.168.1.110` and enter the password "pink84"

Running `sudo -l` as steven shows the privilege to run python. This can be exploited to escalate to root privileges by running:
`sudo python -c 'import pty;pty.spawn("bin/bash");'` This python code spawns a new bash shell to allow root privileges.

![escalatetoroot](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/Escalate_root_redteam_steven_python.jpg)

Now, with access to the root directory the fourth flag was found, again:

![flag4](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Red_Team_Offensive/Images/flag4_redteam.jpg)


The Red Team was able to penetrate the Raven Security Target Machine and retrieve the following confidential data for the four 'flags':

| Flag  | Hash |
| ------------- | ------------- |
| 1  | b9bbcb33e11b80be759c4e844862482d  |
| 2  | fc3fd58dcdad9ab23faca6e9a36e581c  |
| 3  | afc01ab56b50591e7dccf93122770cd2  |
| 4  | 715dea6c055b9fe3337544932f2941ce  |

