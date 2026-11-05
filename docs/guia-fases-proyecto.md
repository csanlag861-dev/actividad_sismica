# Guía de fases del proyecto: Observatorio de actividad sísmica

## Cómo usar esta guía

Avanza fase a fase. En cada solicitud:

1. Selecciona el agente indicado en VS Code.
2. Dale el prompt de la fase y pídele que se limite a ese objetivo.
3. Revisa los archivos modificados y la evidencia antes de pasar a la siguiente.
4. Si hay pruebas, exige que distinga entre **PASÓ**, **FALLÓ** y **NO EJECUTADO**.

Esta guía organiza el trabajo; no sustituye los requisitos de `ENUNCIADO.md`, el contrato de `CONTRATO-API.md` ni las instrucciones de `Guia-entregables-observatorio-sismico.md`.

**Alcance de las peticiones:** no pedir que se creen o actualicen artefactos SDD ni iniciar comandos `/speckit-*`. Pedir cambios concretos en el código o la documentación existente, con criterios verificables.


---

### Fase 0. Familiarizarse con el material de partida

**Objetivo:** comprobar qué viene preparado y entender el recorrido esperado antes de implementar.

**Agente:** Profesor / Mentor Agent para explicaciones; QA Tester para comprobar el frontend de demostración.

**Resultado esperado:**

- El mapa inicial funciona en modo demo y se identifican sus datos como ficticios.
- Se entiende el flujo: USGS → productor → Kafka → consumidor → PostgreSQL → API → Angular.
- Se localizan el contrato de la API, los requisitos, las fuentes y las entregas.

**Prompt sugerido:**

> Ayúdame a revisar el material inicial del Observatorio. Explícame qué componentes están implementados y cuáles debo desarrollar, siguiendo ENUNCIADO.md y la guía de entregables. No modifiques archivos ni generes artefactos SDD. Indícame cómo comprobar el mapa de demostración y qué significa que sus eventos sean ficticios.


---

### Fase 1. Preparar la estructura y el arranque en Docker Compose

**Objetivo:** establecer pronto una base ejecutable, pero construirla de forma incremental junto con los servicios, no como una configuración aislada.

**Agentes:** Deploy y Backend.

**Skill:** `docker-compose-demostracion`.

**Resultado esperado:**

- Un `compose.yaml` con los cinco servicios backend separados: productor, Kafka, consumidor, PostgreSQL y API.
- Red interna, configuración por variables de entorno, volúmenes persistentes y comprobaciones de salud/reintentos cuando correspondan.
- Una primera comprobación reproducible con `docker compose up --build -d`.

**Prompt sugerido:**

> Prepara la estructura inicial de Docker Compose para los cinco servicios obligatorios del enunciado: productor, Kafka, consumidor, PostgreSQL y API. Antes de editar, revisa los archivos existentes y las fuentes del proyecto. Mantén secretos fuera de Git; incluye o actualiza `.env.example` si hace falta. Distingue lo que queda funcionando de lo que son servicios todavía por implementar y valida el arranque disponible. No añadas servicios opcionales ni artefactos SDD.

**Punto de control:** Compose debe ir completándose a medida que se construyen los servicios. No declares el sistema completo porque el archivo exista: comprueba salud, conectividad y persistencia.

**Estado de la estructura inicial (revisión local, 5 de noviembre de 2026):** `compose.yaml`
declara los cinco servicios separados, nombres internos `kafka`/`postgres`, volúmenes para Kafka
y PostgreSQL, retención Kafka y publica únicamente la API en `127.0.0.1:8000`. `.env.example`
contiene una clave de reemplazo, no una credencial real; `.env` está excluido por Git. Esto valida
la configuración declarada, no el funcionamiento de los contenedores. Productor, consumidor y API
siguen siendo placeholders cuyo comando termina con `PENDIENTE`; no cuentan como servicios
implementados, y sus imágenes no se construyen desde el código del proyecto. Por tanto, aún faltan
su implementación, salud/reintentos donde correspondan y la verificación de persistencia y
comunicación en ejecución. No se debe presentar la pila como funcional ni continuar a una
demostración extremo a extremo.

