# h0 Sieppari ruispellossa
*First test or kind of like a pre-test to analyze network traffic*

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
**Sieppaa ja analysoi verkkoliikennettä.**

First i installed `tcpdump`

```bash
sudo apt-get install tcpdump
```

I have a device connected with usb, which tethers the ethernet connection, that i use as a system monitor and debug over ssh, but has no access to the public internet.   
I analyzed the traffic between my laptop and my monitoring device:   
```bash
~$ sudo tcpdump -i usb0 -v
tcpdump: listening on usb0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
09:53:31.059694 IP (tos 0x10, ttl 64, id 54679, offset 0, flags [DF], proto TCP (6), length 216)
    parasite.ssh > 192.168.186.86.42512: Flags [P.], cksum 0x22ac (correct), seq 793131884:793132048, ack 999448198, win 501, options [nop,nop,TS val 3409085973 ecr 1150749954], length 164
09:53:31.060438 IP (tos 0x10, ttl 64, id 54680, offset 0, flags [DF], proto TCP (6), length 248)
    parasite.ssh > 192.168.186.86.42512: Flags [P.], cksum 0xe5fa (correct), seq 164:360, ack 1, win 501, options [nop,nop,TS val 3409085973 ecr 1150749954], length 196
09:53:31.061553 IP (tos 0x48, ttl 64, id 13716, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.186.86.42512 > parasite.ssh: Flags [.], cksum 0x2eac (correct), ack 164, win 411, options [nop,nop,TS val 1150750453 ecr 3409085973], length 0
09:53:31.062078 IP (tos 0x48, ttl 64, id 13717, offset 0, flags [DF], proto TCP (6), length 52)
    192.168.186.86.42512 > parasite.ssh: Flags [.], cksum 0x2de8 (correct), ack 360, win 411, options [nop,nop,TS val 1150750453 ecr 3409085973], length 0
^C
4 packets captured
4 packets received by filter
0 packets dropped by kernel
``` 

So to analyze it:  
`tcpdump` is listening the given interface `usb0` which has Ethernet link.   
- First packet: Indicates that my laptop, with hostname `parasite` sent a packet to `192.168.186.86` port `42512` using `ssh`.  And if i understand correctly , the package contains `164` bytes worth of data.   
- Second packet: Same sender, same destination. `196` bytes worth of data.   
- Third packet: Acknowledgement from `192.168.186.86` . Indicates that `192.168.186.86`  has received the first packet from `parasite` via `ssh`.   
- Fourt packet: Also acknowledgement. Indicates that `192.168.186.86`  has received the second packet from `parasite` via `ssh`.   

Four packets captured, as summarized by `tcpdump` with zero dropped by kernel.

What i really understand in this, is that there is an `ssh` connection between a pc, with hostname `parasite` and a device with ip `192.168.186.86`.

