# Despliegue y operación

**Estado:** borrador pendiente de revisión del estudiante. Describe requisitos y alternativas documentados; no contiene configuración ejecutable ni afirma que el sistema esté desplegado o que las pruebas de despliegue hayan sido realizadas.

## 1. Propósito y límites

Esta especificación concreta las expectativas de despliegue local del backend, publicación del frontend y operación durante la demostración. Sirve de base para planificar la implementación y verificarla; no crea `compose.yaml`, Dockerfile(s), secretos, configuración de Netlify ni instrucciones de ejecución listas para usar.

La arquitectura se limita a lo prescrito en [ENUNCIADO](../source-of-truth/ENUNCIADO.md), §§2 y 9, y a las opciones explicadas en el [README del material base](../source-of-truth/README.md), §§3–4. Las alternativas siguen siendo alternativas cuando las fuentes no seleccionan una. Los contratos HTTP y sus decisiones se mantienen en [05-api-contratos.md](05-api-contratos.md), no se redefinen aquí.

## 2. Arquitectura de despliegue requerida

### Requisitos del profesor

- El backend debe ejecutarse en cinco servicios separados de Docker Compose: productor Python, Kafka, consumidor Python, PostgreSQL y API FastAPI. Se ejecutan localmente en el equipo del estudiante y la entrega debe permitir arrancarlos conjuntamente con `docker compose up --build -d`. (ENUNCIADO, §2; entregables, §12.)
- Angular se compila y publica como SPA en Netlify. No se requiere contenedor de producción para el frontend del paquete de apoyo. (ENUNCIADO, §2; README del material base, §§3–4.)
- Deben existir volúmenes persistentes para los datos y retención limitada de Kafka. (ENUNCIADO, §9.)
- Se configuran comprobaciones de salud y reintentos para dependencias; el orden de arranque no demuestra que Kafka o PostgreSQL estén listos. (ENUNCIADO, §2; README del material base, §3.)
- ngrok proporciona acceso HTTPS público a la API durante la demostración. Kafka y PostgreSQL no se publican mediante el túnel. (ENUNCIADO, §§2 y 9.)

### Opciones que no equivalen a decisiones aprobadas

- Los procesos Python pueden compartir una imagen y usar comandos de inicio diferentes; no es obligatorio hacerlo así. (ENUNCIADO, §2; Guía de entregables, §1.2.)
- ngrok puede ejecutarse instalado en el equipo o como contenedor en la red Compose. La elección del proyecto sigue pendiente. (ENUNCIADO, §9; README del material base, §3.)
- El despliegue local no exige servidor cloud permanente ni disponibilidad 24/7. El backend y el túnel pueden apagarse fuera de la demostración. (ENUNCIADO, §§2 y 9.)

La arquitectura no fija imágenes, nombres definitivos de servicios, topología del broker ni configuración específica de reintentos o de los health checks.

## 3. Contenedores, red y configuración

Los Dockerfile(s) construyen las imágenes; `compose.yaml` declara cómo se ejecutan y conectan los cinco servicios. El documento docente permite compartir una imagen entre productor, consumidor y API Python con comandos de inicio diferentes, como alternativa. (ENUNCIADO, §2; Guía de entregables, §1.2.)

