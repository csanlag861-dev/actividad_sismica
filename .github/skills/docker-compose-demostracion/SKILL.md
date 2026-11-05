---
name: docker-compose-demostracion
description: 'Prepare or troubleshoot reproducible Docker Compose startup and demonstration for the seismic observatory. Use for its five required services, health/retries, persistent volumes, Kafka networking, ngrok, Netlify, or secret-safe configuration.'
---

# Docker Compose y demostración

## Propósito

Preparar el arranque local reproducible y el acceso de demostración según los límites explícitos del proyecto, sin sumar infraestructura no exigida.

## Cuándo utilizarla

- Al crear o cambiar Dockerfiles, `compose.yaml`, variables de entorno o inicialización de datos.
- Al investigar readiness, listeners, conectividad entre contenedores o persistencia.
- Al documentar el túnel ngrok, publicación Netlify y procedimiento de defensa.

## Procedimiento

1. Inspecciona la configuración existente, nombres de servicio, comandos y versiones; no asumas motor Kafka, imágenes ni dependencias no declaradas.
2. Verifica los cinco servicios separados obligatorios: productor, Kafka, consumidor, PostgreSQL y API FastAPI. Los procesos Python pueden compartir imagen si tienen comandos separados.
3. Configura nombres de servicio para comunicación interna, health/readiness y reintentos. El orden de inicio no demuestra que un servicio esté listo. Revisa listeners Kafka anunciados y evita usar `localhost` entre contenedores.
4. Asegura volúmenes persistentes para PostgreSQL y estado/datos necesarios; define retención limitada Kafka y procedimientos reproducibles de inicialización/carga.
5. Expón solo la API de consulta cuando corresponda. Si ngrok corre en host, usa el puerto API publicado; si corre en la red Compose, conecta al nombre de servicio (`http://api:8000`). No tuneles Kafka ni PostgreSQL.
6. Confirma que Uvicorn atiende en `0.0.0.0:8000` dentro del contenedor. ngrok ofrece HTTPS y gestiona el certificado; no añadas terminación TLS al backend sin requisito.
7. Configura la URL HTTPS pública en la configuración pública del frontend, CORS para el origen exacto de Netlify y desarrollo, y preflight `OPTIONS` según el contrato.
8. Mantén authtoken y contraseñas en entorno local/secreto, nunca en Git ni en el frontend. Comprueba `.env.example` sin valores reales.
9. Ejecuta validación de Compose/build/startup disponible y documenta qué fue local y qué depende de cuenta/servicio externo; no afirmes despliegue remoto no observado.

## Criterios de calidad

- Mantén la topología y configuración en el mínimo que exige la demostración; no agregues servicios, herramientas ni automatizaciones opcionales como parte del cumplimiento.
- Arranque documentado con `docker compose up --build -d` y cinco servicios obligatorios separados.
- Persistencia, readiness/reintentos, retención y comunicación entre servicios son verificables.
- Broker y base no quedan expuestos por el túnel; secretos fuera de archivos públicos/control de versiones.
- El panel y la API son accesibles por el flujo previsto sin exigir cloud permanente, Kubernetes u otras tecnologías extra.
- Se distinguen comprobaciones realizadas de pasos externos pendientes.

## Agentes que la utilizan

Deploy es responsable; Backend la usa para configuración de servicios; Coordinador revisa dependencias y límites; QA Tester verifica arranque y recuperación.

## Referencias

- [Enunciado](../../../docs/source-of-truth/ENUNCIADO.md)
- [Guía del material base](../../../docs/source-of-truth/README.md)
- [FAQ del alumnado](../../../docs/source-of-truth/FAQ-ALUMNADO.md)
- [Contrato de API](../../../docs/source-of-truth/CONTRATO-API.md)
