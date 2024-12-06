Writeup: AnonymousPingu
 **Autor:** David Cardozo
 **Fecha de Desarrollo:** 05/12/24
 **Plataforma:** DockerLabs
 **Nivel de Dificultad:** Facil
 
![[Pasted image 20241206000729.png]]
### **Temáticas Tratadas:**
- FTP
- Upload File
- Escalacion Horizontal

---

## 1. Descripción General

En esta máquina, logramos subir un archivo malicioso a través del servicio FTP, el cual luego ejecutamos mediante el servidor web para obtener una shell reversa. Posteriormente, realizamos una escalación de privilegios progresiva, pasando por diferentes usuarios en el sistema, hasta finalmente obtener acceso como usuario `root`.

---

## 2. Reconocimiento

### **Reconocimiento Inicial**
- **Escaneo de puertos:** 21, 80 
- **Servicios encontrados:** FTP, HTTP
![[Pasted image 20241205234007.png]]
En esta máquina, identificamos que el servicio FTP permitía acceso con el usuario anonymous, lo que nos permitió explorar el sistema y subir un archivo malicioso. mediante el comando `put`, en la carpte `upload` la cual nos permitia la subida del archivo `.php`. 

![[Pasted image 20241205234342.png]]

Al analizar el servicio FTP, notamos que no era necesario realizar fuzzing, ya que directamente podíamos observar el directorio al cual teníamos acceso desde la web. Esto facilitó la identificación del punto exacto para cargar y ejecutar nuestro archivo malicioso. Subimos un archivo que nos permitió obtener una reverse shell, asegurándonos de configurar previamente nuestra máquina en escucha en el puerto 443 antes de ejecutarlo. Este enfoque directo simplificó el proceso de intrusión inicial.

![[Pasted image 20241205234516.png]]
![[Pasted image 20241205234531.png]]
![[Pasted image 20241205234551.png]]
Una vez obtenida la reverse shell, procedemos a mejorar la TTY para tener un entorno más funcional. Esto lo logramos utilizando los siguientes comandos

```bash
script /dev/null -c bash
Ctrl + z
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

---

## 3. Explotación

### **Fase de Explotación**
Después de obtener nuestra reverse shell, ejecutamos el comando `sudo -l` para identificar posibles configuraciones que nos permitieran abusar de privilegios elevados. Al analizar los resultados, descubrimos que el usuario `pingu` tenía permisos para ejecutar el comando `man` con privilegios de `sudo`. Esto nos proporcionó una oportunidad clara para escalar privilegios

![[Pasted image 20241205235037.png]]
Por lo que mediante GTOFBins podemos ver cuales comando nos ayudaran a escalar estos privilegios [https://gtfobins.github.io/]. A continuacion los comandos que ultilizaremos.

```
sudo -u pingu man man
!/bin/bash
```

![[Pasted image 20241205235634.png]]
Una vez escalamos privilegios a través de `pingu`, ejecutamos nuevamente el comando `sudo -l` para analizar las configuraciones de otros usuarios. Observamos que el usuario `gladys` tenía permisos para ejecutar los comandos `nmap` y `dpkg` con privilegios elevados. Dado que `dpkg` puede ser utilizado para instalar paquetes y ejecutar scripts con privilegios de `root`, decidimos abusar de esta configuración para escalar aún más nuestros privilegios. A continuacion los comandos para escalar como gladys.

```bash
sudo -u gladys dpkg -l
!/bin/bash
```

![[Pasted image 20241206000032.png]]

---

## 4. Escalada de Privilegios

Después de convertirnos en el usuario gladys, ejecutamos nuevamente el comando sudo -l para revisar qué acciones podíamos realizar con privilegios elevados. En esta ocasión, descubrimos que teníamos permisos para ejecutar el comando chown, lo cual nos permitió manipular la propiedad de archivos y directorios en el sistema.

Aprovechando esto, decidimos escalar privilegios creando un nuevo usuario con permisos de root. Para hacerlo, editamos el archivo /etc/passwd, que contiene la información sobre las cuentas de usuario del sistema.  Comandos para escalar como root.

```bash
sudo -u root chown gladys /etc/passwd
echo 'hola::0:0:root:/root:/bin/bash' >> /etc/passwd
su hola
Dejamos el espacio de contraseña vacio
```

![[Pasted image 20241206000531.png]]

Y listo con eso ya nos hemos convertido en usuario root.

