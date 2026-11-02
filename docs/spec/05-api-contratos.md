# Contratos de API

**Estado:** borrador pendiente de revisión del estudiante. El contrato de `GET /api/eventos` reproduce los requisitos normativos de la fuente del profesor; las formas detalladas de estadísticas, salud, paginación y errores que se sugieren aquí son propuestas del equipo, no aprobadas.

## 1. Propósito y límites

Este documento concreta los contratos HTTP que consumirá el frontend y organiza su verificación. El contrato existente de `GET /api/eventos` es normativo en lo que define; no se modifica ni se sustituye el documento del profesor. El detalle de `GET /api/estadisticas` y `GET /api/salud` no está completo en las fuentes: las formas incluidas en esta spec son propuestas revisables, como confirmó el estudiante, y requieren aprobación antes de implementarse.

Esta spec no define el esquema físico de PostgreSQL, el diseño visual de Angular ni la configuración completa de despliegue. Mantiene las responsabilidades descritas en [02-arquitectura-flujos.md](02-arquitectura-flujos.md) y los conceptos de datos de [03-modelo-datos-calidad.md](03-modelo-datos-calidad.md), sin resolver sus decisiones pendientes.

**Fuentes:** [CONTRATO-API](../source-of-truth/CONTRATO-API.md), secciones «GET /api/eventos», «Otros endpoints a implementar» y «Acceso desde Netlify»; [ENUNCIADO](../source-of-truth/ENUNCIADO.md), §§8–9 y 11; [README del material base](../source-of-truth/README.md), §§1 y 3; [FAQ](../source-of-truth/FAQ-ALUMNADO.md), preguntas 23 y 30–31 como orientación aclaratoria.

## 2. Principios compartidos

| Principio | Clase y fuente |
|---|---|
| La API consulta datos procesados; el navegador no consulta USGS como sustituto del productor, consumidor y persistencia. | **Requisito del profesor:** ENUNCIADO, §§2 y 8; README del material base, §§3 y 5. |
| Los tiempos se expresan en UTC y las unidades se identifican. En `GET /api/eventos`, los timestamps son milisegundos Unix UTC; las coordenadas llevan longitud, latitud y profundidad en km. | **Requisito del profesor:** ENUNCIADO, §§4.1 y 8; CONTRATO-API, «GET /api/eventos». |
| Para intervalos, `start` es inclusivo y `end` exclusivo en `GET /api/eventos`. La propuesta para estadísticas conserva esta misma semántica. | **Requisito del profesor para eventos; propuesta de consistencia para estadísticas:** CONTRATO-API, «GET /api/eventos» y «Otros endpoints a implementar». |
| Si falla la base de datos, estadísticas debe responder con error explícito y no presentar cero eventos como resultado válido. | **Requisito del profesor:** CONTRATO-API, «Otros endpoints a implementar». |
| No incluir credenciales o secretos en las respuestas. La ruta de salud tampoco debe exponerlos. | **Requisito del profesor:** CONTRATO-API, «GET /api/eventos», «Otros endpoints a implementar» y «Acceso desde Netlify». |
| Los filtros estadísticos deben ser equivalentes a los del mapa; la comparación debe aplicar filtros equivalentes a sus periodos reciente e histórico. | **Requisito del profesor:** ENUNCIADO, §8; CONTRATO-API, «Otros endpoints a implementar». |
| CORS autoriza el origen exacto de Netlify y `http://localhost:4200` durante desarrollo; admite GET y OPTIONS, y la cabecera `ngrok-skip-browser-warning` si el frontend la utiliza. | **Requisito del profesor:** CONTRATO-API, «Acceso desde Netlify»; ENUNCIADO, §9. El dominio efectivo de Netlify depende del despliegue y no está fijado aquí. |
| ngrok proporciona acceso HTTPS y no es autenticación. Para esta API pública de consulta no se requieren cookies. | **Requisito del profesor:** CONTRATO-API, «Acceso desde Netlify» y «Transporte y contenedores»; ENUNCIADO, §9. |
| Ante un error de consulta, el cliente no debe presentar como recién actualizados datos antiguos. Debe conservar/identificar la última lectura y señalar el fallo. | **Requisito del profesor:** ENUNCIADO, §§8 y 11; README del material base, §3. La presentación concreta es responsabilidad del frontend. |

