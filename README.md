# 👨‍👩‍👧‍👦 Custodia Familiar

Aplicación web progresiva (PWA) para la gestión y coordinación de la custodia compartida entre padres, con calendario interactivo, sistema de solicitudes y notificaciones en tiempo real.

## ✨ Características

### 📅 Calendario Interactivo
- Vista mensual con navegación intuitiva
- Visualización por roles (Padre/Madre) con código de colores
- Indicador del día actual
- Gestos swipe para cambiar de mes
- Asignación rápida de días de custodia

### 📨 Sistema de Solicitudes
- Envío de solicitudes de cambio de custodia
- Vista centralizada de todas las solicitudes
- Estados: Pendiente, Aceptada, Rechazada
- Aceptar/Rechazar solicitudes de la otra parte
- Cancelar solicitudes propias antes de ser procesadas
- Notificaciones automáticas por Telegram

### 📋 Lista de Eventos
- Vista cronológica de todos los días con custodia asignada
- Filtrado automático de eventos futuros
- Indicador visual del día actual
- Navegación rápida desde la lista al calendario

### 🔔 Notificaciones
- Integración con Telegram mediante Cloud Functions
- Notificaciones instantáneas para:
  - Nuevas solicitudes
  - Solicitudes aceptadas/rechazadas
  - Solicitudes canceladas
  - Asignaciones y eliminaciones de custodia

### 💾 Sincronización en la Nube
- Base de datos en tiempo real con Firebase Firestore
- Sincronización automática entre dispositivos
- Autenticación anónima para acceso rápido
- Persistencia de datos segura

## 🚀 Tecnologías Utilizadas

- **HTML5 + CSS3**: Interfaz responsive y moderna
- **JavaScript Vanilla**: Sin frameworks pesados
- **Firebase Firestore**: Base de datos en tiempo real
- **Firebase Auth**: Autenticación de usuarios
- **Google Cloud Functions**: Backend serverless para Telegram
- **Telegram Bot API**: Sistema de notificaciones
- **Lucide Icons**: Iconografía moderna
- **Google Fonts**: Tipografía (Google Sans + Inter)

## ⚙️ Configuración

### Firebase

El proyecto utiliza Firebase para la sincronización de datos. La configuración está en el archivo HTML:

```javascript
const firebaseConfig = {
    apiKey: "AIzaSyAwgk4_l0zvaCUCgm1CjqqqXjJHr5KPBjc",
    authDomain: "custodia-familiar.firebaseapp.com",
    projectId: "custodia-familiar",
    storageBucket: "custodia-familiar.firebasestorage.app",
    messagingSenderId: "1038952432258",
    appId: "1:1038952432258:web:7608d6a48d8ad8308a6a9b"
};
```

#### Configurar tu propio proyecto Firebase:

