# 👨‍👩‍👧‍👦 Custodia Familiar

Aplicación web progresiva (PWA) para la gestión y coordinación de la custodia compartida entre padres. Sistema completo con calendario interactivo basado en semanas alternas, gestión de solicitudes de cambio, eventos programados y notificaciones en tiempo real.

## ✨ Características Principales

### 🔐 Sistema de Autenticación
- **Login y Registro** con email y contraseña
- **Onboarding inicial** para configurar perfil (rol: Padre/Madre y nombre del niño/a)
- **Autenticación segura** con Firebase Authentication
- **Sesiones persistentes** entre dispositivos

### 📅 Calendario Inteligente
- **Sistema de semanas alternas** automático desde el 5 de enero de 2026
- **Visualización mensual** con código de colores por custodio
- **Indicador del día actual** destacado
- **Gestos swipe** para navegación rápida entre meses
- **Overrides manuales** para modificar días específicos
- **Eventos visuales** con iconos de categorías

### 📨 Gestión de Solicitudes
- **Envío de solicitudes** de cambio de custodia con rango de fechas
- **Vista centralizada** de todas las solicitudes
- **Estados**: Pendiente, Aceptada, Rechazada
- **Acciones rápidas**: Aceptar/Rechazar solicitudes de la otra parte
- **Notificaciones automáticas** por Telegram para cada acción
- **Historial completo** de solicitudes

### 📋 Eventos Programados
- **Lista cronológica** de todas las actividades del calendario
- **8 categorías de eventos**: Médico, Dentista, Escuela, Deporte, Cumpleaños, Parque, General, Vacaciones
- **Iconos visuales** en el calendario
- **Navegación directa** desde la lista al día en el calendario
- **Eliminar eventos** con un clic

### 🔔 Notificaciones en Tiempo Real
- **Integración con Telegram** mediante Cloud Functions
- Notificaciones automáticas para:
  - ✅ Nuevas solicitudes de cambio
  - ✅ Solicitudes aceptadas o rechazadas
  - ✅ Asignación de custodia
  - ✅ Eventos importantes

### 💾 Sincronización Automática
- **Base de datos en tiempo real** con Firebase Firestore
- **Sincronización instantánea** entre dispositivos
- **Persistencia de datos** en la nube
- **Sin pérdida de información**: Los datos existentes se conservan y cargan automáticamente

## 🚀 Tecnologías

| Tecnología | Uso |
|-----------|-----|
| **HTML5 + CSS3** | Interfaz responsive y moderna |
| **JavaScript Vanilla** | Sin dependencias pesadas |
| **Firebase Firestore** | Base de datos NoSQL en tiempo real |
| **Firebase Auth** | Autenticación de usuarios |
| **Google Cloud Functions** | Backend serverless para Telegram |
| **Telegram Bot API** | Sistema de notificaciones |
| **Lucide Icons** | Iconografía moderna |
| **Google Fonts** | Tipografía (Google Sans + Inter) |

## 📦 Estructura de Datos en Firebase

### Colecciones de Firestore

```
users/{userId}
  ├── email: string
  ├── role: "padre" | "madre"
  ├── childName: string
  └── createdAt: timestamp

overrides/{YYYY-MM-DD}
  └── owner: "padre" | "madre"

events/{eventId}
  ├── date: "YYYY-MM-DD"
  ├── title: string
  ├── type: "medico" | "dentista" | "escuela" | "deporte" | "cumple" | "parque" | "general" | "vacaciones"
  └── createdAt: timestamp

requests/{requestId}
  ├── requesterUid: string
  ├── requesterRole: "padre" | "madre"
  ├── startDate: "YYYY-MM-DD"
  ├── endDate: "YYYY-MM-DD"
  ├── reason: string
  ├── status: "pendiente" | "aceptado" | "rechazado"
  └── createdAt: timestamp
```

## ⚙️ Configuración

### 1. Firebase Setup

El proyecto ya está configurado con estas credenciales:

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

#### Reglas de seguridad recomendadas para Firestore:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Usuarios solo pueden leer/escribir sus propios datos
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // Overrides y eventos compartidos
    match /overrides/{document=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }

    match /events/{document=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }

    // Solicitudes accesibles por todos los usuarios autenticados
    match /requests/{document=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null;
    }
  }
}
```

### 2. Telegram Bot

**URL de la Cloud Function actual:**
```
https://sendtelegram-jd4jwaesba-uc.a.run.app
```

#### Para configurar tu propio bot:

1. Crea un bot con [@BotFather](https://t.me/botfather)
2. Obtén el token del bot
3. Obtén tu Chat ID usando [@userinfobot](https://t.me/userinfobot)
4. Crea una Cloud Function en Google Cloud Platform:

```javascript
const functions = require('@google-cloud/functions-framework');
const fetch = require('node-fetch');

