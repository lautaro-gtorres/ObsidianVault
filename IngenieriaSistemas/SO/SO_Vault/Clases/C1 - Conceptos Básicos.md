---
tags: [SO, conceptos, kernel, procesos, UAI]
clase: C1
estado: ⬜
---

# C1 — Conceptos Básicos del Sistema Operativo

← [[🏠 Inicio SO]] | Siguiente: [[C2 - Arquitectura y Servicios del SO]] →

---

## 🎯 Objetivos

Comprender qué es un SO, sus funciones, tipos y componentes fundamentales: kernel, procesos y BCP.

---

## 📌 ¿Qué es un Sistema Operativo?

Distintas definiciones apuntan a lo mismo:

> Conjunto de programas que crean la **interfaz entre el hardware y el usuario**, con dos funciones primordiales:
> 1. **Gestionar el hardware** — administrar eficientemente los recursos.
> 2. **Facilitar el trabajo del usuario** — permitir la comunicación con los dispositivos.

---

## ⚙️ Funciones del SO

| # | Función | Descripción |
|---|---|---|
| 1 | **Eficiencia** | Uso óptimo de los recursos del sistema |
| 2 | **Habilidad para evolucionar** | Permite incorporar nuevas funciones sin interrumpir el servicio |
| 3 | **Administración del hardware** | Asigna procesador y recursos a cada proceso |
| 4 | **Facilitar E/S** | Acceso y manejo de dispositivos de entrada/salida |
| 5 | **Administración de periféricos** | Coordina y organiza la comunicación con los DP |
| 6 | **Manejo de redes** | Gestión de instalación y uso de redes |
| 7 | **Recuperación de errores** | Gestiona errores de hardware y pérdida de datos |
| 8 | **Estadísticas** | Genera registros de errores y operaciones |
| 9 | **Gestión de permisos/usuarios** | Controla el acceso y evita interferencias entre usuarios |
| 10 | **Seguridad** | Protección contra virus, hackers y malware |
| 11 | **Control de ocurrencia** | Establece prioridades ante recursos compartidos |
| 12 | **Administración de memoria** | Asigna y gestiona el uso de memoria |
| 13 | **Control de ejecución** | Acepta trabajos, asigna recursos y los monitorea |

---

## 🗂️ Tipos de Sistemas Operativos

| Tipo | Características | Ejemplos |
|---|---|---|
| Main / Mini | Lotes, tiempo compartido, múltiples usuarios, grandes BD | OS/400, OS/360 (IBM) |
| Servidores | Comparten recursos en red, gran escala | Solaris, Linux, Windows Server |
| Multiprocesadores | Varios procesadores, chips multinúcleo | Windows, Linux |
| PC | Un solo usuario, todo el procesador disponible | Linux, Windows, macOS |
| Bolsillo | Smartphone, tablet, smartwatch, 64 bits | Android, iOS, Windows Mobile |
| Integrados (embebidos) | No instalado por el usuario, en dispositivos cotidianos | QNX, VxWorks |
| Nodos sensores | RAM/ROM/CPU mínimas, inalámbricos, batería | TinyOS |
| Tiempo real | SCADA, aeronáutica, el tiempo es crítico | e-Cos |
| Tarjetas inteligentes | Los más pequeños, tarjetas de débito/crédito | SO muy básico |

---

## 🏗️ Modelo de Capas de un Sistema Computacional

```
┌────────────────────────────┐
│       USUARIO FINAL        │
├────────────────────────────┤
│         PROGRAMADOR        │
├────────────────────────────┤
│  DISEÑADORES DE TRADUCTORES│
├────────────────────────────┤
│    DISEÑADORES DE S.O.     │
├────────────────────────────┤
│ CONSTRUCTORES DE COMPUTAD. │
└────────────────────────────┘
```

---

## 📐 Estructura por Capas del SO