| Aspecto | Requisito o límite |
|---|---|
| Comunicación Compose | Los servicios se comunican por nombre de servicio. `localhost` dentro de un contenedor designa ese contenedor, no el host ni otro servicio. (ENUNCIADO, §9; README del material base, §3.) |
| API | Uvicorn/FastAPI escucha en `0.0.0.0:8000` dentro del contenedor. (ENUNCIADO, §9; CONTRATO-API, «Transporte y contenedores».) |
| Kafka desde el host | Si se accede también desde el host, se documentan sus listeners; las direcciones anunciadas deben permitir la conexión correspondiente. La configuración concreta no está fijada. (ENUNCIADO, §9; FAQ, pregunta 28.) |
| Dependencias | Entregar manifiestos/archivos de dependencias fijadas, inicialización de base y `.env.example` sin credenciales reales. Los nombres de variables deben corresponder a la implementación; los ejemplos de la guía son orientativos, no valores aprobados. (ENUNCIADO, §§9 y 12; Guía de entregables, §§1.2–1.3.) |
| Persistencia | Volúmenes para conservar datos entre reinicios; definir cómo se inicializa la base y cómo se carga el histórico en instrucciones reproducibles. (ENUNCIADO, §§9 y 12; Guía de entregables, §§1.2 y 1.4.) |
| Repositorio | No incluir secretos, authtoken de ngrok, credenciales, `node_modules`, volúmenes binarios o grandes datos regenerables. Documentar dónde se conservan los originales y cómo regenerarlos. (ENUNCIADO, §12; README del material base, §§3–4; Guía de entregables, §§1.3–1.4.) |

No se fijan aquí nombres de variables, listeners, puertos de Kafka, imágenes ni versiones adicionales. La preferencia del estudiante por versiones LTS cuando las fuentes no indiquen versión no desplaza las versiones concretas del frontend proporcionado (aclaración del estudiante; README del material base, §1).

## 4. Alternativas de ngrok y seguridad de red

Las fuentes documentan dos configuraciones posibles; esta spec no selecciona una:

1. **ngrok en el equipo anfitrión:** publicar la API localmente, preferiblemente en `127.0.0.1:8000:8000`, y dirigir ngrok al puerto publicado. (ENUNCIADO, §9; README del material base, §3.)
2. **ngrok en otro contenedor Compose:** conectarlo a la red Compose y dirigirlo a la URL interna de la API, por ejemplo `http://api:8000` si el nombre de servicio elegido fuese `api`. En esta alternativa no hace falta publicar el puerto de API en el host para que ngrok acceda al servicio. (ENUNCIADO, §9; README del material base, §3.)

Requisitos de transporte y seguridad:

- ngrok termina HTTPS público y gestiona el certificado; la API puede usar HTTP dentro de la red local/Compose. Angular utiliza la URL HTTPS pública de ngrok. No se requiere instalar un certificado en FastAPI para este flujo. (ENUNCIADO, §9; CONTRATO-API, «Transporte y contenedores».)
- CORS permite el origen exacto del sitio Netlify y `http://localhost:4200` durante desarrollo; admite GET y OPTIONS y la cabecera `ngrok-skip-browser-warning` si el frontend la utiliza. (ENUNCIADO, §9; CONTRATO-API, «Acceso desde Netlify».)
- CORS no es autenticación; la API pública de consulta no requiere cookies según el contrato. (CONTRATO-API, «Acceso desde Netlify».)
- El túnel expone solo la API de consulta. No se publican rutas administrativas, secretos, PostgreSQL ni Kafka. El authtoken de ngrok no se incluye en Angular ni en Git. (ENUNCIADO, §9; CONTRATO-API, «Acceso desde Netlify».)

La elección host/contenedor, los puertos publicados y la URL real del túnel se mantienen pendientes de implementación y revisión; los ejemplos no acreditan una configuración activa.

## 5. Publicación de frontend en Netlify

Los valores siguientes pertenecen al material de apoyo entregado; son instrucciones/configuración de referencia, no prueba de que exista un sitio Netlify ni de que el proyecto se haya publicado.

| Elemento | Valor/documentación entregada | Fuente |
|---|---|---|
| Build command | `npm run build` | README del material base, §4; `frontend/netlify.toml` entregado. |
| Carpeta de publicación | `dist/observatorio/browser`, relativa a la base configurada | README del material base, §4; `frontend/netlify.toml` entregado. |
| Base directory | `frontend` si se publica el repositorio completo; vacío si la raíz publicada ya es el frontend | README del material base, §4. |
| Versión de Node declarada en configuración | El `netlify.toml` entregado declara `NODE_VERSION = "22"`; no implica que el sitio se haya construido en Netlify. El README del material base describe para desarrollo Node 22.12 o posterior dentro de la rama 22, o Node 24. | README del material base, §1; `frontend/netlify.toml` entregado. |
| Rutas SPA | El archivo `_redirects` entregado permite recargar rutas SPA | README del material base, §4. |

