# Plan de Implementación: M12-03 Fee Cliente — Frontend Angular

> **Para agente IA:** Este documento es el contexto completo para implementar el módulo `prices/fee-cliente` en el proyecto Angular `CisLatam.CISOrchestrator.UI`. Leer completo antes de empezar.

---

## Contexto del Proyecto

- **Repositorio Angular:** `c:\Users\torre\source\repos\CisLatam.CISOrchestrator.UI`
- **Módulo de referencia (REPLICAR PATRÓN):** `src/app/main/prices/terrapay-costs/`
- **Módulo nuevo:** `src/app/main/prices/fee-cliente/`
- **Documento funcional:** `M12-03 - Gestionar Fee Cliente.md` (ver adjunto o en `c:\Users\torre\OneDrive\Documentos\Obsidian Vault\CIS\Proyecto CIS\Documentacion\Modulos\M12-03 - Gestionar Fee Cliente.md`)

### Stack técnico relevante
- Angular (versión del proyecto)
- `xlsx ^0.18.5` ya instalado — usar `import * as XLSX from 'xlsx'`
- `ngx-bootstrap` para modales (`ModalDirective`, `bsModal`)
- `TablesEngineModule` + `TableEngineBuiltInRenderersModule` para grillas
- `AppComponentBase` de `@shared/common/app-component-base` — extender en todos los componentes
- `AppSharedModule` + `AdminSharedModule` — importar en el nuevo módulo

---

## Arquitectura del Módulo

### Estructura completa de carpetas y archivos a crear

```
src/app/main/prices/fee-cliente/
├── fee-cliente-routing.module.ts
├── fee-cliente.module.ts
├── components/
│   ├── consult/
│   │   └── fee-cliente-consult.component.ts       (+ .html si el patrón lo usa separado)
│   ├── create/
│   │   ├── fee-cliente-create.component.ts
│   │   └── fee-cliente-create.component.html
│   ├── detail/
│   │   ├── fee-cliente-detail.component.ts
│   │   └── fee-cliente-detail.component.html
│   ├── edit/
│   │   ├── fee-cliente-edit-modal.component.ts
│   │   └── fee-cliente-edit-modal.component.html
│   └── simulate/
│       ├── fee-cliente-simulate-modal.component.ts
│       └── fee-cliente-simulate-modal.component.html
├── services/
│   └── fee-cliente-api.service.ts
├── config/
│   └── fee-cliente.view-definition.ts
└── shared/
    ├── models/
    │   └── fee-cliente.interface.ts
    └── const/
        └── fee-cliente.const.ts
```

### Archivos a modificar (solo 1)
- `src/app/main/main-routing.module.ts` → agregar entrada lazy route

---

## Routing

### Ruta lazy en main-routing.module.ts
Agregar junto a los demás routes de `prices/`:
```typescript
{
  path: "prices/fee-cliente",
  loadChildren: () =>
    import("./prices/fee-cliente/fee-cliente.module")
      .then((m) => m.FeeClienteModule),
  data: { permission: "" }   // string vacío = sin guard, igual que terrapay-costs
},
```

### Routing interno (fee-cliente-routing.module.ts)
```typescript
const routes: Routes = [
  { path: "", component: FeeClienteConsultComponent, pathMatch: "full" },
  { path: "create", component: FeeClienteCreateComponent },
  { path: "detail/:id", component: FeeClienteDetailComponent },
];
```
**IMPORTANTE:** Create y Detail son páginas completas (full-page navigation), NO modales.
Edit y Simulate son modales (`@ViewChild`) incrustados en los componentes que los usan.

### Navegación programática
- Consult → Create: `this.router.navigate(['create'], { relativeTo: this.route })`
- Consult → Detail: `this.router.navigate(['detail', id], { relativeTo: this.route })`
- Create/Detail → Consult (back): `this.router.navigate(['../'], { relativeTo: this.route })`

---

## Module (fee-cliente.module.ts)

