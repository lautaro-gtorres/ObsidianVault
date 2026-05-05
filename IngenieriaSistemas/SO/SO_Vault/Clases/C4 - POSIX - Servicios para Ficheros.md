---
tags: [SO, POSIX, ficheros, descriptores, mmap, UAI]
clase: C4
estado: ⬜
fuente: SISTEMAS_OPERATIVOS_Clase_II.pdf (páginas 24-54, excluyendo las indicadas)
---

# C4 — POSIX: Servicios para Ficheros

← [[C3 - POSIX - Servicios para Procesos]] | Siguiente: [[C5 - GNU Linux y Ubuntu]] →

---

## 🎯 Objetivos

Conocer los servicios POSIX para gestión de ficheros, directorios y proyección en memoria.

---

## 🗂️ Modelo de ficheros en POSIX

### Tipos de fichero
- **Normales** (`.`) — datos o programas
- **Directorios** — contienen referencias a otros ficheros
- **Especiales** — dispositivos de hardware representados como ficheros

### Nombres de fichero
```
Absoluto:  /usr/include/stdio.h     (empieza con /)
Relativo:  stdio.h                  (desde el directorio actual)
           ../include/stdio.h       (usando . y ..)
```

### Descriptores de fichero (fd)
- Un **entero entre 0 y 64K** que identifica un fichero abierto
- Se obtiene con `open()` o `creat()`
- **Descriptores predefinidos:**

| fd | Nombre |
|---|---|
| 0 | Entrada estándar (stdin) |
| 1 | Salida estándar (stdout) |
| 2 | Salida de error (stderr) |

### Tabla de ficheros y fork
- Cada proceso tiene su **tabla de ficheros abiertos**
- Al hacer `fork`, se **duplica** la tabla del padre
- La tabla intermedia de nodos-i y posiciones es **compartida** entre padre e hijo

### Permisos (protección)
```
dueño   grupo   mundo
 rwx     rwx     rwx

Ejemplo: 755 → rwxr-xr-x
```

---

## 📋 Operaciones genéricas sobre ficheros

| Operación | Descripción |
|---|---|
| **crear** | Crea un fichero con nombre y atributos |
| **borrar** | Borra un fichero por su nombre |
| **abrir** | Abre un fichero para operaciones de acceso |
| **cerrar** | Cierra un fichero abierto |
| **leer** | Lee datos de un fichero a memoria |
| **escribir** | Escribe datos desde memoria a un fichero |
| **posicionar** | Mueve el puntero de acceso al fichero |
| **control** | Manipula los atributos de un fichero |

---

## 🛠️ Servicios POSIX para Ficheros