Comprobaciones observadas en ese equipo:

- `docker compose version`: **NO DISPONIBLE**; la invocación falló porque PowerShell no reconoce
  `docker`. No se ejecutaron `docker compose config`, `docker compose up --build -d`, ni pruebas
  de salud o integración.
- Con Python 3.14.7 y PyYAML 6.0.3 disponibles localmente, se pudo analizar el YAML y comprobar
  estáticamente los cinco nombres de servicio, volúmenes, el bind local de API, la ausencia de
  puertos publicados para Kafka/PostgreSQL, `0.0.0.0:8000` en API y los tres comandos `PENDIENTE`.
  Resultado: **PASÓ** solo esa inspección sintáctica/estructural; no demuestra que Compose pueda
  interpretar interpolaciones, descargar imágenes o arrancar servicios.

Cuando Docker Compose esté disponible y el backend implementado, vuelve a validar la configuración
y ejecuta `docker compose up --build -d`; después comprueba `docker compose ps`, los logs y los
endpoints conforme al contrato. La publicación externa queda pendiente: ngrok requiere cuenta y
authtoken local, y Netlify requiere publicar el SPA y autorizar su origen exacto en CORS. Kafka y
PostgreSQL no deben exponerse mediante ngrok.


---

### Fase 2. Definir el modelo e inicializar PostgreSQL desde FastAPI

**Objetivo:** definir el modelo de datos e implementar su esquema con SQLAlchemy y migraciones Alembic, aplicadas al arrancar el servicio FastAPI después de que PostgreSQL esté listo.

**Agente:** Backend.

**Skills:** `api-fastapi-observatorio` y `consumidor-calidad-postgresql`.

**Resultado esperado:**

- Modelo conceptual, lógico y físico, con diccionario de campos, tipos, unidades y tratamiento de nulos.
- Modelos ORM de SQLAlchemy y restricciones que representen eventos actuales, versiones recibidas, procedencia/estado de ingesta y cuarentena, de acuerdo con el alcance definido.
- Configuración versionada de Alembic y una migración inicial revisada que cree el esquema en PostgreSQL. Las migraciones son la fuente de verdad del esquema; no se sustituirán por `Base.metadata.create_all()`.
- El servicio `api` ejecuta `alembic upgrade head` antes de iniciar Uvicorn. Si la migración falla, el contenedor debe informar el error y no iniciar la API como si la base estuviera lista. No se añade un servicio de migraciones separado.
- Evidencia de inicialización en una base vacía, aplicación de migraciones pendientes y persistencia del esquema al reiniciar; SQL de creación de tablas e índices generado desde las migraciones, incluido en el repositorio y enlazado desde la memoria o notebook.

**Prompt sugerido:**

> Revisa ENUNCIADO.md, la guía de entregables, el contrato de la API y el Compose existente. Define el modelo conceptual, lógico y físico y el diccionario de datos para conservar originales, estado actual por evento, versiones recibidas, procedencia y rechazos con su motivo y referencia al original. Implementa los modelos con SQLAlchemy y configura Alembic para generar y versionar las migraciones; revisa la migración inicial y úsala para crear el esquema, no `Base.metadata.create_all()`. Configura el servicio FastAPI para ejecutar `alembic upgrade head` antes de Uvicorn, después de que PostgreSQL esté saludable, y para detener el arranque con un error visible si la migración falla. No añadas otro servicio Compose. Conserva los nulos, unidades, claves y restricciones que requiere el modelo y la idempotencia. Comprueba la migración en una base vacía, que no haya cambios pendientes después de aplicarla y que el esquema persista tras reiniciar. Incluye en el repositorio el SQL de creación de tablas e índices generado desde las migraciones y enlázalo desde la memoria o notebook. No implementes aún las rutas finales ni cambies el contrato de la API.

