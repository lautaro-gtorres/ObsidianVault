# Guia para Agente IA - Replicar modulo ABM (template reusable)

## Objetivo
Este documento define instrucciones concretas para que un agente de IA replique el patron del modulo `terrapay-costs` y cree un nuevo ABM en otro dominio funcional.

## Resultado esperado
El nuevo modulo debe incluir:
1. Ruta en menu principal.
2. Pantalla de consulta con grilla y filtros.
3. Alta, modificacion, baja logica y activar/desactivar.
4. Importacion xlsx.
5. Exportacion xlsx.
6. Plantilla de importacion.
7. Servicio mock de sesion (si backend no esta listo).

## Alcance tecnico minimo

### Estructura de carpetas
Crear una estructura equivalente a:
1. `components/consult`
2. `components/create`
3. `components/edit`
4. `components/import`
5. `config`
6. `services`
7. `shared/const`
8. `shared/models`

### Archivos clave
1. `*-routing.module.ts`
2. `*.module.ts`
3. `components/consult/*.component.ts`
4. `components/create/*.component.ts` + html
5. `components/edit/*.component.ts` + html
6. `components/import/*.component.ts` + html
7. `services/*-api.service.ts`
8. `config/*.view-definition.ts`
9. `shared/models/*.interface.ts`
10. `shared/const/*.const.ts`

## Contratos recomendados para el servicio API mock

### Metodos
1. `getCosts(input)`
2. `getCostForEdit(id)`
3. `createOrEdit(input)`
4. `delete(id)`
5. `toggleActive(id, active)`
6. `getDropdownData()`
7. `importFromFile(file)`
8. `exportToExcel(input)`
9. `downloadImportTemplate()`

### Reglas de negocio base
1. Duplicados: validar por llave funcional definida por negocio.
2. Baja logica: no borrar fisicamente.
3. Activacion: revertir estado de baja.
4. Importacion: devolver errores por fila.
5. Exportacion: respetar filtros actuales de la grilla.

## Pasos para el agente IA

### Paso 1 - Preparar modulo
1. Crear carpeta del modulo dentro del dominio correspondiente.
2. Agregar ruta lazy en `main-routing.module.ts`.
3. Declarar modulo y componentes.

### Paso 2 - Implementar grilla
1. Configurar `tables-engine` con columnas.
2. Definir filtros y labels.
3. Conectar evento de busqueda y paginacion.

### Paso 3 - Implementar ABM
1. Crear modal de alta.
2. Crear modal de edicion.
3. Implementar baja logica y toggle activo.
4. Agregar confirmaciones y notificaciones.

### Paso 4 - Implementar import/export
1. Importacion xlsx.

2. Validacion de catalogos y duplicados.
3. Exportacion xlsx.
4. Plantilla xlsx. descargable con headers exactos.

### Paso 5 - Documentar
1. Actualizar resumen funcional del modulo.
2. Actualizar estado de implementacion.
3. Agregar guia de migracion backend.

## Checklist de validacion rapida
1. Navega por URL y carga grilla sin errores.
2. Alta crea registro visible de inmediato.
3. Edicion impacta registro existente.
4. Baja oculta o marca inactivo segun filtro.
5. Activar/Desactivar funciona con confirmacion.
6. Importar agrega registros validos y reporta errores.
7. Exportar baja archivo con filtros aplicados.
8. Plantilla descarga headers correctos.

## Criterios de calidad para el agente
1. No romper contratos de componentes existentes.
2. Mantener nombres consistentes entre modelo, servicio y vista.
3. No introducir dependencias innecesarias.
4. Entregar cambios compilables.
5. Incluir mensajes de error claros para usuario.

## Prompt recomendado para reutilizar
Usa este prompt como base para otro ABM:

"Replica el patron de `prices/terrapay-costs` para el nuevo modulo `<NOMBRE_MODULO>`.
Implementa ruta, modulo, grilla con tables-engine, modales de alta/edicion, baja logica,
activar/desactivar, importacion CSV, exportacion CSV y plantilla.
Usa un servicio mock de sesion con los mismos contratos y valida duplicados por llave funcional.
No cambies APIs publicas existentes fuera del modulo nuevo.
Al final, actualiza documentacion funcional y tecnica del nuevo modulo."
