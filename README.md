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
- **Diseño responsive** optimizado para móviles

### 📨 Gestión de Solicitudes
- **Envío de solicitudes** de cambio de custodia con rango de fechas
- **Vista centralizada** de todas las solicitudes
- **Estados**: Pendiente, Aceptada, Rechazada
- **Acciones rápidas**: Aceptar/Rechazar solicitudes de la otra parte
- **🆕 CANCELAR SOLICITUDES**: Cancela tus propias solicitudes antes de que sean procesadas
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
  - ✅ Solicitudes canceladas
  - ✅ Asignación de custodia
  - ✅ Eventos importantes

### 💾 Sincronización Automática
- **Base de datos en tiempo real** con Firebase Firestore
- **Sincronización instantánea** entre dispositivos
- **Persistencia de datos** en la nube
- **Sin pérdida de información**: Los datos existentes se conservan y cargan automáticamente

### 🎨 Diseño Moderno
- **Tema claro** con colores suaves y agradables
- **Gradiente morado/violeta** como fondo
- **Tarjetas blancas** con sombras sutiles
- **Alta legibilidad** con contraste adecuado
- **Completamente responsive** - se adapta perfectamente a móviles, tablets y escritorio

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
      // Permite eliminar solicitudes (para la función de cancelar)
      allow delete: if request.auth != null;
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

5. Despliega la función y actualiza la URL en el archivo HTML

## 🎯 Instalación y Uso

### Instalación Rápida

1. Descarga el archivo `Custodia_Familiar_Light.html`
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
- **Responsive**: El calendario se adapta perfectamente a pantallas móviles

#### 📨 Vista Solicitudes
- **Crear solicitud**: Rellena el formulario con fechas y motivo
- **Gestionar solicitudes**: 
  - Si eres el receptor → Acepta o rechaza
  - Si eres el solicitante → **🆕 Puedes cancelar la solicitud**
- **Cancelar**: Botón "Cancelar solicitud" visible solo en tus propias solicitudes pendientes
- **Historial**: Ve todas las solicitudes pendientes en tiempo real

#### 📋 Vista Eventos
- **Ver todos los eventos**: Lista ordenada cronológicamente
- **Filtro automático**: Solo muestra eventos futuros y actuales
- **Navegación rápida**: Haz clic en cualquier evento para ir a ese día en el calendario

## 🆕 Nueva Función: Cancelar Solicitudes

### ¿Cómo funciona?

1. **Solo tus solicitudes**: El botón de cancelar aparece únicamente en las solicitudes que TÚ has creado
2. **Solo pendientes**: Solo puedes cancelar solicitudes que aún no han sido aceptadas o rechazadas
3. **Confirmación**: Se pide confirmación antes de cancelar
4. **Notificación**: Se envía una notificación automática por Telegram cuando cancelas
5. **Eliminación definitiva**: La solicitud se elimina completamente de la base de datos

### Ejemplo de uso

```
Situación: Solicitaste cambiar la custodia del 15-20 de marzo, 
pero ya no es necesario.

Acción: 
1. Ve a la vista "Solicitudes"
2. Localiza tu solicitud pendiente
3. Presiona "🗑️ CANCELAR SOLICITUD"
4. Confirma la acción
5. ¡Listo! La solicitud desaparece y se notifica por Telegram
```

## 🎨 Características del Diseño

### Paleta de Colores

- **Fondo**: Gradiente morado/violeta (#667eea → #764ba2)
- **Tarjetas**: Blanco puro (#ffffff)
- **Texto principal**: Gris oscuro (#1e293b)
- **Texto secundario**: Gris medio (#64748b)
- **Padre**: Azul (#3b82f6)
- **Madre**: Rosa (#ec4899)
- **Acentos**: Tonos pastel para mejor legibilidad

### Responsive Design

```css
/* Móviles (< 640px) */
- Espaciado reducido
- Texto más pequeño pero legible
- Iconos adaptados
- Botones táctiles grandes

/* Tablets (640px - 1024px) */
- Aprovecha el espacio horizontal
- Tarjetas más amplias

/* Escritorio (> 1024px) */
- Máximo ancho de 1200px
- Diseño centrado
- Espaciado generoso
```

## 🔒 Seguridad y Privacidad

- ✅ Autenticación requerida para todos los datos
- ✅ Datos encriptados en tránsito (HTTPS)
- ✅ Reglas de seguridad de Firestore configurables
- ✅ Permisos de eliminación controlados
- ✅ Sin tracking de terceros
- ✅ Datos almacenados en servidores de Google (Firebase)

## 📱 Características Técnicas

- **Responsive Design**: Funciona perfectamente en móviles, tablets y escritorio
- **PWA Ready**: Se puede instalar como aplicación
- **Touch Optimized**: Gestos táctiles nativos (swipe)
- **Real-time**: Actualizaciones instantáneas entre dispositivos
- **Lightweight**: Sin frameworks pesados, código vanilla optimizado
- **Accessible**: Contraste adecuado y tamaño de texto legible

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

### No puedo cancelar una solicitud
- Solo puedes cancelar TUS PROPIAS solicitudes
- Solo se pueden cancelar solicitudes con estado "pendiente"
- Verifica que las reglas de Firestore permitan `delete` en la colección `requests`

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

## 🎉 Changelog

### Versión 2.0 (Febrero 2026)
- ✅ **Nuevo diseño en colores claros** - Tema light con gradientes suaves
- ✅ **Función de cancelar solicitudes** - Cancela tus propias solicitudes pendientes
- ✅ **Responsive mejorado** - Optimización para móviles con media queries
- ✅ **Mejor legibilidad** - Contraste mejorado y tamaños de texto ajustados
- ✅ **Confirmaciones** - Diálogos de confirmación antes de acciones críticas

### Versión 1.0 (Enero 2026)
- Sistema de login/registro
- Calendario con semanas alternas
- Gestión de solicitudes
- Eventos con categorías
- Notificaciones por Telegram
- Sincronización en tiempo real

## ⭐ Agradecimientos

- [Firebase](https://firebase.google.com/) - Backend y autenticación
- [Lucide Icons](https://lucide.dev/) - Sistema de iconos
- [Google Fonts](https://fonts.google.com/) - Tipografías
- [Telegram](https://telegram.org/) - Sistema de notificaciones

---

**Desarrollado con ❤️ para facilitar la coordinación familiar**

*Última actualización: Febrero 2026 - Versión 2.0*
