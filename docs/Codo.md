# Machine info
**Name:** Codo  
**OS:** Linux    
**Difficulty:**  Easy  
**Complexity:**  Low


## Phase 1: Reconnaissance 
To begin, we spawn in the machine and wait for it to assign an IP address. In this case, it appears we will be attacking `192.168.132.23`  

## Phase 2: Scanning
To begin, I run my enumeration script - which simply runs a quick nmap, a more detailed nmap, and nmap on the top 100 udp ports - then outputs into a folder I created named nmapscans.  
(enumeration 1)  
While the more detailed scan is running, I opt to take some time to check out the web server on port 80.  
(enumeration 2)  
Visiting `http://192.168.132.23` - we are met with a webpage titled "CODOLOGIC". We can take note that this is powered by "Codoforum". There appears to be a link, register, and login page in the upper right hand corner. We can additionally see that the user 'admin' made a post called "Welcome to Codoforum". In addition to the NMAP scan, I had started a simple GoBuster scan to enumerate sub directories of the web server.  
(enumeration 3)  
We can see that there a few interesting directories that we can explore. I begin by checking out `sys`. After combing through these files, there was no juicy information. Moving onto `cache`, and `sites` the same holds true.  
(enumeration 4)  
(enumeration 5)  
(enumeration 6)  
Finally, we get to `admin`. I checked the source to see if I could gather a version or build number, with no luck.  
(enumeration 7)  
(enumeration 8)  
To our luck, after trying a few basic common passwords manually - `admin:admin` appears to get us into the login! From here we are able to determine the version `V.5.1.105`.  
(enumeration 9)  


## Phase 3: Vulnerability verification & Exploitation
In the terminal, I check to see if searchsploit has any known exploits - which it does! `CodoForum v5.1 - Remote Code Execution (RCE)` exists - so I opt to try it out and make a copy.  
(enumeration 10)  
Reviewing the code, it seems pretty straight forward - it takes arguments in the form of a `target-url`, `username`, `password`, `listener-ip` and `port`. 
(enumeration 11)  
In the terminal, I make sure that python3 will work with this script.  
(Enumeration 12)  
Finally, I attempt to run it:  
(enumeration 13)  
Failure. No worries - it looks like we are able to do this manually! I navigate to https://www.revshells.com and pull the `PHP Ivan Sincek` reverse shell, using `/bin/bash`. I opt to use `443` as the listening port because it is less likely to be blocked and noticed. Uploading this via the directions in the terminal, we land a shell as the user `www-data` - Nice!  
(exploitation 1)  
(exploitation 2)  
I take a moment and upgrade my shell after determining python3 is located at `/usr/bin/python3`.  
(exploitation 3)  
Checking out the `/etc/passwd` file, we can see that the user `offsec` exists.  
(post-exploitation-1)  
Moving to `/` we can don't see anything out of the ordinary.  
(post-exploitation-2)  
Checking out `/var/www` we can see the directories we previously were exploring via browser. 
(post-exploitation-3)  
After poking around here - I move to the temp directory, make a folder named `k1ret` to keep it tidy and transfer over `linpeas.sh`, `PwnKit` & `lse.sh`
(post-exploitation-4)  
(post-exploitation-5)  
Running through linpeas, we notice that one of the files `/var/www/html/sites/default/config.php` has a password notated.  
(post-exploitation-6)  



## Phase 4: Maintaining access & privlege escalation
I open up this config file and see that this is actually the password to for mariadb.  
(post-exploitation-7)  
Connecting to mariadb - I list the databases and choose to review codoforums in case anything stands out.  
(post-exploitation-8)  
(post-exploitation-9)  
After chasing this - I realized that this was not the intended path for priv esc. Out of curiousity, I tried the password against the root user and discovered I could switch the user to root with this password!   
(post-exploitation-10)  
From here, we have numerous ways that we can maintain access on the system. We could generate our own public/private key pair for ssh since root is permitted to sign in via SSH (we can change it if it wasnt). We could set up a cron job that verifies our connection to a C2 server and reconnects if lost. It is truly dealers choice. 
  
## Phase 5: Reporting & Documentation  
Initially, we were able to leverage default credentials to access the admin dashboard for Codoforum. From here we could likely have found a PoC that worked with our credentials, but were able to land a reverse shell manually via a malicious php file that was uploaded as the forum logo. We were then able to discover the mariadb password configured `config.php` in  was also utilized by the root user and achieve root access without furhter complex privlege. There are several ways in which the system admin could have protected against these attacks including updating Codoforums to a patched version, using complex passwords on all accounts, and through the practice of not re-using passwords.
