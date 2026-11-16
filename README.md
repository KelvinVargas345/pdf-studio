# PDF Studio

Aplicación web progresiva (PWA), responsive y local-first.

## Ejecutar
Sirve esta carpeta mediante un servidor HTTP (no abras index.html directamente):
- `python3 -m http.server 8000`
- Abre `http://localhost:8000`

Para instalarla en iPhone: publica la carpeta en un hosting HTTPS, abre la URL en Safari, toca Compartir > Añadir a pantalla de inicio.

## Funciones implementadas
- Unir PDF
- Extraer rangos de páginas
- Eliminar páginas
- Rotar páginas
- Imágenes JPG/PNG a PDF
- PDF a JPG (una descarga por página)
- Añadir marca de agua
- Numerar páginas
- Re-guardar PDF con object streams (no garantiza una reducción del tamaño)

## Límites importantes
Esta primera versión procesa archivos en el navegador y no los sube a un servidor. La compresión real depende del contenido y puede no reducir el tamaño. PDF-lib no implementa cifrado PDF; por eso no se ofrece una falsa protección por contraseña. PDF a Word/Excel/PowerPoint, OCR, desbloqueo de PDF y firma criptográfica requieren integrar motores especializados y pruebas adicionales. No se presentan como funciones terminadas.

Las librerías PDF-lib y PDF.js se cargan desde CDN, por lo que se necesita internet para cargarlas en el primer inicio. Para un despliegue de producción, conviene fijar versiones y alojar las dependencias localmente.