```typescript
@NgModule({
  declarations: [
    FeeClienteConsultComponent,
    FeeClienteCreateComponent,
    FeeClienteDetailComponent,
    FeeClienteEditModalComponent,
    FeeClienteSimulateModalComponent,
  ],
  imports: [
    AppSharedModule,
    AdminSharedModule,
    FeeClienteRoutingModule,
    TablesEngineModule,
    TableEngineBuiltInRenderersModule,
  ],
  providers: [FeeClienteApiService],
})
export class FeeClienteModule {}
```

---

## Permisos (CRÍTICO para pruebas)

**NUNCA agregar `canActivate` guards en el routing.**

En todos los templates, usar el patrón `|| true` igual que terrapay-costs:
```html
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Create') || true"
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Edit') || true"
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Delete') || true"
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Activate') || true"
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Export') || true"
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Detail') || true"
*ngIf="isGranted('Pages.Menu.Prices.FeeCliente.Simulate') || true"
```
Esto hace que todos los botones sean siempre visibles durante las pruebas de frontend.

---

## Interfaces TypeScript (fee-cliente.interface.ts)

```typescript
export interface FeeClienteHeaderDto {
  id: number;
  paisId: number;
  paisNombre: string;
  paisDestinoGrupoId: number;
  paisDestinoGrupoNombre: string;
  tipoTransaccionId: number;
  tipoTransaccionNombre: string;
  productoId: number;
  productoNombre: string;
  tipoServicioId: number;
  tipoServicioNombre: string;
  nombreInstitucionId: number | null;
  nombreInstitucionNombre: string | null;
  canalId: number;
  canalNombre: string;
  isDeleted: boolean;
  usuBaja: string | null;
  fechaBaja: string | null;
  usuActiva: string | null;
  fechaActiva: string | null;
}

export interface FeeClienteRangeDto {
  id: number;
  feeClienteHeaderId: number;
  montoDesdeUsd: number;
  montoHastaUsd: number;
  costoTerrapayFijo: number | null;
  costoTerrapayPorc: number | null;
  markUp: number | null;
  gananciaCisMin: number | null;
  feeClienteMin: number | null;
  comisionPorc: number | null;
  comisionFija: number | null;
  bonifica: boolean;
  porcentajeBonificacion: number | null;
  isDeleted: boolean;
}

export interface FeeClienteDto {
  header: FeeClienteHeaderDto;
  ranges: FeeClienteRangeDto[];
}

export interface GetFeeClienteInput {
  paisId: number | null;
  paisDestinoGrupoId: number | null;
  tipoTransaccionId: number | null;
  productoId: number | null;
  tipoServicioId: number | null;
  nombreInstitucionId: number | null;
  canalId: number | null;
  /** -1 = todos, 1 = activos, 0 = inactivos */
  activeFilter: number | null;
  sorting: string | null;
  skipCount: number;
  maxResultCount: number;
}

export interface CreateFeeClienteInput {
  paisId: number;
  paisDestinoGrupoId: number;
  tipoTransaccionId: number;
  productoId: number;
  tipoServicioId: number;
  nombreInstitucionId: number | null;
  canalId: number;
  ranges: UpdateFeeClienteRangeInput[];
}

export interface UpdateFeeClienteRangeInput {
  rangeId: number;
  comisionPorc: number | null;
  comisionFija: number | null;
  bonifica: boolean;
  porcentajeBonificacion: number | null;
}

export interface SimulateFeeClienteInput {
  headerId: number;
  rangeId: number;
  monedaId: number;
  montoEnviar: number;
  comisionFija: number | null;
  comisionPorc: number | null;
}

export interface SimulateFeeClienteResult {
  gananciaFeeML_Porc: number | null;
  gananciaFeeML_Fijo: number | null;
  gananciaFxML: number | null;
  montoABonificar: number | null;
  rentabilidadTotalML: number | null;
  rentabilidadTotalME: number | null;
}

export interface FeeClienteLookupDto {
  id: number;
  displayName: string;
}

export interface FeeClienteDropdownData {
  paises: FeeClienteLookupDto[];
  paisesDestinoGrupo: FeeClienteLookupDto[];
  tiposTransaccion: FeeClienteLookupDto[];
  productos: FeeClienteLookupDto[];
  tiposServicio: FeeClienteLookupDto[];
  instituciones: FeeClienteLookupDto[];
  canales: FeeClienteLookupDto[];
  monedas: FeeClienteLookupDto[];
}

export interface FeeClientePagedResultDto {
  items: FeeClienteHeaderDto[];
  totalCount: number;
}

/** Fila mapeada para tables-engine */
export interface FeeClienteHeaderRow extends Record<string, unknown> {
  id: number;
  paisNombre: string;
  paisDestinoGrupoNombre: string;
  tipoTransaccionNombre: string;
  productoNombre: string;
  tipoServicioNombre: string;
  nombreInstitucionNombre: string | null;
  canalNombre: string;
  isDeleted: boolean;
  estadoLabel: string;
}

/** Estado de filtros para view-definition builder */
export interface FeeClienteFilterState {
  paises: FeeClienteLookupDto[];
  paisesDestinoGrupo: FeeClienteLookupDto[];
  tiposTransaccion: FeeClienteLookupDto[];
  productos: FeeClienteLookupDto[];
  tiposServicio: FeeClienteLookupDto[];
  instituciones: FeeClienteLookupDto[];
  canales: FeeClienteLookupDto[];
  isLoading: boolean;
  hasError: boolean;
}

/** Modelo de filtros para tables-engine */
export interface FeeClienteFilterModel extends Record<string, string | number | boolean | null | undefined> {
  paisId: string | null;
  paisDestinoGrupoId: string | null;
  tipoTransaccionId: string | null;
  productoId: string | null;
  tipoServicioId: string | null;
  nombreInstitucionId: string | null;
  canalId: string | null;
  activeFilter: string | null;
}
```