Antes de compilar/publicar el modo API, se actualiza `config.json` con la URL HTTPS real de ngrok y se ajusta CORS al dominio exacto de Netlify. El frontend debe verificarse desde otro dispositivo. El ejemplo es SPA y no requiere SSR. (README del material base, §§3–4; ENUNCIADO, §§2 y 9.)

No se afirma que haya cuenta, dominio, sitio ni publicación real de Netlify; [VALIDACION.md](../source-of-truth/VALIDACION.md) indica que no se desplegó una cuenta real ni se abrió un túnel ngrok del alumnado.

## 6. Operación y demostración

**Procedimiento requerido para la defensa**, descrito por el material docente, no ejecutado en esta spec:

1. Arrancar Docker Compose y comprobar que los servicios están operativos.
2. Comprobar que PostgreSQL contiene los datos necesarios y que la ingesta funciona.
3. Activar ngrok y verificar que la URL de API configurada en Angular corresponde al túnel.
4. Abrir el panel y comprobar datos, filtros y estado de actualización.
5. Preparar la reproducción histórica para cuando se solicite.

Fuente: Guía de entregables, §3.2; ENUNCIADO, §§11–12. La reproducción demostrativa debe estar ordenada, acelerada, pasar por Kafka y quedar separada/identificada respecto a live; el mecanismo concreto se mantiene en la spec de ingesta/arquitectura. (ENUNCIADO, §11; [04-ingesta-procesamiento.md](04-ingesta-procesamiento.md), §8.)

El equipo, los contenedores y el túnel deben estar activos para disponer de datos nuevos en la demostración. Netlify puede servir la interfaz aunque el backend local esté apagado, pero el panel debe mostrar esa indisponibilidad claramente. No se exige disponibilidad 24/7. (ENUNCIADO, §9; Guía de entregables, §§3.1–3.2.)

Al detener servicios, las instrucciones de entrega deben explicar cómo detenerlos y qué datos persisten al reiniciar; la persistencia mediante volúmenes sí es requisito. La secuencia operativa detallada y sus comandos concretos son decisiones de implementación. (ENUNCIADO, §§9 y 12; Guía de entregables, §1.4.)

## 7. Observabilidad y recuperación

| Situación | Comportamiento exigido/documentado | Detalle técnico pendiente |
|---|---|---|
| Dependencia no lista al iniciar | Configurar health checks y reintentos; el orden de arranque no garantiza disponibilidad. (ENUNCIADO, §2.) | Probes, periodicidad, plazos y condición de readiness concretos no están fijados. |
| Reinicio del consumidor | Reanudar sin duplicar efectos ya persistidos; la escritura persistida precede a la confirmación del offset. (ENUNCIADO, §§7 y 11.) | Modo de commit y comandos de reinicio quedan para implementación; se preserva el requisito de idempotencia. |
| Reinicio del sistema y datos | Los volúmenes persisten los datos; instrucciones explican qué sobrevive al reinicio. (ENUNCIADO, §9; Guía de entregables, §§1.2 y 1.4.) | Nombres/alcance de volúmenes, respaldo y procedimientos de recuperación concretos no especificados. |
| Backend/túnel inaccesible | El panel informa el fallo y conserva o identifica la última lectura; al volver el túnel, recupera consultas. (ENUNCIADO, §§8 y 11.) | Esquema de errores y señales conforme a 05; no se fija política de reintentos del frontend aquí. |
| Productor detenido | Distinguir última ingesta de generación de respuesta y del último acceso del panel; avisar de datos desactualizados. (ENUNCIADO, §8; CONTRATO-API, «GET /api/eventos».) | El umbral numérico de obsolescencia no está fijado en las fuentes; depende de la decisión de API en 05 §5. |
| Servicio recuperado | Tras la vuelta del túnel/API, el panel vuelve a consultar automáticamente. (ENUNCIADO, §11.) | Intervalo concreto de recuperación y estrategia de retry distintos de la actualización indicada por fuentes no se inventan aquí. |
| Cuotas/servicios externos | Las cuentas gratuitas tienen cuotas; el mapa base OpenStreetMap depende de Internet y está sujeto a su política. (ENUNCIADO, §9; FUENTES-Y-LICENCIAS, «Visualización».) | No se fijan costes, cuotas actuales ni política de sustitución del mapa. |

