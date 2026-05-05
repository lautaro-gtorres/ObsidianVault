---
tags: [SO, arquitectura, llamadas-al-sistema, interrupciones, UAI]
clase: C2
estado: ⬜
---

# C2 — Arquitectura y Servicios del SO

← [[C1 - Conceptos Básicos]] | Siguiente: [[C3 - POSIX - Servicios para Procesos]] →

---

## 🎯 Objetivos

- Comprender qué es un servicio del SO
- Entender los mecanismos de una llamada al sistema
- Conocer la interfaz POSIX y sus principales servicios

---

## ⚡ Ejecución del Sistema Operativo

Una vez finalizado el arranque, el SO **solo se ejecuta en respuesta a interrupciones**.

El SO se activa ante:

| Causa | Descripción |
|---|---|
| Petición de servicio | Un proceso necesita algo del SO (ej: leer un archivo) |
| Interrupción de periférico | Un dispositivo de E/S avisa que terminó una operación |
| Interrupción de reloj | Tick del temporizador del sistema |
| Excepción hardware | Error de hardware (ej: división por cero, fallo de página) |

### Fases de activación

```
Proceso A ejecutando
        ↓  [Se solicita el SO]
SO: Salva estado del Proceso A
SO: Realiza la función pedida
SO: Planificador → elige qué proceso sigue
        ↓
Proceso B (u otro) ejecutando
```

---

## 🔐 Activación de Servicios — El problema de seguridad

Una **invocación directa** a una rutina del SO plantea problemas de seguridad (el proceso podría saltar a cualquier dirección del kernel).

**Solución**: usar una **interrupción software (trap)**:

```
Rutina de biblioteca
├── Instrucciones de máquina que preparan la llamada al SO
├── Instrucción de trap  ← genera una interrupción controlada
└── Instrucciones para procesar los resultados de la llamada
```

La instrucción `trap` transfiere el control al SO de forma **segura y controlada**.

---

## 📞 Llamadas al Sistema (System Calls)

Las llamadas al sistema son la **interfaz entre las aplicaciones y el SO**.

- Disponibles originalmente como funciones en ensamblador
- Actualmente expuestas en lenguajes de alto nivel (C, C++, Python...)
- Cada función de la API **se corresponde con un servicio del SO**

### Servicios típicos

| Categoría | Descripción |
|---|---|
| Gestión de procesos | Crear, destruir, esperar procesos |
| Gestión de procesos ligeros | Hilos (threads) |
| Gestión de señales y temporizadores | Enviar señales, alarmas |
| Gestión de memoria | Asignar y liberar memoria |
| Gestión de ficheros y directorios | Crear, leer, escribir, borrar archivos |

### Ejemplos POSIX
- `read` — leer datos de un fichero
- `fork` — crear un nuevo proceso

---

## 🔄 Flujo completo de una invocación

```
Programa de usuario
    read(fd, b, lo)          ← llamada en C de alto nivel
         ↓
Biblioteca de sistema        ← envoltorio (wrapper)
    lw $a0, fd
    lw $a1, b
    lw $a2, lo
    li $v0, 14               ← número de servicio = READ_SYSCALL
    syscall                  ← instrucción TRAP
         ↓
Sistema Operativo
    14 → read_syscall        ← tabla de rutinas de servicio
         ↓
    Ejecuta la lectura
         ↓
    Devuelve resultado
         ↓
Retorno al programa de usuario
```

### Selección del servicio

Como hay **una sola instrucción trap** para múltiples servicios, se necesita un mecanismo para identificar cuál:
- Se pasa siempre un **identificador numérico** del servicio
- Junto con los parámetros correspondientes

### Métodos de paso de parámetros

| Método | Descripción |
|---|---|
| **En registros** | Los parámetros se cargan directamente en registros del CPU |
| **Tabla de memoria** | Los parámetros están en memoria; se pasa la dirección en un registro |
| **Pila** | Los parámetros se apilan; el SO los extrae |

### Rutina de tratamiento (dentro del SO)

Al recibir la interrupción, el SO debe:
1. **Recuperar** los parámetros enviados por el proceso
2. **Identificar** el servicio solicitado
3. **Determinar** la dirección de la rutina de servicio (indexando la tabla)
4. **Transferir** el control a esa rutina

---

## 🌐 Interfaz del Programador

La interfaz del programador ofrece la **visión de máquina extendida** que tiene el usuario del SO.

Cada SO puede ofrecer una o varias interfaces:

| SO | Interfaces |
|---|---|
| Linux | POSIX |
| Windows | Win32, POSIX |
| macOS | POSIX (certificado Unix 03) |

---

## 📋 Estándar POSIX

> POSIX = *Portable Operating System Interface* — Estándar IEEE para interfaces de SO.

| Aspecto | Detalle |
|---|---|
| **Objetivo** | Portabilidad de aplicaciones entre plataformas y SO distintos |
| **¿Es una implementación?** | **NO** — solo define una interfaz (contrato) |
| **¿Quién lo usa?** | Linux, macOS, AIX, Solaris, etc. |

### Estándares POSIX

| Estándar | Contenido |
|---|---|
| 1003.1 | Servicios básicos del SO |
| 1003.1a | Extensiones a los servicios básicos |
| 1003.1b | Extensiones de tiempo real |
| 1003.1c | Extensiones de procesos ligeros (threads) |
| 1003.2 | Shell y utilidades |
| 1003.2b | Utilidades adicionales |

### Unix 03 (Single Unix Specification)
Evolución que engloba POSIX y otros estándares (X/Open XPG4, ISO C). Incluye no solo la interfaz de programación sino también:
- Servicios ofrecidos
- Intérprete de mandatos (shell)
- Utilidades disponibles

### Características de la API POSIX

- Nombres de funciones **cortos y en minúsculas**: `fork`, `read`, `close`
- Las funciones retornan **0 en éxito** o **-1 en error**
  - El error detallado se consulta en la variable `errno`
- Los recursos del SO se referencian mediante **descriptores** (números enteros)

---

## 🔗 Relaciones con otras clases

| Clase | Relación |
|---|---|
| [[C1 - Conceptos Básicos]] | El kernel es quien recibe y procesa las llamadas al sistema |
| [[C3 - POSIX - Servicios para Procesos]] | fork, exec, exit son llamadas POSIX para procesos |
| [[C4 - POSIX - Servicios para Ficheros]] | open, read, write son llamadas POSIX para ficheros |

---

## 📝 Notas de clase

*(Espacio para tus apuntes personales)*

---

## ✅ Checklist

- [ ] El SO solo se ejecuta ante interrupciones
- [ ] Causas de activación del SO
- [ ] Por qué la invocación directa es insegura
- [ ] Rol de la instrucción trap
- [ ] Flujo completo de una llamada al sistema
- [ ] Métodos de paso de parámetros (3)
- [ ] Rutina de tratamiento: 4 pasos
- [ ] POSIX: qué es, qué NO es, estándares
- [ ] Características de la API POSIX
