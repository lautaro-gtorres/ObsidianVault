---
tags: [SO, Linux, GNU, Ubuntu, UAI]
clase: C5
estado: ⬜
fuentes:
  - https://www.profesionalreview.com/2022/12/10/posix/
  - https://es.wikipedia.org/wiki/GNU/Linux
  - https://www.godaddy.com/resources/es/crearweb/que-es-ubuntu-y-para-que-sirve
---

# C5 — GNU/Linux y Ubuntu

← [[C4 - POSIX - Servicios para Ficheros]] | ← [[🏠 Inicio SO]]

---

## 🎯 Objetivos

Comprender qué es GNU/Linux, cómo se relaciona con POSIX, y qué es Ubuntu como distribución.

---

## 🐧 GNU/Linux

### ¿Qué es?

**GNU/Linux** es un sistema operativo de código abierto formado por dos componentes:

| Componente | Descripción |
|---|---|
| **Linux** | El **kernel** (núcleo), creado por Linus Torvalds en 1991 |
| **GNU** | El **conjunto de herramientas** y utilidades del sistema (compilador gcc, bash, libc, etc.), iniciado por Richard Stallman en 1983 |

> El nombre correcto es **GNU/Linux** porque el kernel Linux por sí solo no es un SO completo — necesita las herramientas GNU para ser usable.

### Historia resumida

```
1983 → Richard Stallman lanza el proyecto GNU (GNU's Not Unix)
1991 → Linus Torvalds publica el kernel Linux v0.01
1992 → Se combinan GNU + Linux → primer sistema operativo libre completo
Hoy  → Miles de distribuciones basadas en GNU/Linux
```

### Características principales

- **Código abierto y libre** — cualquiera puede ver, modificar y distribuir el código
- **Multiusuario** — varios usuarios pueden usar el sistema simultáneamente
- **Multitarea** — ejecuta múltiples procesos en paralelo
- **Multiplataforma** — corre en x86, ARM, RISC-V, mainframes, etc.
- **Estabilidad y seguridad** — base de la mayoría de servidores del mundo
- **Compatible con POSIX** — implementa el estándar POSIX completamente

### Relación con POSIX

GNU/Linux implementa el estándar POSIX, lo que significa que todas las llamadas al sistema vistas en [[C3 - POSIX - Servicios para Procesos|C3]] y [[C4 - POSIX - Servicios para Ficheros|C4]] funcionan en Linux:

```
POSIX (estándar IEEE) ←→ Linux (implementación)
fork(), exec(), exit()         ✅
open(), read(), write()        ✅
mmap(), stat(), dup()          ✅
```

### Distribuciones principales

Una **distribución** (distro) es GNU/Linux + software adicional empaquetado para un uso específico:

| Distribución | Orientada a |
|---|---|
| **Ubuntu** | Usuarios de escritorio y servidores |
| **Debian** | Estabilidad, base de muchas otras distros |
| **Fedora** | Tecnologías de punta (Red Hat) |
| **Arch Linux** | Usuarios avanzados, minimalismo |
| **CentOS / RHEL** | Servidores empresariales |
| **Raspberry Pi OS** | Dispositivos embebidos (ARM) |
| **Android** | Smartphones (kernel Linux modificado) |

---

## 🟠 Ubuntu

### ¿Qué es Ubuntu?

**Ubuntu** es una distribución GNU/Linux basada en Debian, desarrollada y mantenida por **Canonical Ltd.** Está orientada a ser:
- Fácil de usar (escritorio)
- Potente como servidor
- Totalmente gratuita

> El nombre "Ubuntu" proviene del concepto zulú/xhosa: *"soy lo que soy por lo que todos somos"* — refleja su filosofía de comunidad.

### Versiones y ciclo de lanzamiento

| Tipo | Frecuencia | Soporte |
|---|---|---|
| **LTS** (Long Term Support) | Cada 2 años (años pares, abril) | 5 años |
| **Estándar** | Cada 6 meses | 9 meses |

