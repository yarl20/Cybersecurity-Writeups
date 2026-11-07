<img width="580" height="235" alt="image" src="https://github.com/user-attachments/assets/2c4039bc-e0de-40a0-ae46-d1f855718463" /><img width="509" height="137" alt="image" src="https://github.com/user-attachments/assets/c2c3ea59-3112-4ae3-bf63-c88109d4094a" /># OverTheWire Bandit — Niveles 0 al 10

## Nivel 0 → 1

### Objetivo
Conectarse al servidor de OverTheWire Bandit mediante SSH y localizar la contraseña del siguiente nivel en el archivo `readme`.

### Entorno utilizado
- Sistema operativo: Kali Linux
- Protocolo: SSH
- Servidor: bandit.labs.overthewire.org
- Puerto: 2220
- Usuario: bandit0

### Procedimiento

#### 1. Conexión SSH
```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```
El comando ssh establece una conexión remota cifrada. El parámetro -p 2220 especifica el puerto del servidor.

#### 2. Enumeración de archivos
```bash
ls
```
El resultado mostró un archivo llamado readme.

#### 3. Lectura del archivo
```bash
cat readme
```
El comando cat permitió visualizar el contenido del archivo, donde se encontraba la contraseña del usuario bandit1. La contraseña se omite intencionalmente.

### Resultado
Nivel completado correctamente. Se obtuvo la información necesaria para continuar al siguiente nivel.

### Conocimientos adquiridos
- Conexión remota mediante SSH.
- Uso de puertos personalizados.
- Enumeración de archivos con ls.
- Lectura de archivos con cat.

### Evidencias
<img width="565" height="238" alt="image" src="https://github.com/user-attachments/assets/eb392d1a-f4e3-4fdc-915a-431ad16265c7" />
<img width="500" height="136" alt="image" src="https://github.com/user-attachments/assets/5080716d-956f-4ca4-8d7c-3a68ccdc3ab4" />
<img width="627" height="122" alt="image" src="https://github.com/user-attachments/assets/ad03f8ab-de5c-44e8-94fb-2dba92856b5f" />

--------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Nivel 1 → 2

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en un archivo llamado `-` dentro del directorio personal de bandit1.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cat

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un archivo cuyo nombre es únicamente un guion (`-`).

**3. Lectura del archivo**

```bash
cat ./-
```

El prefijo `./` permite identificar el archivo como una ruta relativa. Esto evita que `cat` interprete el guion como una referencia a la entrada estándar.

### Resultado
Se obtuvo correctamente la contraseña para acceder al usuario bandit2. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Manejo de archivos con nombres especiales.
- Uso de rutas relativas en Linux.
- Diferencia entre una ruta de archivo y la entrada estándar.
- Uso de ls -la para enumerar archivos.

### Evidencias
<img width="564" height="235" alt="image" src="https://github.com/user-attachments/assets/74b68eb1-e82d-48b3-bf0f-e899be3ac938" />
<img width="509" height="137" alt="image" src="https://github.com/user-attachments/assets/060c3370-1b61-4547-b029-368eadd69749" />
<img width="220" height="21" alt="image" src="https://github.com/user-attachments/assets/04223ae7-d4f9-457d-ad00-f8686b2454d3" />


--------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Nivel 2 → 3

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en un archivo llamado `--spaces in this filename--` dentro del directorio personal de bandit2.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cat

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un archivo llamado `--spaces in this filename--`, cuyo nombre contiene espacios y comienza con dos guiones (`--`).

**3. Lectura del archivo**

```bash
cat ./"--spaces in this filename--"
```

Las comillas dobles (`" "`) permiten que el sistema interprete el nombre completo del archivo como un único argumento, evitando que los espacios sean interpretados como separadores.

El prefijo `./` indica que el archivo se encuentra en el directorio actual y evita que los guiones iniciales sean interpretados como opciones del comando `cat`.

### Resultado
Se identificó el archivo que contiene la contraseña para acceder al usuario bandit3 y se ejecutó el comando para leerlo. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Manejo de archivos cuyos nombres contienen espacios.
- Uso de comillas dobles para interpretar nombres de archivos.
- Uso de rutas relativas en Linux.
- Manejo de archivos que comienzan con guiones.
- Uso de ls -la para enumerar archivos.

