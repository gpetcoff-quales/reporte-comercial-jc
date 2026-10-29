# DATA_SCHEMA — data.json del tablero "Base Instalada y Servicios"

`index.html` lee **un solo archivo**, `data.json`, de la misma carpeta (`fetch('data.json')`). Todo lo que muestra el tablero (tarjetas, grilla interanual, oportunidades, Servicios, historia del cliente) se calcula en el navegador a partir de este archivo.

- Codificación: UTF-8, JSON estándar.
- Fechas: texto ISO. Día `AAAA-MM-DD`, mes `AAAA-MM`, fecha y hora ISO 8601 con zona.
- Números: en los conjuntos que vienen de **consultas** (Airtable, Zoho) son números JSON. En los que vienen de **archivos** (CSV) son **texto** con punto decimal (`"3730.0"`), igual que en el tablero original; la página los convierte con `Number()`.
- Vacíos: `null` en las consultas y `""` en los archivos.
- Si falta un conjunto, las partes que lo usan muestran "—" o "No se pudo cargar", y el resto del tablero sigue funcionando.

---

## 1. Estructura general

```json
{
  "generated_at": "2026-10-29T17:01:48.080Z",
  "title": "Base Instalada y Servicios",
  "datasets": {
    "<id>": {
      "title": "Nombre corto",
      "description": "Qué es y de dónde sale",
      "source": "query | file",
      "origin": "Airtable · list_records_for_table | nombre_del_archivo.csv",
      "fetched_at": "ISO 8601 (solo source = query)",
      "updated_at": "ISO 8601 (solo source = file)",
      "columns": ["col1", "col2"],
      "rows": [ { "col1": "...", "col2": 0 } ]
    }
  },
  "seguimiento": { }
}
```

| Campo | Tipo | Obligatorio | Uso |
|---|---|---|---|
| `generated_at` | texto ISO 8601 | sí | Fecha de la copia. Es la que se usa en "Datos actualizados" cuando un conjunto no trae su propia fecha. |
| `title` | texto | no | Solo informativo. |
| `datasets` | objeto | sí | Un objeto por conjunto de datos, con la clave = id (ver sección 2). |
| `datasets.<id>.rows` | lista de objetos | **sí** | Las filas. Cada fila tiene las columnas de su conjunto. |
| `datasets.<id>.columns` | lista de texto | sí | Orden de las columnas (el mismo que las claves de cada fila). |
| `datasets.<id>.fetched_at` / `updated_at` | texto ISO 8601 | recomendado | Lo muestra el tooltip "Datos actualizados" por origen y se pone naranja si tiene más de 30 h. |
| `datasets.<id>.title`, `description`, `source`, `origin` | texto | no | Documentación; la página no los usa para calcular. |
| `seguimiento` | objeto | no | Seguimiento manual de Servicios, solo lectura (ver sección 4). Si no está, se toma vacío. |

---

## 2. Conjuntos de datos (`datasets`)

Todos tienen claves exactas y sensibles a mayúsculas. Las columnas marcadas **clave** son las que se usan para cruzar conjuntos.

### 2.1 `proyectos`: Listado de proyectos (Airtable › Gestión Servicios › Listado de Proyectos)

Una fila por proyecto, activos y dados de baja, toda la historia. Hoy tiene 2.154 filas.

| Columna | Tipo | Ejemplo | Notas |
|---|---|---|---|
| `codigo` | texto | `"C0189P0002"` | **Clave.** Formato `CxxxxPyyyy`. Los internos son `C99xx…` y QG SAU es `C0234…`; la página excluye a ambos. |
| `cliente` | texto | `"C0189 - Rapanui"` | Código y nombre del cliente. |
| `proyecto` | texto | `"Rapanui - Extensión Tablero Comercial"` | |
| `pais` | texto | `"Argentina"` | |
| `lider` | texto | `"nestrada"` | Usuario. |
| `coordinador` | texto | `"ngalletti"` | Usuario (head). |
| `comercial` | texto o null | `"sifran"` | |
| `tipo_facturacion` | texto | `"Hitos"` | |
| `tipo_servicio` | texto | `"Mantenimiento Evolutivo"` | |
| `moneda` | texto | `"ARS"` | `USD`, `EUR`, `ARS`, … Tiene que existir en `tipos_cambio` para pasar a USD. |
| `fecha_alta` | texto `AAAA-MM-DD` | `"2023-02-15"` | |
| `fecha_baja` | texto `AAAA-MM-DD` o null | `"2023-06-30"` | |
| `activo` | texto | `"Si"` / `"No"` | Estado de Proyecto. |
| `propuesta` | texto | `"PC-QG-01242-01"` | Código de la propuesta comercial. Cruza con el CRM por el patrón `PC-QG-#####-##`. Puede ser `"Sin PC"` u otro texto. |
| `area` | texto | `"SERV"` | |
| `empresa` | texto | `"QG S.R.L"` | |