Ejemplo de nomenclatura: **Ubuntu 24.04 LTS** → año 2024, mes abril, soporte largo.

### Entornos de escritorio

| Sabor | Entorno | Descripción |
|---|---|---|
| Ubuntu | **GNOME** | Moderno, orientado a táctil |
| Kubuntu | KDE Plasma | Altamente personalizable |
| Xubuntu | XFCE | Liviano, para hardware antiguo |
| Lubuntu | LXQt | El más liviano |
| Ubuntu Server | Sin GUI | Solo línea de comandos |

### ¿Para qué sirve Ubuntu?

**Escritorio:**
- Navegación web, ofimática (LibreOffice), multimedia
- Desarrollo de software
- Educación

**Servidor:**
- Servidores web (Apache, Nginx)
- Bases de datos (MySQL, PostgreSQL)
- Cloud computing (AWS, Azure, Google Cloud usan Ubuntu)
- Contenedores (Docker, Kubernetes)

**Áreas especializadas:**
- Ubuntu IoT / Ubuntu Core → dispositivos embebidos
- Ubuntu Studio → producción de audio y video

### Comandos básicos (shell POSIX)

Estas son las llamadas al sistema de [[C4 - POSIX - Servicios para Ficheros|C4]] accesibles desde la terminal:

```bash
# Gestión de ficheros
ls -l           # listar (usa stat internamente)
cp src dst      # copiar (usa open, read, write, creat)
mv src dst      # mover / renombrar
rm archivo      # borrar (usa unlink)
cat archivo     # mostrar contenido (usa open, read, close)

# Gestión de procesos
ps aux          # listar procesos (lee /proc con open/read)
kill -9 PID     # terminar proceso
top             # monitorear procesos en tiempo real
./programa &    # ejecutar en segundo plano (fork + exec)

# Permisos (chmod usa el campo st_mode de stat)
chmod 755 archivo
chmod u+x script.sh
chown usuario:grupo archivo
```

### El shell como interfaz POSIX

El shell (`bash`, `zsh`, `sh`) es el **intérprete de comandos** mencionado en [[C2 - Arquitectura y Servicios del SO#Estándar POSIX|C2]]. Cada comando del shell ejecuta llamadas al sistema POSIX internamente:

```
usuario escribe: ls /home
shell llama a:   fork() → exec("ls", "/home") → wait()
ls llama a:      opendir("/home") → readdir() → stat() → closedir()
resultado:       se imprime en stdout (fd=1)
```

---

## 🔗 Relaciones con otras clases

| Clase | Relación |
|---|---|
| [[C1 - Conceptos Básicos]] | Linux implementa el modelo de capas: kernel, procesos, BCP |
| [[C2 - Arquitectura y Servicios del SO]] | Linux implementa POSIX completo (llamadas al sistema) |
| [[C3 - POSIX - Servicios para Procesos]] | fork/exec/exit disponibles en cualquier terminal Linux |
| [[C4 - POSIX - Servicios para Ficheros]] | open/read/write/mmap funcionan en Linux nativamente |

---

## 📝 Notas de clase

*(Espacio para tus apuntes personales)*

---

## ✅ Checklist

- [ ] Diferencia entre Linux (kernel) y GNU/Linux (SO completo)
- [ ] Contribuciones de Linus Torvalds y Richard Stallman
- [ ] Características principales de GNU/Linux
- [ ] Por qué GNU/Linux es compatible con POSIX
- [ ] Concepto de distribución y ejemplos
- [ ] Qué es Ubuntu y quién lo desarrolla
- [ ] LTS vs versión estándar
- [ ] Sabores de Ubuntu y sus entornos de escritorio
- [ ] Usos de Ubuntu (escritorio, servidor, cloud, IoT)
- [ ] Cómo los comandos de shell invocan llamadas al sistema POSIX