La política precisa de reintentos, umbrales de alerta, objetivos de disponibilidad y supervisión queda pendiente; esta spec no impone valores no prescritos.

## 8. Criterios de aceptación y trazabilidad

Los escenarios siguientes son verificaciones futuras; **no son pruebas ya ejecutadas**. Los identificadores son los existentes en [01-requisitos-aceptacion.md](01-requisitos-aceptacion.md).

| Escenario | Resultado esperado | Requisito existente | Fuente | Evidencia verificable |
|---|---|---|---|---|
| Arranque Compose | `docker compose up --build -d` levanta conjuntamente los cinco servicios backend separados. | REQ-ARCH-001, REQ-DEP-001 | ENUNCIADO, §2; Guía de entregables, §1.2 | Ejecutar desde un entorno reproducible y comprobar los cinco servicios y su conectividad. |
| Persistencia tras reiniciar | Los datos requeridos sobreviven al reinicio conforme a los volúmenes configurados; la documentación explica qué se conserva. | REQ-DEP-001, REQ-DATA-001 | ENUNCIADO, §9; Guía de entregables, §§1.2 y 1.4 | Guardar/consultar datos, reiniciar los servicios y verificar su persistencia. |
| Dependencias no listas | Servicios esperan/reintentan conforme a health checks y se recuperan cuando Kafka/PostgreSQL están disponibles; no se confía solo en el orden de arranque. | REQ-ARCH-003 | ENUNCIADO, §2; README del material base, §3 | Prueba controlada con dependencia retrasada y revisión de estado/logs. |
| Reinicio de consumidor | La reanudación no duplica efectos persistidos. | REQ-NF-002, REQ-TEST-004 | ENUNCIADO, §§7 y 11 | Interrumpir/reiniciar en un punto controlado y comparar registros/offsets. |
| Acceso HTTPS por ngrok | El navegador usa la URL HTTPS del túnel y llega a la API local; no se presenta la conexión como despliegue ya realizado. | REQ-DEP-002, REQ-DEP-003 | ENUNCIADO, §9; CONTRATO-API, «Transporte y contenedores» | Activar un túnel de prueba y verificar una respuesta de API por HTTPS. |
| Preflight CORS y consulta desde origen autorizado | OPTIONS y GET funcionan desde el origen exacto configurado; cabecera de ngrok se admite si el frontend la usa. | REQ-DEP-002 | ENUNCIADO, §9; CONTRATO-API, «Acceso desde Netlify» | Probar preflight y GET desde origen Netlify publicado y desde `http://localhost:4200` en desarrollo. |
| Superficie pública limitada | No se accede públicamente a Kafka, PostgreSQL, rutas administrativas ni secretos por túnel. | REQ-ARCH-002, REQ-DEP-003, REQ-NF-004 | ENUNCIADO, §§2 y 9; CONTRATO-API, «Acceso desde Netlify» | Inspeccionar puertos/rutas expuestos y probar que solo la API prevista es accesible. |
| Sin secretos en frontend/Git | No aparecen credenciales/authtoken en Angular, `config.json` ni historial de archivos de entrega; `.env.example` contiene solo valores de ejemplo no secretos. | REQ-NF-004, REQ-DEP-001, REQ-DOC-001 | ENUNCIADO, §12; CONTRATO-API, «Acceso desde Netlify»; Guía de entregables, §1.3 | Revisión de configuración y repositorio antes de publicar, sin imprimir valores secretos. |
| Publicación SPA Netlify | La configuración entregada de build y publicación se aplica; la web se abre desde otro dispositivo y solicita la API por HTTPS. | REQ-DEP-004, REQ-DOC-001 | ENUNCIADO, §§2, 9 y 12; README del material base, §4 | Build/publicación real y comprobación desde otro dispositivo; no se declara realizada por esta spec. |
| Backend/túnel inaccesible | El panel presenta el error y conserva/identifica la lectura anterior; al restaurar acceso vuelve a consultar. | REQ-F-012, REQ-TEST-005, REQ-DEP-003 | ENUNCIADO, §§8–9 y 11; README del material base, §3 | Cerrar y reabrir el túnel con una lectura previa y comprobar aviso, marca temporal y recuperación. |
| Operación de defensa | README de entrega explica arranque, verificación de servicios/datos, activación de túnel, comprobación del panel y preparación de replay. | REQ-DEP-003, REQ-DOC-001, REQ-DOC-005 | ENUNCIADO, §§11–12; Guía de entregables, §3.2 | Seguir el procedimiento desde un estado detenido y comprobar cada paso durante una prueba futura. |