## 3. `GET /api/eventos`

### 3.1 Contrato obligatorio de la fuente

Los siguientes aspectos son normativos, no propuestas de esta spec. Fuente: CONTRATO-API, «GET /api/eventos».

| Aspecto | Requisito del profesor |
|---|---|
| Estado y tipo de contenido | HTTP 200 y `Content-Type: application/json` cuando la consulta es válida. |
| Forma | GeoJSON `FeatureCollection`. Sin resultados, respuesta válida con `features: []`, no error. |
| Versión de evento | Cada evento aparece una sola vez, en su versión actual. |
| Periodo predeterminado | Terremotos ocurridos en las últimas 24 horas. |
| Metadata establecida | `generated`, `last_ingestion`, `mode`, `count` y `truncated`. `generated` indica generación de respuesta, no demuestra que la ingesta esté activa. |
| Timestamps | `generated`, `last_ingestion`, `time` y `updated` en milisegundos Unix UTC. |
| Feature y propiedades mínimas | `Feature` con `id`, geometría `Point` válida y propiedades `mag`, `magType`, `place`, `time`, `updated`, `country_id` y `country_name`. |
| Coordenadas | Orden GeoJSON longitud, latitud, profundidad; profundidad en km. |
| Desconocidos y JSON | Magnitud, profundidad o país desconocidos se representan como `null`; nunca NaN ni Infinity. La magnitud nula no satisface `min_mag` y se contabiliza aparte. |
| Filtros | `start` ISO8601 inclusivo, `end` ISO8601 exclusivo, `min_mag` y `country_id`. |
| Limitación de resultados | Si se limita el tamaño, se documenta el máximo; cuando se trunca, `truncated: true`, se informa el total completo y se ofrece un mecanismo de paginación. El frontend debe hacer visible que muestra datos parciales y no calcular estadísticas globales a partir de una muestra truncada. |
| Compatibilidad con el material de apoyo | El componente inicial espera una colección pequeña y completa. No se debe cambiar esa expectativa sin revisar la compatibilidad. |

### 3.2 Detalles no fijados por la fuente y propuesta de paginación

La fuente no fija el valor máximo de resultados, los parámetros de paginación, el orden, la semántica de `count` cuando hay páginas, ni cómo estabilizar las páginas mientras llegan revisiones live. Esos detalles no son requisitos docentes.

**Propuesta de contrato del equipo, pendiente de aprobación:**

- Usar `limit` y un `cursor` opaco para solicitar páginas sucesivas. El cursor se vincula a los filtros efectivos y a una ordenación determinista por tiempo del evento descendente e `id` como desempate; los valores concretos de `limit` por defecto y máximo quedan pendientes.
- En la respuesta, conservar toda la metadata normativa y proponer `metadata.total` como total de coincidencias de la consulta completa, `metadata.count` como cantidad de features de esta página y `metadata.next_cursor` como cursor siguiente o `null`. `truncated` sería `true` cuando esta respuesta no contenga por sí sola el conjunto completo de coincidencias, incluso en la última página; solo sería `false` si una respuesta contiene el conjunto completo.
- En solicitudes de continuación, el cliente mantiene los mismos filtros y `limit`; la validez del cursor y el comportamiento ante cambios de datos entre páginas requieren revisión. La fuente no exige consistencia de snapshot.
- El frontend indicaría explícitamente que ve una página/muestra parcial, por ejemplo «mostrando X de Y eventos», y ofrecería navegación para continuar. No calcularía estadísticas globales desde las features recibidas; `/api/estadisticas` consultaría el conjunto correspondiente a sus propios filtros, sin derivar agregaciones de la página visible.
- Si el conjunto cabe en una respuesta completa, `truncated` será `false` y la colección seguirá siendo completa. La integración con el componente de apoyo —que espera una colección pequeña y completa— queda como compatibilidad pendiente; esta propuesta no cambia esa expectativa.

La elección de cursor frente a otro mecanismo, el límite numérico, `count`/`total`, el orden y la consistencia de navegación deben aprobarse antes de implementar paginación. No se atribuyen a CONTRATO-API.