### Evidencias
<img width="563" height="246" alt="image" src="https://github.com/user-attachments/assets/1f803f61-f9e5-4e68-b01e-3daa3783723a" />
<img width="621" height="134" alt="image" src="https://github.com/user-attachments/assets/90232072-3e9b-4eca-8d0d-55472fc3c5ab" />
<img width="436" height="16" alt="image" src="https://github.com/user-attachments/assets/3d17ab37-834e-4003-923c-0175e6a8ecfb" />


--------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Nivel 3 → 4

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en un archivo oculto dentro del directorio `inhere`, ubicado en el directorio personal de bandit3.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cd, cat

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un directorio llamado `inhere` dentro del directorio personal del usuario bandit3.

**3. Acceso al directorio**

```bash
cd inhere
```

El comando `cd` permite cambiar de directorio. En este caso, se utilizó para ingresar a la carpeta `inhere`.

**4. Identificación de archivos ocultos**

```bash
ls -la
```

Se identificó un archivo oculto llamado `...Hiding-From-You`.

En Linux, los archivos cuyos nombres comienzan con un punto (`.`) se consideran ocultos y no aparecen al ejecutar `ls` sin la opción `-a`.

**5. Lectura del archivo oculto**

```bash
cat ./...Hiding-From-You
```

El comando `cat` permite visualizar el contenido del archivo.

El prefijo `./` indica que el archivo se encuentra en el directorio actual, mientras que los tres puntos iniciales forman parte del nombre del archivo.

### Resultado
Se localizó el archivo oculto que contiene la contraseña para acceder al usuario bandit4 y se ejecutó el comando para leerlo. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Identificación de archivos y directorios ocultos en Linux.
- Uso del comando cd para navegar entre directorios.
- Uso de ls -la para mostrar archivos ocultos y sus permisos.
- Lectura de archivos mediante el comando cat.
- Uso de rutas relativas para acceder a archivos ocultos.

### Evidencias
<img width="578" height="235" alt="image" src="https://github.com/user-attachments/assets/ebba01be-769f-4a4e-b865-a281689b5907" />
<img width="461" height="133" alt="image" src="https://github.com/user-attachments/assets/a58817fd-4aec-4687-a00a-110ecbbad88d" />
<img width="559" height="113" alt="image" src="https://github.com/user-attachments/assets/daac00dc-06bd-4369-af75-a581d1601dcc" />
<img width="424" height="18" alt="image" src="https://github.com/user-attachments/assets/9c18ad9d-4602-4946-9d9e-660caf6eefe9" />


--------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Nivel 4 → 5

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en el único archivo legible por humanos dentro del directorio `inhere`, ubicado en el directorio personal de bandit4.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cd, file, cat

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un directorio llamado `inhere` dentro del directorio personal del usuario bandit4.

**3. Acceso al directorio**

```bash
cd inhere
ls -la
```

Se encontraron diez archivos, desde `-file00` hasta `-file09`, cuyos nombres comienzan con un guion.

**4. Identificación del tipo de archivo**

Para determinar cuáles contienen información legible, se utilizó el comando `file`:

```bash
file ./-file*
```

El comando `file` permite identificar el tipo de contenido de los archivos.

El comodín `*` selecciona todos los archivos cuyos nombres coinciden con el patrón indicado, mientras que el prefijo `./` evita que los guiones iniciales sean interpretados como opciones.

En la práctica, también se verificaron individualmente los archivos mediante comandos como:

```bash
file ./-file00
file ./-file01
file ./-file02
```

Tras analizar los archivos, se identificó que `-file07` contenía texto ASCII:

```text
./-file07: ASCII text
```

**5. Lectura del archivo**

```bash
cat ./-file07
```

El comando `cat` permite visualizar el contenido del archivo identificado.

El prefijo `./` indica que el archivo está ubicado en el directorio actual y evita que el guion inicial de su nombre sea interpretado como una opción.

### Resultado
Se identificó correctamente el archivo `-file07` como el único archivo de texto ASCII y se ejecutó el comando para obtener la contraseña del usuario bandit5. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Identificación de tipos de archivos mediante el comando file.
- Diferencia entre archivos de texto y datos binarios.
- Uso de comodines (*) para seleccionar múltiples archivos.
- Enumeración de archivos mediante ls -la.
- Manejo de archivos cuyos nombres comienzan con guiones.
- Lectura de archivos mediante el comando cat.