---

## Constantes (fee-cliente.const.ts)

### 19 rangos predefinidos
**VERIFICAR** en el documento original `M12-03 - Gestionar Fee Cliente.md` la sección "Tabla de Rangos Predefinidos". Si no está disponible, usar estos 19:
```typescript
export const PREDEFINED_RANGES: Array<{ desde: number; hasta: number }> = [
  { desde: 1, hasta: 50 },
  { desde: 51, hasta: 100 },
  { desde: 101, hasta: 200 },
  { desde: 201, hasta: 300 },
  { desde: 301, hasta: 400 },
  { desde: 401, hasta: 500 },
  { desde: 501, hasta: 600 },
  { desde: 601, hasta: 700 },
  { desde: 701, hasta: 800 },
  { desde: 801, hasta: 900 },
  { desde: 901, hasta: 1000 },
  { desde: 1001, hasta: 1500 },
  { desde: 1501, hasta: 2000 },
  { desde: 2001, hasta: 3000 },
  { desde: 3001, hasta: 4000 },
  { desde: 4001, hasta: 5000 },
  { desde: 5001, hasta: 6000 },
  { desde: 6001, hasta: 10000 },
  { desde: 10001, hasta: 999999999 },
];
```

### Opciones estáticas de dropdowns
```typescript
export const TIPO_TRANSACCION_OPTIONS = [
  { value: 1, label: "P2P" },
  { value: 2, label: "P2B" },
  { value: 3, label: "B2B" },
  { value: 4, label: "B2P" },
] as const;

export const PRODUCTO_OPTIONS = [
  { value: 1, label: "RIA" },
  { value: 2, label: "Terrapay" },
  { value: 3, label: "WU" },
] as const;

export const TIPO_SERVICIO_OPTIONS = [
  { value: 1, label: "Banco" },
  { value: 2, label: "Wallet" },
  { value: 3, label: "Tarjeta de Pago" },
] as const;

export const CANAL_OPTIONS = [
  { value: 1, label: "Digital" },
  { value: 2, label: "Presencial" },
] as const;

export const ESTADO_FILTER_OPTIONS = [
  { value: "-1", label: "Todos" },
  { value: "1", label: "Activos" },
  { value: "0", label: "Inactivos" },
] as const;

export const BONIFICA_OPTIONS = [
  { value: true, label: "SI" },
  { value: false, label: "NO" },
] as const;

// Instituciones agrupadas por tipo de servicio (mock)
export const INSTITUCIONES_POR_TIPO_SERVICIO: Record<number, Array<{ id: number; nombre: string }>> = {
  1: [{ id: 10, nombre: "Banco Nación" }, { id: 11, nombre: "Banco Estado" }],  // Banco
  2: [{ id: 20, nombre: "Nequi" }, { id: 21, nombre: "Daviplata" }],             // Wallet
  3: [{ id: 30, nombre: "Visa" }, { id: 31, nombre: "Mastercard" }],             // Tarjeta de Pago
};

// Países destino / grupos mock
export const PAISES_DESTINO_GRUPO_MOCK = [
  { id: 1, displayName: "Grupo Colombia" },
  { id: 2, displayName: "Grupo SEPA" },
  { id: 3, displayName: "Grupo LATAM" },
];

// Países CIS mock
export const PAISES_CIS_MOCK = [
  { id: 1, displayName: "Chile" },
  { id: 2, displayName: "Bolivia" },
  { id: 3, displayName: "Argentina" },
];
```

