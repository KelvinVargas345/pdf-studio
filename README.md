# PDF Studio

Aplicación web progresiva (PWA), responsive y local-first. Versión 3.2.

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
- Vista previa del archivo elegido (primera página del PDF, con botones para pasar de página; miniatura de la imagen) y del resultado generado, antes de descargar
- Márgenes pensados para celular: respeta el área segura del iPhone (notch e indicador de inicio) y el botón de procesar ocupa todo el ancho
- Imágenes JPG/PNG/HEIC a PDF (una por página, ajustada a A4)
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
- Imágenes a PDF (v3): el navegador decodifica la imagen (JPG progresivo o CMYK, PNG, WebP y HEIC en Safari), corrige la orientación EXIF, la reduce a un máximo de 2000 px y la coloca en una página A4. Así ya no falla con las fotos del iPhone, que llegan en HEIC o sin tipo MIME reconocido.
- Vista previa (v3): al elegir archivos se muestra la primera página del PDF (con anterior/siguiente) o la imagen; tras procesar, se muestra la primera página del PDF generado o la imagen resultante.
- v3.1: el CSS ya no anula el atributo `hidden` (se veía contenido viejo y un icono de imagen rota) y la zona de subida ya no se corta ni se solapa con la descripción. Márgenes para celular: respeta el área segura del iPhone (notch e indicador de inicio) y el botón de procesar ocupa todo el ancho. El paso de página de la vista previa seguía sin dibujar (lienzo en blanco al pasar a la página 2 y al volver): la v3.1 no lo arregló.
- v3.2: la vista previa dibuja cada página en un lienzo nuevo (el lienzo reutilizado quedaba en blanco tras el primer dibujo), así que pasar de página y volver ya redibuja. El service worker ya no sirve páginas viejas de su caché: con internet, cada visita trae lo último del servidor (y lo guarda para sin conexión); sin conexión, sirve la copia guardada. Esto evita que una visita tras un despliegue siga viendo la versión anterior.

## Límites honestos
- Word, Excel y PowerPoint se crean a partir del texto del PDF: se conserva el texto y su orden, no el diseño exacto, las tablas complejas ni las imágenes. Un PDF escaneado (solo imágenes, sin texto) produce un documento vacío: usa OCR primero para leerlo.
- El OCR depende de la calidad de la imagen; con texto borroso o manuscrito puede equivocarse.
- Desbloquear no adivina contraseñas: necesita la clave correcta.
- La compresión real depende del contenido y puede no reducir el tamaño.
- HEIC fuera de Safari (por ejemplo, en Chrome de escritorio) no se puede decodificar: la app avisa y sugiere compartir la foto como JPG desde el iPhone o activar «Más compatible» en Ajustes > Cámara > Formatos.
- La vista previa de Word, Excel, PowerPoint y texto OCR no se puede dibujar en la página: se muestra un aviso y el archivo se descarga igual.

Las librerías se cargan desde CDN, por lo que se necesita internet para cargarlas en el primer uso. Para un despliegue de producción, conviene fijar versiones y alojar las dependencias localmente.