### Evidencias
<img width="573" height="243" alt="image" src="https://github.com/user-attachments/assets/a09c8335-67ae-4b5b-94a9-3097de6b7e7b" />
<img width="461" height="132" alt="image" src="https://github.com/user-attachments/assets/fb08061b-a9aa-4025-bfb1-2da9f0385a9e" />
<img width="465" height="241" alt="image" src="https://github.com/user-attachments/assets/4d3f0bee-08ad-4674-ac97-5b901fb39c2d" />
<img width="623" height="336" alt="image" src="https://github.com/user-attachments/assets/e79d5ebf-7ca1-4419-9cbf-d24d36cacf81" />

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## Nivel 5 → 6

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en un archivo dentro del directorio `inhere`, que cumple las siguientes características:

- Es legible por humanos.
- Tiene un tamaño de 1033 bytes.
- No es ejecutable.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cd, find, cat

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un directorio llamado `inhere` dentro del directorio personal del usuario bandit5.

**3. Acceso al directorio**

```bash
cd inhere
ls -la
```

Se encontraron veinte subdirectorios, desde `maybehere00` hasta `maybehere19`.

Debido a la cantidad de directorios y archivos, se utilizó el comando `find` para localizar el archivo que cumpliera las condiciones solicitadas.

**4. Búsqueda del archivo**

```bash
find . -type f -size 1033c ! -executable
```

Explicación del comando:

- `find .`: realiza una búsqueda recursiva desde el directorio actual.
- `-type f`: busca únicamente archivos regulares.
- `-size 1033c`: selecciona archivos cuyo tamaño es exactamente 1033 bytes.
- `! -executable`: excluye los archivos ejecutables por el usuario actual.

Como resultado, se identificó el siguiente archivo:

```text
./maybehere07/.file2
```

El archivo se encontraba oculto dentro del subdirectorio `maybehere07`.

**5. Lectura del archivo**

```bash
cat ./maybehere07/.file2
```

El comando `cat` permite visualizar el contenido del archivo identificado.

La ruta `./maybehere07/.file2` indica que el archivo se encuentra dentro del directorio `maybehere07`. Su nombre comienza con un punto (`.`), por lo que se considera oculto en Linux.

### Resultado
Se identificó correctamente el archivo `.file2` dentro del directorio `maybehere07` mediante el uso del comando `find` con filtros de tipo, tamaño y permisos de ejecución.

Posteriormente, se ejecutó el comando `cat` para leer el archivo que contiene la contraseña correspondiente al usuario bandit6. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Búsqueda recursiva de archivos mediante el comando find.
- Filtrado de archivos según su tipo y tamaño.
- Uso del operador de negación (!) en comandos de Linux.
- Identificación de archivos no ejecutables.
- Localización de archivos ocultos dentro de subdirectorios.
- Uso de rutas relativas para acceder a archivos.
- Lectura de archivos mediante el comando cat.

### Evidencias
<img width="571" height="237" alt="image" src="https://github.com/user-attachments/assets/201de9fb-9dfb-42ce-913b-997c6164971f" />
<img width="490" height="137" alt="image" src="https://github.com/user-attachments/assets/a6445985-c927-4c87-aa0a-6ea848cf8c8b" />
<img width="478" height="385" alt="image" src="https://github.com/user-attachments/assets/de1c2c58-45f9-4326-a380-278a5900282c" />
<img width="537" height="32" alt="image" src="https://github.com/user-attachments/assets/a1395398-2eb1-4341-bd88-aeeb53eca8a0" />
<img width="415" height="16" alt="image" src="https://github.com/user-attachments/assets/6cd441c0-64fc-47fc-a82b-16b07c7bbb54" />

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## Nivel 6 → 7

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en un archivo ubicado en cualquier parte del servidor, que cumple las siguientes características:

- Pertenece al usuario bandit7.
- Pertenece al grupo bandit6.
- Tiene un tamaño exacto de 33 bytes.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, find, cat
- Redirección de errores: 2>/dev/null

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se realizó una enumeración del directorio personal del usuario bandit6. No se identificó directamente el archivo solicitado, por lo que fue necesario realizar una búsqueda en todo el sistema.

