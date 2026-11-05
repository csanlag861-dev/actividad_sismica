---
name: angular-leaflet-observatorio
description: 'Develop or troubleshoot the seismic observatory Angular and Leaflet panel. Use for API integration, GeoJSON layers, map coordinates, filters, event tables, charts, accessible status, or frontend build behavior.'
---

# Angular y Leaflet del Observatorio

## Propósito

Completar y mantener el panel existente preservando sus datos, mapa base, modos, convenciones de coordenadas y contrato con la API.

## Cuándo utilizarla

- Al modificar la aplicación Angular, el mapa Leaflet o la conexión a la API.
- Al añadir filtros, tabla, análisis, leyenda, estado o tratamiento de respuestas.
- Al depurar capas duplicadas, coordenadas desplazadas, errores de carga o build.

## Procedimiento

1. Inspecciona componentes, configuración, dependencias y patrones existentes antes de editar. La base actual es SPA Angular 20.3, ngx-leaflet 20.0 y Leaflet 1.9.4; verifica `package.json` antes de cambiar versiones.
2. Consulta `CONTRATO-API.md`. El panel final consulta la API y datos procesados; el modo demo con cuatro eventos ficticios solo verifica la base cartográfica.
3. Mantén claro el orden GeoJSON `[longitud, latitud, profundidad]` frente a Leaflet manual `[latitud, longitud]`. No hagas clasificación de país en el navegador; consume la clasificación backend.
4. Actualiza la capa de eventos como una única versión por refresco, sin acumular puntos. Construye/valida la nueva respuesta antes de reemplazar la anterior para conservar la última lectura ante error.
5. Mantén controles y resultados sincronizados entre mapa, tabla y estadísticas. Incluye «Sin país asignado», nulos/ND, truncamiento y modo replay cuando los metadatos existan.
6. Comunica por separado última consulta exitosa del panel, generación de respuesta y última ingesta. Presenta errores/desactualización claramente, evita peticiones solapadas y cancela temporizadores/peticiones al destruir componentes siguiendo los patrones del proyecto.
7. Mantén accesibilidad de la tabla y controles, atribución de OpenStreetMap/Natural Earth y contenido de popup como texto, no HTML no confiable.
8. No añadas biblioteca de gráficos u otra dependencia sin justificarla frente a la petición y al paquete existente.
9. Ejecuta la comprobación más pequeña configurada y el build frontend pertinente; reporta el resultado observado.

## Criterios de calidad

- API es la fuente de datos de la entrega; demo está identificada como ficticia.
- Coordenadas, filtros, nulos, estado de frescura y truncamiento se representan sin alterar significado.
- Las capas no se duplican y un fallo no borra la última lectura válida.
- El panel es accesible y conserva atribuciones/licencias cartográficas.
- Build y pruebas reportados son los realmente ejecutados.

## Agentes que la utilizan

Frontend es responsable de la implementación; Backend y Coordinador la consultan al acordar contratos; QA Tester la usa para pruebas visibles y de integración.

## Referencias

- [Contrato de API](../../../docs/source-of-truth/CONTRATO-API.md)
- [Guía del material base](../../../docs/source-of-truth/README.md)
- [Fuentes y licencias](../../../docs/source-of-truth/FUENTES-Y-LICENCIAS.md)
- [Validación de la base](../../../docs/source-of-truth/VALIDACION.md)