### Llave funcional para duplicados
```typescript
export function buildFeeClienteKey(input: {
  paisId: number;
  paisDestinoGrupoId: number;
  tipoTransaccionId: number;
  productoId: number;
  tipoServicioId: number;
  nombreInstitucionId: number | null;
  canalId: number;
}): string {
  return `${input.paisId}-${input.paisDestinoGrupoId}-${input.tipoTransaccionId}-${input.productoId}-${input.tipoServicioId}-${input.nombreInstitucionId ?? 0}-${input.canalId}`;
}
```

### Constantes generales
```typescript
export const FEE_CLIENTE_CONSTANTS = {
  DEFAULT_PAGE_SIZE: 20,
  VALIDATION: {
    MIN_COMISION_PORC: 0,
    MAX_COMISION_PORC: 100,
    MIN_BONIFICACION: 0,
    MAX_BONIFICACION: 100,
    MIN_COMISION_FIJA: 0,
  },
} as const;
```

---

## Servicio Mock (fee-cliente-api.service.ts)

### Datos iniciales mock
3 registros de FeeClienteDto con sus 19 rangos cada uno:
- Chile (id:1) + Grupo Colombia (id:1) + P2P (id:1) + Terrapay (id:2) + Wallet (id:2) + Nequi (id:20) + Presencial (id:2) → isDeleted: false
- Chile (id:1) + Grupo SEPA (id:2) + P2P (id:1) + Terrapay (id:2) + Banco (id:1) + null + Digital (id:1) → isDeleted: false
- Bolivia (id:2) + Grupo Colombia (id:1) + P2B (id:2) + WU (id:3) + Banco (id:1) + null + Presencial (id:2) → isDeleted: true

Para los rangos, pre-cargar CostoTerrapay con los valores del prototipo:
- Rangos 1-50 a 601-700: usar mezcla de fijo/porcentaje como se muestra en la pantalla 2 del prototipo
- Si no hay costo disponible, usar `costoTerrapayFijo: 1.5` como default

### Estructura del servicio
```typescript
@Injectable()
export class FeeClienteApiService {
  private _nextId = 100;
  private _nextRangeId = 1000;
  private readonly _data: FeeClienteDto[] = [...INITIAL_MOCK_DATA];

  getFees(input: GetFeeClienteInput): Observable<FeeClientePagedResultDto>
  getFeeForEdit(id: number): Observable<FeeClienteDto>
  createFee(input: CreateFeeClienteInput): Observable<void>
  updateRange(input: UpdateFeeClienteRangeInput): Observable<void>
  delete(id: number): Observable<void>
  activate(id: number): Observable<void>
  simulateFee(input: SimulateFeeClienteInput): Observable<SimulateFeeClienteResult>
  getDropdownData(): Observable<FeeClienteDropdownData>
  exportToExcel(input: GetFeeClienteInput): Observable<Blob>
}
```