**3. Búsqueda del archivo**

```bash
find / -type f -user bandit7 -group bandit6 -size 33c
```

Explicación del comando:

- `find /`: realiza una búsqueda recursiva desde la raíz del sistema.
- `-type f`: busca únicamente archivos regulares.
- `-user bandit7`: filtra archivos pertenecientes al usuario bandit7.
- `-group bandit6`: selecciona archivos pertenecientes al grupo bandit6.
- `-size 33c`: busca archivos de exactamente 33 bytes.

Durante la ejecución aparecieron múltiples mensajes `Permission denied`, debido a que el usuario bandit6 no tiene permisos para acceder a determinados directorios del sistema.

**4. Redirección de errores**

Para evitar que los mensajes de permisos dificultaran la identificación del archivo, se utilizó:

```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

El operador `2>` redirige la salida de errores estándar (stderr), mientras que `/dev/null` descarta la información enviada.

De esta manera, se ocultaron los mensajes de error y se obtuvo el siguiente resultado:

```text
/var/lib/dpkg/info/bandit7.password
```

**5. Lectura del archivo**

```bash
cat /var/lib/dpkg/info/bandit7.password
```

El comando `cat` permite visualizar el contenido del archivo identificado.

Se utilizó una ruta absoluta, ya que el archivo se encontraba fuera del directorio personal del usuario bandit6.

### Resultado
Se identificó correctamente el archivo `bandit7.password` dentro de la ruta `/var/lib/dpkg/info/`, mediante filtros de propietario, grupo y tamaño.

Posteriormente, se ejecutó el comando para leer el archivo que contiene la contraseña correspondiente al usuario bandit7. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Búsqueda recursiva de archivos en todo el sistema Linux.
- Filtrado de archivos según su propietario y grupo.
- Identificación de archivos mediante su tamaño exacto.
- Uso del comando find con múltiples condiciones.
- Interpretación de errores de permisos en Linux.
- Redirección de errores mediante 2>/dev/null.
- Diferencia entre rutas absolutas y relativas.
- Lectura de archivos mediante el comando cat.

### Evidencias
<img width="581" height="235" alt="image" src="https://github.com/user-attachments/assets/d2773608-118b-4dd9-8b75-4ae52aa98c28" />
<img width="462" height="119" alt="image" src="https://github.com/user-attachments/assets/51d3758d-1a6d-4c4e-b333-8d77fd139421" />
<img width="590" height="643" alt="image" src="https://github.com/user-attachments/assets/bb5abfb7-548a-4dac-bc5e-0d4b52baf492" />
<img width="682" height="34" alt="image" src="https://github.com/user-attachments/assets/a77901d2-54e7-4917-aeb0-80f9ddc29a8a" />

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## Nivel 7 → 8

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en el archivo `data.txt`, junto a la palabra `millionth`.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cat, grep

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un archivo llamado `data.txt` dentro del directorio personal del usuario bandit7.

**3. Exploración del contenido**

```bash
cat data.txt
```

El comando `cat` permite visualizar el contenido del archivo.

Se observó que `data.txt` contenía numerosas líneas de texto, cada una formada por una palabra y una cadena de caracteres.

Debido a la gran cantidad de información, resultaba poco práctico localizar manualmente la línea solicitada.

**4. Búsqueda de la contraseña**

```bash
grep "millionth" data.txt
```

Explicación del comando:

- `grep`: busca líneas que coinciden con un patrón de texto.
- `"millionth"`: indica la palabra que se desea localizar.
- `data.txt`: especifica el archivo donde se realizará la búsqueda.

El comando permite filtrar el contenido del archivo y mostrar únicamente las líneas que contienen la palabra `millionth`.

De esta manera, se localiza la línea donde se encuentra la contraseña del siguiente nivel.

### Resultado
Se identificó el archivo `data.txt` y se utilizó el comando `grep` para buscar la palabra `millionth`, asociada a la contraseña del usuario bandit8.

La credencial se omite por seguridad.

### Conocimientos adquiridos
- Búsqueda de patrones de texto mediante grep.
- Filtrado de información dentro de archivos.
- Identificación de información específica en archivos extensos.
- Uso del comando cat para visualizar archivos de texto.
- Uso de ls -la para enumerar archivos.
- Diferencia entre visualizar todo el contenido y realizar una búsqueda filtrada.

### Evidencias
<img width="559" height="239" alt="image" src="https://github.com/user-attachments/assets/062b436c-393d-40ac-86b1-ac0d0ff40dfc" />
<img width="534" height="136" alt="image" src="https://github.com/user-attachments/assets/c1050e1e-dbc2-4bd7-a0f9-6bec16c8bf43" />
<img width="463" height="727" alt="image" src="https://github.com/user-attachments/assets/e93af00f-ef2e-4a6c-94ee-455bf8834f0e" />
<img width="353" height="15" alt="image" src="https://github.com/user-attachments/assets/dd773244-9ed6-4577-bf6a-ce7025728564" />

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## Nivel 8 → 9

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en el archivo `data.txt`, siendo la única línea de texto que aparece una sola vez dentro del archivo.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cat, sort, uniq

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un archivo llamado `data.txt` dentro del directorio personal del usuario bandit8.

**3. Exploración del contenido**

```bash
cat data.txt
```

El comando `cat` permite visualizar el contenido del archivo.

Se observó que `data.txt` contenía numerosas líneas de texto, muchas de ellas repetidas. Debido a la cantidad de información, resultaba poco práctico identificar manualmente la única línea que aparecía una sola vez.

**4. Identificación de la línea única**

```bash
sort data.txt | uniq -u
```

Explicación del comando:

- `sort data.txt`: ordena las líneas del archivo, agrupando las que tienen el mismo contenido.
- `|`: envía la salida del primer comando como entrada del siguiente.
- `uniq -u`: muestra únicamente las líneas que aparecen una sola vez.

El comando `uniq` compara líneas consecutivas, por lo que es necesario ordenar previamente el contenido mediante `sort` para identificar correctamente las líneas repetidas.

De esta manera, se filtra la información y se identifica la única línea que contiene la contraseña del siguiente nivel.

### Resultado
Se identificó el archivo `data.txt` y se ejecutó correctamente la combinación de comandos `sort` y `uniq -u` para localizar la línea única correspondiente a la contraseña del usuario bandit9.

La credencial se omite por seguridad.

### Conocimientos adquiridos
- Ordenamiento de líneas de texto mediante sort.
- Identificación de líneas únicas utilizando uniq.
- Uso del parámetro -u para filtrar elementos no repetidos.
- Combinación de comandos mediante tuberías (|).
- Procesamiento y filtrado de archivos de texto.
- Importancia de ordenar los datos antes de utilizar uniq.
- Uso de ls -la para enumerar archivos.

### Evidencias
<img width="575" height="240" alt="image" src="https://github.com/user-attachments/assets/06e8b372-4c10-4c04-956d-b0cef7685843" />
<img width="528" height="132" alt="image" src="https://github.com/user-attachments/assets/fcfef369-32a6-4e0d-a92c-81854c80958c" />
<img width="313" height="723" alt="image" src="https://github.com/user-attachments/assets/c09bfbdc-92c6-4b6e-bd16-fc62df6b5edd" />
<img width="341" height="17" alt="image" src="https://github.com/user-attachments/assets/912db18d-365b-4313-82ff-e98c5d42177c" />

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## Nivel 9 → 10

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en el archivo `data.txt`, entre las pocas cadenas de texto legibles que están precedidas por varios signos de igualdad (`=`).

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cat, strings, grep

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un archivo llamado `data.txt` dentro del directorio personal del usuario bandit9.

**3. Exploración del contenido**

```bash
cat data.txt
```

El comando `cat` permite visualizar el contenido de un archivo.

Se observó que `data.txt` contenía principalmente datos binarios, representados en la terminal mediante caracteres especiales y símbolos difíciles de interpretar.

Debido a que el contenido no era completamente legible, fue necesario utilizar una herramienta que permitiera extraer las cadenas de texto.

**4. Extracción y filtrado de cadenas legibles**

```bash
strings data.txt | grep "=="
```

Explicación del comando:

- `strings data.txt`: extrae las secuencias de caracteres imprimibles presentes en el archivo.
- `|`: envía la salida del primer comando como entrada del siguiente.
- `grep "=="`: filtra las líneas que contienen dos signos de igualdad consecutivos.

El comando `strings` permite identificar texto legible dentro de archivos que contienen datos binarios.

Posteriormente, `grep` filtra la información para mostrar las cadenas que contienen el patrón indicado.

De esta manera, se pueden localizar las líneas relevantes entre las cuales se encuentra la contraseña correspondiente al siguiente nivel.

### Resultado
Se identificó el archivo `data.txt` y se ejecutó la combinación de comandos `strings` y `grep` para localizar cadenas legibles precedidas por signos de igualdad.

Este procedimiento permite identificar la contraseña correspondiente al usuario bandit10. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Identificación de información legible dentro de archivos binarios.
- Extracción de cadenas de texto mediante strings.
- Búsqueda de patrones utilizando grep.
- Combinación de comandos mediante tuberías (|).
- Filtrado de información relevante en archivos con contenido no legible.
- Diferencia entre archivos de texto y contenido binario.
- Uso de ls -la para enumerar archivos.

### Evidencias
<img width="557" height="241" alt="image" src="https://github.com/user-attachments/assets/5540cc97-1099-4564-8468-063650346356" />
<img width="529" height="131" alt="image" src="https://github.com/user-attachments/assets/523f592d-13de-44a9-b26e-81a86c6f09a3" />
<img width="774" height="454" alt="image" src="https://github.com/user-attachments/assets/61e92ee4-d489-4f2e-b5a6-5572cff481f5" />

---------------------------------------------------------------------------------------------------------------------------------------------------------------

## Nivel 10 → 11

### Objetivo
Encontrar la contraseña del siguiente nivel, almacenada en el archivo `data.txt`, cuyo contenido se encuentra codificado en Base64.

### Herramientas utilizadas
- Kali Linux
- SSH
- Comandos: ls, cat, base64

### Procedimiento

**1. Conexión SSH**

```bash
ssh bandit10@bandit.labs.overthewire.org -p 2220
```

**2. Enumeración de archivos**

```bash
ls -la
```

Se identificó un archivo llamado `data.txt` dentro del directorio personal del usuario bandit10.

**3. Exploración del contenido**

```bash
cat data.txt
```

El comando `cat` permite visualizar el contenido de un archivo.

Se observó que `data.txt` contenía una cadena de caracteres codificada en Base64, formada por letras, números y símbolos propios de este formato.

Base64 es un método de codificación que permite representar información binaria mediante caracteres de texto. No es un método de cifrado, ya que su contenido puede decodificarse sin necesidad de una clave secreta.

**4. Decodificación del archivo**

```bash
base64 -d data.txt
```

Explicación del comando:

- `base64`: herramienta utilizada para codificar y decodificar información en formato Base64.
- `-d`: indica que se desea decodificar el contenido.
- `data.txt`: especifica el archivo que contiene la información codificada.

El comando permite transformar la cadena Base64 en su contenido original, donde se encuentra la contraseña correspondiente al siguiente nivel.

### Resultado
Se identificó el archivo `data.txt` y se ejecutó correctamente el comando `base64 -d` para decodificar su contenido.

Este procedimiento permite recuperar la contraseña correspondiente al usuario bandit11. La credencial se omite por seguridad.

### Conocimientos adquiridos
- Identificación de información codificada en Base64.
- Decodificación de archivos mediante el comando base64.
- Uso del parámetro -d para decodificar información.
- Diferencia entre codificación y cifrado.
- Visualización de archivos mediante cat.
- Procesamiento de información desde la terminal de Linux.
- Uso de ls -la para enumerar archivos.

### Evidencias
<img width="564" height="238" alt="image" src="https://github.com/user-attachments/assets/f7509ee7-5de6-485d-a540-f9b1dee84648" />
<img width="524" height="133" alt="image" src="https://github.com/user-attachments/assets/110fd198-c760-4e70-8f4f-1eaf0712bc4a" />
<img width="569" height="36" alt="image" src="https://github.com/user-attachments/assets/8c719640-d948-4b1a-bbc6-1f86428534c1" />
<img width="318" height="16" alt="image" src="https://github.com/user-attachments/assets/3b0bca11-106b-4b62-9542-b9a1e9495d90" />



