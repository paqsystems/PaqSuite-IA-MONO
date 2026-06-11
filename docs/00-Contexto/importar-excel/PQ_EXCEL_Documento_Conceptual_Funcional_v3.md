# Documento Conceptual-Funcional - Importación Excel

## 1. Alcance
Este documento define el comportamiento funcional general del proceso de importación de información desde archivos Excel al sistema, mediante procesos predefinidos, con estructura fija por proceso, en entorno multiusuario.

## 2. Principios generales
- Cada proceso de importación tiene un diseño fijo.
- El usuario no define el esquema ni mapea columnas manualmente.
- El sistema debe permitir exportar una plantilla modelo por proceso.
- El archivo importado debe ajustarse al diseño del proceso.
- El archivo importado puede contener columnas adicionales a las requeridas en el diseño.
- La identificación de columnas se realiza por nombre y no por posición.
- El orden de las columnas no afecta la importación.
- Toda ejecución debe registrarse como un lote independiente.
- El staging debe ser persistente, auditado y multiusuario.
- La fila de encabezado es siempre la fila 1.

## 3. Regla definitiva de encabezado
- La fila 1 contiene siempre los nombres de las columnas.
- No se permite parametrización de fila de encabezado en esta etapa.
- No se permite detección automática de encabezado.
- No se admiten filas previas con títulos, subtítulos o descripciones.
- Si el archivo no cumple esta regla, debe ser corregido por el usuario antes de importar.

## 4. Regla definitiva para nombres de encabezados
Los nombres de columnas del Excel deben cumplir todas las siguientes reglas:

### 4.1 Coincidencia
- La coincidencia con la plantilla es estricta y exacta.
- No se permiten alias.
- No se permiten equivalencias automáticas.
- No se corrigen errores tipográficos.

### 4.2 Caracteres permitidos
Se permiten únicamente:
- letras sin tildes
- números
- espacios

### 4.3 Caracteres no permitidos
No se permiten:
- tildes
- puntos
- comas
- guiones bajos
- barras
- signos de puntuación
- símbolos especiales
- caracteres “raros”

### 4.4 Ejemplos
Encabezados válidos:
- Codigo
- Fecha Emision
- Importe Total
- Cantidad 1

Encabezados inválidos:
- Código
- Fecha Emisión
- Importe_Total
- Importe.
- Codigo-Articulo

## 5. Flujo funcional
1. El usuario ingresa al proceso específico.
2. El sistema permite exportar la plantilla modelo.
3. El usuario completa el archivo.
4. El usuario selecciona un archivo `.xlsx`.
5. El sistema muestra las hojas disponibles.
6. El usuario elige una única hoja.
7. El sistema valida la estructura del archivo.
8. Si la estructura es válida, crea un lote de importación.
9. El sistema lee la información y la vuelca a staging.
10. El sistema ejecuta validaciones de formato en frontend y validaciones de negocio en backend.
11. El sistema presenta el resultado en una grilla DevExtreme.
12. El usuario confirma el procesamiento final cuando corresponda (según política del proceso — ver §6.1).
13. El sistema aplica la lógica final sobre el destino solo sobre las filas habilitadas para procesar.
14. El sistema deja auditoría completa del lote.

## 6. Reglas funcionales consolidadas
- Se acepta únicamente formato `.xlsx`.
- Si el archivo no es Excel válido, se rechaza con mensaje claro.
- El usuario siempre selecciona la hoja a procesar.
- Las columnas extra se ignoran siempre.
- Si falta una columna obligatoria, es error de archivo y se bloquea el proceso.
- Si existen columnas duplicadas, es error de archivo.
- Si existen encabezados vacíos, es error de archivo.
- Si existen celdas combinadas, es error de archivo.
- Las filas completamente vacías se ignoran y se contabilizan como descartadas.
- Una fila se considera vacía si todas las columnas obligatorias están vacías.
- Si una fila tiene menos columnas, se completa con `NULL`.
- Si una fila tiene más valores que columnas definidas, se ignoran los valores extra.
- Los duplicados dentro del Excel no se controlan en el importador.
- Los `NULL` se aceptan y su obligatoriedad real se valida en backend.
- Si el tipo de dato es incorrecto, es error de fila.
- Para fechas se toma el valor fecha real de Excel y luego se normaliza internamente.
- Para números decimales se toma el valor numérico real de Excel.
- Los campos tipo código deben venir como texto en Excel.
- Si un texto supera la longitud máxima, es error de fila.
- Las fórmulas se leen por su valor resultante.
- Las filas y columnas ocultas se procesan normalmente.
- Los saltos de línea dentro de una celda se aceptan.
- Los valores con delimitadores se tratan como texto plano.
- Los archivos grandes se procesan en forma asíncrona.
- La cancelación solo se permite antes del procesamiento final.
- Debe existir historial de importaciones.

### 6.1 Procesamiento ante filas con error (por proceso)

Cada **proceso de importación** define si la existencia de **al menos una fila con error** (validación de formato o de negocio en staging) **permite o no** ejecutar el procesamiento final sobre el resto de los registros válidos.

Parámetro de configuración en `PQ_EXCEL_PROCESOS`: **`PermiteProcesamientoParcial`**.

