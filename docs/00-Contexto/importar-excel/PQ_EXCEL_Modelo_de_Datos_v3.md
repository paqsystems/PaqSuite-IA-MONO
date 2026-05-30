# Modelo de Datos - Importación Excel

## 1. Objetivo
Definir la base conceptual del modelo de datos para soportar procesos de importación Excel en entorno multiusuario, con staging persistente, auditoría, historial y reglas estructurales fijas por proceso.

## 2. Regla definitiva de encabezado
- La fila de encabezado es siempre la fila 1.
- No se almacena parametrización de fila de encabezado.
- El parser asume fila 1 como fuente única de nombres de columnas.

## 3. Regla definitiva de nombres de columnas
La definición de columnas esperadas por proceso debe almacenar el nombre visible exacto que se exporta en la plantilla y contra el cual se valida el archivo importado.

### 3.1 Restricción del nombre visible
Se permiten únicamente:
- letras sin tildes
- números
- espacios

No se permiten:
- tildes
- puntuación
- símbolos especiales
- guiones bajos
- separadores especiales

## 4. Consecuencia para el modelo
Cada campo esperado por proceso debe contemplar dos conceptos:
- nombre visible Excel
- nombre interno técnico

Ejemplo conceptual:
- `NombreColumnaExcel`: `Fecha Emision`
- `NombreCampoInterno`: `fecha_emision`

## 5. Entidades conceptuales principales
- Proceso de importación
- Campo definido por proceso
- Lote de importación
- Fila importada
- Error de fila
- Historial de importaciones

## 6. Tablas previstas
Se propondrá una estructura con prefijo `PQ_EXCEL_`, incluyendo:
- configuración de procesos
- definición de campos
- cabecera de importaciones
- detalle de filas importadas
- errores por fila si se decide separar
- historial consultable

## 7. Impacto de las definiciones adoptadas
Las reglas ya cerradas producen estas simplificaciones de diseño:

### 7.1 Encabezado fijo
- evita configuración adicional
- simplifica el parser
- reduce ambigüedad

### 7.2 Coincidencia exacta de columnas
- evita alias
- evita lógica difusa
- facilita la validación estructural

### 7.3 Encabezados sin tildes ni símbolos
- reduce problemas de encoding
- simplifica comparaciones
- facilita interoperabilidad con APIs, JSON y SQL

### 7.4 Separación entre nombre visible y nombre interno
- mejora experiencia de usuario en Excel
- mantiene orden técnico interno
- permite evolucionar procesos sin exponer nombres técnicos

## 8. Decisiones operativas que impactan el modelo
- cada importación debe tener un identificador único
- el staging debe ser persistente
- el historial debe quedar disponible desde la primera etapa
- debe registrarse si una fila fue ajustada automáticamente
- debe registrarse el estado de cada lote y de cada fila
- debe registrarse la hoja elegida y el archivo original

## 9. Nota de evolución futura
En etapas futuras podría analizarse:
- múltiples hojas por proceso
- archivos múltiples
- mayor detalle de auditoría por ajuste automático
- separación específica de errores en tabla independiente