### Filtrado en getFees
- Filtrar por cada campo del input si no es null
- `activeFilter`: -1 = devolver todos, 1 = isDeleted=false, 0 = isDeleted=true
- Paginación: `slice(skipCount, skipCount + maxResultCount)`
- Retornar `of(result).pipe(delay(300))`

### Lógica de delete (baja lógica)
```typescript
delete(id: number): Observable<void> {
  const fee = this._data.find(f => f.header.id === id);
  if (fee) {
    fee.header.isDeleted = true;
    fee.header.fechaBaja = new Date().toISOString();
    fee.header.usuBaja = "mock-user";
    fee.ranges.forEach(r => r.isDeleted = true);
  }
  return of(undefined).pipe(delay(300));
}
```

### Lógica de activate
```typescript
activate(id: number): Observable<void> {
  const fee = this._data.find(f => f.header.id === id);
  if (fee) {
    fee.header.isDeleted = false;
    fee.header.fechaActiva = new Date().toISOString();
    fee.header.usuActiva = "mock-user";
    fee.ranges.forEach(r => r.isDeleted = false);
  }
  return of(undefined).pipe(delay(300));
}
```

### Lógica de updateRange
- Encontrar el rango por rangeId
- Actualizar comisionPorc, comisionFija, bonifica, porcentajeBonificacion
- Si comisionPorc tiene valor → comisionFija = null; si comisionFija tiene valor → comisionPorc = null
- Recalcular markUp, gananciaCisMin, feeClienteMin (usar valores mock hardcoded con TODO comment)
- En producción aquí se insertaría en HST_FEECLIENTE

### Lógica de simulateFee (mock)
```typescript
simulateFee(input: SimulateFeeClienteInput): Observable<SimulateFeeClienteResult> {
  // TODO: Integrar con fórmulas reales de M12-02, M12-04, M12-05
  const range = this._findRange(input.headerId, input.rangeId);
  const comision = input.comisionPorc
    ? input.montoEnviar * (input.comisionPorc / 100)
    : (input.comisionFija ?? 0);
  const mockResult: SimulateFeeClienteResult = {
    gananciaFeeML_Porc: input.comisionPorc ?? null,
    gananciaFeeML_Fijo: input.comisionFija ?? null,
    gananciaFxML: comision * 0.1,
    montoABonificar: range?.bonifica ? comision * ((range.porcentajeBonificacion ?? 0) / 100) : 0,
    rentabilidadTotalML: comision,
    rentabilidadTotalME: comision * 1.05,
  };
  return of(mockResult).pipe(delay(500));
}
```

### Export con XLSX
```typescript
exportToExcel(input: GetFeeClienteInput): Observable<Blob> {
  const filtered = this._applyFilters(input);
  const rows = filtered.map(f => f.header);
  const wb = XLSX.utils.book_new();
  const headers = ['País', 'País Destino (Grupo)', 'Tipo Transacción', 'Producto',
                   'Tipo Servicio', 'Nombre Institución', 'Canal', 'Estado'];
  const data = rows.map(h => [
    h.paisNombre, h.paisDestinoGrupoNombre, h.tipoTransaccionNombre,
    h.productoNombre, h.tipoServicioNombre, h.nombreInstitucionNombre ?? '',
    h.canalNombre, h.isDeleted ? 'Inactivo' : 'Activo'
  ]);
  const ws = XLSX.utils.aoa_to_sheet([headers, ...data]);
  XLSX.utils.book_append_sheet(wb, ws, 'Fee Cliente');
  const wbArray = XLSX.write(wb, { bookType: 'xlsx', type: 'array' });
  const blob = new Blob([wbArray], {
    type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
  });
  return of(blob).pipe(delay(400));
}
```

---

## View Definition (fee-cliente.view-definition.ts)

