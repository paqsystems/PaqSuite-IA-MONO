# Modelo de Datos para Almacenamiento de Pivots Personalizados
**Proyecto:** Paqsuite  
**Documento:** Diseño de persistencia de pivots guardados  
**Versión:** 1.0  
**Fecha:** 2026-03-16  
**Última actualización:** 2026-06-11 — nota de nomenclatura canónica (alineación PaqSuite-IA-Tango)

---

## Nota de nomenclatura (implementación canónica)

En **PaqSuite-IA-Tango** (referencia implementada) la persistencia física usa **`pq_pivots_config`** en Dictionary DB con prefijo `pq_pivots_`. Este documento conserva el boceto conceptual inicial (`dbo.PQ_PIVOTS`, PascalCase) y las **reglas funcionales** (§2, §7, §20); no reemplaza el DDL canónico.

| Ámbito | Fuente |
|--------|--------|
| Migraciones, modelos, API CRUD de diseños | **`pq_pivots_config`** — [modelo_datos_pivots_y_catalogo.md](modelo_datos_pivots_y_catalogo.md) |
| Reglas de negocio, JSON de configuración, borrado lógico | Este documento + [especificacion_tecnica §17](especificacion_tecnica_consultas_pivotables.md) |
| Mapeo conceptual → físico | `PivotID` → `pivot_id`; `ConsultaID` → `consulta_id`; `ConfiguracionJson` → `configuracion_json`; `UsuarioCreadorID` → `user_id` / `usuario_creador_id` (FK según TR) |

**TR e implementación:** citar `modelo_datos_pivots_y_catalogo.md` como fuente de verdad del esquema; este archivo como complemento normativo funcional (patrón TR-020 Informes en Tango, que referencia ambos documentos).

---

# 1. Objetivo

Definir el modelo de datos propuesto para almacenar pivots personalizados del sistema, siguiendo una lógica análoga a la utilizada para grillas guardadas.

El objetivo del modelo es permitir:

- guardar configuraciones de pivot
- reutilizar pivots entre usuarios
- modificar pivots propios
- clonar pivots existentes mediante “Guardar como”
- eliminar pivots propios
- mantener trazabilidad y compatibilidad con la definición técnica de cada consulta

---

# 2. Reglas funcionales heredadas

El almacenamiento de pivots debe respetar estas reglas:

1. todos los pivots son visibles para todos los usuarios con acceso a la consulta
2. solo el creador puede modificar un pivot existente mediante **Guardar**
3. cualquier usuario puede crear una nueva variante mediante **Guardar como**
4. solo el creador puede eliminar su pivot
5. la lógica funcional debe ser equivalente a la usada para grillas guardadas (HU-GEN-03)
6. los diseños propios (`isOwner: true`) se marcan en el selector con sufijo **` (*)`** (i18n `pivotLayout.ownerMarker`); no se almacena en el campo `Nombre`
7. **Plantilla inicial** (`configId: null`) representa pivot vacía en UI; al seleccionarla se resetea el diseño sin filas/columnas/valores/filtros internos asignados
8. con plantilla inicial activa, **Guardar** equivale a **Guardar como** (POST con nombre nuevo)
9. debe registrarse último diseño usado por usuario (tabla auxiliar análoga a `pq_grid_layout_last_used`, p. ej. `pq_pivots_config_last_used`)

---

# 3. Enfoque de modelado recomendado

Se propone un modelo híbrido:

- **cabecera relacional**
- **configuración serializada en JSON**

## Motivo
La cabecera relacional facilita:

- búsquedas
- filtros
- auditoría
- permisos
- listados por consulta
- control de creador

La configuración en JSON facilita:

- flexibilidad
- evolución del modelo
- menor complejidad estructural
- compatibilidad con cambios de layout del pivot

---

# 4. Entidades propuestas

## 4.1 Cabecera de Pivot Guardado
Representa la identidad y metadatos del pivot.

## 4.2 Configuración de Pivot Guardado
Representa la estructura concreta del pivot:
- filas
- columnas
- valores
- agregaciones
- subtotales
- totales
- ordenamientos
- filtros internos
- opciones visuales

## 4.3 Historial o auditoría (opcional)
Permite guardar trazabilidad de cambios relevantes.

---

# 5. Tabla principal propuesta

## 5.1 Nombre sugerido

> **Implementación canónica (Dictionary DB):** tabla **`pq_pivots_config`** — ver [modelo_datos_pivots_y_catalogo.md](modelo_datos_pivots_y_catalogo.md).  
> El identificador `dbo.PQ_PIVOTS` siguiente es el **boceto conceptual v1.0**; no usar en migraciones ni código.

```txt
dbo.PQ_PIVOTS
```

---

# 6. Estructura propuesta de la tabla cabecera

## Tabla: `dbo.PQ_PIVOTS`

