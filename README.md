# TryHackMe-GoldenEye
H4ppy H4ck1ng Br0


![image](https://user-images.githubusercontent.com/64806211/127406438-801cc0a5-3311-4532-89fa-6c51ef9a77d2.png)
 
 
 
 You can find the room here:
 
 https://tryhackme.com/room/mrrobot
 
 
 
 ### **Recon__** 
 
 
 
nmap -A 10.10.227.36 shows that there are 997 filtered port and port 22, 80 and 443 are filtered. This means that there is some kind of firewall blocking the nmap scans


Lets open the website anyway. the http site give a browser based shell with only few commands. The https site has a self signed certificate, and it also does the same thing



![hdh](https://user-images.githubusercontent.com/64806211/127406829-e28231df-20b3-4f41-a636-9ffcb6107fae.png)
 
 
 
If we use wakeup command, then the port seems to be opened. Nothin interesting from the page source too

 
 
 
  ### **File enumeration__** 
  
  
  lets run a gobuster on this. gobuster dir -u http://10.10.227.36/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

 
 
 There is a /images page but we dont have permissions to access it. THere are lot of such pages with 301 return codes. /readme and /license have some text but not helpful

 
 
 
 /robots has the hint for first key. robots
 
 
 
![dhcc](https://user-images.githubusercontent.com/64806211/127407129-268b1fc7-ad9f-4fc2-b07b-54edc12c14bf.png)




  
  
  