const TELEGRAM_BOT_TOKEN = 'TU_BOT_TOKEN_AQUI';
const TELEGRAM_CHAT_ID = 'TU_CHAT_ID_AQUI';

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

5. Despliega y actualiza la URL en el archivo HTML

## 🎯 Instalación y Uso

### Instalación Rápida

1. Descarga el archivo `Custodia_Familiar_Final.html`
2. Ábrelo en tu navegador web
3. Crea una cuenta o inicia sesión
4. Configura tu perfil (Padre/Madre y nombre del niño/a)
5. ¡Listo! Todos tus datos existentes se cargarán automáticamente

### Uso de las 3 Vistas

#### 📅 Vista Calendario
- **Ver custodia**: El calendario muestra automáticamente el patrón de semanas alternas
- **Modificar días**: Haz clic en cualquier día para ver detalles y añadir eventos
- **Navegación**: Usa las flechas o gestos swipe para cambiar de mes
- **Eventos**: Los iconos de colores indican actividades programadas

#### 📨 Vista Solicitudes
- **Crear solicitud**: Rellena el formulario con fechas y motivo
- **Gestionar solicitudes**: 
  - Si eres el receptor → Acepta o rechaza
  - Si eres el solicitante → Espera respuesta
- **Historial**: Ve todas las solicitudes pendientes en tiempo real

#### 📋 Vista Eventos
- **Ver todos los eventos**: Lista ordenada cronológicamente
- **Filtro automático**: Solo muestra eventos futuros y actuales
- **Navegación rápida**: Haz clic en cualquier evento para ir a ese día en el calendario

## 🎨 Personalización

### Cambiar el patrón de semanas

Modifica la fecha base en el código:

```javascript
const BASE_MONDAY = new Date(2026, 0, 5, 12, 0, 0); // 5 de enero de 2026
```

### Añadir nuevas categorías de eventos

1. Añade el tipo al objeto `ICON_MAP`:
```javascript
const ICON_MAP = {
    medico: "stethoscope",
    tuCategoria: "tu-icono-lucide"
};
```

2. Añade el botón en el modal:
```html
<button class="icon-btn evt-tuCategoria" onclick="selectIcon('tuCategoria', this)">
    <i data-lucide="tu-icono"></i>
</button>
```

3. Añade el estilo CSS:
```css
.evt-tuCategoria { background: #tu-color; }
```

## 🔒 Seguridad y Privacidad

- ✅ Autenticación requerida para todos los datos
- ✅ Datos encriptados en tránsito (HTTPS)
- ✅ Reglas de seguridad de Firestore configurables
- ✅ Sin tracking de terceros
- ✅ Datos almacenados en servidores de Google (Firebase)

## 📱 Características Técnicas

- **Responsive Design**: Funciona en móviles, tablets y escritorio
- **PWA Ready**: Se puede instalar como aplicación
- **Offline-first**: Datos cacheados localmente
- **Real-time**: Actualizaciones instantáneas entre dispositivos
- **Touch Gestures**: Swipe para navegación en móviles

## 🐛 Solución de Problemas

### No veo mis datos
- Verifica que hayas iniciado sesión con la cuenta correcta
- Revisa la consola del navegador (F12) para errores de Firebase
- Asegúrate de tener conexión a internet

### No llegan notificaciones de Telegram
- Verifica que la URL de la Cloud Function sea correcta
- Comprueba que el bot esté activo en Telegram
- Revisa los logs de la Cloud Function en Google Cloud Console

### Las solicitudes no se actualizan
- Comprueba las reglas de seguridad de Firestore
- Verifica que ambos usuarios tengan los permisos correctos
- Refresca la página para forzar sincronización

## 📄 Licencia

Este proyecto es de código abierto bajo licencia MIT.

## 🤝 Contribuciones

¡Las contribuciones son bienvenidas!

1. Fork el proyecto
2. Crea tu rama de feature (`git checkout -b feature/MejoraNueva`)
3. Commit tus cambios (`git commit -m 'Añadir nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/MejoraNueva`)
5. Abre un Pull Request

## 📞 Soporte

- 🐛 **Issues**: [GitHub Issues](https://github.com/tu-usuario/custodia-familiar/issues)
- 💬 **Preguntas**: Abre una discusión en GitHub
- 📧 **Email**: Para consultas privadas

## ⭐ Agradecimientos

- [Firebase](https://firebase.google.com/) - Backend y autenticación
- [Lucide Icons](https://lucide.dev/) - Sistema de iconos
- [Google Fonts](https://fonts.google.com/) - Tipografías
- [Telegram](https://telegram.org/) - Sistema de notificaciones

---

**Desarrollado con ❤️ para facilitar la coordinación familiar**

*Última actualización: Febrero 2026*