## 4. Propuesta de `GET /api/estadisticas`

### 4.1 Base normativa y alcance de la propuesta

**Requisito del profesor:** ofrecer conteos por hora UTC y país, distribuciones, comparación histórica, filtros equivalentes a los del mapa y, para cada resultado, periodo, filtros, unidad y registros sin valor. Si falla la base, devolver un error explícito en lugar de cero como resultado válido. (CONTRATO-API, «Otros endpoints a implementar»; ENUNCIADO, §8.)

**Aclaración confirmada por el estudiante:** el payload detallado no está en las fuentes y el equipo debe preparar una propuesta para revisión. El siguiente ejemplo propone una forma de representar la capacidad, no es contenido normativo, contrato aprobado ni resultado estadístico real.

### 4.2 Forma ilustrativa propuesta

Los textos entre `<...>` son marcadores explicativos, no valores ni un JSON ejecutable.

```json
{
  "period": {
    "start": "<inicio ISO8601 UTC>",
    "end": "<fin ISO8601 UTC>",
    "timezone": "UTC",
    "start_inclusive": true,
    "end_exclusive": true
  },
  "filters": {
    "min_mag": "<umbral efectivo o null>",
    "country_id": "<ADM0_A3 o null>",
    "event_type": "<tipo efectivo; no implica un nuevo parámetro>"
  },
  "completeness": {
    "status": "<complete | limited | unknown>",
    "limitations": ["<limitación conocida, si existe>"]
  },
  "hourly_counts": [
    {
      "hour_start": "<inicio de ventana UTC>",
      "count": "<entero: eventos>",
      "unit": "eventos"
    }
  ],
  "by_country": [
    {
      "country_id": "<ADM0_A3 o null>",
      "country_name": "<nombre o Sin país asignado>",
      "count": "<entero: eventos>",
      "unit": "eventos"
    }
  ],
  "distributions": {
    "magnitude": {
      "unit": "escala indicada por magType",
      "groups": [
        {
          "magType": "<tipo de magnitud>",
          "bins": [
            {
              "lower_inclusive": "<límite>",
              "upper_exclusive": "<límite>",
              "count": "<entero: eventos>"
            }
          ],
          "missing_count": "<entero: eventos sin magnitud>"
        }
      ]
    },
    "depth": {
      "unit": "km",
      "bins": [
        {
          "lower_inclusive": "<límite>",
          "upper_exclusive": "<límite>",
          "count": "<entero: eventos>"
        }
      ],
      "missing_count": "<entero: eventos sin profundidad>"
    }
  },
  "comparison": {
    "recent": {
      "period": "<periodo reciente>",
      "filters": "<filtros efectivos>",
      "count": "<entero: eventos>",
      "unit": "eventos"
    },
    "historical": {
      "period": "[2025-01-01T00:00:00Z, 2026-01-01T00:00:00Z)",
      "filters": "<filtros efectivos equivalentes>",
      "count": "<entero: eventos>",
      "denominator_days": "<todos los días del periodo, incluidos los de cero eventos>",
      "daily_average": "<eventos por día>"
    },
    "limitations": ["<por ejemplo, posible retraso de datos recientes>"]
  }
}
```

**Propuestas aún no aprobadas asociadas al ejemplo:**

- `start` y `end` de estadísticas usarían la semántica inclusiva/exclusiva de `/api/eventos`; ambos endpoints aplicarían `min_mag` y `country_id` con idéntico significado. El periodo predeterminado para estadísticas no está fijado; se propone el mismo intervalo reciente que usa el mapa, pendiente de revisión.
- `event_type` en el ejemplo es metadato de los filtros efectivos del conjunto, no propuesta de añadir un parámetro HTTP: el material fija `earthquake` para la ingesta live y el estudiante confirma ese tipo en el subconjunto histórico de trabajo.
- `hourly_counts` agruparía por hora de ocurrencia UTC, no por recepción del mensaje, tal como requiere el análisis del enunciado. Los conteos se expresarían en eventos. La granularidad exacta por país o subregión debe revisarse frente a la opción indicada por ENUNCIADO, §8.
- `by_country` incluiría «Sin país asignado» con `country_id: null`; no imputaría país. La forma exacta de filtrar exclusivamente la categoría sin país no está definida por el contrato de eventos y queda pendiente.
- Magnitud no tiene una unidad universal única entre `magType`; se propone separar las distribuciones por tipo en vez de mezclar escalas. Profundidad se expresa en km. La forma y anchura de bins no está especificada y requiere propuesta/revisión.
- Cada distribución contabilizaría sus valores ausentes por separado. Los totales y unidades se referirían a eventos actuales, no a mensajes o revisiones.
- `completeness` y `limitations` son una propuesta para comunicar si la cobertura del periodo está completa, limitada o es desconocida. No se fija aquí una métrica o umbral automático de completitud.

