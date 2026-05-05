# Estado de Implementacion y Siguientes Pasos - M12-04

## Estado actual
El modulo `prices/fx-cis` esta operativo como soft ABM de sesion y permite validar flujo funcional end-to-end desde UI sin dependencia de backend.

## Implementado

### Navegacion y vista
1. Ruta lazy agregada en `main-routing.module.ts` en path `prices/fx-cis`.
2. Entrada de menu agregada en `app-navigation.service.ts` bajo el grupo Precios.
3. Pantalla principal con `TableEngineViewService` + columnas, filtros y paginacion.
4. Animacion `[@routerTransition]` y `ViewEncapsulation.None` aplicados al componente principal.

### ABM
1. Alta (modal Agregar FX CIS con seleccion de Pais, Grupo, Tipo Servicio, Canal y carga de rangos).
2. Modificacion (modal con campos deshabilitados y solo Spread% editable por rango).
3. Baja logica: cambia estado a Inactivo sin borrar el registro.
4. Activar: revierte el estado a Activo con confirmacion ABP.

### Rangos y formulas
1. Tres rangos fijos: Rango 1 (1-1000), Rango 2 (1001-10000), Rango 3 (10001-Infinito).
2. Tasas Banco Proveedor (Venta/Compra) cargadas desde mock por pais.
3. Recalculo en tiempo real al modificar Spread%:
   - Tipo Cambio Cliente (Venta) = Tasa Banco Venta * (1 + spread).
   - Diferencia Tipo de Cambio (Venta) = TC Cliente Venta - TC Banco Venta.
   - Tipo Cambio Cliente (Compra) = Tasa Banco Compra * (1 - spread).
   - Diferencia Tipo de Cambio (Compra) = TC Cliente Compra - TC Banco Compra.
4. Validacion: Spread% obligatorio para todos los rangos antes de guardar (0-100).

### Gestion de Grupos de Paises
1. Modal Gestionar Grupos con listado de grupos existentes.
2. Crear grupo con nombre y seleccion de paises (checkbox).
3. Modificar nombre y paises del grupo.
4. Eliminar grupo con validacion de uso en FX CIS.
5. Grilla con columnas: Pais Destino, Cod. Pais, Nombre Grupo.

### Exportacion
1. Exportacion del grid filtrado a `.xlsx` usando SheetJS en el frontend (`manage-fx-cis.component.ts`).
2. Columnas exportadas: Pais, Pais Destino (Grupo), Tipo Servicio, Canal, Estado.
3. Nombre de archivo incluye fecha de generacion.

> **IMPORTANTE — Generacion de Excel en frontend es TEMPORAL:**
> El uso de la libreria `xlsx` (SheetJS) en `manage-fx-cis.component.ts` es un workaround de demo/prueba.
> En la implementacion final **el backend debe generar los archivos Excel** y el frontend solo descarga la URL del endpoint.
> Eliminar `import * as XLSX from 'xlsx'` y toda la logica de generacion de `.xlsx` de `exportToExcel()` al integrar con backend real.

### Confirmaciones y notificaciones
1. Eliminar y Activar usan `message.confirm()` de ABP (no `confirm()` nativo).
2. Notificaciones de exito y error con `notify.success()` / `notify.error()`.

### Datos para pruebas
1. Dataset inicial con 10 registros (Argentina, Bolivia, Chile, Colombia, Paraguay).
2. Catalogos mock: 5 paises CIS, 5 grupos, 3 tipos de servicio, 2 canales, 2 estados.
3. Tasas banco mock por pais (sellRate: 850.50, buyRate: 840.20).
4. Rangos mock con formulas precalculadas al 2.5% de spread.

### Servicio
1. `FxCisApiService` — Clase abstracta con contrato completo.
2. `FxCisApiMockService` — Implementacion en memoria con todos los metodos.
3. Metodos implementados:
   - `getFxCisList()` — Filtrado + paginacion + ordenamiento por pais.
   - `getFilterOptions()` — Catalogos de filtros.
   - `getFxCisDetail()` — Detalle con rangos para edicion.
   - `createFxCis()` — Alta en lista en memoria.
   - `updateFxCis()` — Actualizacion en memoria.
   - `deleteFxCis()` — Soft-delete (estado Inactivo).
   - `activateFxCis()` — Reactivacion (estado Activo).
   - `getBankRates()` — Tasas banco proveedor por pais.
   - `getGroupsByCountry()` — Grupos por pais.
   - `getCountriesForGroupManagement()` — Paises disponibles para grupos.
   - `createGroup()` / `updateGroup()` / `deleteGroup()` — CRUD de grupos.

