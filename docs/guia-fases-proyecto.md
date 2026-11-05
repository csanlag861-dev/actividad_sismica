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


---

### Fase 2. Definir el modelo de datos y la inicialización de PostgreSQL

**Objetivo:** decidir cómo conservar eventos actuales, revisiones y rechazos antes de implementar la persistencia del consumidor.

**Agente:** Backend.

**Skill:** `consumidor-calidad-postgresql`.

**Resultado esperado:**

- Esquema e inicialización reproducible de PostgreSQL.
- Identificación única del evento y de cada versión recibida.
- Tablas o mecanismos equivalentes para originales, estado actual, procedencia y cuarentena, según el alcance acordado.
- Un diccionario de campos con tipos, unidades y tratamiento de nulos.

**Prompt sugerido:**

> Diseña e implementa la persistencia PostgreSQL mínima que exige el enunciado para eventos, versiones y rechazos. Revisa antes el modelo y los requisitos de idempotencia. Explica las claves, restricciones e índices elegidos; conserva los originales y no conviertas valores ausentes en cero. Incluye una forma reproducible de inicializar la base y pruebas pequeñas de inserción y consulta. No cambies el contrato de la API ni generes artefactos SDD.


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
- Pruebas de evento nuevo, duplicado, revisión nueva y revisión antigua.

**Prompt sugerido:**

> Implementa el consumidor Kafka hacia PostgreSQL. Revisa los requisitos de versiones, transacciones, offsets e idempotencia. Debe conservar cada versión recibida y actualizar el estado actual solo si la versión es posterior; un reenvío no debe duplicar efectos. Conserva rechazos y motivos. Añade pruebas para evento nuevo, duplicado, actualización posterior y mensaje antiguo recibido después. Ejecuta las pruebas pertinentes y reporta esperado, observado y evidencia.


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

**Prompt sugerido:**

> Implementa la API FastAPI respetando exactamente `docs/source-of-truth/CONTRATO-API.md`. Consulta PostgreSQL, aplica los filtros y unidades del contrato y devuelve errores explícitos si la base falla. Incluye pruebas de respuesta vacía, filtros, nulos y error de base de datos. No inventes datos para que las rutas parezcan funcionar. Indica qué pruebas ejecutaste y sus resultados.


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
- [ ] Los reenvíos y revisiones se procesan sin duplicados ni regresiones.
- [ ] La API respeta su contrato y reporta errores de forma explícita.
- [ ] El panel aplica filtros y muestra claramente estados, nulos y errores.
- [ ] La carga histórica y la reproducción están documentadas y diferenciadas.
- [ ] Las pruebas ejecutadas tienen evidencia; las no ejecutadas están identificadas.
- [ ] README, memoria, conclusiones y procedimiento de demostración coinciden con la implementación.

**Importante:** no hace falta terminar una fase al 100 % para empezar a probar la siguiente. Mantén una comprobación de extremo a extremo temprana y ve sustituyendo los componentes mínimos por implementaciones completas.
