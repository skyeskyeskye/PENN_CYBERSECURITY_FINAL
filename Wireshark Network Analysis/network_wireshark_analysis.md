# Network Analysis


## Time Thieves
At least two users on the network have been wasting time on YouTube. Usually, IT wouldn't pay much mind to this behavior, but it seems these people have created their own web server on the corporate network. So far, Security knows the following about these time thieves:

  - They have set up an Active Directory network.
  - They are constantly watching videos on YouTube.
  - Their IP addresses are somewhere in the range 10.6.12.0/24.

You must inspect your traffic capture to answer the following questions:

(1) What is the domain name of the users' custom site?
   
### frank-n-ted.com

Source:

![franknted](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/Screenshot%20(4).png)

(2) What is the IP address of the Domain Controller (DC) of the AD network?
    
### 10.6.12.12

Source:

![IPandDC](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/Wireshark_Shows_IP_and_DC.png)

(3) What is the name of the malware downloaded to the 10.6.12.203 machine? Once you have found the file, export it to your Kali machine's desktop.

### June11.dll

Source:
![June11](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/Wireshark_Malware_Capture.jpg)
 
(4) Upload the file to VirusTotal.com. What kind of malware is this classified as?

### Trojan/YAKES-- this type of malware is where the malware attempts to connect to and download from remote servers by arriving on a system unknowingly when visiting a site.

![virustotal1](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/Virustotal1.jpg)
![virustotal2](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/Virusttotal2.jpg)
 
## Vulnerable Windows Machines

The Security team received reports of an infected Windows host on the network. They know the following:

  - Machines in the network live in the range 172.16.4.0/24.
  - The domain mind-hammer.net is associated with the infected computer.
  - The DC for this network lives at 172.16.4.4 and is named Mind-Hammer-DC.
  - The network has standard gateway and broadcast addresses.

Inspect your traffic to answer the following questions:

(1) Find the following information about the infected Windows machine:
 
### Host name:   Rotterdam-PC
 
### IP address:  172.16.4.205
 
### MAC address: 00:59:07:b0:a4
 
 Source:
![ip src](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/infectedwindowshostnamemacaddressIPallinonescreenshot.jpg)

(2) What is the username of the Windows user whose computer is infected?

### matthijs.devries

Source: Filtering 'kerberos.CNameString' && ip.addr==172.16.4.205 then finding CNameString in the results and setting Wireshark to add 'CNameString' as a column.

![CName](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/InfectedWindowsUserName.jpg)
 
(3) What are the IP addresses used in the actual infection traffic?
   
### 172.16.4.205, 185.243.115.84, 166.64.111.64

Source: This was based on the number of packets these IPs received and after assessing each IP, individually, to verify the type of traffic.

I selected: `Wireshark-->Statistcs-->Endpoint` with the filter of `ip.addr==172.168.4.205 && http.request.method == POST`

![packet](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/HTTPPostTrafficEndpointsBymostPackets.jpg)
 
As a bonus, retrieve the desktop background of the Windows host.

Not sure if this is correct, but I selected `file-->export-->http` and then searched for jpegs and img and users from the wireshark packets that looked familiar and found this image:

![bonus](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/commit/d87e68715ad6df2f3b610ed5be5d120f9fa0c1d6)


## Illegal Downloads

IT was informed that some users are torrenting on the network. The Security team does not forbid the use of torrents for legitimate purposes, such as downloading operating systems. However, they have a strict policy against copyright infringement.
IT shared the following about the torrent activity:

  - The machines using torrents live in the range 10.0.0.0/24 and are clients of an AD domain.
  - The DC of this domain lives at 10.0.0.2 and is named DogOfTheYear-DC.
  - The DC is associated with the domain dogoftheyear.net.

Your task is to isolate torrent traffic and answer the following questions:

(1) Find the following information about the machine with IP address 10.0.0.201:

### MAC address: 00:16:17:18:66:c8
  
### Windows username: elmer.blanco

Source:
  
![mac and windows username](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/WiresharkPart3Answer1.jpg)

### OS version: Windows 10
  
Source: 

![OS](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/OSVersionQuestion3.jpg)


(2) Which torrent file did the user download?

### “Betty_Boop_Rhythm_On_The_Reservation”

Source:

![bittorrent](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/BettyBoop.jpg)

![bboopstills](https://github.com/skyeskyeskye/PENN_CYBERSECURITY_FINAL/blob/main/Wireshark%20Network%20Analysis/Images/BettyBoopImage.jpg)

Useful Resources: https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/

