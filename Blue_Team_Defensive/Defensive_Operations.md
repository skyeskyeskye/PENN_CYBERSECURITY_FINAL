# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology
_TODO: Fill out the information below._
_TODO: use `nmap -sV 102.168.1.1-110` or nmap -sS IPs_of_every_machine to verify all of these from the Remote Desktop Machines.
  
The following machines were identified on the network:
- Azure VM
  - **Operating System:** Windows
  - **Purpose:** Acts as the Remote Desktop Host
  - **IP:** 
- Target 1
  - **Operating System:** Linux
  - **Purpose:** Apache Webserver running a Wordpress Site
  - **IP Address:** 192.168.1.110/24
- Kali 
  - **Operating System:** Linux
  - **Purpose:** Penetration Tester VM
  - **IP Address:** 192.168.1.90/24
- ELK
  - **Operating System:** Linux
  - **Purpose:** Run ELK stack to Kibana
  - **IP Address:** 192.168.1.100/24
- Capstone
  - **Operating System:** Linux
  - **Purpose:** VM used solely for testing alerts
  - **IP Address:** 196.168.1.105/24

### Description of Targets

The target of this attack was: `Target 1` at the IP Addresss: `192.168.1.110`.

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Alert 1: Excessive HTTP Errors

![HTTPErrorsAlert](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/ExcessiveHTTPErrors_Kibana.jpg)
    {Note: This screenshot was taken pre-attack}

Alert 1 is implemented as follows:
  - **Metric**: http.response.status_code
  - **Threshold**: above 400
  - **Vulnerability Mitigated**: Brute Force Attempts (through SSH) that result in excessive amount of error codes
  - **Reliability**: High- however, it should be noted, while the baseline came from previous traffic experience, there can be times where there is more traffic to the webserver with codes over 400 just as a fluke and not indicative of an attack.

#### Alert 2: HTTP Request Size Monitor

![HTTPRequestSize](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/HTTPRequestSize_Kibana.jpg)
    {Note: This screenshot was taken pre-attack}

Alert 2 is implemented as follows:
  - **Metric**: http.request.bytes
  - **Threshold**: when the sum over all documents is above 3500 for the last minute
  - **Vulnerability Mitigated**: SQL Injection and HTTP Request Smuggling
  - **Reliability**: Medium- there can be triggers over the baseline because there isn't enough data collected (yet) to know what the best baseline is. For example, there might be times when there is a legitimate, work-related reason to have an increase in http.request.bytes. (Trigger might need to be adjusted higher or lower over time to be more reliable.)

#### Alert 3: CPU Usage Monitor

![CPUUsageAlert](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/CPUUsageMonitor_Kibana.jpg)
    {Note: this screenshot was taken pre-attack}

Alert 3 is implemented as follows:
  - **Metric**: system.process.cpu.pct 
  - **Threshold**: when max over all documents is above 0.5 for the last 5 minutes
  - **Vulnerability Mitigated**: Denial of Service, Brute Force, Malicious Software
  - **Reliability**: Medium- any unexpected spike in CPU usage should be assessed to verify the reason for such an increase in usage. The reason can be a nefarious attack, but other reasons can be update installations, outdated indexes that are not optimized, changes to third party applications, etc. 

_TODO Note: Explain at least 3 alerts. Add more if time allows._

### Suggestions for Going Further (Optional)
_TODO_: 
- Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain _how_ to implement each patch.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:


- Vulnerability 1: Brute Force Attacks resulting in Excessive HTTP Errors
  - **Patch**: (1) Install a WordPress Firewall Plugin. These can be a DNS Level Website Firewall or an Application Level Firewall. (2) Make sure WordPress is up to date- Run `wp core version` to verify your current version of WordPress and then run `wp core update` to install the newest version.
  - **Why It Works**: (1) The firewall service will filter and block bad traffic from accessing the site. (2) Having an updated WordPress installation should provide security from older known vulnerablities that have been patched in the most recent version of WordPress.


- Vulnerability 2: Unauthorized Access to MySQL
  - **Patch**: Secure the wp-config.php file by either (1) Using newly generated Secret Keys, (2) Moving the wp-config.php to somewhere other than the root folder of the site, or (3) Block access to the wp-config.php by creating an htacess file inside the same directory (`.htaccess` -- making sure to not have a .txt file end) then edit the file adding the code: 

  `1 <files wp-config.php>
   2 order allow,den
   3 deny from all
   4 </files>`
   
which will deny access to the config file. (source: https://webdesign.tutsplus.com/tutorials/how-to-secure-your-wordpress-wp-configphp--cms-27737)
	
  - **Why It Works**: By preventing access to the wp-config.php file, important and sensitive data (in our case the username: `root` and password: `R@v3nSecurity`) would not be readily available to attackers. 


- Vulnerability 3
  - **Patch**: TODO: E.g., _install `special-security-package` with `apt-get`_
  - **Why It Works**: TODO: E.g., _`special-security-package` scans the system for viruses every day_


Useful Resources:
https://www.wpbeginner.com/wp-tutorials/how-to-protect-your-wordpress-site-from-brute-force-attacks-step-by-step/

https://wpengine.com/resources/prevent-sql-injection-attack-wordpress/