| Campo | Tipo sugerido | Nulo | Descripción |
|---|---|---:|---|
| PivotID | bigint identity / bigint | No | Identificador interno del pivot |
| ConsultaID | varchar(100) | No | Identificador lógico de la consulta pivotable |
| Nombre | varchar(200) | No | Nombre visible del pivot |
| Descripcion | varchar(500) | Sí | Descripción opcional |
| ConfiguracionJson | nvarchar(max) | No | Configuración serializada del pivot |
| VersionDefinicionConsulta | int | No | Versión de la definición técnica usada |
| UsuarioCreadorID | bigint / varchar(100) | No | Usuario creador |
| FechaCreacion | datetime2 | No | Fecha de creación |
| UsuarioUltModID | bigint / varchar(100) | No | Usuario de última modificación |
| FechaUltMod | datetime2 | No | Fecha de última modificación |
| PivotOrigenID | bigint | Sí | Pivot del cual se clonó con Guardar como |
| Eliminado | bit | No | Borrado lógico |
| FechaEliminacion | datetime2 | Sí | Fecha de eliminación |
| UsuarioEliminacionID | bigint / varchar(100) | Sí | Usuario que eliminó |
| Activo | bit | No | Indica disponibilidad operativa |

---

# 7. Consideraciones sobre claves

## Clave primaria
- `PivotID`

## Clave foránea sugerida
- `PivotOrigenID` → `dbo.PQ_PIVOTS.PivotID`

## Relaciones externas
Dependen del diseño existente de usuarios.
Puede vincularse a:

- tabla de usuarios del Dictionary DB
- identificador textual de usuario/autenticación
- GUID de identidad externa

---

# 8. Índices recomendados

## Índice 1
Por consulta y estado activo

```txt
IX_PQ_PIVOTS_ConsultaID_Activo_Eliminado
(ConsultaID, Activo, Eliminado)
```

## Índice 2
Por creador

```txt
IX_PQ_PIVOTS_UsuarioCreadorID
(UsuarioCreadorID)
```

## Índice 3
Por nombre dentro de consulta

```txt
IX_PQ_PIVOTS_ConsultaID_Nombre
(ConsultaID, Nombre)
```

## Índice 4
Por origen de clonación

```txt
IX_PQ_PIVOTS_PivotOrigenID
(PivotOrigenID)
```

---

# 9. Regla sobre nombres

**Decisión A1 (SPEC-001-08, paridad grilla):** el `nombre` del diseño guardado es **único por `consulta_id`** (un solo registro activo por nombre en la consulta, independiente del creador).

- Intento de **Guardar como** con nombre duplicado → error i18n (`pivotLayout.duplicateName`); no sobrescribir diseño ajeno.
- El sufijo ` (*)` es solo visual para diseños propios; no forma parte del `nombre` persistido.

---

# 10. Configuración JSON propuesta

La columna `ConfiguracionJson` debe almacenar la estructura concreta del pivot guardado.

## Estructura sugerida

```json
{
  "filas": ["Cliente"],
  "columnas": ["Mes"],
  "valores": [
    {
      "campo": "ImporteNeto",
      "agregacion": "SUMA"
    }
  ],
  "filtrosInternos": [],
  "ordenamiento": [
    {
      "campo": "Cliente",
      "direccion": "asc"
    }
  ],
  "mostrarSubtotales": true,
  "mostrarTotalesGenerales": true,
  "expandido": true,
  "opcionesVisuales": {
    "mostrarCamposVacios": false,
    "compacto": false
  }
}
```

## Regla
La estructura serializada debe ser compatible con la definición técnica de la consulta.

---

# 11. Datos que deben quedar persistidos en JSON

Como mínimo:

- filas
- columnas
- valores
- agregaciones elegidas
- filtros internos del pivot
- ordenamientos
- subtotales
- totales generales
- expandido / contraído
- opciones visuales relevantes
- campos auxiliares de renderizado, si hicieran falta

---

# 12. Versionado

El registro debe guardar:

- `VersionDefinicionConsulta`

## Motivo
Cuando cambie la definición técnica de la consulta, el sistema debe poder:

- validar compatibilidad
- migrar estructura guardada
- rechazar pivots obsoletos si no son compatibles
- informar al usuario si el pivot debe recrearse

---

# 13. Borrado lógico

Se recomienda **borrado lógico**, no borrado físico.

Campos asociados:

- `Eliminado`
- `FechaEliminacion`
- `UsuarioEliminacionID`

## Motivos
- auditoría
- trazabilidad
- recuperación eventual
- análisis de uso
- consistencia histórica

---

# 14. Operaciones funcionales y su impacto en datos

## 14.1 Guardar
Actualiza un pivot existente.

### Validaciones
- el pivot existe
- está activo
- no está eliminado
- el usuario actual es el creador

### Acciones
- actualizar `ConfiguracionJson`
- actualizar `UsuarioUltModID`
- actualizar `FechaUltMod`

---

## 14.2 Guardar como
Crea un nuevo pivot partiendo de otro existente o de una configuración no guardada (incluye diseño sobre **plantilla inicial** / pivot vacía).

### Acciones
- insertar nuevo registro
- copiar configuración actual
- asignar nuevo creador
- registrar `PivotOrigenID` si aplica
- registrar fechas de creación y modificación inicial

### Regla especial
Si el usuario tiene activa la **plantilla inicial** (`configId: null`) y pulsa **Guardar**, el sistema debe ejecutar este flujo (Guardar como), no un PUT inexistente.