**Punto de control:** una base vacía debe poder inicializarse de forma reproducible mediante el arranque de `api`; una migración fallida no debe quedar oculta. Revisa el SQL generado y explica grano, claves, relaciones e índices según las consultas previstas. La estructura de tablas que muestra el enunciado es orientación: justifica el modelo elegido sin perder los datos originales, las versiones ni los rechazos.


---

### Fase 3. Implementar el productor USGS y Kafka

**Objetivo:** consultar la fuente reciente y publicar novedades o revisiones sin generar mensajes redundantes innecesarios.

**Agente:** Backend.

**Skill:** `ingesta-usgs-aiokafka`.

**Resultado esperado:**

- Consulta periódica del feed de USGS.
- Registro de procedencia, hora de ingesta y modo (`live` o `replay`).
- Uso del identificador del evento como clave Kafka.
- Gestión explícita de fallos HTTP, límites, JSON inválido y reintentos.
- Una prueba que demuestre que un mensaje llega a Kafka.

**Prompt sugerido:**

> Implementa el productor del feed reciente de USGS según ENUNCIADO.md. Revisa los archivos existentes y la skill de ingesta antes de cambiar código. Conserva trazabilidad de fuente y hora de consulta, detecta versiones por `id` y `updated`, publica con clave por evento y espera confirmación de Kafka. Añade pruebas focalizadas para una respuesta válida y para errores de fuente; informa resultados observados. No hagas aún el despliegue público ni crees artefactos SDD.


---

### Fase 4. Implementar el consumidor, validación e idempotencia

**Objetivo:** procesar mensajes, conservar versiones y actualizar el evento actual sin duplicarlo ni permitir que una revisión antigua lo sobrescriba.

**Agente:** Backend.

**Skill:** `consumidor-calidad-postgresql`.

**Resultado esperado:**

- Validación y transformación documentadas.
- Escritura transaccional e idempotente.
- Confirmación del offset después de persistir.
- Rechazos conservados con su motivo y referencia al original.
- Uso del esquema ya creado por las migraciones Alembic y de los modelos/mapeos SQLAlchemy del backend, sin crear tablas por una vía paralela.
- Pruebas de evento nuevo, duplicado, revisión nueva y revisión antigua.

**Prompt sugerido:**

> Implementa el consumidor Kafka hacia PostgreSQL usando el esquema y los modelos SQLAlchemy definidos en la Fase 2; no crees ni modifiques tablas al margen de Alembic. Revisa los requisitos de versiones, transacciones, offsets e idempotencia. Debe conservar cada versión recibida y actualizar el estado actual solo si la versión es posterior; un reenvío no debe duplicar efectos. Conserva rechazos y motivos. Añade pruebas para evento nuevo, duplicado, actualización posterior y mensaje antiguo recibido después. Ejecuta las pruebas pertinentes y reporta esperado, observado y evidencia.


---

### Fase 5. Añadir integración geográfica y análisis de calidad de datos

**Objetivo:** enriquecer eventos con el catálogo geográfico y medir la calidad con resultados reproducibles.

**Agentes:** Backend y Documentador.

**Skills:** `consumidor-calidad-postgresql` y `analisis-calidad-sismica`.

**Resultado esperado:**

- Asignación espacial mediante geometría, no mediante texto del lugar.
- Eventos sin país conservados como categoría independiente.
- Reglas explícitas para fronteras, nulos y coordenadas inválidas.
- Evidencia antes/después y denominadores para el análisis de calidad, que irá en la memoria o notebook.

**Prompt sugerido:**

> Integra los eventos con el GeoJSON de países usando una operación espacial y las reglas de ENUNCIADO.md. No infieras el país desde `place` ni asignes automáticamente un país cercano a eventos marítimos. Revisa las reglas para fronteras y conserva los casos sin asignación. Después prepara un análisis de calidad reproducible con código o consultas, denominadores y resultados observados. No generes un sistema automático de informes: documenta el análisis en la memoria o notebook existente.


---

