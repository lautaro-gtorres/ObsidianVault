# FX CIS - Implementación Frontend (Mocks)

## Resumen

Módulo **M12-04 Gestionar FX CIS** implementado en `src/app/main/prices/fx-cis/` con arquitectura basada en el patrón `table-engine` reutilizable.

---

## Estructura creada

```
src/app/main/prices/fx-cis/
├── fx-cis.module.ts
├── fx-cis-routing.module.ts
├── presentation/
│   └── manage-fx-cis/
│       ├── manage-fx-cis.component.ts              ← Pantalla principal (table-engine)
│       ├── config/
│       │   └── manage-fx-cis.view-definition.ts    ← Columnas, filtros, constantes
│       ├── interfaces/
│       │   └── manage-fx-cis.interface.ts          ← Tipos locales
│       ├── services/
│       │   ├── fx-cis-api.service.ts               ← Clase abstracta (contrato)
│       │   └── fx-cis-api.service.mock.ts          ← Implementación por defecto con mocks
│       └── modals/
│           ├── create-fx-cis-modal/
│           │   └── create-fx-cis-modal.component.ts    ← Agregar FX CIS + rangos
│           ├── edit-fx-cis-modal/
│           │   └── edit-fx-cis-modal.component.ts      ← Modificar Spread%
│           └── manage-groups-modal/
│               └── manage-groups-modal.component.ts    ← Gestionar Grupos de Países
└── shared/
    ├── interfaces/
    │   └── fx-cis.dto.ts                           ← Todos los DTOs del dominio
    └── mocks/
        ├── fx-cis-list.mock.ts                     ← ~10 registros de ejemplo
        ├── fx-cis-filter-options.mock.ts           ← Opciones de filtros
        └── fx-cis-range-details.mock.ts            ← 3 rangos con fórmulas
```

---

## Archivos modificados

- `src/app/main/main-routing.module.ts` — Agregada ruta `prices/fx-cis` (sin restricción de permisos)

---

## Funcionalidades implementadas

### 1. Pantalla principal (Gestionar FX CIS)

- ✅ Integración con `TableEngineViewService` + `TableEngineService`
- ✅ Filtros: País, País Destino (Grupo), Tipo Servicio, Canal, Estado
- ✅ Tabla paginada con columnas: País, Grupo, Tipo Servicio, Canal, Estado
- ✅ Badges custom para Estado (Activo/Inactivo)
- ✅ Acciones por fila: Modificar, Eliminar (soft-delete), Activar
- ✅ Acciones globales: Agregar, Exportar, Gestionar Grupos
- ✅ Ordenamiento ascendente por País
- ✅ Manejo de estados: LOADING, EMPTY, SUCCESS, ERROR

### 2. Modal Agregar FX CIS

- ✅ Selección de País, Grupo, Tipo Servicio, Canal
- ✅ Carga de tasas bancarias (mock)
- ✅ Generación automática de 3 rangos con fórmulas:
  - Monto Desde/Hasta USD
  - Tipo Cambio Banco (Venta/Compra)
  - Spread % (editable, 0-100)
  - Tipo Cambio Cliente = Tasa Banco * (1 ± spread)
  - Diferencia Tipo de Cambio
- ✅ Validación de Spread obligatorio
- ✅ Guardado mock (agrega a la lista)

### 3. Modal Modificar FX CIS

- ✅ Campos deshabilitados (País, Grupo, Tipo, Canal)
- ✅ Carga de tasas bancarias actualizadas
- ✅ Solo Spread% editable en cada rango
- ✅ Recálculo automático de columnas derivadas
- ✅ Guardado mock

### 4. Modal Gestionar Grupos de Países

- ✅ Combo de grupos existentes
- ✅ Crear / Modificar / Eliminar grupo
- ✅ Grilla de países con checkboxes
- ✅ Columnas: País Destino, Cód. País, Nombre Grupo
- ✅ Validación: un país solo en un grupo por país CIS

### 5. Servicios

- ✅ `FxCisApiService` — Clase abstracta con contrato completo
- ✅ `FxCisApiMockService` — Implementación por defecto con datos estáticos
- ✅ Métodos mockeados:
  - `getFxCisList()` — Filtrado + paginación
  - `getFilterOptions()` — Catálogos
  - `getFxCisDetail()` — Detalle con rangos
  - `createFxCis()` — Creación
  - `updateFxCis()` — Actualización
  - `deleteFxCis()` — Soft-delete (Inactivo)
  - `activateFxCis()` — Reactivación
  - `getBankRates()` — Tasas banco proveedor
  - `getGroupsByCountry()` — Grupos por país
  - `getCountriesForGroupManagement()` — Países para grupos
  - `createGroup()` / `updateGroup()` / `deleteGroup()` — CRUD grupos

---