### `creat` — Crear fichero

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int creat(char *name, mode_t mode);
```

| Arg | Descripción |
|---|---|
| `name` | Nombre del fichero |
| `mode` | Bits de permiso (ej: `0751`) |

**Retorna:** descriptor de fichero o `-1` si error.

**Comportamiento:**
- Si no existe → crea un fichero vacío (abierto para escritura)
- Si ya existe → lo **trunca** sin cambiar los permisos

```c
fd = creat("datos.txt", 0751);
// equivalente a:
fd = open("datos.txt", O_WRONLY | O_CREAT | O_TRUNC, 0751);
```

---

### `unlink` — Borrar fichero

```c
#include <unistd.h>
int unlink(const char *path);
```

**Retorna:** `0` o `-1` si error.

Decrementa el **contador de enlaces** del fichero. Si llega a 0, borra el fichero y libera sus recursos.

---

### `open` — Abrir fichero

```c
#include <fcntl.h>
int open(char *name, int flags, ...);
```

| Flag | Descripción |
|---|---|
| `O_RDONLY` | Solo lectura |
| `O_WRONLY` | Solo escritura |
| `O_RDWR` | Lectura y escritura |
| `O_APPEND` | El puntero se desplaza al final |
| `O_CREAT` | Crea si no existe |
| `O_TRUNC` | Trunca si se abre para escritura |

**Retorna:** descriptor de fichero o `-1` si error.

```c
fd = open("/home/juan/datos.txt", O_RDONLY);
fd = open("/home/juan/datos.txt", O_WRONLY | O_CREAT | O_TRUNC, 0750);
```

---

### `close` — Cerrar fichero

```c
int close(int fd);
```

**Retorna:** `0` o `-1` si error. El proceso pierde la asociación al fichero.

---

### `read` — Leer fichero

```c
#include <sys/types.h>
ssize_t read(int fd, void *buf, size_t n_bytes);
```

| Arg | Descripción |
|---|---|
| `fd` | Descriptor de fichero |
| `buf` | Zona donde almacenar los datos leídos |
| `n_bytes` | Número de bytes a leer |

**Retorna:** bytes realmente leídos o `-1` si error. (Puede leer menos si llega al EOF o se interrumpe por señal.)

Después de la lectura, el **puntero del fichero avanza** con los bytes transferidos.

---

### `write` — Escribir fichero

```c
ssize_t write(int fd, void *buf, size_t n_bytes);
```

**Retorna:** bytes realmente escritos o `-1` si error.

Si se rebasa el fin de fichero, el fichero **aumenta de tamaño**.

---

### `lseek` — Mover el puntero de posición

```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
```

| `whence` | Posición calculada |
|---|---|
| `SEEK_SET` | `offset` (desde el inicio) |
| `SEEK_CUR` | posición actual + `offset` |
| `SEEK_END` | tamaño del fichero + `offset` |

**Retorna:** nueva posición del puntero o `-1` si error.

---

### `fnctl` — Modificar atributos

```c
int fnctl(int fildes, int cmd /*, arg*/ ...);
```

Modifica los atributos de un fichero ya abierto. Retorna `0` o `-1`.

---

### `dup` — Duplicar descriptor

```c
int dup(int fd);
```

Crea un nuevo descriptor que apunta al **mismo fichero**, compartiendo el mismo puntero de posición y modo de acceso. El nuevo descriptor tendrá el menor valor numérico libre. Retorna el nuevo `fd` o `-1`.

---

### `ftruncate` — Ajustar tamaño

```c
#include <unistd.h>
int ftruncate(int fd, off_t length);
```

Establece el tamaño del fichero a `length`. Si `length == 0`, el fichero queda vacío.

---

### `stat` / `fstat` — Información sobre un fichero

```c
int stat (char *name,   struct stat *buf); // por nombre
int fstat(int fd,       struct stat *buf); // por descriptor
```

La estructura `stat` contiene:

| Campo | Descripción |
|---|---|
| `st_mode` | Modo del fichero (tipo + permisos) |
| `st_ino` | Número de nodo-i |
| `st_dev` | Dispositivo |
| `st_nlink` | Número de enlaces |
| `st_uid` / `st_gid` | UID y GID del propietario |
| `st_size` | Tamaño en bytes |
| `st_atime` | Último acceso |
| `st_mtime` | Última modificación |
| `st_ctime` | Último cambio de metadatos |

**Macros para verificar el tipo:**
```c
S_ISDIR(s.st_mode)   // cierto si directorio
S_ISCHR(s.st_mode)   // cierto si especial de caracteres
S_ISBLK(s.st_mode)   // cierto si especial de bloques
S_ISREG(s.st_mode)   // cierto si fichero normal
S_ISFIFO(s.st_mode)  // cierto si pipe o FIFO
```

---

### `utime` — Modificar fechas

```c
#include <utime.h>
int utime(char *name, struct utimbuf *times);
```

Cambia las fechas de último acceso (`actime`) y última modificación (`mctime`).

---

## 📁 Servicios POSIX para Directorios

Un directorio es un fichero con registros de tipo `DIR`. Se puede **leer pero no escribir directamente** desde un programa.

### Estructura `DIR` (dirent)
```c
d_ino;     // Nodo-i
d_off;     // Posición en el fichero del directorio
d_reclen;  // Tamaño del registro
d_type;    // Tipo del elemento
d_name[];  // Nombre del fichero (longitud variable)
```

### Funciones

```c
DIR  *opendir (const char *dirname);          // abrir directorio
int   readdir_r(DIR *dirp, struct dirent *entry, struct dirent **result); // leer entrada
long  telldir  (DIR *dirp);                   // posición actual
void  seekdir  (DIR *dirp, long int loc);     // avanzar a posición
void  rewinddir(DIR *dirp);                   // volver al principio
int   closedir (DIR *dirp);                   // cerrar
```

---

## 🗺️ Proyección en memoria — `mmap`

`mmap` establece una **proyección entre el espacio de direcciones de un proceso y un archivo**: permite acceder al fichero como si fuera un array en memoria.

```c
void *mmap(void *direc, size_t lon, int prot, int flags, int fd, off_t desp);
```

| Arg | Descripción |
|---|---|
| `direc` | Dirección donde proyectar (NULL = el SO elige) |
| `lon` | Bytes a proyectar |
| `prot` | Protección de la zona |
| `flags` | Propiedades de la región |
| `fd` | Descriptor del fichero a proyectar |
| `desp` | Desplazamiento inicial en el archivo |

### Tipos de protección (`prot`)

| Constante | Descripción |
|---|---|
| `PROT_READ` | Se puede leer |
| `PROT_WRITE` | Se puede escribir |
| `PROT_EXEC` | Se puede ejecutar |
| `PROT_NONE` | Sin acceso |

### Propiedades de región (`flags`)

| Constante | Descripción |
|---|---|
| `MAP_SHARED` | Compartida; modificaciones afectan al fichero; los hijos comparten |
| `MAP_PRIVATE` | Privada; el fichero no se modifica; hijos obtienen copias |
| `MAP_FIXED` | Proyectar exactamente en la dirección indicada |

### `munmap` — Desproyectar

```c
void munmap(void *direc, size_t lon);
```

Libera la proyección del espacio de direcciones desde `direc` hasta `direc + lon`.

---

## 💡 Ejemplos integrados

### Copia de un fichero (con read/write)

```c
// Patrón básico
fd_ent = open(argv[1], O_RDONLY);
fd_sal = creat(argv[2], 0644);