### 2.2 `historia`: Historia del cliente (mismo listado de Airtable)

Las mismas filas que `proyectos` (todos los clientes), con estas columnas. La página muestra solo las del cliente elegido, filtrando por `cliente`.

| Columna | Tipo | Ejemplo |
|---|---|---|
| `codigo` | texto | `"C0189P0002"` |
| `cliente` | texto | `"C0189 - Rapanui"` (tiene que coincidir exactamente con `proyectos.cliente`) |
| `proyecto` | texto | |
| `fecha_alta` | texto `AAAA-MM-DD` | |
| `fecha_baja` | texto o null | |
| `activo` | texto | `"Si"` / `"No"` |
| `tipo_servicio` | texto | |
| `propuesta` | texto | `"PC-QG-01242-01"` |
| `proyecto_previo` | texto o null | `"C0189P0001 - Rapanui - …"` |

### 2.3 `horas`: Horas proyectadas (Airtable › Horas - Proyección › Proyectado_Anual)

Una fila por proyecto, colaborador y mes.

| Columna | Tipo | Ejemplo |
|---|---|---|
| `codigo` | texto | `"C0013P0021"` (**clave** con `proyectos.codigo`) |
| `colaborador` | texto | `"dcozzitorto"` (usuario) |
| `mes` | texto `AAAA-MM` | `"2026-11"` |
| `horas` | número | `8` |
| `comentario` | texto o null | `"LIDER"` |

### 2.4 `montos`: Montos y tarifas (Airtable › Horas - Proyección › Info_Inicial_Proyectos_Anual)

| Columna | Tipo | Ejemplo | Notas |
|---|---|---|---|
| `proyecto` | texto o null | `"C0143P0010 - Labyes - Conexion ERP…"` | Empieza con el código del proyecto. |
| `moneda` | texto o null | `"ARS"` | |
| `monto_vendido` | número o null | `1081808` | En la moneda del proyecto. |
| `monto_devengado` | número o null | `1081808` | |
| `tarifa` | número o null | `0` | |

### 2.5 `colaboradores` (Airtable › Horas - Proyección › Vista_Colaboradores)

| Columna | Tipo | Ejemplo |
|---|---|---|
| `usuario` | texto | `"macosta"` |
| `nombre` | texto | `"Martin Acosta"` |

### 2.6 `roles` (Airtable › Horas - Proyección › Vista - Perfil y Rol Activo)

| Columna | Tipo | Ejemplo |
|---|---|---|
| `usuario` | texto | `"scalderon"` |
| `rol` | texto | `"Executive Assistant"` |
| `seniority` | texto | `"Sr"` |
| `vigente` | texto | `"🟢"` |
| `hasta` | texto `AAAA-MM-DD` | `"2030-12-31"` |

### 2.7 `crm`: Oportunidades (Zoho CRM › Deals)

Oportunidades de tipo `Base Instalada…` y `New Name…`, toda la historia. Los nombres de columna son los de la API de Zoho, sin cambios.

