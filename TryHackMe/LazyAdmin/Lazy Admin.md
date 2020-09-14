# Lazy Admin WriteUp
### Español & English



## Enumeración

#### Nmap - Enumeración de puertos
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

#### Wfuzz - Enumeración de directorios

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





