# **Lame writeup for OSCP/PREP by MCRUZ**
### Initial Scan with nmap


``sudo nmap -p- --min-rate 10000 -oA initial $ip``

> PORT     STATE SERVICE\
21/tcp   open  ftp\
22/tcp   open  ssh\
139/tcp  open  netbios-ssn\
445/tcp  open  microsoft-ds\
3632/tcp open  distccd