| Columna | Tipo | Ejemplo | Notas |
|---|---|---|---|
| `id` | texto | `"3852010000046141095"` | **Clave** con `crm_fases.oportunidad_id`. Siempre texto: es un número largo. |
| `Deal_Name` | texto | `"Saint Gobain - A buscar"` | |
| `Account_Name.Account_Name` | texto | `"Saint Gobain"` | Cuenta. Si no hay PC, se cruza por nombre con `proyectos.cliente`. |
| `Stage` | texto | `"S0-Prospect"` | Etapa: `S0…`–`S4…` (de 0 a 4), `WI…` (ganada = 5), `LO…` (perdida = 6). |
| `Type` | texto | `"Base Instalada - Servicios"` | Las `New Name…` cuentan solo si están en WI y tienen proyecto C. |
| `Amount` | número o null | `0` | En `Currency`. |
| `Currency` | texto | `"EUR"` | |
| `Closing_Date` | texto `AAAA-MM-DD` | `"2027-12-31"` | Fecha de cierre (estimada o real). |
| `Created_Time` | texto ISO | `"2025-09-15T16:14:07+02:00"` | |
| `N_mero_de_Oportunidad` | texto o null | `"PC-QG-02380-01"` | Código de PC, **clave** con `proyectos.propuesta`. |
| `Total_horas_proyecto` | número o null | `0` | Horas vendidas. |
| `Comercial` | texto o null | `"mcabezas"` | |
| `Contact_Name.Full_Name` | texto o null | | |
| `Pa_s_de_la_Oportunidad` | texto o null | `"España"` | |
| `ltimo_status` | texto o null | | Último status. |
| `Last_Activity_Time` | texto ISO o null | | |
| `Reason_For_Loss__s` | texto o null | `"Situación interna"` | |
| `Detalle_de_la_p_rdida` | texto o null | | |

### 2.8 `crm_fases`: Historial de fases (Zoho CRM › DealHistory)

Un cambio de fase por fila. Se usa para la fecha real de WI y LO.

| Columna | Tipo | Ejemplo |
|---|---|---|
| `oportunidad_id` | texto | `"3852010000061489001"` (= `crm.id`) |
| `fase` | texto | `"WI-Cerrado ganado"` |
| `fecha` | texto `AAAA-MM-DD` | `"2026-10-29"` |
| `hora` | texto ISO | `"2026-10-29T11:56:58+01:00"` |

### 2.9 `devengado_monto`: Horas y monto devengado (Tableau › Services - Situación Proyecto, archivo)

Una fila por proyecto y mes, desde 2019. Los meses que todavía no pasaron traen lo proyectado (hasta 2027). **Todos los valores son texto.**

| Columna | Tipo | Ejemplo | Notas |
|---|---|---|---|
| `codigo` | texto | `"C0085P0002"` | **Clave** con `proyectos.codigo`. |
| `mes` | texto `AAAA-MM` | `"2019-03"` | |
| `hs` | texto numérico | `"5.0"` | Horas devengadas. |
| `devengado` | texto numérico | `"3730.0"` | Monto en la moneda del proyecto (`proyectos.moneda`). |

### 2.10 `maestro`: Maestro de Proyectos (archivo)

Define head, segmento, país y nombre del cliente que se muestran. **Todo es texto.**

| Columna | Ejemplo | Notas |
|---|---|---|
| `pais` | `"Argentina"` | |
| `segmento` | `"Data Team"` | |
| `cliente` | `"AbbVie"` | Nombre sin código. |
| `codigo` | `"C0001P0033"` | **Clave** con `proyectos.codigo`. |
| `proyecto` | `"AbbVie - Soporte IFT Pediatria"` | |
| `activo` | `"SI"` / `"NO"` | |
| `head` | `"pcorrea"` | |
| `lider` | `"N/A"` | |
| `anio_inicio` | `"2017"` | |
| `anio_fin` | `"2018"` | |

### 2.11 `control`: Sheet de control del mes (archivo)

**Todo es texto.**

| Columna | Ejemplo |
|---|---|
| `region` | `"Europa + Asia"` |
| `codigo` | `"C0203P0014"` |
| `cliente` | `"Women Hub"` |
| `tipo_proyecto` | `"Data Team"` |
| `proyecto` | `"Women Hub - Data Team - 1 SEM 2026"` |
| `sponsor` | `"Asnar, Andrea"` |
| `head` | `"Cappa, Julián"` |
| `tecnologia` | `"Azure, data bri."` |
| `oportunidades` | texto libre |
| `reunion` | `"2026-10-16"` |
| `hs_vendidas_oct` | `"0"` |
| `hs_proy_oct_airtable` | `"0"` |