### 4.3 Comparación histórica y filtros efectivos

**Aclaración del estudiante:** el subconjunto histórico de trabajo es 2025, mundial, magnitud ≥4 y tipo `earthquake`; sirve para trabajo, filtros, pruebas y comparación histórica, no para restringir la ingesta live. (Propuesta inicial: ENUNCIADO, §4.2; comparación: §8; aclaración en [00-principios-y-alcance.md](00-principios-y-alcance.md#fuentes-de-datos).)

**Requisito del profesor:** comparar periodos con filtros equivalentes; en la media diaria histórica, incluir todos los días del periodo, también los que tienen cero eventos. La comparación reciente puede estar incompleta por retrasos. (ENUNCIADO, §8; FAQ, pregunta 23 como explicación.)

**Propuesta pendiente:** la consulta base de estadísticas aplica los filtros recibidos con la misma semántica que `/api/eventos`. Para la comparación, la respuesta incluiría los filtros efectivos de ambos lados y aplicaría a reciente e histórico el mismo periodo comparable de magnitud/tipo/ámbito. El conjunto histórico disponible está limitado a `mag >= 4`, por lo que la comparación usaría como mínimo ese umbral; si el usuario pide uno más estricto, se aplicaría a ambos periodos. Si el filtro de mapa/consulta es menos estricto, el panel debe mostrar que el subresultado de comparación usa `mag >= 4`, o la comparación debe declararse no disponible para evitar una comparación engañosa. La forma de presentar esta diferencia requiere aprobación.

Para el subconjunto completo de 2025, el denominador es 365 días (2025 tiene 365 días), con días sin eventos incluidos; este es el denominador ilustrado en FAQ, pregunta 23. Si se filtra por país/región, el mismo filtro se aplica a los dos periodos y el denominador sigue incluyendo los 365 días, incluidos aquellos con cero eventos para ese ámbito. No se atribuye al profesor una decisión de payload o de comportamiento ante filtros incompatibles con la cobertura histórica.

El filtro de tipo `earthquake` y el ámbito mundial describen el subconjunto histórico confirmado, no una restricción nueva de la ingesta live.

## 5. Propuesta de `GET /api/salud`

**Requisito del profesor:** la ruta informa estado de API y fecha de última ingesta; no incluye credenciales. (CONTRATO-API, «Otros endpoints a implementar».) **Aclaración del estudiante:** el payload detallado debe proponerse para revisión.

**Propuesta de forma, no aprobada; valores entre `<...>` son marcadores, no JSON ejecutable:**

```json
{
  "status": "<ok | degraded>",
  "last_successful_usgs_ingestion_at": "<timestamp UTC o null>",
  "freshness": {
    "status": "<fresh | stale | unknown>",
    "evaluated_at": "<timestamp UTC>",
    "threshold": "<valor/unidad aprobado o null>"
  }
}
```

La respuesta permite distinguir que la API responde, la última ingesta correcta conocida y si la actualización está dentro del umbral elegido. Si no se conoce la última ingesta, se propondría `null` y estado `unknown`, no una fecha inventada. La disponibilidad del endpoint ya es una señal de accesibilidad de la API; el cliente también conserva la hora de su último acceso correcto y el periodo de los datos consultados, como pide ENUNCIADO, §8.

El profesor exige avisar cuando los datos estén desactualizados, pero no establece un umbral de obsolescencia ni el cálculo de `fresh`/`stale`. No se propone un valor numérico: el umbral, los estados y cómo se obtienen requieren revisión. La última consulta correcta a USGS no demuestra por sí sola que todos los datos del periodo estén completos.

## 6. Errores y estados HTTP

### 6.1 Lo exigido por las fuentes

- Una consulta válida de eventos puede responder con una colección vacía; esto no debe confundirse con un fallo. (CONTRATO-API, «GET /api/eventos».)
- Si falla la base de datos, estadísticas devuelve un error explícito, no una respuesta exitosa con cero eventos. (CONTRATO-API, «Otros endpoints a implementar».)
- El frontend avisa si API/túnel no responde y conserva o identifica la última lectura. (ENUNCIADO, §§8 y 11.)
- Las fuentes no fijan un formato común de error ni códigos concretos para parámetros inválidos o indisponibilidad de base.

### 6.2 Propuesta de error, pendiente de aprobación

Como propuesta revisable, los errores de parámetros podrían responder HTTP 422 y la indisponibilidad de base HTTP 503; el formato podría ser:

```json
{
  "error": {
    "code": "<código estable>",
    "message": "<explicación segura para el cliente>"
  }
}
```

Los valores de ejemplo son marcadores, no respuestas implementadas. La selección de códigos HTTP, campos y códigos de error no es requisito del profesor mientras no se apruebe. No se devolverían detalles internos, credenciales o secretos. Los fallos no se representarían como `200` con `features: []` o conteos cero: una colección vacía solo es válida cuando la consulta terminó correctamente. (La forma exacta de errores sigue pendiente; la distinción de resultado válido y fallo deriva del CONTRATO-API, «GET /api/eventos» y «Otros endpoints a implementar».)

## 7. Matriz de filtros y consistencia

La semántica de parámetros de eventos procede de CONTRATO-API, «GET /api/eventos». La equivalencia de filtros para el análisis y la comparación procede de ENUNCIADO, §8, y CONTRATO-API, «Otros endpoints a implementar». La extensión exacta de filtros estadísticos a continuación es propuesta pendiente de aprobación.

| Filtro/alcance | `/api/eventos` | `/api/estadisticas` (propuesta) | Comparación reciente ↔ histórica |
|---|---|---|---|
| Periodo | `start` inclusivo y `end` exclusivo; default últimas 24 h. | Mismo significado de `start`/`end`; default por decidir. | Periodos identificados explícitamente; histórico cubre 2025 y reciente se compara bajo filtros equivalentes. |
| Magnitud | `min_mag`; magnitud nula no cumple el filtro y se contabiliza aparte. | Mismo filtro y tratamiento en los agregados base. | Histórico de trabajo es `mag >= 4`; aplicar al reciente el mismo umbral, además de cualquier umbral más estricto. Si la selección del mapa es menos estricta, mostrar el filtro efectivo de comparación o declarar la comparación no disponible; política UX pendiente. |
| País/región | `country_id` selecciona la entidad según el contrato; semántica especial para pedir solo nulos no está definida. | Aplicar el mismo `country_id` a cada agregado; incluir categoría sin país en agrupaciones. | Aplicar el mismo ámbito geográfico a ambos lados; el denominador incluye todos los días del periodo histórico aunque no haya eventos en algunos. |
| Tipo de evento | El endpoint representa terremotos según el contrato; no se establece un parámetro de tipo. | La base de trabajo del proyecto analiza `earthquake`. Un filtro adicional de tipo no se propone como requisito. | El subconjunto histórico confirmado es `earthquake`; aplicar tipo equivalente al periodo reciente. |
| Live frente a histórico | El default de eventos es últimas 24 h y no hereda `min_mag >= 4` ni el ámbito mundial histórico. | Los filtros de consulta del panel siguen el mapa. | Subconjunto histórico de trabajo: 2025, mundial, `mag >= 4`, `earthquake`. **No restringe la ingesta live.** |

El conjunto histórico anual de trabajo comprende los días con y sin eventos. La media diaria usa el total del periodo histórico dividido por todos sus días; no se elimina un día por tener conteo cero. La comparación debe identificar explícitamente filtros, periodo, unidad y denominador. (ENUNCIADO, §§4.2 y 8; aclaración estudiantil DEC-P-05 en [00](00-principios-y-alcance.md#fuentes-de-datos).)

## 8. Ejemplos y casos de aceptación

Los casos siguientes son criterios propuestos para comprobar requisitos existentes; **no son pruebas ya ejecutadas**. Los ejemplos no contienen datos reales.

| Caso | Resultado esperado | Trazabilidad | Evidencia futura |
|---|---|---|---|
| Consulta válida de eventos sin coincidencias | HTTP 200, GeoJSON `FeatureCollection` y `features: []`; metadata de respuesta presente. | CONTRATO-API, «GET /api/eventos»; `REQ-API-001`. | Prueba de contrato con filtros válidos que no produzcan eventos. |
| Extremos de fecha | Un evento en `start` se incluye; uno exactamente en `end` se excluye. | CONTRATO-API, «GET /api/eventos»; `REQ-API-001`, `REQ-DATA-002`. | Fixture con tiempos en ambos extremos y comparación de IDs. |
| `min_mag` y magnitud nula | El nulo no satisface el filtro y se cuenta aparte en análisis donde corresponda. | CONTRATO-API, «GET /api/eventos»; ENUNCIADO, §8; `REQ-API-001`, `REQ-DATA-002`, `REQ-TEST-003`. | Fixture con magnitud conocida inferior/superior y `null`; revisar respuesta y contabilidad. |
| Evento sin país asignado | Se conserva con país nulo y aparece en análisis como «Sin país asignado», sin imputar país cercano. | ENUNCIADO, §§4.3 y 8; `REQ-API-001`, `REQ-DATA-003`, `REQ-TEST-003`. | Evento válido fuera de polígonos y revisión de API/agregación. |
| Valores desconocidos | Magnitud, profundidad o país desconocidos son JSON `null`, no NaN/Infinity; la geometría de evento sigue siendo válida. | CONTRATO-API, «GET /api/eventos»; `REQ-API-001`, `REQ-DATA-002`. | Validación JSON de fixture con campos desconocidos. |
| Respuesta truncada y páginas | Si se adopta límite, informa el total completo, `truncated` y continuación según el mecanismo finalmente aprobado; el cliente identifica la página parcial y no calcula estadísticas globales con ella. | CONTRATO-API, «GET /api/eventos»; `REQ-API-001`, `REQ-F-008`. | Prueba contractual una vez aprobados límite y paginación; verificar navegación y que no se confunda página con total. |
| Fallo de base de datos | Estadísticas responde error explícito, no `200` con ceros. | CONTRATO-API, «Otros endpoints a implementar»; `REQ-NF-003`, `REQ-API-002`. | Desconectar base en entorno controlado y verificar respuesta de error. |
| Estadísticas con periodo, filtros, unidades y nulos | Resultado identifica periodo, filtros efectivos, unidad de cada valor y número de registros sin valor; comparación usa filtros equivalentes y denominador histórico completo. | ENUNCIADO, §8; CONTRATO-API, «Otros endpoints a implementar»; `REQ-API-002`, `REQ-DATA-004`, `REQ-F-009`, `REQ-F-010`. | Contrato aprobado y contraste de agregados con consulta independiente. |
| Salud sin credenciales | Informa estado de API y última ingesta; no expone secretos. El estado stale/fresh solo se afirma con umbral acordado. | CONTRATO-API, «Otros endpoints a implementar»; ENUNCIADO, §8; `REQ-API-003`, `REQ-NF-004`, `REQ-F-012`. | Revisar respuestas y simular ingesta antigua después de aprobar contrato/umbral. |
| Filtros coherentes en mapa y estadísticas | Mismos filtros efectivos de fecha, magnitud y país/región producen conjuntos/denominadores coherentes; comparación declara sus filtros y límites históricos. | ENUNCIADO, §8; CONTRATO-API, «GET /api/eventos» y «Otros endpoints a implementar»; `REQ-F-010`, `REQ-API-001`, `REQ-API-002`, `REQ-DATA-004`. | Ejecutar consultas paralelas con mismos filtros y contrastar resultados de referencia. |

## 9. Decisiones y revisión

| Tema | Requisito oficial | Propuesta del equipo | Decisión pendiente |
|---|---|---|---|
| `GET /api/eventos` | Es obligatorio cumplir los campos, filtros, fechas, nulos, unicidad, metadata y reglas de truncado descritas en §3.1. | Paginación por `limit` + cursor opaco, orden estable, `metadata.total`, `count` por página y `next_cursor`. | Aprobar o sustituir mecanismo, semántica de metadata, límite numérico y consistencia entre páginas. La colección pequeña/completa esperada por el frontend de apoyo debe conservarse o revisarse explícitamente. |
| `GET /api/estadisticas` | Capacidades mínimas de conteos horarios/país, distribuciones, comparación, filtros equivalentes, periodo, unidad, nulos y error explícito de base. | Forma de respuesta de §4, conteos por evento actual, nulos explícitos, grupos de magnitud por `magType` y transparencia de cobertura. | **Pendiente de aprobación del estudiante:** esquema y semánticas de respuesta de estadísticas; periodo default, bins, granularidad país/subregión y comportamiento cuando los filtros exceden la cobertura histórica. |
| `GET /api/salud` | Estado de API y fecha de última ingesta, sin credenciales. | Estado, timestamp de última ingesta y frescura (`fresh`/`stale`/`unknown`). | **Pendiente de aprobación del estudiante:** esquema y valores de salud; coordinación con el estado que muestra el cliente. |
| Errores | Estadísticas no debe responder como si cero fuera resultado válido si falla la base. | 422 para parámetros inválidos, 503 para base no disponible y cuerpo JSON común. | **Pendiente de aprobación:** formato de error, códigos HTTP y códigos estables; no están fijados por las fuentes. |
| Obsolescencia | El panel debe avisar de datos desactualizados. | Campo de frescura y umbral configurable/mostrado. | **Pendiente de decisión:** umbral y cálculo. El profesor no fija valor ni unidad temporal; no inferir uno. |
| Filtros de país nulo | Los eventos sin país se conservan y se contabilizan como «Sin país asignado». | Incluirlos en agrupaciones con `country_id: null`; no se propone aún un valor de filtro para seleccionarlos exclusivamente. | Acordar si y cómo filtrar solo los eventos sin país de forma idéntica en mapa y estadísticas. |
| Tiempo y cobertura de comparación | Filtros equivalentes y denominador que incluye días históricos sin eventos; reciente puede ser incompleto por retrasos. | Filtros efectivos devueltos en la respuesta; magnitud de comparación no menor que 4 para ajustarse al subconjunto histórico confirmado. | Aprobar cómo presenta el panel una comparación cuyo filtro efectivo difiere de una selección de mapa menos estricta y cómo comunicar cobertura/limitaciones. |
| CORS y origen Netlify | Permitir el origen exacto de Netlify y `http://localhost:4200`, GET/OPTIONS y cabecera de ngrok cuando corresponda. | Mantener el origen exacto como configuración de despliegue, no comodín. | El dominio real de Netlify se conocerá al desplegar; confirmar el origen publicado y si el frontend envía la cabecera. |

Las decisiones propuestas no alteran [CONTRATO-API.md](../source-of-truth/CONTRATO-API.md) ni se consideran aprobadas por aparecer en este borrador. Una aprobación debe reflejarse después en los artefactos canónicos correspondientes.

### Trazabilidad a requisitos existentes

No se crean ni renombran identificadores en esta spec.

| Requisito existente | Aplicación en esta spec |
|---|---|
| `REQ-API-001` | Contrato obligatorio de `/api/eventos`, propuesta de paginación y sus casos de aceptación. |
| `REQ-API-002`, `REQ-NF-003` | Capacidades mínimas y propuesta revisable de `/api/estadisticas`; error explícito ante fallo de base. |
| `REQ-API-003`, `REQ-F-012`, `REQ-TEST-005` | Requisito mínimo y propuesta de `/api/salud`, estado del panel y obsolescencia pendiente. |
| `REQ-F-008`, `REQ-F-009`, `REQ-F-010` | Compatibilidad del mapa, estadísticas, filtros equivalentes y comparación. |
| `REQ-F-005`, `REQ-DATA-004` | Subconjunto histórico confirmado, comparación y denominador. |
| `REQ-DATA-002`, `REQ-DATA-003`, `REQ-TEST-003` | Timestamps, unidades, nulos y categoría sin país. |
| `REQ-NF-004` | No exposición de credenciales/secretos. |