| Valor | Comportamiento |
|-------|----------------|
| **`false`** (default) | Si el lote tiene **≥ 1 fila con error**, **no** se habilita el procesamiento final del lote. El usuario debe corregir el Excel y reimportar, o resolver los errores hasta dejar el lote sin filas erróneas. |
| **`true`** | Si el lote tiene filas con error **y** filas válidas, el usuario **puede** confirmar el procesamiento: solo se aplican al destino las **filas válidas**; las filas con error quedan en staging sin procesar. El lote puede cerrar en estado **`procesada_parcial`**. |

**Casos borde (cerrados)**

| Situación | `PermiteProcesamientoParcial` | Resultado |
|-----------|-------------------------------|-----------|
| **Todas** las filas tienen error | `true` o `false` | **No** se habilita el procesamiento (no hay filas válidas que aplicar). |
| **Cero** filas con error | `true` o `false` | Se procesa **todo** el conjunto; estado final **`procesada`** (no `procesada_parcial`). |
| Mezcla: ≥ 1 error y ≥ 1 válida | `false` | No se habilita el procesamiento. |
| Mezcla: ≥ 1 error y ≥ 1 válida | `true` | Se procesan solo las válidas; estado **`procesada_parcial`**. |

**Alcance de la regla**

- Aplica a **errores por fila** tras la carga en staging (tipo de dato, obligatoriedad de negocio, longitud, reglas del `HandlerBackend`, etc.).
- **No** aplica a **errores estructurales de archivo** (columnas faltantes, encabezados inválidos, hoja incorrecta, etc.): esos bloquean el lote antes del staging y no admiten procesamiento parcial.

**UI**

- Con `PermiteProcesamientoParcial = false` y filas con error: deshabilitar acción **Procesar** / **Confirmar** y mostrar mensaje explícito (p. ej. «Existen filas con error; corrija el archivo antes de procesar»).
- Con `PermiteProcesamientoParcial = true`: permitir confirmar procesando solo filas válidas; informar cantidad de filas omitidas por error.

**Auditoría**

- Registrar en el lote: `CantidadFilasConError`, `CantidadFilasProcesadas` y estado final (`procesada` vs `procesada_parcial`).

## 7. Normalizaciones configurables

**Fila ajustada automáticamente** (`FilaAjustadaAutomaticamente` en staging): fila a la que el importador le aplicó **trim** de espacios (§7.1) o **eliminación de caracteres no imprimibles** (§7.2) porque los parámetros del lote/proceso lo indican. Es un **marcador de auditoría** («el sistema modificó el valor leído del Excel antes de validar»); **no** es un error ni bloquea el procesamiento por sí solo. La fila sigue siendo válida o con error según las validaciones de formato y negocio sobre el valor **ya normalizado**.

**UI (esta etapa):** **no** se informa al usuario que una fila fue ajustada (sin columna, ícono ni mensaje en la grilla). El flag queda solo en staging para trazabilidad técnica.

### 7.1 Espacios en blanco
Parámetro:
- `MantenerEspaciosEnBlanco` = false por defecto

Si está en false:
- se aplica trim automático al inicio y fin
- si hubo cambio, `FilaAjustadaAutomaticamente = true`

Si está en true:
- no se aplica trim

### 7.2 Caracteres especiales
Parámetro:
- `MantenerCaracteresEspeciales` = false por defecto

Si está en false:
- se eliminan caracteres no imprimibles
- si hubo cambio, `FilaAjustadaAutomaticamente = true`

Si está en true:
- se conserva el contenido original

## 8. Presentación de resultados
- La visualización estándar es DevExtreme DataGrid.
- **Fuera de UI (esta etapa):** indicar filas con normalización automática (`FilaAjustadaAutomaticamente`); ver §7.
- Debe existir una columna fija de errores.
- Los errores de una fila se muestran concatenados en una sola columna.
- La fila con error debe marcarse con fondo suave.
- Los errores deben quedar también como tooltip de la fila.

## 9. Multiusuario y concurrencia
- Toda importación debe manejarse como un lote independiente.
- Distintos usuarios pueden ejecutar importaciones simultáneamente.
- Incluso el mismo proceso puede ejecutarse varias veces en paralelo.
- Los datos intermedios no deben mezclarse entre ejecuciones.

## 10. Historial
Debe existir una pantalla de historial de importaciones con al menos:
- fecha y hora
- usuario
- proceso
- archivo
- hoja
- estado
- cantidad de filas leídas
- filas válidas
- filas con error
- filas procesadas

## 11. Criterio de diseño final para encabezados
El encabezado del Excel debe ser humano y claro, pero controlado:
- no técnico
- sin tildes
- sin símbolos
- con coincidencia exacta respecto de la plantilla exportada

## 12. Generación de plantilla modelo

- Se realiza en base a la tabla "PQ_EXCEL_PROCESOS_CAMPOS"
- la fila 1 (encabezado), destacarla con Fondo Azul y letras blancas
- el atributo "ordenCampo" indica el orden de generación de las columnas 
   (1->A, 2->B, ....)  , solo alterado si existe una columna con "Activo" = false 
- Como título se coloca el "NombreColumnaExcel"
- si el atributo "Observaciones" no está vacío, se carga como comentario de la celda
- se formatea cada columna según el atributo "TipoDato"
- Los atributos "LargoMaximo" y "CantidadDecimales" se utilizan como validación de celda
- Definir si los datos booleanos se exigen 0 y 1, "N"/"S" , o "VERDADERO"/"FALSO", según conveniencia para el procesamiento posterior.