while ((n_read = read(fd_ent, buffer, BUFSIZE)) > 0) {
    write(fd_sal, buffer, n_read);
}
close(fd_ent);
close(fd_sal);
```

### Copia con mmap (más eficiente)

```c
fd1 = open("f1", O_RDONLY);
fd2 = open("f2", O_CREAT | O_TRUNC | O_RDWR, 0640);
fstat(fd1, &dstat);
ftruncate(fd2, dstat.st_size);    // ajustar tamaño del destino

vec1 = mmap(0, dstat.st_size, PROT_READ,  MAP_SHARED, fd1, 0);
vec2 = mmap(0, dstat.st_size, PROT_READ,  MAP_SHARED, fd2, 0);
close(fd1); close(fd2);

for (i = 0; i < dstat.st_size; i++) *q++ = *p++;  // copiar byte a byte

munmap(fd1, dstat.st_size);
munmap(fd2, dstat.st_size);
```

---

## 🔗 Relaciones con otras clases

| Clase | Relación |
|---|---|
| [[C2 - Arquitectura y Servicios del SO]] | Todas estas funciones son llamadas al sistema POSIX |
| [[C3 - POSIX - Servicios para Procesos]] | fork hereda la tabla de ficheros abiertos |
| [[C5 - GNU Linux y Ubuntu]] | En Linux se usan exactamente estas llamadas |

---

## 📝 Notas de clase

*(Espacio para tus apuntes personales)*

---

## ✅ Checklist

- [ ] Tipos de fichero en POSIX
- [ ] Descriptores predefinidos (0, 1, 2)
- [ ] Permisos: formato rwx y octal
- [ ] creat / open (flags O_RDONLY, O_WRONLY, O_RDWR, O_APPEND, O_CREAT, O_TRUNC)
- [ ] close / read / write
- [ ] lseek: SEEK_SET, SEEK_CUR, SEEK_END
- [ ] dup, fnctl, ftruncate
- [ ] stat / fstat: campos de struct stat
- [ ] Macros S_ISDIR, S_ISREG, etc.
- [ ] Servicios de directorios: opendir, readdir, closedir
- [ ] mmap: args, tipos de prot, tipos de flags
- [ ] munmap
