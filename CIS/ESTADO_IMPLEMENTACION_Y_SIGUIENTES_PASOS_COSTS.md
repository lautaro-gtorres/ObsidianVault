# Estado de Implementacion y Siguientes Pasos - M12-02

## Estado actual
El modulo `prices/terrapay-costs` esta operativo como soft ABM de sesion y permite validar flujo funcional end-to-end desde UI.

## Implementado

### Navegacion y vista
1. Ruta de modulo agregada en `main-routing`.
2. Pantalla principal con tables-engine.
3. Filtros, paginacion, acciones y modales operativos.

### ABM
1. Alta.
2. Modificacion.
3. Baja logica.
4. Activar/Desactivar.

### Importacion/exportacion
1. Importacion de archivo `.xlsx` con lectura binaria via SheetJS y validacion de filas.
2. Deteccion de duplicados por llave funcional.
3. Exportacion `.xlsx` con filtros aplicados (generada en el frontend con SheetJS).
4. Descarga de plantilla `.xlsx` (generada en el frontend con SheetJS).

> **IMPORTANTE — Generacion de Excel en frontend es TEMPORAL:**
> El uso de la libreria `xlsx` (SheetJS) en el frontend (`terrapay-costs-api.service.ts`) es un workaround de demo/prueba.
> En la implementacion final **el backend debe generar los archivos Excel** y el frontend solo descarga la URL.
> Eliminar `import * as XLSX from 'xlsx'` y toda logica de generacion/parseo de `.xlsx` del servicio frontend al integrar con backend real.

### Datos para pruebas
1. Dataset inicial con data real de negocio.
2. Catalogos mock de paises, monedas, tipos y companias.
3. Auditoria funcional simulada en memoria.

## Pendiente para productivo

### Backend
1. Entidad persistente de costos Terrapay.
2. Migracion EF Core para tabla principal e historico.
3. Reglas de negocio de duplicados en backend.

### Application layer
1. AppService real para CRUD + activar + importar + exportar + plantilla.
2. DTOs estables para consumo del frontend.
3. Respuesta de importacion con errores por fila.

### Frontend integracion real
1. Regenerar proxies NSwag.
2. Reemplazar `TerrapayCostsApiService` mock por service proxy real.
3. Mantener contratos actuales de pantalla para minimizar retrabajo.
4. **Eliminar generacion de Excel en frontend:** remover dependencia `xlsx` (SheetJS) de `terrapay-costs-api.service.ts`. Reemplazar `exportToExcel()`, `downloadImportTemplate()` e `importFromFile()` por llamadas al backend real (endpoint de exportacion/importacion). Ver nota en seccion Importacion/exportacion.

### Seguridad
1. Registrar permisos del modulo y acciones.
2. Configurar visibilidad por perfil en menu y botones.
3. Quitar bypasses temporales de autorizacion si existieran.

### QA
1. Casos de alta/modificacion/baja/activacion.
2. Importacion: exitos, duplicados, formato invalido.
3. Exportacion y plantilla con estructura valida.

## Definicion de listo (DoD) sugerida
1. ABM persistente en base real.
2. Import/export integrados a backend.
3. Permisos productivos activos.
4. Suite minima de pruebas de regresion aprobada.
