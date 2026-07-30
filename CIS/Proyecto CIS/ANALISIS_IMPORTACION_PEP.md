# Análisis de Importación - Lista PEP 2026

**Archivo:** Lista_PEP_corte_8.04.2026.xlsx  
**Registros:** 3,279  
**Fecha:** 22-06-2026

---

## Campos en el Excel a Cargar

| Campo | Tipo | Ejemplo |
|---|---|---|
| Tipo de identificación | Text | Cédula |
| Identificación | Number | 103120524 |
| Nombre | Text | Rafael Angel Ramirez Badilla |
| Fecha de nacimiento | Date | 8/27/1943 |
| Puesto | Text | Regidores Propietarios Y Suplentes |
| Institución | Text | Municipalidad De San Jose |
| Fecha inicio de puesto | Date | 5/1/2020 |
| Fecha fin de puesto | Date | (vacío) |
| Fecha fin de PEP | Date | (vacío) |
| Es PEP vigente | Text | Sí/No |

---

## Campos en BD Actual (PepList)

```
documentNumber           (string)  - Número documento
lastName                 (string)  - Apellido
secondLastName           (string)  - Segundo apellido
names                    (string)  - Nombres
employmentPosition       (string)  - Puesto
enabled                  (boolean) - Habilitado
fromWu                   (boolean) - Proviene de Western Union
documentLocation         (string)  - Localización documento
countryEmployment        (string)  - País empleo
typePEP                  (string)  - Tipo de PEP
countryEmploymentPositionId (number) - ID de posición
cisCountryId             (number)  - ID de país CIS
id                       (number)  - ID único
```

---

## Campos Usados en Envíos/Pagos (Cuando Activa PEP)

```
IDENTIDAD Y DATOS PERSONALES:
- personaExpuestaPoliticamente      (Sí/No)
- nombreRelPEP                       (Nombre relación PEP)
- remitenteCategoriaPep              (Categoría PEP)
- tipoRelacionPEP                    (Tipo relación)
- relpePField                        (Campo relación)

INFORMACIÓN LABORAL:
- remitenteCargo                     (Cargo)
- remitentePosicionEmpleo            (Posición empleo)
- remitenteNivelPosicionEmpleo       (Nivel posición)
- remitenteNombreEmpresa             (Nombre empresa/institución)
- remitenteFechaIngresoTrabajo       (Fecha ingreso)

UBICACIÓN:
- remitenteCodigoPaisTrabajo         (Código país trabajo)
- remitenteEmpleadorPais             (País empleador)
- remitenteEmpleadorProvincia        (Provincia empleador)
- remitenteEmpleadorCiudad           (Ciudad empleador)

DATOS ADICIONALES:
- remitenteActividadEconomica        (Actividad económica)
- remitenteDestinoRecursos           (Destino recursos)
- remitenteOrigenRecursos            (Origen recursos)
- remitenteMotivoTransaccion         (Motivo transacción)

INFORMACIÓN DE IDENTIDAD:
- complianceXmlComplianceIdType      (Tipo documento)
- complianceXmlComplianceIdNumber    (Número documento)
- complianceXmlComplianceDateOfBirth (Fecha nacimiento)
- complianceXmlComplianceGender      (Género)
- complianceXmlComplianceNationality (Nacionalidad)
```

---

## Mapeo de Campos: Excel → BD Actual

| Excel | BD Actual | Observación |
|---|---|---|
| Identificación | documentNumber | ✓ Mapeo directo |
| Tipo de identificación | - | ✗ **No existe en BD** |
| Nombre | lastName + secondLastName + names | ⚠️ Requiere separación |
| Fecha de nacimiento | - | ✗ **No existe en BD** |
| Puesto | employmentPosition | ✓ Mapeo directo |
| Institución | countryEmployment | ⚠️ Incompleto (no es país) |
| Fecha inicio de puesto | - | ✗ **No existe en BD** |
| Fecha fin de puesto | - | ✗ **No existe en BD** |
| Fecha fin de PEP | - | ✗ **No existe en BD** |
| Es PEP vigente | enabled | ✓ Mapeo (Sí→true, No→false) |

---

## Comparación: Excel → BD Actual vs Excel → Envíos/Pagos

### Campos que están en Excel

| Campo Excel | ¿En BD Actual? | ¿Se usa en Envíos/Pagos? |
|---|---|---|
| Tipo de identificación | ✗ No | ✓ Sí - complianceXmlComplianceIdType |
| Identificación | ✓ Sí | ✓ Sí - complianceXmlComplianceIdNumber |
| Nombre | ✓ Sí (separado) | ✓ Sí - nameXmlReceiver* |
| Fecha de nacimiento | ✗ No | ✓ Sí - complianceXmlComplianceDateOfBirth |
| Puesto | ✓ Sí | ✓ Sí - remitenteCargo, remitentePosicionEmpleo |
| Institución | ✓ Sí (incompleto) | ✓ Sí - remitenteNombreEmpresa |
| Fecha inicio de puesto | ✗ No | ✓ Sí - remitenteFechaIngresoTrabajo |
| Fecha fin de puesto | ✗ No | ✗ No |
| Fecha fin de PEP | ✗ No | ✗ No |
| Es PEP vigente | ✓ Sí | ✓ Sí - personaExpuestaPoliticamente |

---

## Vacíos Detectados