### Header
```typescript
export const FEE_CLIENTE_HEADER: TableEngineHeader = {
  title: "Gestionar Fee Cliente",
  description: "Administración de FEE Cliente por país y corredor",
};
```

### Columnas de la grilla
```typescript
export function buildFeeClienteColumns(): TableEngineColumn[] {
  return [
    { field: "paisNombre", header: "País", sortable: true, width: "100px" },
    { field: "paisDestinoGrupoNombre", header: "País Destino (Grupo)", width: "150px" },
    { field: "tipoTransaccionNombre", header: "Tipo Transacción", width: "130px" },
    { field: "productoNombre", header: "Producto", width: "100px" },
    { field: "tipoServicioNombre", header: "Tipo Servicio", width: "120px" },
    { field: "nombreInstitucionNombre", header: "Nombre Institución", width: "140px" },
    { field: "canalNombre", header: "Canal", width: "90px" },
    {
      field: "estadoLabel", header: "Estado", width: "80px",
      format: "badgeCustom",
      badgeClassResolver: (row) => row["isDeleted"] ? "badge badge-danger" : "badge badge-success",
      badgeTextResolver: (row) => row["isDeleted"] ? "Inactivo" : "Activo",
    },
  ];
}
```

### Filtros
```typescript
export function buildFeeClienteFilterRows(state: FeeClienteFilterState): TableEngineFilterRows {
  // Fila 1: Pais, PaisDestinoGrupo, TipoTransaccion, Producto
  // Fila 2: TipoServicio, NombreInstitucion, Canal, Estado
  // Cada filtro es type: "select" con allowEmptyOption: true, emptyOptionLabel: "Seleccione una opción"
  // Estado: type: "select", allowEmptyOption: false, usando ESTADO_FILTER_OPTIONS
}
```

---

## Consult Component (fee-cliente-consult.component.ts)

Extender `AppComponentBase`. Inyectar: `FeeClienteApiService`, `ActivatedRoute`, `Router`.

### Propiedades
```typescript
columns: TableEngineColumn[] = buildFeeClienteColumns();
filterRows: TableEngineFilterRows = [];
header = FEE_CLIENTE_HEADER;
toolbarLabels = FEE_CLIENTE_TOOLBAR_LABELS;
emptyMessage = FEE_CLIENTE_EMPTY_MESSAGE;
rows: FeeClienteHeaderRow[] = [];
totalCount = 0;
isLoading = false;
filterState: FeeClienteFilterState = { paises: [], paisesDestinoGrupo: [], ... };
private _lastInput: GetFeeClienteInput = { ...defaultInput };
```

### Métodos
- `ngOnInit()` → `loadDropdownData()` + `search()`
- `loadDropdownData()` → llamar `getDropdownData()` → actualizar filterState → rebuild filterRows
- `search(filterModel?, pageEvent?)` → construir input → llamar `getFees()` → mapear a rows
- `navigateCreate()` → `router.navigate(['create'], { relativeTo: route })`
- `openDetail(id)` → `router.navigate(['detail', id], { relativeTo: route })`
- `deleteFee(id)` → `message.confirm(...)` → `delete(id)` → `search()` → notify success
- `activateFee(id)` → `message.confirm(...)` → `activate(id)` → `search()` → notify success
- `exportToExcel()` → `exportToExcel(lastInput)` → blob download

### Template
- `<app-tables-engine>` con todas las propiedades
- Botones con `|| true` para permisos
- Columna de acción condicional: si `!row.isDeleted` mostrar [Eliminar] + [Detalle]; si `row.isDeleted` mostrar [Activar]
- Fila inactiva: aplicar clase CSS usando `rowClassResolver: (row) => row.isDeleted ? 'table-danger' : ''`

---

## Create Component (fee-cliente-create.component.ts)

Full-page component. Extender `AppComponentBase`. Inyectar: `FeeClienteApiService`, `Router`, `ActivatedRoute`.

