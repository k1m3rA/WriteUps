# Lazy Admin WriteUp
### Español & English



## Enumeración // Enumeration

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
Como podemos observar hay un servicio de http en el puerto 80 al que podemos acceder desde nuestro navegador.



