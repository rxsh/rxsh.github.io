---
title: "EasyPeasy"
date: 2023-06-08 17:00:23 +0800
categories: [TryHackMe]
tags: [TryHackMe]
---
![alt text](/assets/img/posts/EasyPeasyIMG/machine.png)

<p><center>WriteUp detallado explotando la máquina EasyPeasy de la plataforma de TryHackMe</center></p>.

## Reconocimiento

Realizamos un escaneo con nmap para ver los puertos abiertos.

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.25.102 -oG allPorts
```

![OPEN PORTS](/assets/img/posts/EasyPeasyIMG/image.png){: width="600"}

Aplicamos el siguiente comando para ver las versiones que estan corriendos dichos servicios con sus respectivos puertos que se encuentran abiertos y le pasamos un conjunto de script por default de nmap.

```bash
nmap -sCV -p80,6498,65524 10.10.25.102 -oN targeted
```

![VERSIONS PORTS](/assets/img/posts/EasyPeasyIMG/image-1.png){: width="600"}

Ahora vamos a buscar directorios para ver si encontramos una ruta escondida que nos ayude a encontrar algo.

```bash
gobuster dir -u http://10.10.25.102/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
```

## Explotación

![DIRECTORIO ESCONDIDO](/assets/img/posts/EasyPeasyIMG/image-2.png){: width="600"}

Encontramos el directorio `/hidden` entrando veremos una página con una imagen solamente y no vemos nada en su código fuente así que nuevamente repetiremos el proceso pero ahora cambiando a la ruta actual.

```bash
gobuster dir -u http://10.10.25.102/hidden/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
```
Encontramos una ruta dentro de `hidden` y al entrar a este nuevo directorio vemos que en su código fuente nos da una flag pero encriptada en `base64` por lo que se puede apreciar.

![DIRECTORIO ESCONCIDO EN HIDDEN](/assets/img/posts/EasyPeasyIMG/image-3.png){: width="600"}


![FLAG EN BASE64](/assets/img/posts/EasyPeasyIMG/image-4.png){: width="600"}

En nuestra terminal decodeamos en base64 para ver que mensaje nos da.

```bash
echo ZmxhZ3tmMXJzN19mbDRnfQ== | base64 -d; echo
```

![FLAG DESENCRIPTADA](/assets/img/posts/EasyPeasyIMG/image-5.png){: width="600"}

Ya habiendo desencriptado dicho texto ya nos da la primera flag.
Ahora tenemos que seguir buscando por otros lados porque ya no encontramos mas nada en ese directorio.

Entrando a la URL con el puerto `http://VICTIM-MACHINE:65524` vemos que es una página default de Apache 2 pero leyendo bien encontramos la flag3 escondida.

![flag3](/assets/img/posts/EasyPeasyIMG/image-6.png){: width="600"}

```bash
gobuster dir -u http://10.10.25.102:65524 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,sh,html,txt
```

Al buscar directorios aparece la ruta de `http://VICTIM_MACHINE:65524/robots.txt`. Vemos que hay en la parte de User-Agent hay un texto medio raro que no podemos identificar bien que es pero es un texto hasheado en MD5 por lo que entramos a [MD5Hashing](https://md5hashing.net/) para pasarlo desencriptarlo.

![CADENA ENCRYPT](/assets/img/posts/EasyPeasyIMG/image-8.png){: width="600"}

![flag2 decrypt](/assets/img/posts/EasyPeasyIMG/image-7.png){: width="600"}

Listo. Tenemos la segunda flag, por lo que tenemos que buscar donde esta la tercera.
En la misma ruta de `http://VICTIM_MACHINE:65524` viendo el código fuente encontramos que esta escondido una flag media sospechosa que al parecer tambien esta encriptada pero no sabemos con que.

![base?](/assets/img/posts/EasyPeasyIMG/image-9.png){: width="600"}

Al parecer es base62 por lo que entramos a la siguiente página para desencriptarlo [base62-Decode](https://www.dcode.fr/base62-encoding)

![decode base62](/assets/img/posts/EasyPeasyIMG/image-10.png){: width="600"}

Viendo esto suponemos que es una ruta y lo añadimos a la ruta actual.

![code](/assets/img/posts/EasyPeasyIMG/image-11.png){: width="600"}

Pasamos el hash en un archivo en nuestra máquina para a través de john desencriptar con el diccionario con nos da la máquina en la plataforma de thm.

```bash
john --wordlist=easypeasy_1596838725703.txt --format=gost hash.txt 
```

Aplicamos el comando y vemos que el texto era una contraseña encriptada y esta misma contraseña nos va a servir mas adelante.

![decrypt password](/assets/img/posts/EasyPeasyIMG/image-12.png){: width="600"}

Descargamos la imagen, la pasamos a nuestro directorio de trabajo y ejecutaremos el siguiente comando

```bash
steghide extract -sf binarycodepixabay.jpg
```

Insertamos la contraseña que acabamos de desencriptar y vemos que nos crea un archivo `secrettext.txt` donde encontramos un username y password.

![secrettext.txt](/assets/img/posts/EasyPeasyIMG/image-13.png){: width="600"}


Como nos podemos dar cuenta la contraseña esta en formato binario y entonces buscamos la página de [cyberchef](https://gchq.github.io/CyberChef/) para pasarlo a formato texto y poder ver la contraseña.

![contraseñaBINARY](/assets/img/posts/EasyPeasyIMG/image-14.png){: width="600"}

Ya teniendo la contraseña y el usuario podemos conectarnos vía ssh en el puerto que vemos que corre el servicio ssh.

```bash
ssh -p 6498 boring@10.10.25.102
```

![BORING](/assets/img/posts/EasyPeasyIMG/image-15.png){: width="600"}

YYYY TENEMOS LA FLAG DE USER pero vemos que nos dice que esta rotada, buscamos en internet **rotated online decode** y pasamos esa flag para que nos de la contraseña correcta.

Encontramos la siguiente página que nos sirve para la flag rota [rot13](https://rot13.com/)

![userflagROT13](/assets/img/posts/EasyPeasyIMG/image-16.png){: width="600"}

![flagUSER](/assets/img/posts/EasyPeasyIMG/image-17.png){: width="400"}

## Privilege Escalation

---------------------------------------------------------

Ahora vamos a subir privilegios para encontrar la flag de root.

Viendo los permisos SUID y si hay tareas CRON o algunas credenciales ssh vemos que no encontramos nada peeeero en el directorio `var/www/` encontramos un archivo escondido

![escondidoFile](/assets/img/posts/EasyPeasyIMG/image-18.png){: width="600"}

![reverseShell](/assets/img/posts/EasyPeasyIMG/image-19.png){: width="600"}

Nos conectamos como ROOT!!!

![root!](/assets/img/posts/EasyPeasyIMG/image-20.png){: width="600"}

Buscamos en el directorio root haciendo un `ls -la` y vemos el fichero `.root.txt`

![rootFlag](/assets/img/posts/EasyPeasyIMG/image-21.png){: width="600"}

¡¡Obtenemos una flag, que estaba oculta en .root.txt!!

¡Ahora que tenemos todas nuestras respuestas, enviémoslas y hemos completado con éxito nuestro CTF!

Sigue intentándolo, sigue trabajando :)

¡Gracias por leer!

Feliz Hacking :D

¡Sígueme más para obtener más consejos y trucos! 🙏

```
Autor: rxsh
```