| Nivel | Nombre | Función |
|---|---|---|
| 1 | Gestión del Procesador | Comparte CPU: sincronización, conmutación, control de interrupciones |
| 2 | Gestión de Memoria | Reparte memoria disponible, controla violaciones de acceso |
| 3 | Gestión de Procesos | Crea/destruye procesos, intercambio de mensajes, detección y arranque |
| 4 | Gestión de Dispositivos | Maneja E/S, asigna/libera dispositivos, planifica E/S |
| 5 | Gestión de Información | Sistema de archivos, protección, lectura/escritura, directorios |

---

## 🧠 El Kernel (Núcleo)

> El núcleo es una colección de módulos de software que se ejecutan en modo **privilegiado**, con acceso pleno a todos los recursos del sistema.

Características clave:
- Reside permanentemente en **memoria principal**
- Es la parte del SO **más usada**, aunque representa una fracción pequeña del total
- Realiza el **mínimo** de procesamiento por interrupción y delega el resto a procesos del sistema

### Funciones del Kernel

**Gestión de procesos:**
- Manejo de interrupciones
- Creación y destrucción de procesos
- Cambio de estado de procesos
- Despacho, suspensión y reanudación
- Sincronización y comunicación entre procesos
- Manipulación de los BCPs

**Soporte general:**
- Apoyo para actividades de E/S
- Asignación y liberación de memoria
- Sistema de archivos
- Shell e intérprete de comandos
- API

---

## 🔄 Proceso y BCP

### ¿Qué es un Proceso?
Un **proceso** es un programa en ejecución, caracterizado por:
- La ejecución de una secuencia de instrucciones
- Un estado actual
- Un conjunto de recursos del sistema asociados

### Bloque de Control de Proceso (BCP)

El BCP es el registro donde el SO guarda **toda la información** que necesita sobre un proceso.

| Campo del BCP | Descripción |
|---|---|
| **PID** | Identificador único del proceso |
| **Estado** | Ejecutando / Listo / Bloqueado |
| **Prioridad** | Nivel de preferencia para ser despachado |
| **Contador de Programa (PC)** | Dirección de la próxima instrucción |
| **Punteros de Memoria** | Ubicación en memoria |
| **Valores de Registros del CPU** | Contexto del procesador al momento de interrupción |
| **Datos del Propietario** | Usuario dueño del proceso |
| **Señales** | Señales pendientes |
| **Permisos** | Privilegios del proceso |
| **Estadísticas** | Tiempo de CPU consumido, etc. |

### Ciclo de vida del BCP
1. **Creación**: el SO crea el BCP cuando nace el proceso
2. **Uso**: el BCP es actualizado en cada cambio de estado
3. **Interrupción**: el PC y registros se guardan en el BCP; el proceso pasa a Bloqueado/Listo
4. **Reanudación**: se cargan los valores del BCP en los registros y se retoma la ejecución
5. **Destrucción**: el BCP se borra cuando el proceso termina

> El BCP es la herramienta que hace posible la **multiprogramación**.

---

## 🔗 Relaciones con otras clases

| Clase | Relación |
|---|---|
| [[C2 - Arquitectura y Servicios del SO]] | Los procesos solicitan servicios al SO vía llamadas al sistema |
| [[C3 - POSIX - Servicios para Procesos]] | POSIX implementa `fork`, `exec`, `exit` para gestionar procesos |
| [[C5 - GNU Linux y Ubuntu]] | Linux implementa todos estos conceptos en la práctica |

---

## 📝 Notas de clase

*(Espacio para tus apuntes personales)*

---

## ✅ Checklist

- [ ] Definición de SO y sus dos funciones primordiales
- [ ] 13 funciones del SO
- [ ] Tipos de SO y ejemplos
- [ ] Modelo de capas del sistema computacional
- [ ] 5 niveles de la estructura por capas del SO
- [ ] Concepto y características del kernel
- [ ] Funciones del kernel
- [ ] Concepto de proceso
- [ ] Campos del BCP y ciclo de vida
