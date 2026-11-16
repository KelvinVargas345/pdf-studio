# PDF Studio

Aplicación web progresiva (PWA), responsive y local-first. Versión 2.

## Ejecutar
Sirve esta carpeta mediante un servidor HTTP (no abras index.html directamente):
- `python3 -m http.server 8000`
- Abre `http://localhost:8000`

Para instalarla en iPhone: publica la carpeta en un hosting HTTPS, abre la URL en Safari, toca Compartir > Añadir a pantalla de inicio.

## Funciones
- Unir PDF
- Extraer rangos de páginas
- Eliminar páginas
- Rotar páginas
- Imágenes JPG/PNG a PDF
- PDF a JPG (una descarga por página)
- Añadir marca de agua
- Numerar páginas
- Re-guardar PDF con object streams (no garantiza una reducción del tamaño)
- PDF a Word (.docx, con el texto extraído del PDF)
- PDF a Excel (.xlsx, una hoja por página)
- PDF a PowerPoint (.pptx, una diapositiva por página)
- OCR a texto (.txt, español + inglés)
- Proteger PDF con contraseña (cifrado real AES-256)
- Desbloquear PDF (con la contraseña correcta)
- Firmar PDF (firma visual con nombre escrito o imagen)

## Cómo funcionan las conversiones (v2)
Todo se procesa en el navegador, sin subir archivos a un servidor. Los motores se descargan solo cuando se usan, por eso la primera vez que usas una conversión puede tardar unos segundos:
- Word: pdf.js extrae el texto y docx.js (8.5.0) genera el .docx.
- Excel: pdf.js extrae el texto y SheetJS (0.18.5) genera el .xlsx.
- PowerPoint: pdf.js extrae el texto y PptxGenJS (3.12.0) genera el .pptx.
- OCR: Tesseract.js (5.0.0); la primera vez descarga el modelo de idioma (español ~8 MB).
- Proteger / Desbloquear: QPDF compilado a WebAssembly (0.3.0), cifrado AES-256 estándar, el mismo que usan Adobe Acrobat y los lectores del sistema. Sin la contraseña correcta no se puede desbloquear: no hay forma de recuperar una clave olvidada.
- Firmar: pdf-lib dibuja la firma en la página elegida.

## Límites honestos
- Word, Excel y PowerPoint se crean a partir del texto del PDF: se conserva el texto y su orden, no el diseño exacto, las tablas complejas ni las imágenes. Un PDF escaneado (solo imágenes, sin texto) produce un documento vacío: usa OCR primero para leerlo.
- El OCR depende de la calidad de la imagen; con texto borroso o manuscrito puede equivocarse.
- Desbloquear no adivina contraseñas: necesita la clave correcta.
- La compresión real depende del contenido y puede no reducir el tamaño.

Las librerías se cargan desde CDN, por lo que se necesita internet para cargarlas en el primer uso. Para un despliegue de producción, conviene fijar versiones y alojar las dependencias localmente.
