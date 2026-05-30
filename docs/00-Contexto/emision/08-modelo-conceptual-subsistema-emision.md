# Modelo Conceptual del Subsistema de Emisión

Este diagrama muestra las entidades funcionales principales del subsistema y sus relaciones.

- El eje central es el **Proceso**
- Cada proceso define su **Dataset**
- Cada proceso puede tener múltiples **Reportes**
- Cada proceso puede tener múltiples **Plantillas de Mail**
- Cada proceso puede tener múltiples **Formatos de Integración**
- Cada proceso define **Modos de Emisión**
- Cada proceso define **Canales de Salida**
- Cada emisión genera **Historial**
- La emisión segmentada genera **Detalle por Segmento**

---

```mermaid
classDiagram

class Proceso {
  +IdProceso
  +Nombre
  +Descripcion
  +TipoProceso
  +Activo
}

class DatasetSchema {
  +IdDatasetSchema
  +IdProceso
  +Nombre
  +TipoEstructura
  +Definicion
  +Activo
}

class Reporte {
  +IdReporte
  +IdProceso
  +IdDatasetSchema
  +Codigo
  +Nombre
  +EsEstandar
  +EsPrincipal
  +EsPrivado
  +Activo
}

class PlantillaMail {
  +IdPlantillaMail
  +IdProceso
  +Codigo
  +Nombre
  +AsuntoTemplate
  +BodyHtmlTemplate
  +EsPrincipal
  +EsEstandar
  +Activo
}

class VariableMail {
  +IdVariableMail
  +IdProceso
  +CodigoVariable
  +Nombre
  +Descripcion
  +Activo
}

class FormatoIntegracion {
  +IdFormatoIntegracion
  +IdProceso
  +Codigo
  +Nombre
  +TipoArchivo
  +Extension
  +Generador
  +EsPrincipal
  +Activo
}

class ModoEmision {
  +CodigoModo
  +Nombre
}

class CanalSalida {
  +CodigoCanal
  +Nombre
}

class ClaveSegmentacion {
  +IdClaveSegmentacion
  +IdProceso
  +Codigo
  +Nombre
  +CampoClave
  +CampoDescripcion
  +CampoMail
  +Activo
}

class ConfiguracionProceso {
  +IdProceso
  +IdReportePrincipal
  +IdPlantillaMailPrincipal
  +IdFormatoIntegracionPrincipal
  +PermiteConsolidado
  +PermiteSegmentado
  +VisibleEnMobile
  +ModoMobile
}

class Emision {
  +IdEmision
  +IdProceso
  +IdReporte
  +IdPlantillaMail
  +IdFormatoIntegracion
  +ModoEmision
  +CanalSalida
  +FechaEmision
  +Usuario
  +Resultado
}

class EmisionDetalle {
  +IdEmisionDetalle
  +IdEmision
  +ValorClaveSegmento
  +DescripcionSegmento
  +MailDestino
  +Resultado
}

class Usuario {
  +IdUsuario
  +Nombre
}

Proceso "1" --> "1..*" DatasetSchema : define
Proceso "1" --> "0..*" Reporte : tiene
Proceso "1" --> "0..*" PlantillaMail : tiene
Proceso "1" --> "0..*" FormatoIntegracion : tiene
Proceso "1" --> "0..*" ClaveSegmentacion : permite
Proceso "1" --> "1" ConfiguracionProceso : configura

Reporte --> DatasetSchema : usa
PlantillaMail --> VariableMail : usa
Proceso --> VariableMail : expone

ConfiguracionProceso --> Reporte : principal
ConfiguracionProceso --> PlantillaMail : principal
ConfiguracionProceso --> FormatoIntegracion : principal

Emision --> Proceso : ejecuta
Emision --> Reporte : puede usar
Emision --> PlantillaMail : puede usar
Emision --> FormatoIntegracion : puede usar
Emision --> Usuario : emitida_por
Emision "1" --> "0..*" EmisionDetalle : detalle

Proceso --> ModoEmision : soporta
Proceso --> CanalSalida : soporta