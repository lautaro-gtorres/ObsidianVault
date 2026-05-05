---
tags: [SO, POSIX, procesos, fork, exec, exit, UAI]
clase: C3
estado: ⬜
fuente: SISTEMAS_OPERATIVOS_Clase_II.pdf (páginas 17-23)
---

# C3 — POSIX: Servicios para Procesos

← [[C2 - Arquitectura y Servicios del SO]] | Siguiente: [[C4 - POSIX - Servicios para Ficheros]] →

---

## 🎯 Objetivos

Conocer los principales servicios POSIX para la gestión de procesos: `fork`, `exec` y `exit`.

> Ver contexto general de llamadas al sistema en [[C2 - Arquitectura y Servicios del SO#Llamadas al Sistema|C2 — Llamadas al Sistema]]

---

## 🌿 Servicio `fork` — Duplicar un proceso

```c
pid_t fork(void);
```

### ¿Qué hace?
**Duplica el proceso** que invoca la llamada. Crea un proceso hijo que es una copia exacta del padre.

```
Antes del fork:
┌──────────────────┐
│    Proceso A     │
└──────────────────┘

Después del fork:
┌──────────────────┐   ┌──────────────────┐
│    Proceso A     │   │   Proceso A'     │
│    (padre)       │   │    (hijo)        │
└──────────────────┘   └──────────────────┘
```

### Características
- Padre e hijo siguen ejecutando el **mismo programa** desde el mismo punto
- El hijo **hereda los ficheros abiertos** del padre (se copian los descriptores)
- Se **desactivan las alarmas** pendientes en el hijo

### Valor de retorno

| Contexto | Retorna |
|---|---|
| Error | `-1` |
| En el **proceso padre** | PID del proceso hijo (>0) |
| En el **proceso hijo** | `0` |

```c
pid_t pid = fork();
switch (pid) {
    case -1: /* error */
        exit(-1);
    case 0: /* proceso hijo */
        // aquí el hijo hace su trabajo
        break;
    default: /* proceso padre */
        printf("Soy el padre, mi hijo tiene PID %d\n", pid);
}
```

---

## 🔄 Servicio `exec` — Cambiar la imagen del proceso

```c
int execl (const char *path, const char *arg, ...);
int execv (const char *path, char * const argv[]);
int execve(const char *path, char * const argv[], char * const envp[]);
int execvp(const char *file, char * const argv[]);
```

### ¿Qué hace?
**Cambia la imagen del proceso actual**: reemplaza el programa que se está ejecutando por otro.

```
Antes del exec:
┌──────────────────┐
│    Proceso A     │ ← ejecutando programa A
└──────────────────┘

Después del exec:
┌──────────────────┐
│    Proceso A     │ ← ahora ejecuta programa B (misma PID)
└──────────────────┘
```

### Diferencia entre variantes
| Parámetro | Descripción |
|---|---|
| `path` | Ruta absoluta al ejecutable |
| `file` | Nombre del ejecutable — lo busca en los directorios del `PATH` |
| `argv[]` | Array de argumentos |
| `envp[]` | Array de variables de entorno |

### Características
- Devuelve **-1 en error**; si tiene éxito, **no retorna** (el programa anterior desaparece)
- El **mismo proceso** ejecuta otro programa (misma PID)
- Los **ficheros abiertos** permanecen abiertos
- Las señales con manejador toman la acción por defecto

---

## 🔗 Patrón `fork` + `exec`

El patrón estándar en UNIX para ejecutar un nuevo programa:

```c
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char** argv) {
    pid_t pid;
    pid = fork();              // 1. Duplicar el proceso
    switch (pid) {
        case -1:
            exit(-1);
        case 0:                // 2. El hijo reemplaza su imagen
            if (execvp(argv[1], &argv[1]) < 0) {
                perror("error");
            }
            break;
        default:               // 3. El padre continúa
            printf("Proceso padre\n");
    }
    return 0;
}
```

Ejemplo de uso: `prog cat f1` → el padre hace fork, el hijo ejecuta `cat f1`.

---

## 🚪 Servicio `exit` — Finalizar un proceso

```c
void exit(int status);
```

### ¿Qué hace?
**Finaliza la ejecución del proceso**.

### Acciones que realiza el SO al recibir `exit`:
1. Se **cierran** todos los descriptores de ficheros abiertos
2. Se **liberan** todos los recursos del proceso
3. Se **libera el BCP** del proceso

> El BCP se destruye aquí — ver [[C1 - Conceptos Básicos#Ciclo de vida del BCP|BCP en C1]]

```c
// Ejemplo completo con fork + exec + exit
int main() {
    pid_t pid;
    int status;

    pid = fork();
    if (pid == 0) {          /* proceso hijo */
        execlp("ls", "ls", "-l", NULL);
        exit(-1);            // solo llega aquí si exec falló
    }
    else {                   /* proceso padre */
        printf("Fin del padre\n");
    }
    return 0;                // return 0 invoca implícitamente exit(0)
}
```

---

## 📊 Resumen de servicios de procesos

| Servicio | Firma | Retorna | Función |
|---|---|---|---|
| `fork` | `pid_t fork(void)` | PID hijo / 0 / -1 | Duplica el proceso actual |
| `exec*` | `int exec*(path, ...)` | -1 o no retorna | Reemplaza imagen del proceso |
| `exit` | `void exit(status)` | — | Termina el proceso |

---

## 🔗 Relaciones con otras clases

| Clase | Relación |
|---|---|
| [[C1 - Conceptos Básicos#BCP]] | `exit` destruye el BCP; `fork` crea un nuevo BCP |
| [[C2 - Arquitectura y Servicios del SO#Llamadas al Sistema]] | fork/exec/exit son llamadas al sistema POSIX |
| [[C4 - POSIX - Servicios para Ficheros]] | El hijo hereda los ficheros abiertos del padre (fork) |

---

## 📝 Notas de clase

*(Espacio para tus apuntes personales)*

---

## ✅ Checklist

- [ ] `fork`: qué hace, valor de retorno en padre vs hijo
- [ ] Qué hereda el hijo del padre
- [ ] `exec`: variantes y diferencia entre path vs file
- [ ] Por qué exec no retorna en caso de éxito
- [ ] Patrón fork + exec: cuándo usarlo
- [ ] `exit`: qué libera el SO al llamarlo
- [ ] Relación entre exit y el BCP
