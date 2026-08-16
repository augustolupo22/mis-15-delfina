# Tarjeta de Invitación - Mis 15

Tarjeta digital de invitación para quinceañera con confirmación de asistencia integrada a Google Sheets.

## Estructura del Proyecto

```
TARJETA/
├── index.html          ← Archivo principal (todo en uno)
├── assets/             ← Carpeta de imágenes
│   ├── hero.jpg        ← Foto principal de portada
│   ├── foto1.jpg       ← Fotos de la galería
│   ├── foto2.jpg
│   ├── foto3.jpg
│   ├── foto4.jpg
│   ├── foto5.jpg
│   └── foto6.jpg
└── music.mp3           ← Archivo de música de fondo (opcional)
```

## Personalización

### Cambiar nombre, fecha y datos del evento

Abrí `index.html` y buscá los textos para reemplazar:

- **Nombre:** Buscá `Delfina` y reemplazalo
- **Fecha del contador:** Modificá la línea `const EVENT_DATE = new Date(2026, 9, 3, 21, 0, 0);`
- **Datos del salón:** Editá los textos en la sección "Ubicación y Horarios"
- **Alias bancario:** Buscá `DELFINA.POGGI11` y cambialo
- **Hashtag:** Buscá `#Mis15Delfina`

### Cambiar colores

En la sección `tailwind.config` dentro del `<head>`:

```javascript
colors: {
    nude: { ... },    // Tonos piel/beige
    gold: { ... },    // Dorados
    olive: { ... }    // Verdes oliva
}
```

### Agregar fotos

1. Poné tus imágenes en la carpeta `assets/`
2. En `index.html`, reemplazá los `src="assets/foto1.jpg"` etc.

---

## Integración con Google Sheets

### Paso 1: Crear la planilla

1. Andá a [Google Sheets](https://sheets.google.com)
2. Creá una nueva planilla
3. El script creará automáticamente una pestaña llamada `Respuestas` y estos encabezados:

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| **Nombre** | **Asistencia** | **Cantidad de acompañantes** | **Nombres de acompañantes** | **Menú** | **Menú acompañantes** | **Mensaje** | **Canción sugerida** |

### Paso 2: Crear el Apps Script

1. En tu planilla, andá a **Extensiones > Apps Script**
2. Borrá todo el código que aparezca
3. Pegá este código:

```javascript
function doPost(e) {
  try {
    const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = spreadsheet.getSheetByName('Respuestas') || spreadsheet.insertSheet('Respuestas');
    const data = JSON.parse(e.postData.contents);

    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        'Nombre', 'Asistencia', 'Cantidad de acompañantes', 'Nombres de acompañantes',
        'Menú', 'Menú acompañantes', 'Mensaje', 'Canción sugerida'
      ]);
    }
    
    // Si es una canción sugerida, se guarda en la columna H.
    if (data.tipo === 'cancion') {
      sheet.appendRow([
        '', '', '', '', '', '', '', data.cancion
      ]);
      
      return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
        .setMimeType(ContentService.MimeType.JSON);
    }
    
    // Si es confirmación de asistencia
    sheet.appendRow([
      data.nombre,
      data.asistencia,
      data.acompaniantes,
      data.nombresAcompanantes,
      data.menu,
      data.menusAcompanantes,
      data.mensaje,
      ''
    ]);
    
    return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({status: 'error', message: error.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService.createTextOutput('OK').setMimeType(ContentService.MimeType.TEXT);
}
```

4. Guardá con el nombre "Recibir RSVP"
5. Clic en el ícono de "Proyectar" (Deploy) > **Nuevo despliegue**
6. Tipo: **API Web**
7. Acceso: **Cualquier persona**
8. Clic en **Desplegar**
9. **Copiá la URL que te dé** (algo como `https://script.google.com/macros/s/AKfycbx.../exec`)

### Paso 3: Pegar la URL en tu web

En `index.html`, buscá esta línea y pegá tu URL:

```javascript
const GOOGLE_SHEETS_WEBHOOK_URL = 'TU_URL_DE_GOOGLE_APPS_SCRIPT_AQUI';
```

### Paso 4: Probar

1. Abrí tu tarjeta en el navegador
2. Completá el formulario
3. Verificá que aparezcan los datos en tu planilla de Google Sheets

---

## Despliegue en Vercel

### Opción 1: Desde GitHub

1. Creá un repositorio en GitHub
2. Subí todos los archivos
3. Andá a [vercel.com](https://vercel.com)
4. Importá el repositorio
5. Deploy automático

### Opción 2: Desde Vercel CLI

```bash
npm i -g vercel
vercel
```

---

## Notas

- La música de fondo requiere interacción del usuario para empezar a reproducirse (política de navegadores)
- Las fotos deben ser cuadradas para mejor resultado en la galería
- El formulario funciona sin conexión (guarda localmente) mientras tanto
- Recordá cambiar la URL del webhook antes de publicar