### Fase 6. Implementar la API FastAPI

**Objetivo:** ofrecer desde PostgreSQL las rutas de salud, eventos y estadísticas que necesita el panel.

**Agente:** Backend.

**Skill:** `api-fastapi-observatorio`.

**Resultado esperado:**

- `/api/eventos` conforme al contrato GeoJSON.
- `/api/salud` y `/api/estadisticas`.
- Filtros coherentes, fechas y unidades correctas, nulos representados como `null`.
- Errores de base de datos explícitos, sin respuestas que parezcan válidas.
- CORS configurado para desarrollo y el origen de publicación cuando se conozca.
- Consultas implementadas sobre SQLAlchemy y esquema PostgreSQL inicializado mediante las migraciones Alembic de la Fase 2.

**Prompt sugerido:**

> Implementa las rutas de la API FastAPI respetando exactamente `docs/source-of-truth/CONTRATO-API.md`. Reutiliza SQLAlchemy y el esquema que Alembic inicializa antes de arrancar Uvicorn; no cambies el contrato ni crees tablas por separado. Consulta PostgreSQL, aplica los filtros y unidades del contrato y devuelve errores explícitos si la base falla. Incluye pruebas de respuesta vacía, filtros, nulos y error de base de datos. No inventes datos para que las rutas parezcan funcionar. Indica qué pruebas ejecutaste y sus resultados.


---

### Fase 7. Completar el panel Angular

**Objetivo:** conectar el frontend entregado a la API y desarrollar los elementos interactivos y analíticos requeridos.

**Agente:** Frontend.

**Skill:** `angular-leaflet-observatorio`.

**Resultado esperado:**

- Conexión al backend y manejo visible de errores/desactualización.
- Filtros aplicados de forma coherente a mapa, tabla y estadísticas.
- Tabla accesible, gráficos, distribuciones y comparación histórica.
- Nulos y resultados truncados tratados conforme al contrato.
- El modo demo sigue identificado como sintético.

**Prompt sugerido:**

> Extiende el frontend Angular proporcionado para consumir la API según `CONTRATO-API.md` y los requisitos de ENUNCIADO.md. Antes de editar, revisa la implementación actual. Añade filtros, tabla y visualizaciones requeridas sin consultar USGS directamente; conserva el modo demo claramente identificado. Maneja timeout, errores y datos desactualizados de forma visible. Ejecuta la comprobación frontend más pequeña que cubra el cambio y resume los resultados.


---

### Fase 8. Incorporar el histórico y la reproducción

**Objetivo:** permitir una comparación histórica equivalente y demostrar el flujo Kafka sin depender de que aparezcan terremotos nuevos.

**Agentes:** Backend y QA Tester.

**Skills:** `ingesta-usgs-aiokafka` y `pruebas-aceptacion-replay`.

**Resultado esperado:**

- Extracción documentada del periodo histórico elegido, con consultas y recuentos.
- Respeto de límites de resultados, subdivisión de consultas y deduplicación entre intervalos.
- Reproducción identificada como tal y separada de datos live.
- Comparación con filtros y periodos equivalentes.

**Prompt sugerido:**

> Implementa o revisa la carga histórica y la reproducción según ENUNCIADO.md. Documenta filtros, periodo, consultas, fecha y número de registros; maneja límites de la fuente sin truncar silenciosamente. Asegura que el modo replay sea identificable y no se mezcle de forma confusa con live. Añade una prueba reproducible y reporta qué se observó.


---

### Fase 9. Validar aceptación y recuperación ante fallos

**Objetivo:** demostrar que el sistema funciona de extremo a extremo y que se comporta correctamente ante duplicados, revisiones, reinicios y desconexiones.

**Agente:** QA Tester.

**Skill:** `pruebas-aceptacion-replay`.

**Resultado esperado:**

- Pruebas con resultado esperado, observado, estado y evidencia.
- Revisión de persistencia tras reiniciar.
- Prueba del túnel desconectado y su recuperación, cuando el entorno lo permita.
- Lista clara de pruebas no ejecutadas y por qué.