## Datos de mocks

### Tabla principal (10 registros)

| País | Grupo | Tipo Servicio | Canal | Estado |
|------|-------|---------------|-------|--------|
| Chile | Grupo 1 | Banco | Digital | Activo |
| Chile | Grupo 2 | Wallet | Presencial | Activo |
| Bolivia | Grupo 3 | Tarjeta de Pago | Digital | Activo |
| Argentina | Grupo 1 | Banco | Presencial | Inactivo |
| Paraguay | Grupo 2 | Wallet | Digital | Activo |
| Colombia | Grupo 4 | Banco | Digital | Activo |
| Chile | Grupo 5 | Tarjeta de Pago | Presencial | Inactivo |
| Bolivia | Grupo 1 | Wallet | Digital | Activo |
| Argentina | Grupo 3 | Banco | Digital | Activo |
| Paraguay | Grupo 4 | Tarjeta de Pago | Presencial | Inactivo |

### Rangos (fórmulas aplicadas)

| Rango | Desde | Hasta | Banco Venta | Banco Compra | Spread% | Cliente Venta | Dif Venta | Cliente Compra | Dif Compra |
|-------|-------|-------|-------------|--------------|---------|---------------|-----------|----------------|------------|
| 1 | 1 | 1,000 | 850.50 | 840.20 | 2.5 | 871.76 | 21.26 | 819.20 | -21.00 |
| 2 | 1,001 | 10,000 | 850.50 | 840.20 | 2.5 | 871.76 | 21.26 | 819.20 | -21.00 |
| 3 | 10,001 | Infinito | 850.50 | 840.20 | 2.5 | 871.76 | 21.26 | 819.20 | -21.00 |

---

## Pendiente por desarrollar

### Backend / Service Proxies

- [ ] Crear DTOs en el backend (`CisLatam.CISOrchestrator.Web`)
- [ ] Generar service proxies con NSwag (`nswag/refresh.bat` o similar)
- [ ] Implementar endpoints:
  - `GET /api/fx-cis` — Listado con filtros y paginación
  - `GET /api/fx-cis/{id}` — Detalle con rangos
  - `POST /api/fx-cis` — Crear
  - `PUT /api/fx-cis/{id}` — Modificar
  - `DELETE /api/fx-cis/{id}` — Soft-delete
  - `POST /api/fx-cis/{id}/activate` — Activar
  - `GET /api/fx-cis/filter-options` — Catálogos
  - `GET /api/fx-cis/bank-rates/{countryId}` — Tasas banco
  - `GET /api/fx-cis/groups` — Grupos por país
  - `POST /api/fx-cis/groups` — Crear grupo
  - `PUT /api/fx-cis/groups/{id}` — Modificar grupo
  - `DELETE /api/fx-cis/groups/{id}` — Eliminar grupo
- [ ] Tabla `HST_FXCIS` para historial de cambios

### Frontend

- [ ] Reemplazar `FxCisApiMockService` por el service proxy real
- [ ] Implementar exportación a Excel (`exportToExcel()`)
- [ ] Conectar permisos reales (`Pages.Menu.Prices.FxCis`)
- [ ] Implementar historial de cambios (visualización)
- [ ] Integrar con módulo M12-05 Tasas Banco y Proveedores
- [ ] Validaciones de negocio:
  - Validar que no exista duplicado (País + Grupo + Tipo + Canal)
  - Validar costos Terrapay al crear/modificar grupos
  - Validar que Spread esté entre 0 y 100%
- [ ] Pruebas unitarias de componentes y servicios
- [ ] Internacionalización (i18n) de textos

### UX / UI

- [ ] Agregar indicador visual para registros inactivos (texto en rojo)
- [ ] Implementar búsqueda en tiempo real de grupos
- [ ] Mejorar responsive en grilla de rangos
- [ ] Agregar tooltips con fórmulas en columnas calculadas

---

## Cómo usar

1. Navegar a: `/#/app/main/prices/fx-cis`
2. La pantalla carga con filtros vacíos y tabla vacía
3. Seleccionar filtros y presionar **Buscar**
4. Usar **Agregar** para crear nuevos registros
5. Usar dropdown de **Acciones** en cada fila para Modificar/Eliminar/Activar
6. Usar **Gestionar Grupos** para administrar grupos de países

---

## Notas técnicas

- **Arquitectura**: Basada en `TableEngine` genérico (`src/app/shared/engines/tables-engine/`)
- **Servicio mock**: Es el provider por defecto en `FxCisModule`
- **Para cambiar a servicio real**: Modificar `fx-cis.module.ts` y reemplazar `useClass: FxCisApiMockService` por el servicio proxy generado
- **Sin dependencias de backend**: El módulo funciona completamente standalone

---

## Branch

`M12-04-Gestionar-FX-CIS`
