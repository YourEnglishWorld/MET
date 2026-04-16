# Your English World - MET Quiz

## Configuración Requerida

Este documento explica cómo configurar Google Analytics y Google Apps Script para que el quiz funcione correctamente.

---

## 1. Google Analytics

### Crear propiedad GA4

1. Ve a [Google Analytics](https://analytics.google.com/) e inicia sesión con la cuenta de Diana (`yourenglishworld.dianagranados@gmail.com`)

2. Crea una nueva propiedad GA4:
   - Haz clic en "Administración" (⚙️)
   - En la columna "Propiedad", haz clic en "Crear propiedad"
   - Sigue los pasos del asistente
   - Nombre: "Your English World Quiz"

3. Obtén el Measurement ID:
   - En la propiedad recién creada, ve a "Flujos de datos"
   - Crea un nuevo flujo web
   - Copia el Measurement ID (formato: G-XXXXXXXXXX)

4. Actualiza `index.html`:
   - Reemplaza `G-XXXXXXXXXX` con tu Measurement ID real

---

## 2. Google Apps Script + Google Sheets

### Crear la hoja de cálculo

1. Ve a [Google Sheets](https://sheets.google.com/) e inicia sesión con la cuenta de Diana

2. Crea una nueva hoja:
   - Haz clic en "En blanco" para crear una hoja nueva
   - Nombre: "YEW Quiz - Registros"
   - En la primera fila, agrega estos encabezados:
     ```
     Timestamp | Nombre | Email | Categoría | Acción | Detalle
     ```

3. Comparte la hoja con edición:
   - Haz clic en "Compartir"
   - Agrega el email de Diana
   - Selecciona "Editor"

### Crear el Apps Script

1. Ve a [Google Apps Script](https://script.google.com/)

2. Crea un nuevo proyecto:
   - Haz clic en "Nuevo proyecto"
   - Borra el código existente

3. Copia y pega este código:

```javascript
const SHEET_NAME = 'Sheet1';
const NOTIFICATION_EMAIL = 'yourenglishworld.dianagranados@gmail.com';

const doPost = (e) => {
  const lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    const doc = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = doc.getSheetByName(SHEET_NAME);

    const data = JSON.parse(e.postData.contents);
    
    sheet.appendRow([
      data.timestamp || new Date().toISOString(),
      data.name || '',
      data.email || '',
      data.category || '',
      data.action || '',
      data.detail || ''
    ]);

    if (data.action === 'CONSULTA') {
      sendNotificationEmail(data);
    }

    return ContentService
      .createTextOutput(JSON.stringify({ result: 'success' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', error: error.message }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
};

const sendNotificationEmail = (data) => {
  const subject = `Nueva consulta - YEW Quiz`;
  const body = `
Nueva consulta recibida
========================

Nombre: ${data.name}
Email: ${data.email}
Categoría: ${data.category}

Consulta:
${data.detail}

---
Enviado automáticamente desde Your English World Quiz
  `;
  
  MailApp.sendEmail(NOTIFICATION_EMAIL, subject, body);
};

const doGet = () => {
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
};
```

4. Configurar implementaciones:
   - Haz clic en "Implementaciones" > "Nueva implementación"
   - Tipo: "Aplicación web"
   - Descripción: "YEW Quiz API"
   - Ejecutar como: "Yo"
   - ¿Quién tiene acceso?: "Cualquier persona"

5. Copia la URL de la aplicación web (format: `https://script.google.com/macros/s/XXXXX/exec`)

6. Actualiza `script.js`:
   - Reemplaza `YOUR_GOOGLE_APPS_SCRIPT_URL` con la URL de tu aplicación web

### Conectar con la hoja de cálculo

1. En el Apps Script, haz clic en "Servicios" (+)
2. Agrega "Google Sheets API"
3. Haz clic en "Añadir"

---

## 3. Verificación

Después de configurar todo:

1. Abre el quiz en GitHub Pages
2. Ingresa un nombre y email de prueba
3. Verifica que los datos aparezcan en Google Sheets
4. Envía una consulta de prueba y verifica que llegue el email

---

## Notas Importantes

- **Google Analytics** funciona automáticamente una vez configurado el Measurement ID
- **Google Sheets** guarda todos los registros de actividad
- **Notificaciones por email** se envían solo para consultas
- La primera vez que se ejecuta el Apps Script, puede pedir permisos

¿Necesitas ayuda con algún paso? Contáctame para asistencia.
