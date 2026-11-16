# PDF Studio

Aplicación web progresiva (PWA), responsive y local-first. Versión 3.6.

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
- v3.2: el service worker ya no sirve páginas viejas de su caché: con internet, cada visita trae lo último del servidor (y lo guarda para sin conexión); sin conexión, sirve la copia guardada. Esto evita que una visita tras un despliegue siga viendo la versión anterior. El intento de arreglar el paso de página con un lienzo nuevo por dibujo NO funcionó: la v3.2 en vivo seguía dejando el lienzo en blanco al pasar a la página 2 y al volver (verificado 18:11 AST).
- v3.3: la vista previa ya no vuelve a dibujar una página: cada página se dibuja una sola vez y su lienzo queda guardado (máximo 12 páginas en memoria); al pasar de página se muestra el lienzo ya dibujado. El patrón que se observó en vivo es que solo el primer dibujo de cada documento pinta algo y los siguientes salen en blanco aunque no den error, así que no redibujar es el arreglo. Al cambiar de archivo se limpia la memoria de lienzos. NO funcionó en vivo (verificado 18:29 AST): la página 1 se dibuja, al pasar a la 2 el lienzo queda en blanco y al volver tampoco se ve la 1, con los dos PDFs de prueba; ni siquiera el lienzo guardado de la página 1 se ve al volver.
- v3.4 (diagnóstico): al cargar un PDF, la vista previa dibuja TODAS sus páginas seguidas en un bucle cerrado (como hace el conversor PDF a JPG, que sí dibuja todas las páginas) y guarda cada lienzo; al navegar solo se muestra el lienzo guardado. Debajo de la vista previa se muestra una línea con los píxeles dibujados de cada página («Dibujo previo: Pág 1: N px · Pág 2: N px»), para saber si el dibujo pinta o no. El pie de la app muestra la versión («PDF Studio v3.4») para confirmar qué código está corriendo en vivo. HALLAZGO 18:35 AST: los dibujos SÍ pintan (Pág 1: 978 px, Pág 2: 458 px; segundo PDF 1097/497 px) y la navegación funciona, pero el lienzo no se repinta cuando se cambia de página con la vista abajo en la pantalla (se ve en blanco hasta que se sube el desplazamiento y el contenido reaparece). El fallo nunca fue el dibujo: es el repintado del lienzo al cambiar de página.
- v3.5: la vista previa ya no muestra un lienzo: cada página se dibuja una sola vez al cargar (como en v3.4) y se guarda como imagen (data URL), y al navegar se muestra esa imagen en el elemento de imagen, que se repinta siempre. La línea de diagnóstico con los píxeles por página sigue visible en esta versión para confirmar el dibujo en vivo. NO funcionó en el navegador de pruebas (verificado 18:42 AST): la imagen también queda en blanco al cambiar de página con la vista bajada y solo aparece al subir el desplazamiento. Puede ser un artefacto del navegador de pruebas y no un fallo real; la prueba decisiva es en el iPhone (Safari).
- v3.6 (preparada, sin publicar): (1) tras cambiar de página, la app mueve el desplazamiento 1 px y lo devuelve (en dos frames) para forzar el repintado de la vista previa, que es lo mismo que hace subir el desplazamiento a mano; (2) la entrada de archivos ya no está oculta con `display:none` (en iPhone eso impedía abrir el selector de archivos al tocar la zona de subida) — ahora está invisible pero presente, y la zona funciona con la etiqueta nativa; (3) el botón Procesar queda deshabilitado sin archivo y, si se pulsa sin archivo, dice «Selecciona al menos un archivo primero» en vez del error genérico; (4) si un motor no carga (sin conexión o CDN bloqueado), el mensaje lo dice claro en vez del error genérico; (5) la página ya no se puede desplazar de lado en el celular y el texto de la zona de subida se parte bien; (6) el service worker pasa a v7 (caché limpia). Publicarla tras el reporte del iPhone 18:46 AST (no procesaba en ninguna pestaña, no se veía el archivo cargado, Procesar daba el error genérico y el texto de subida se veía cortado; el pie de la captura no tenía etiqueta de versión, o sea código viejo en caché).

## Límites honestos
- Word, Excel y PowerPoint se crean a partir del texto del PDF: se conserva el texto y su orden, no el diseño exacto, las tablas complejas ni las imágenes. Un PDF escaneado (solo imágenes, sin texto) produce un documento vacío: usa OCR primero para leerlo.
- El OCR depende de la calidad de la imagen; con texto borroso o manuscrito puede equivocarse.
- Desbloquear no adivina contraseñas: necesita la clave correcta.
- La compresión real depende del contenido y puede no reducir el tamaño.
- HEIC fuera de Safari (por ejemplo, en Chrome de escritorio) no se puede decodificar: la app avisa y sugiere compartir la foto como JPG desde el iPhone o activar «Más compatible» en Ajustes > Cámara > Formatos.
- La vista previa de Word, Excel, PowerPoint y texto OCR no se puede dibujar en la página: se muestra un aviso y el archivo se descarga igual.

Las librerías se cargan desde CDN, por lo que se necesita internet para cargarlas en el primer uso. Para un despliegue de producción, conviene fijar versiones y alojar las dependencias localmente.