### Propiedades
```typescript
headerCreated = false;   // Controla las 2 fases del create
saving = false;
loadingDropdowns = false;
loadingCostos = false;
dropdownData: FeeClienteDropdownData;
headerForm = { paisId: null, paisDestinoGrupoId: null, ... };
ranges: FeeClienteRangeDto[] = [];  // 19 rangos una vez que headerCreated=true

// Para el modal de simulación
@ViewChild('simulateModal') simulateModal: FeeClienteSimulateModalComponent;
```

### Métodos
- `ngOnInit()` → loadDropdownData()
- `onTipoServicioChange()` → filtrar instituciones disponibles según tipoServicioId
- `crearCabecera()` → validar todos los campos requeridos → verificar duplicados → si OK: `headerCreated=true` + `buildRanges()`
- `buildRanges()` → crear 19 FeeClienteRangeDto con PREDEFINED_RANGES + cargar CostoTerrapay del servicio mock
- `onComisionPorcChange(range)` → si tiene valor: `range.comisionFija = null` + recalcular
- `onComisionFijaChange(range)` → si tiene valor: `range.comisionPorc = null` + recalcular
- `onBonificaChange(range)` → si NO: `range.porcentajeBonificacion = null`
- `openSimulate(range)` → `simulateModal.show(headerId=0, rangeId=range.id)`  *(headerId=0 en modo create)*
- `guardar()` → validar que todos los rangos tengan al menos ComisionPorc o ComisionFija → `createFee(input)` → notify → navigate back
- `cancelar()` → `message.confirm("¿Desea cancelar? Se perderán los datos")` → navigate back

### Nota importante
En modo "create", el headerId aún no existe. Para el simulador desde create, pasar `headerId: -1` y en el servicio manejar ese caso (usar datos del formulario directamente o simplemente hacer cálculo sin headerId).

---

## Detail Component (fee-cliente-detail.component.ts)

Full-page component. Extender `AppComponentBase`. Inyectar: `FeeClienteApiService`, `ActivatedRoute`, `Router`.

### Propiedades
```typescript
isLoading = false;
feeCliente: FeeClienteDto | null = null;
@ViewChild('editModal') editModal: FeeClienteEditModalComponent;
@ViewChild('simulateModal') simulateModal: FeeClienteSimulateModalComponent;
```

### Métodos
- `ngOnInit()` → leer `route.params.id` → `getFeeForEdit(id)` → asignar
- `openEdit(range)` → `editModal.show(range, feeCliente.header.id)`
- `openSimulate(range)` → `simulateModal.show(feeCliente.header.id, range.id)`
- `onEditSaved()` → recargar `getFeeForEdit(id)`
- `exportarDetalle()` → llamar export con filtro por id
- `volver()` → `router.navigate(['../../'], { relativeTo: route })`  *(o simplemente `['../']`)*

---

## Edit Modal (fee-cliente-edit-modal.component.ts)

Patrón idéntico a `CreateTerrapayCostModalComponent`. Usar `ModalDirective`, `@ViewChild('modal')`.

### Props
```typescript
@Output() onSave = new EventEmitter<void>();
active = false;
saving = false;
headerId: number;
range: FeeClienteRangeDto | null = null;
editRange = { comisionPorc: null, comisionFija: null, bonifica: false, porcentajeBonificacion: null };
```

### Métodos
- `show(range: FeeClienteRangeDto, headerId: number)` → copiar valores editables al editRange → `this.active = true; modal.show()`
- `onComisionPorcChange()` → mutual exclusion
- `onComisionFijaChange()` → mutual exclusion
- `onBonificaChange()` → if NO: porcentajeBonificacion = null
- `save()` → `updateRange(input)` → notify success → `onSave.emit()` → `close()`
- `close()` → `modal.hide(); active = false`

### Template
- Mostrar campos del header (read-only, de la prop `range.feeClienteHeaderId` → necesita pasar info adicional)
- Mostrar todos los campos del rango disabled excepto los 4 editables
- Modal de tamaño `modal-lg`

---

## Simulate Modal (fee-cliente-simulate-modal.component.ts)

