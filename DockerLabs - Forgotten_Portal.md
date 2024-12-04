**Writeup:** Forgotten_Portal
 **Autor:** David Cardozo
 **Fecha de Desarrollo:** 4/12/2024
 **Plataforma:** DockerLab  
 **Nivel de Dificultad:** Medio  
### **Temáticas Tratadas:**
- SSH
- Binario Tar
- Base64

---

## 1. Descripción General:

La máquina en cuestión presenta los puertos 22 (SSH) y 80 (HTTP) abiertos. Tras realizar un escaneo de dominios, logramos identificar el archivo access_log en el servidor web, el cual contenía una clave codificada en Base64 asociada a un usuario con acceso SSH. Decodificando esta clave, obtuvimos las credenciales para acceder al servidor como dicho usuario.

Una vez dentro del sistema con acceso SSH, procedimos a escalar privilegios al usuario bob. A partir de ahí, mediante la explotación de una vulnerabilidad, logramos obtener acceso completo y elevar nuestros privilegios a nivel de root, logrando así la toma total del sistema.

---

## 2. Reconocimiento:

### **Reconocimiento Inicial**:
- **Escaneo de puertos:** 22, 80
- **Servicios encontrados:** SSH, HTTP

![[Pasted image 20241204142433.png]]

### Pagina Web:

![[Pasted image 20241204142636.png]]

### Fuzzzing:
Procedimos a realizar un escaneo de los dominios activos en la página web de la máquina con la IP **172.17.0.2**. Durante el análisis, encontramos un dominio particularmente relevante denominado access_log, lo que nos llamó la atención y nos llevó a investigar más a fondo.

![[Pasted image 20241204142734.png]]

Al inspeccionar el contenido de dicho dominio, descubrimos que contenía un valor cifrado en Base64. Al decodificarlo, obtuvimos la contraseña de un usuario denominado Alice, lo que nos permitió avanzar en la explotación de la máquina.

![[Pasted image 20241204142919.png]]
![[Pasted image 20241204143031.png]]

---

## 3. Explotación:

### **Fase de Explotación**:
En esta fase, utilizamos la clave obtenida previamente para acceder al sistema a través de SSH, iniciando sesión como el usuario Alice.

![[Pasted image 20241204143251.png]]
Al navegar por el sistema, encontramos una carpeta que nos llamó la atención. Al acceder a ella, descubrimos un archivo que contenía información relevante, lo que nos permitió escalar privilegios al usuario Bob.

![[Pasted image 20241204143358.png]]

Al observar que las claves id_rsa eran idénticas para todos los usuarios, decidimos descargar la clave id_rsa del usuario Alice. Posteriormente, ajustamos los permisos de la clave y la utilizamos para acceder al sistema como el usuario Bob, utilizando la passphrase asociada.

![[Pasted image 20241204143551.png]]

![[Pasted image 20241204143711.png]]

Al notar que las claves id_rsa eran compartidas entre todos los usuarios, descargamos la clave id_rsa del usuario Alice. Luego, ajustamos los permisos de la clave y la utilizamos para acceder al sistema como el usuario Bob, mediante la passphrase asociada.

![[Pasted image 20241204143931.png]]

---

## 4. Escalada de Privilegios Usuario Bob:

### **Escalada Local**:
Como usuario Bob, al ejecutar el comando `sudo -l`, descubrimos que podíamos ejecutar el comando `tar` sin necesidad de privilegios de root. Aprovechando esta información y consultando la página de GTFOBins, utilizamos este binario para escalar privilegios a root.

![[Pasted image 20241204144225.png]]
![[Pasted image 20241204144204.png]]

Finalmente, al ejecutar el comando proporcionado por GTFOBins, logramos escalar nuestros privilegios y obtener acceso como usuario root.

![[Pasted image 20241204144306.png]]

---

## 5. Obtención de la Bandera:

- **Flag User:** `CYBERLAND{us3r_l3v3l_fl4g}`
- **Flag Root:** `CYBERLAND{r00t_4cc3ss_gr4nt3d}`

---