## 9. Decisiones pendientes

Las filas no convierten alternativas en decisiones aprobadas. Las decisiones generales se pueden planificar; los valores concretos deben resolverse antes de configurar o publicar la parte afectada.

| Decisión pendiente | Requisitos afectados | Cuándo resolver |
|---|---|---|
| Ejecutar ngrok en host o en contenedor | REQ-DEP-002, REQ-DEP-003 | Durante la planificación de despliegue; antes de definir puertos/red y ejecutar el túnel. |
| Listeners/topología Kafka para red Compose y, si aplica, acceso host | REQ-ARCH-001, REQ-DEP-001 | En la decisión de red de despliegue; antes de probar conexiones desde cada contexto. |
| Nombres y valores definitivos de servicios, variables de configuración y puertos | REQ-DEP-001, REQ-DEP-002 | Al implementar Compose/configuración; contrastar nombres con código y documentación. |
| Dominio real de Netlify/ngrok y orígenes CORS | REQ-DEP-002, REQ-DEP-003, REQ-DEP-004 | Tras provisionar los servicios, antes de publicar/configurar el frontend; no inventar dominios. |
| Política concreta de health checks/reintentos distinta del requisito general | REQ-ARCH-003, REQ-DEP-001 | Al implementar los servicios y dependencias; verificar recuperación sin fijar aquí umbrales. |
| Procedimiento exacto de creación/inicialización de PostgreSQL y carga histórica | REQ-DEP-001, REQ-F-004, REQ-DATA-001 | Resolver durante planificación/tarea de persistencia e ingesta; respetar la spec de datos e ingesta. |
| Orden operativo detallado, logs, comprobaciones y parada/recuperación | REQ-DEP-003, REQ-DOC-001, REQ-DOC-005 | Definir al redactar instrucciones de ejecución y validarlo durante la preparación de defensa. |
| Umbral de datos desactualizados y señal de salud | REQ-F-012, REQ-API-003 | Depende de la decisión pendiente de 05 §5; resolver allí antes de implementar el aviso, sin introducir un umbral de UI independiente. |
| Política/periodo concreto de retención Kafka y alcance de volúmenes | REQ-DEP-001 | Antes de configurar Compose y persistencia; la obligación de retención limitada y persistencia está fijada, no sus valores. |

### Límites de evidencia

[VALIDACION.md](../source-of-truth/VALIDACION.md) declara compilación de producción del frontend de apoyo y comprobaciones del catálogo, pero no una publicación real en Netlify, un túnel ngrok del alumnado, ni una API/pipeline implementados y probados. Por tanto, las filas de aceptación de esta spec describen trabajo futuro, no resultados. (VALIDACION.md, «Comprobaciones realizadas» y «Límites de esta comprobación».)