### 2.12 `situacion`: Situación Proyecto 2026 (Tableau, archivo)

Base del Gross Margin de Servicios. **Todo es texto.**

| Columna | Ejemplo |
|---|---|
| `codigo` | `"C0013P0018"` |
| `proyecto` | `"Grobocopatel Hnos - Assessment de Arquitectura 25"` |
| `region` | `"LATAM"` |
| `segmento` | `"Data Team"` |
| `anio` | `"2026"` |
| `mes` | `"2026-06"` |
| `horas` | `"74.0"` |
| `devengado` | `"5182026.12"` |
| `costo` | `"3653388.56"` |
| `margen` | `"1528637.56"` |

### 2.13 `tipos_cambio`: Tipos de cambio a USD (archivo)

| Columna | Tipo | Ejemplo | Notas |
|---|---|---|---|
| `moneda` | texto | `"EUR"` | Tiene que estar cada moneda usada. Si falta `USD`, se toma 1. |
| `usd_por_unidad` | texto numérico | `"1.1234"` | USD por 1 unidad de la moneda. |
| `fuente` | texto | `"BCE tipo de referencia"` | |
| `fecha` | texto `AAAA-MM-DD` | `"2026-10-22"` | |

---

## 3. Lo que calcula la página (no va en data.json)

La página arma estos conjuntos en el navegador con las mismas fórmulas del tablero original:

| Cálculo | A partir de | Qué es |
|---|---|---|
| `oportunidades_crm` | `crm`, `proyectos`, `tipos_cambio` | Oportunidades con etapa 0–6, cliente de Airtable, código de proyecto y monto en USD. Excluye QG SAU. |
| `acciones_crm` | `oportunidades_crm` | Último status y motivos de pérdida. |
| `gm_proyecto` | `situacion` | Gross Margin por proyecto. |
| `base` | `proyectos`, `horas`, `montos`, `control`, `maestro`, `devengado_monto` | Base Instalada: una fila por proyecto, con región, segmento, head y montos. |
| `pcs` | `base`, `oportunidades_crm`, `devengado_monto`, `crm_fases` | Una fila por propuesta comercial (PC). |

La búsqueda del PDF de la propuesta en Google Drive (`pc_pdf`) solo funciona dentro de claude.ai. Fuera de claude.ai queda el botón "Buscar en Gmail".

---

## 4. `seguimiento` (opcional, solo lectura)

Columnas manuales de la tabla de Servicios. Es un objeto cuya clave es el código de proyecto:

```json
"seguimiento": {
  "C0203P0014": {
    "fecha_cierre": "2026-10-26",
    "mail_enviado": "Si",
    "responsable": "Julián Cappa",
    "comentario": "Texto libre",
    "editado_at": "2026-10-28T14:00:00Z",
    "editado_por": "u_xxx"
  }
}
```

| Campo | Tipo | Valores |
|---|---|---|
| `fecha_cierre` | texto `AAAA-MM-DD` | |
| `mail_enviado` | texto | `"Si"`, `"No"` o `""` |
| `responsable` | texto | |
| `comentario` | texto | |
| `editado_at`, `editado_por` | texto | Informativo. |

Fuera de claude.ai estas columnas se ven pero no se pueden editar. Hoy el seguimiento está vacío.

---

## 5. Cómo regenerar data.json

1. Bajá cada fuente con las columnas de arriba, con los mismos nombres y tipos:
   - Airtable: tablas y campos indicados en cada `description`.
   - Zoho CRM: consultas COQL sobre `Deals` y `DealHistory` filtradas por `Type like 'Base Instalada%' or Type like 'New Name%'`.
   - CSV de Tableau, Maestro y control.
2. Armá el objeto de la sección 1, actualizá `generated_at` y la fecha de cada conjunto.
3. Reemplazá `data.json` en la carpeta publicada. No hace falta tocar `index.html`.

Para probarlo en tu equipo, serví la carpeta con un servidor web (por ejemplo `python -m http.server` y abrí `http://localhost:8000`). Si abrís `index.html` como archivo, el navegador bloquea la lectura de `data.json`.