### Props
```typescript
@Output() onClose = new EventEmitter<void>();
active = false;
calculating = false;
headerId: number;
rangeId: number;
range: FeeClienteRangeDto | null = null;
headerData: FeeClienteHeaderDto | null = null;
simulateInput = { monedaId: null, montoEnviar: null, comisionFija: null, comisionPorc: null };
result: SimulateFeeClienteResult | null = null;
monedas: FeeClienteLookupDto[] = [];
```

### Métodos
- `show(headerId, rangeId)` → cargar range y header del servicio → cargar monedas → `modal.show()`
- `calcular()` → validar inputs → `simulateFee(input)` → asignar resultado
- `limpiar()` → resetear simulateInput y result
- `volver()` → `modal.hide(); active = false; onClose.emit()`

### Template
- Sección 1: Datos principales del header (7 campos, todos disabled)
- Sección 2: Datos del rango (MontoDesde, MontoHasta, CostoTerrapayFijo, CostoTerrapayPorc, Bonifica, %Bonifica) — todos disabled
- Sección 3: Inputs (Moneda select, MontoEnviar input, ComisionFija input, ComisionPorc input) + [Calcular] [Limpiar] [Volver]
- Sección 4: Resultados (si `result != null`) — todos disabled

---

## Checklist de Verificación

1. `ng build` o `ng serve` compila sin errores
2. Navegar a `/#/app/main/prices/fee-cliente` carga grilla con 3 registros mock
3. Filtros funcionan (filtrar por país, estado)
4. Botón Agregar → navega a create, muestra 7 dropdowns
5. Completar cabecera → botón Crear → aparecen 19 rangos con costos precargados
6. Completar comisiones → Bonifica SI → %Bonifica se habilita
7. Mutual exclusion: llenar ComisionPorc deshabilita ComisionFija y vice versa
8. Simulador desde Create abre modal correctamente
9. Botón Guardar crea registro → vuelve a consult con el nuevo registro
10. Botón Detalle → navega a pantalla de detalle read-only con 19 rangos
11. Botón Modificar en rango → abre modal, guarda → recarga detalle
12. Botón Simulador en detalle → abre modal, Calcular muestra resultados
13. Botón Eliminar en consult → confirm dialog → registro aparece en rojo/salmon con botón Activar
14. Botón Activar → confirm → registro vuelve a activo
15. Botón Exportar en consult → descarga archivo .xlsx
16. Bolivia (registro inactivo) se muestra con highlight y solo botón Activar

---

## Notas Finales y TODO Comments

1. **19 Rangos:** El documento menciona 19 rangos pero la tabla visible en el prototipo muestra 18. El implementador debe verificar la sección "Tabla de Rangos Predefinidos" en el documento original. Si no está disponible, se usan 19 rangos con `[5001,6000]` y `[6001,10000]` separados.

2. **Fórmulas financieras (TODO):** `gananciaCisMin` y `feeClienteMin` no están completamente especificadas en la documentación disponible. Usar valores hardcoded del prototipo (0.12 USD y 1.32 USD) con `// TODO: implementar fórmula real - ver M12-04 y M12-05`.

3. **CostoTerrapay pre-cargado (TODO):** En mock, hardcodear valores por rango. En producción: consultar M12-02 usando los parámetros del header.

4. **HST_FEECLIENTE:** Solo se mockeará en memoria (array en el servicio). En producción: insertar en tabla `HST_FEECLIENTE` con campos INI/FIN.

5. **NombreInstitucion:** Campo opcional en la llave funcional (puede ser null). En el form de create, habilitarlo solo cuando TipoServicio tiene instituciones asociadas (usar `INSTITUCIONES_POR_TIPO_SERVICIO`).

6. **MarkUp fórmula:** `markUp = costoTerrapayFijo ? (feeClienteMin / costoTerrapayFijo) - 1 : null` — esto viene del documento, pero depende de feeClienteMin que es a su vez calculado. En mock, calcular ambos con valores hardcoded.