**Prompt sugerido:**

> Ejecuta las pruebas de aceptación aplicables al sistema. Prioriza evento nuevo, duplicado, revisiones nueva y antigua, nulos, coordenadas inválidas, cuarentena, evento sin país, reinicio, replay y desconexión/recuperación del túnel. Para cada prueba informa resultado esperado, observado, estado PASÓ/FALLÓ/NO EJECUTADO y evidencia. No cambies código de producción ni marques como pasada una prueba que no hayas ejecutado.


---

### Fase 10. Publicar y preparar la demostración

**Objetivo:** dejar instrucciones reproducibles para arrancar los servicios, abrir la API por HTTPS y publicar Angular.

**Agentes:** Deploy y QA Tester.

**Skill:** `docker-compose-demostracion`.

**Resultado esperado:**

- Arranque de los cinco servicios en Compose y datos persistentes.
- ngrok exponiendo únicamente la API; secretos fuera de Git y del frontend.
- Publicación SPA en Netlify y CORS limitado a los orígenes necesarios.
- Pasos de recuperación, parada y demostración documentados.

**Prompt sugerido:**

> Prepara y valida el procedimiento de demostración local, ngrok y Netlify siguiendo ENUNCIADO.md y las guías existentes. Revisa que Compose use nombres de servicio y que no se expongan Kafka ni PostgreSQL. No incluyas credenciales reales. Distingue comprobaciones locales de despliegues externos que no puedas ejecutar. Actualiza el README existente si sus instrucciones ya no coinciden.


---

### Fase 11. Completar memoria, evidencias y defensa

**Objetivo:** reunir las decisiones, resultados y pruebas en una entrega que otra persona pueda revisar y reproducir.

**Agentes:** Documentador y Profesor / Mentor Agent.

**Skills:** `documentacion-didactica`, `analisis-calidad-sismica` y `defensa-tecnica`.

**Resultado esperado:**

- Memoria o notebook con fuentes, modelo, calidad, análisis y conclusiones.
- Tres conclusiones basadas en resultados reales y filtros explícitos.
- Mediciones de tiempo y memoria con al menos tres tamaños de entrada.
- Evidencias de aceptación y límites conocidos.
- Capacidad de explicar individualmente el flujo y una decisión técnica.

**Prompt sugerido:**

> Revisa la memoria o notebook frente a la guía de entregables y los resultados realmente obtenidos. Identifica apartados ausentes, cifras sin evidencia, filtros o denominadores no declarados y pruebas marcadas sin ejecutar. No inventes resultados ni crees documentos duplicados. Después ayúdame a preparar una defensa oral del recorrido de un evento y de una decisión técnica propia.


---

## Criterio para considerar el proyecto listo

- [ ] El flujo completo funciona desde la ingesta hasta el panel.
- [ ] Compose levanta los cinco servicios y los datos sobreviven a un reinicio.
- [ ] FastAPI aplica las migraciones Alembic pendientes al iniciar; los fallos de migración son visibles y la API no arranca como si el esquema estuviera preparado.
- [ ] El SQL de creación de tablas e índices generado desde las migraciones está en el repositorio y enlazado desde la memoria o notebook.
- [ ] Los reenvíos y revisiones se procesan sin duplicados ni regresiones.
- [ ] La API respeta su contrato y reporta errores de forma explícita.
- [ ] El panel aplica filtros y muestra claramente estados, nulos y errores.
- [ ] La carga histórica y la reproducción están documentadas y diferenciadas.
- [ ] Las pruebas ejecutadas tienen evidencia; las no ejecutadas están identificadas.
- [ ] README, memoria, conclusiones y procedimiento de demostración coinciden con la implementación.

**Importante:** no hace falta terminar una fase al 100 % para empezar a probar la siguiente. Mantén una comprobación de extremo a extremo temprana y ve sustituyendo los componentes mínimos por implementaciones completas.