1. Crea un proyecto en [Firebase Console](https://console.firebase.google.com/)
2. Habilita **Firestore Database** en modo de prueba
3. Habilita **Authentication** con el proveedor "Anónimo"
4. Copia las credenciales de tu proyecto
5. Reemplaza `firebaseConfig` en el archivo HTML con tus credenciales

#### Estructura de datos en Firestore:

```
users/{userId}
  ├── custodyData: {
  │     "2026-1-15": "father",
  │     "2026-1-16": "mother",
  │     ...
  │   }
  ├── pendingRequests: [
  │     {
  │       year: 2026,
  │       month: 1,
  │       day: 20,
  │       requestedBy: "father",
  │       proposedCustodian: "mother",
  │       status: "pending"
  │     }
  │   ]
  └── lastModified: timestamp
```

### Telegram Bot

Las notificaciones se envían mediante una Cloud Function de Google Cloud.

**URL del endpoint:**
```
https://sendtelegram-jd4jwaesba-uc.a.run.app
```

#### Configurar tu propio bot de Telegram:

1. Crea un bot con [@BotFather](https://t.me/botfather) en Telegram
2. Obtén el token del bot
3. Obtén tu Chat ID (puedes usar [@userinfobot](https://t.me/userinfobot))
4. Crea una Cloud Function en Google Cloud:

```javascript
const functions = require('@google-cloud/functions-framework');
const fetch = require('node-fetch');

const TELEGRAM_BOT_TOKEN = 'TU_BOT_TOKEN';
const TELEGRAM_CHAT_ID = 'TU_CHAT_ID';

functions.http('sendTelegram', async (req, res) => {
    res.set('Access-Control-Allow-Origin', '*');

    if (req.method === 'OPTIONS') {
        res.set('Access-Control-Allow-Methods', 'POST');
        res.set('Access-Control-Allow-Headers', 'Content-Type');
        return res.status(204).send('');
    }

    const { text } = req.body;

    const url = `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`;

    try {
        await fetch(url, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                chat_id: TELEGRAM_CHAT_ID,
                text: text,
                parse_mode: 'HTML'
            })
        });

        res.status(200).send({ success: true });
    } catch (error) {
        res.status(500).send({ error: error.message });
    }
});
```

5. Despliega la función y actualiza la URL en el código HTML

## 📦 Instalación

### Opción 1: Uso directo
1. Descarga el archivo `Custodia_Familiar_v2.html`
2. Ábrelo en tu navegador web
3. ¡Listo! La aplicación funcionará inmediatamente

### Opción 2: Configuración personalizada
1. Clona este repositorio:
   ```bash
   git clone https://github.com/tu-usuario/custodia-familiar.git
   cd custodia-familiar
   ```

2. Configura Firebase (ver sección de configuración)

3. Configura Telegram Bot (opcional, ver sección de configuración)

4. Abre el archivo HTML en tu navegador

## 📱 Uso

### Cambiar de Rol
- Usa los botones en la cabecera para alternar entre **Padre** y **Madre**
- Cada rol tiene una vista personalizada del calendario

### Asignar Custodia
1. Haz clic en un día vacío del calendario
2. Selecciona el custodio (Padre o Madre)
3. Confirma la asignación

### Solicitar Cambios
1. Haz clic en un día ya asignado
2. Cambia el custodio en el selector
3. Presiona "Solicitar cambio"
4. La otra parte recibirá una notificación

### Gestionar Solicitudes
1. Ve a la pestaña "Solicitudes" en la navegación inferior
2. Revisa las solicitudes pendientes
3. **Si eres el receptor**: Acepta o rechaza
4. **Si eres el solicitante**: Puedes cancelar antes de que sea procesada

### Ver Eventos Programados
1. Ve a la pestaña "Eventos"
2. Visualiza todos los días de custodia confirmados
3. Haz clic en cualquier evento para ir al calendario

## 🎨 Personalización

### Colores
Puedes personalizar los colores en las variables CSS al inicio del archivo:

```css
:root {
    --father-primary: #0ea5e9;  /* Color del padre */
    --mother-primary: #f472b6;  /* Color de la madre */
    --text-main: #0f172a;       /* Color del texto */
}
```

### Idioma
Los textos están en español. Para cambiar el idioma, modifica:
- Array `monthNames` para los nombres de meses
- Array de días de la semana en `.weekdays`
- Textos de botones y mensajes en el HTML

## 🔒 Seguridad

- La autenticación es anónima por defecto
- Los datos se almacenan por usuario único
- Firestore Rules recomendadas:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## 📄 Licencia

Este proyecto es de código abierto y está disponible bajo la licencia MIT.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Haz fork del proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## 📞 Soporte

Si encuentras algún problema o tienes sugerencias:
- Abre un [Issue](https://github.com/tu-usuario/custodia-familiar/issues)
- Envía un Pull Request con mejoras

## ⭐ Agradecimientos

- [Lucide Icons](https://lucide.dev/) por los iconos
- [Firebase](https://firebase.google.com/) por la infraestructura backend
- [Telegram](https://telegram.org/) por la API de mensajería
- [Google Fonts](https://fonts.google.com/) por las tipografías

---

**Desarrollado con ❤️ para facilitar la coordinación familiar**
