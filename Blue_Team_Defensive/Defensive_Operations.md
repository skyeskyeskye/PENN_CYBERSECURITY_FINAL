# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology
  
The following machines were identified on the network:
- Azure VM
  - **Operating System:** Windows
  - **Purpose:** Acts as the Remote Desktop Host
  - **IP:** 192.168.1.1
- Target 1
  - **Operating System:** Linux 3.2-4.9
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

![Network_Diagram](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/FIXEDFinalProjectDiagram.png)

### Description of Targets

The target of this attack was: `Target 1` at the IP Addresss: `192.168.1.110`.

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Alert 1: Excessive HTTP Errors

![HTTPErrorsAlert](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Screenshot%202022-05-21%20184605_HTTP_Response_Code_Alert.jpg)

Alert 1 is implemented as follows:
  - **Metric**: http.response.status_code
  - **Threshold**: When the count is grouped over top 5 of above 400 errors for 5 minutes
  - **Vulnerability Mitigated**: Brute Force Attempts (through SSH) that result in excessive amount of error codes
  - **Reliability**: High- however, it should be noted, while the baseline came from previous traffic experience, there can be times where there is more traffic to the webserver with codes over 400 just as a fluke and not indicative of an attack.

#### Alert 2: HTTP Request Size Monitor

![HTTPRequestSize](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Screenshot%202022-05-21%20184413_KIBANA_HTTP_Request_Bytes_Alert.jpg)

Alert 2 is implemented as follows:
  - **Metric**: http.request.bytes
  - **Threshold**: when the sum over all documents is above 3500 for the last minute
  - **Vulnerability Mitigated**: SQL Injection and HTTP Request Smuggling
  - **Reliability**: Relatively Low- there can be triggers over the baseline because there isn't enough data collected (yet) to know what the best baseline is. When observing this metric trigger on Kibana, regardless of attack or non-attack periods of time, it triggered quite often.  There might be times when there is a legitimate, work-related reason to have an increase in http.request.bytes. (Trigger might need to be adjusted higher to be more reliable.)

#### Alert 3: CPU Usage Monitor

![CPUUsageAlert](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Screenshot%202022-05-21%20184731_KIBANA_CPU_Usage_Alert.jpg)

Alert 3 is implemented as follows:
  - **Metric**: system.process.cpu.pct 
  - **Threshold**: when max over all documents is above 0.5 for the last 5 minutes
  - **Vulnerability Mitigated**: Denial of Service, Brute Force, Malicious Software
  - **Reliability**: Medium- any unexpected spike in CPU usage should be assessed to verify the reason for such an increase in usage. The reason can be a nefarious attack, but other reasons can be update installations, outdated indexes that are not optimized, changes to third party applications, etc. For example, even on just startup of the machines, the CPU usage metric triggered an alert. 

This is what Kibana shows whent the index thresholds are triggered:

![Kibana](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Screenshot%202022-05-17%20181351.jpg)

This is a closeup during the attack:
![Kibana Close Up](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Kibana_CloseUp_Event.jpg)

This is more information from an event during the attack where specifically the HTTP Request Size alert was triggered:

![RequestTrigger](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Kibana_Closeup_requestbytes.jpg)

This is more information from an event during the attack where specifically the HTTP Error alert was triggered:
![HTTPError](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Blue_Team_Defensive/Images/Kibana_Closeup_HTTPError.jpg)


### Suggestions for Going Further 

Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. Here are some patches that could mitigate the vulnerabilities/exploits that would trigger the alerts that have been set.


The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:


- Vulnerability 1: Brute Force Attacks resulting in Excessive HTTP Errors
  - **Patch**: (1) Install a WordPress Firewall Plugin. These can be a DNS Level Website Firewall or an Application Level Firewall. (2) Make sure WordPress is up to date- Run `wp core version` to verify your current version of WordPress and then run `wp core update` to install the newest version.
  - **Why It Works**: (1) The firewall service will filter and block bad traffic from accessing the site. (2) Having an updated WordPress installation should provide security from older known vulnerablities that have been patched in the most recent version of WordPress.


- Vulnerability 2: Unauthorized Access to MySQL
  - **Patch**: Secure the wp-config.php file by either (1) Using newly generated Secret Keys, (2) Moving the wp-config.php to somewhere other than the root folder of the site, or (3) Block access to the wp-config.php by creating an htacess file inside the same directory (`.htaccess` -- making sure to not have a .txt file end) then edit the file adding the code: 
 

  `1 <files wp-config.php>` 
  
  `2 order allow,den`
  
  `3 deny from all`
  
  `4 </files>`
   
   
which will deny access to the config file. (source: https://webdesign.tutsplus.com/tutorials/how-to-secure-your-wordpress-wp-configphp--cms-27737)
	
  - **Why It Works**: These "patches" help prevent unauthorized access to the wp-config.php file. By preventing access to the wp-config.php file, important and sensitive data (in our case the username: `root` and password: `R@v3nSecurity`) would not be readily available to attackers. 


- Vulnerability 3: Weak Passwords
  - **Patch**: Enact a strong password policy. Particularly, make sure that passwords can not be the same as username credentials.
  - **Why It Works**: The initial access to any of the data on the wordpress site came from easily identifying users and simply guessing one of those user's password. A strong password policy would have prevented that initial access and slowed down or stopped the attack.

- Vulnerability 4: Allowing the WordPress Site to be Scanned
   - **Patch**: Prevent WPScan from probing the site- modify NGINX configuration to block WPScan:
 
		`location ^~ /xmlrpc.php {`
			
			`deny all;`
		
			`error_page 403 =404 /;`
		
		`}` 
	
   - **Why It Works**: The location, xmlrpc, being accessible can allow attackers to enumerate WordPress authors and brute force WordPress logins.  This would prevent access to that file.


Useful Resources:
https://www.wpbeginner.com/wp-tutorials/how-to-protect-your-wordpress-site-from-brute-force-attacks-step-by-step/

https://wpengine.com/resources/prevent-sql-injection-attack-wordpress/

https://shift8web.ca/2019/01/how-to-block-your-wordpress-site-from-being-scanned-by-wpscan-with-nginx/