## Bugs corregidos en esta sesion
1. `EditFxCisModalComponent`: metodo `nonOnChanges()` (typo) reemplazado por `ngOnChanges(changes: SimpleChanges)` con interfaz `OnChanges` correctamente implementada. Sin esta correccion el modal de modificacion no cargaba datos.
2. Confirmaciones con `confirm()` nativo reemplazadas por `message.confirm()` de ABP.
3. Icono `fa-file-excel-o` (Font Awesome 4) corregido a `fa-file-excel` (Font Awesome 5/6).
4. `_notifyService` (NotifyService inyectado manualmente) reemplazado por `notify` de `AppComponentBase`.

## Estructura de carpetas

```
src/app/main/prices/fx-cis/
├── fx-cis.module.ts
├── fx-cis-routing.module.ts
├── presentation/
│   └── manage-fx-cis/
│       ├── manage-fx-cis.component.ts
│       ├── config/
│       │   └── manage-fx-cis.view-definition.ts
│       ├── interfaces/
│       │   └── manage-fx-cis.interface.ts
│       ├── services/
│       │   ├── fx-cis-api.service.ts             (clase abstracta)
│       │   └── fx-cis-api.service.mock.ts        (implementacion en memoria)
│       └── modals/
│           ├── create-fx-cis-modal/
│           │   └── create-fx-cis-modal.component.ts
│           ├── edit-fx-cis-modal/
│           │   └── edit-fx-cis-modal.component.ts
│           └── manage-groups-modal/
│               └── manage-groups-modal.component.ts
└── shared/
    ├── interfaces/
    │   └── fx-cis.dto.ts
    └── mocks/
        ├── fx-cis-list.mock.ts
        ├── fx-cis-filter-options.mock.ts
        └── fx-cis-range-details.mock.ts
```

## Pendiente para productivo

### Backend
1. Entidad persistente `FXCIS` con campos: Id, PaisId, GrupoId, TipoServicio, Canal, Estado, campos de auditoria ABP.
2. Entidad `FXCIS_Rango` con campos: Id, FXCISId, RangeId, SpreadIni, SpreadFin, TipoCambioClienteVenta, TipoCambioClienteCompra, DiferenciaVenta, DiferenciaCompra.
3. Tabla `HST_FXCIS` para historial de cambios con campos: Id, Spread%INI, Spread%FIN, FXCISId, FXCISRangoid, CreationTime, CreatorUserId, LastModificationTime, LastModifierUserId, IsDeleted, DeleterUserId, DeletionTime, FechaBaja, FechaActiva, UsuBaja, UsuActiva.
4. Entidad `GrupoPaises` con relacion N:M a paises.
5. Migraciones EF Core para todas las tablas.
6. Integracion con modulo M12-05 Tasas Banco y Proveedores para obtener cotizaciones reales.

### Application layer
1. AppService real: `FxCisAppService` con metodos CRUD + activar + exportar.
2. DTOs de entrada y salida estables.
3. Validacion de duplicados por llave funcional (Pais + Grupo + TipoServicio + Canal).
4. Logica de historico en HST_FXCIS al crear, modificar, eliminar y activar.
5. Validacion de costos Terrapay iguales en todos los paises del grupo.

### Frontend integracion real
1. Regenerar proxies NSwag tras implementar el backend.
2. Reemplazar `FxCisApiMockService` por el service proxy generado (cambiar `useClass` en `fx-cis.module.ts`).
3. **Implementar exportacion a Excel real via endpoint del backend** y eliminar la generacion de xlsx en el frontend (`manage-fx-cis.component.ts`). Ver nota en seccion Exportacion.
4. Conectar visualizacion de historico (M12 04.04 Visualizar Historico FX CIS).

### Seguridad
1. Registrar permiso `Pages.Menu.Prices.FxCis` y subpermisos por accion.
2. Reemplazar `|| true` en guardas de botones por permisos reales.
3. Configurar visibilidad por perfil en menu y botones de accion.

### QA
1. Casos de alta con una, varias y todas las opciones de Grupo/Tipo/Canal.
2. Validacion de Spread entre 0 y 100%.
3. Recalculo correcto de formulas al modificar Spread.
4. Eliminacion y activacion con confirmacion.
5. Exportacion `.xlsx` generada por backend (reemplazar implementacion temporal de frontend).
6. Gestion de grupos: crear, modificar, eliminar (con y sin FX CIS asociados).

## Definicion de listo (DoD) sugerida
1. ABM persistente en base de datos real con historico HST_FXCIS.
2. Tasas banco proveedor tomadas del modulo M12-05 en tiempo real.
3. Exportacion a Excel desde backend.
4. Permisos productivos activos por rol.
5. Suite minima de pruebas de regresion aprobada.

## Como usar el modulo ahora
1. Navegar a: `/#/app/main/prices/fx-cis`
2. La pantalla carga con filtros vacios y grilla vacia.
3. Seleccionar filtros y presionar **Buscar** para ver registros.
4. Usar **Agregar** para crear nuevos FX CIS.
5. Usar dropdown **Acciones** en cada fila para Modificar / Eliminar / Activar.
6. Usar **Gestionar Grupos** para administrar grupos de paises.
7. Usar **Exportar** para descargar el grid actual en CSV.