---

## 14.3 Eliminar
Realiza borrado lógico.

### Validaciones
- el pivot existe
- el usuario actual es el creador

### Acciones
- `Eliminado = 1`
- `Activo = 0`
- completar auditoría de eliminación

---

# 15. Visibilidad global

Todos los pivots activos y no eliminados deben ser visibles para todos los usuarios que tengan acceso a la consulta correspondiente.

## Consecuencia
Al listar pivots guardados conviene mostrar, además del nombre:

- creador
- fecha de creación
- fecha de última modificación

Esto ayuda a interpretar si el pivot es:

- propio
- ajeno
- reciente
- antiguo
- derivado de otro

---

# 16. Tabla de auditoría opcional

## Nombre sugerido

```txt
dbo.PQ_PIVOTS_AUD
```

## Uso
Guardar eventos relevantes:

- creación
- modificación
- eliminación
- clonación

## Estructura sugerida

| Campo | Tipo sugerido | Descripción |
|---|---|---|
| PivotAudID | bigint identity | PK |
| PivotID | bigint | Pivot afectado |
| Evento | varchar(50) | CREAR / MODIFICAR / ELIMINAR / CLONAR |
| UsuarioID | bigint / varchar(100) | Usuario que realizó la acción |
| FechaEvento | datetime2 | Fecha del evento |
| DatosPreviosJson | nvarchar(max) | Estado anterior opcional |
| DatosNuevosJson | nvarchar(max) | Estado nuevo opcional |

## Observación
Esta tabla es opcional para una primera etapa, pero muy recomendable.

---

# 17. SQL conceptual sugerido

```sql
CREATE TABLE dbo.PQ_PIVOTS
(
    PivotID bigint IDENTITY(1,1) NOT NULL,
    ConsultaID varchar(100) NOT NULL,
    Nombre varchar(200) NOT NULL,
    Descripcion varchar(500) NULL,
    ConfiguracionJson nvarchar(max) NOT NULL,
    VersionDefinicionConsulta int NOT NULL,
    UsuarioCreadorID varchar(100) NOT NULL,
    FechaCreacion datetime2 NOT NULL,
    UsuarioUltModID varchar(100) NOT NULL,
    FechaUltMod datetime2 NOT NULL,
    PivotOrigenID bigint NULL,
    Eliminado bit NOT NULL CONSTRAINT DF_PQ_PIVOTS_Eliminado DEFAULT (0),
    FechaEliminacion datetime2 NULL,
    UsuarioEliminacionID varchar(100) NULL,
    Activo bit NOT NULL CONSTRAINT DF_PQ_PIVOTS_Activo DEFAULT (1),
    CONSTRAINT PK_PQ_PIVOTS PRIMARY KEY (PivotID),
    CONSTRAINT FK_PQ_PIVOTS_PivotOrigen
        FOREIGN KEY (PivotOrigenID)
        REFERENCES dbo.PQ_PIVOTS (PivotID)
);
GO

CREATE INDEX IX_PQ_PIVOTS_ConsultaID_Activo_Eliminado
    ON dbo.PQ_PIVOTS (ConsultaID, Activo, Eliminado);
GO

CREATE INDEX IX_PQ_PIVOTS_UsuarioCreadorID
    ON dbo.PQ_PIVOTS (UsuarioCreadorID);
GO

CREATE INDEX IX_PQ_PIVOTS_ConsultaID_Nombre
    ON dbo.PQ_PIVOTS (ConsultaID, Nombre);
GO

CREATE INDEX IX_PQ_PIVOTS_PivotOrigenID
    ON dbo.PQ_PIVOTS (PivotOrigenID);
GO
```

---

# 18. Posible procedimiento de listado

El listado de pivots disponibles para una consulta debería devolver solo:

- `Activo = 1`
- `Eliminado = 0`

Y ordenar por ejemplo por:

1. nombre
2. fecha de última modificación descendente

---

# 19. Posibles extensiones futuras

Más adelante podría agregarse:

- marcar pivots favoritos
- marcar pivot predeterminado
- distinguir pivots del sistema vs pivots creados por usuarios
- compartir por rol o grupo
- versionado histórico de cada pivot
- métricas de uso

---

# 20. Resumen normativo

1. Se recomienda una tabla cabecera relacional con configuración JSON.
2. El borrado debe ser lógico.
3. Debe conservarse autoría, fechas y origen de clonación.
4. Todos los pivots son visibles globalmente para los usuarios habilitados.
5. Solo el creador puede modificar o eliminar.
6. Toda configuración debe guardar la versión de definición técnica de la consulta.
7. El diseño debe ser análogo al criterio usado para grillas guardadas.
8. Plantilla inicial (`configId: null`) = pivot vacía; Guardar en ese estado = Guardar como.
9. Diseños propios se identifican con sufijo ` (*)` en selector (`pivotLayout.ownerMarker`).
10. UI del pivot debe incluir Actualizar (re-fetch) e i18n completo DevExtreme (ver `patron-i18n-pivot-devextreme.md`).

---

# Fin del documento