### En BD Actual que Sí Necesita Envíos/Pagos

| Campo que Falta en BD | Usado en Envíos/Pagos | Del Excel |
|---|---|---|
| **Tipo de Identificación** | complianceXmlComplianceIdType | Sí, está en Excel |
| **Fecha de Nacimiento** | complianceXmlComplianceDateOfBirth | Sí, está en Excel |
| **Fecha Inicio Puesto** | remitenteFechaIngresoTrabajo | Sí, está en Excel |
| **País de Trabajo** | remitenteCodigoPaisTrabajo | Parcial (solo institución) |
| **Nacionalidad** | complianceXmlComplianceNationality | No está en Excel |
| **Género** | complianceXmlComplianceGender | No está en Excel |

---

## Comparación Funcional

### ¿Qué Falta en BD para Completar Envíos/Pagos?

```
Datos que trae Excel pero BD no captura:
  • Tipo de identificación (Cédula, Pasaporte, etc.)
  • Fecha de nacimiento
  • Fecha inicio de puesto

Datos que Envíos/Pagos necesita pero ni BD ni Excel tienen:
  • Género
  • Nacionalidad (si es diferente a país de institución)
  • Provincia del empleador
  • Ciudad del empleador
  • Dirección del empleador
  • Teléfono de trabajo
  • Email de trabajo
```

---

## Diferencias Críticas

### 1. Tipo de Identificación
- **Excel:** Tiene (Cédula, Pasaporte, etc.)
- **BD:** No captura
- **Envíos/Pagos:** Lo necesita

### 2. Fecha de Nacimiento
- **Excel:** Tiene
- **BD:** No captura
- **Envíos/Pagos:** Lo necesita

### 3. País/Ubicación
- **Excel:** Solo institución (Municipalidad De San Jose)
- **BD:** Campo genérico (countryEmployment)
- **Envíos/Pagos:** Necesita país específico + provincia + ciudad

### 4. Nombre
- **Excel:** Nombre completo sin separar
- **BD:** Espera 3 campos separados (apellido, segundo apellido, nombres)
- **Envíos/Pagos:** Necesita nombre separado por componentes

### 5. Fechas de Puesto
- **Excel:** Tiene inicio y fin
- **BD:** No captura
- **Envíos/Pagos:** Necesita fecha de ingreso

---

## Tabla Resumida de Capacidades

| Dato | Excel | BD | Envíos/Pagos |
|---|---|---|---|
| ID | ✓ | ✓ | ✓ |
| Tipo ID | ✓ | ✗ | ✓ |
| Nombre | ✓ | ✓ (separado) | ✓ (separado) |
| Fecha Nac | ✓ | ✗ | ✓ |
| Puesto | ✓ | ✓ | ✓ |
| Institución | ✓ | ✓ (parcial) | ✓ (completo) |
| Fecha Inicio Puesto | ✓ | ✗ | ✓ |
| País | ⚠️ (derivado) | ⚠️ (genérico) | ✓ |
| Género | ✗ | ✗ | ✓ (requerido) |
| Nacionalidad | ✗ | ✗ | ✓ (requerido) |

**Leyenda:** ✓ Tiene | ✗ No tiene | ⚠️ Tiene pero incompleto

---

## Campos que NO se pueden Ignorar

Para usar lista PEP correctamente en Envíos/Pagos se requiere:

```
CRÍTICOS (Sin estos no funciona):
  □ Tipo de identificación     → Excel ✓ | BD ✗
  □ Fecha de nacimiento        → Excel ✓ | BD ✗
  □ País de trabajo            → Excel ⚠️ | BD ⚠️
  □ Puesto/Cargo              → Excel ✓ | BD ✓
  □ Institución/Empresa       → Excel ✓ | BD ✓

ALTAMENTE RECOMENDADOS (Mejoran calidad):
  □ Fecha inicio de puesto     → Excel ✓ | BD ✗
  □ Género                     → Excel ✗ | BD ✗
  □ Nacionalidad               → Excel ✗ | BD ✗
```

---

## Resumen de Diferencias

| Concepto | Excel | BD Actual | Envíos/Pagos |
|---|---|---|---|
| **Campos totales** | 10 | 12 | 114+ |
| **Campos críticos faltantes en BD** | - | 3 | 6 |
| **Información de fechas** | ✓ Completa | ✗ No | ✓ Necesaria |
| **Información de identificación** | ✓ Tipo + Número | ✗ Solo Número | ✓ Tipo + Número + Fecha Nac |
| **Información de ubicación** | ⚠️ Institución | ⚠️ Genérico | ✓ Completa (país, provincia, ciudad) |
| **Información demográfica** | ✗ | ✗ | ✓ Género, Nacionalidad |

---

## Conclusión

La lista PEP del Excel no puede cargarse directamente a la BD actual sin pérdida de información. Hay campos críticos que:

1. **Están en el Excel pero no en la BD:** Tipo de ID, Fecha de Nacimiento, Fecha de Inicio de Puesto
2. **Están en BD pero incompletos:** País/Ubicación (solo institución, no país específico)
3. **Se necesitan para Envíos/Pagos pero no están en ningún lado:** Género, Nacionalidad, Detalles de ubicación

La BD debería capturar **como mínimo** los 3 primeros campos del Excel para poder usarlos adecuadamente en la funcionalidad de Envíos/Pagos.
