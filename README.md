# 🤖 Autopilot - WhatsApp AI Assistant

Automatización inteligente de respuestas de WhatsApp usando Tasker, AutoNotification y OpenAI Responses API con memoria conversacional.

[![Tasker](https://img.shields.io/badge/Tasker-6.6.7--beta-green.svg)](https://tasker.joaoapps.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-Responses%20API-412991.svg)](https://platform.openai.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## 📋 Descripción

**Autopilot** es un perfil de Tasker que automatiza respuestas en WhatsApp mediante inteligencia artificial. Lee notificaciones, agrupa mensajes consecutivos del mismo contacto y genera respuestas contextualizadas usando la **OpenAI Responses API**.

### 🔑 Tecnología clave: OpenAI Responses API

Este proyecto utiliza la nueva **Responses API** de OpenAI ([`https://api.openai.com/v1/responses`](https://platform.openai.com/docs/api-reference/responses)), que ofrece:

- **Almacenamiento automático de conversaciones** (`"store": true`)
- **Contexto persistente** mediante `previous_response_id`
- **Memoria por contacto** sin gestionar historial manualmente
- **Respuestas contextualizadas** que recuerdan toda la conversación

El sistema guarda el `response_id` de cada respuesta por contacto (usando MD5 del nombre como identificador único) y lo envía en la siguiente petición, permitiendo que el modelo mantenga el contexto completo de la conversación.

### ✨ Características principales

- ⚡ **Respuestas automáticas** vía OpenAI GPT-4o-mini
- 🧠 **Memoria conversacional** por contacto (mantiene contexto entre mensajes gracias a Responses API)
- 📦 **Agregación de mensajes** (debounce de 3s configurable, separados por coma)
- 🔒 **Control de concurrencia** con semáforos por sesión (previene race conditions)
- 🌐 **Manejo robusto de errores** de red con mensajes amigables
- 📝 **Sin archivos de configuración externos** (todo incluido en el XML)
- 🎯 **Filtrado de grupos** (solo responde a mensajes directos)
- 🔄 **Recuperación automática** tras errores

## 📦 Requisitos

### Software necesario

1. **Tasker** (versión 6.6.7-beta o superior)
   - [Descargar desde Play Store](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) (versión de pago)
   - Asegúrate de tener la versión beta habilitada para soporte completo de JavaScriptlet

2. **AutoNotification** (plugin de Tasker)
   - [Descargar desde Play Store](https://play.google.com/store/apps/details?id=com.joaomgcd.autonotification)
   - Versión de pago requerida para usar AutoNotification Reply

3. **WhatsApp** o **WhatsApp Business**
   - Debe tener permisos de notificación activos

### Servicios externos

4. **OpenAI API Key**
   - Crea una cuenta en [OpenAI Platform](https://platform.openai.com/)
   - Genera una API key en [API Keys](https://platform.openai.com/api-keys)
   - Se requiere crédito en la cuenta (el modelo `gpt-4o-mini` es económico: ~$0.15/1M tokens de entrada)

## 🚀 Instalación

### Paso 1: Importar el perfil a Tasker

1. Descarga el archivo [`autopilot.prf.xml`](autopilot.prf.xml) desde este repositorio
2. Copia el archivo a la carpeta de Tasker en tu dispositivo:
   - Ruta recomendada: `/sdcard/Tasker/profiles/` o `/storage/emulated/0/Tasker/profiles/`
3. Abre Tasker en tu dispositivo Android
4. Toca el **botón de menú** (☰) → **Importar** → **Importar un proyecto**
5. Navega hasta donde guardaste `autopilot.prf.xml` y selecciónalo
6. Confirma la importación

**Nota**: Si Tasker no muestra la opción de importar, puedes:
- Hacer **long press** en la pestaña de **Profiles** → **Importar**
- O usar un explorador de archivos para abrir el `.prf.xml` directamente y seleccionar "Tasker" como aplicación

### Paso 2: Configurar AutoNotification

1. Abre **AutoNotification** en tu dispositivo
2. Ve a **Settings** → **Notification Access**
3. Activa el permiso de **Notification Access** para AutoNotification
4. Verifica que WhatsApp esté en la lista de aplicaciones monitorizadas

### Paso 3: Configurar variables globales en Tasker

#### Obligatoria: API Key de OpenAI

1. En Tasker, ve a la pestaña **VARS** (Variables)
2. Toca el **+** para agregar una nueva variable
3. Nombre: `OPENAI_API_KEY`
4. Valor: Tu API key de OpenAI (comienza con `sk-...`)
5. Asegúrate de que sea una **variable global** (sin el símbolo `%` al inicio en la lista)

#### Opcionales: Personalización

**Ajustar tiempo de debounce**:
```
Nombre: DEBOUNCE_SECONDS
Valor: 3
Rango: 3-120 (segundos)
```
- Por defecto: 3 segundos
- Controla cuánto tiempo espera para agregar mensajes consecutivos del mismo contacto

**Instrucciones personalizadas para el modelo**:
```
Nombre: PROMPT
Valor: Eres un asistente personal amigable y profesional. Responde de forma concisa y útil.
```
- Por defecto: Usa un prompt genérico incluido en el perfil
- Define el comportamiento y tono del asistente
- Puedes personalizarlo completamente según tus necesidades

### Paso 4: Activar el perfil

1. En Tasker, ve a la pestaña **PROFILES**
2. Busca el perfil **"WhatsApp Autopilot AI"**
3. Asegúrate de que esté **activado** (switch verde)
4. Si está desactivado, toca el switch para activarlo

## 🧪 Pruebas

### Caso de prueba 1: Respuesta individual
1. Pídele a alguien que te envíe un mensaje de WhatsApp
2. Espera 3 segundos (debounce)
3. Deberías recibir una respuesta automática del asistente

### Caso de prueba 2: Agregación de mensajes
1. Pídele a alguien que envíe 3-4 mensajes consecutivos rápidamente (1-2s entre cada uno)
2. Espera 3 segundos después del último mensaje
3. Deberías recibir **una única respuesta** que considere todos los mensajes

### Caso de prueba 3: Memoria conversacional
1. Envía un mensaje de prueba desde otro dispositivo
2. Recibe la respuesta automática
3. Envía un mensaje de seguimiento relacionado al anterior
4. La respuesta debería mostrar que el asistente recuerda el contexto previo

### Caso de prueba 4: Múltiples contactos simultáneos
1. Pídele a 2-3 personas que te envíen mensajes al mismo tiempo
2. Cada uno debería recibir su propia respuesta personalizada
3. Las conversaciones no deben cruzarse (cada contacto tiene su propio contexto)

### Caso de prueba 5: Manejo de errores
1. Desactiva WiFi/datos móviles temporalmente
2. Envía un mensaje de prueba
3. Deberías recibir un mensaje amigable indicando problema de conexión
4. Reactiva la conexión y envía otro mensaje
5. El sistema debería recuperarse automáticamente

## 🔧 Troubleshooting

### Problema: No responde a mensajes

**Posibles causas y soluciones:**

1. **Verificar API Key**:
   - Asegúrate de que `%OPENAI_API_KEY` esté correctamente configurada en Tasker
   - Verifica que tenga crédito en tu cuenta de OpenAI

2. **Verificar permisos**:
   - AutoNotification debe tener permiso de **Notification Access**
   - Tasker debe tener permisos de **Acceso a archivos** (para logs)

3. **Verificar que el perfil esté activo**:
   - Ve a **PROFILES** en Tasker
   - El perfil "WhatsApp Autopilot AI" debe tener el switch verde

4. **Revisar logs**:
   - En Tasker, ve a **More** → **Run Log**
   - Busca errores relacionados con "Responder GPT-4o-mini"

### Problema: Responde múltiples veces al mismo mensaje

**Solución**: Esto indica que los semáforos no se están limpiando correctamente.

1. Reinicia Tasker: **More** → **Exit** → Reabre Tasker
2. Si persiste, reimporta el perfil desde el XML original

### Problema: No mantiene contexto entre mensajes

**Posibles causas:**

1. **response_id no se está guardando**:
   - Verifica en logs que aparezca "PlanB: Guardar resp_..." con un ID válido
   - El ID debe comenzar con `resp_`

2. **Diferentes session_id para el mismo contacto**:
   - El nombre del contacto debe ser consistente
   - Si cambia el nombre en WhatsApp, se creará una nueva sesión

### Problema: Mensajes de error técnicos aparecen en WhatsApp

**Solución**: El formateo de errores de red debería convertir mensajes técnicos en amigables.

- Si ves `UnknownHostException` o similar en WhatsApp, verifica que la acción de formateo de errores (act41) esté presente
- Reimporta el perfil si falta esta acción

### Problema: Timeout en respuestas

**Causas comunes:**

1. **Debounce muy alto**:
   - Si `%DEBOUNCE_SECONDS` > 30, considera reducirlo a 3-10 segundos

2. **Conexión lenta**:
   - El timeout HTTP es de 45 segundos
   - Verifica tu conexión a Internet

3. **Buffer muy grande**:
   - Si alguien envía 50+ mensajes rápidamente, el payload puede ser muy grande
   - El sistema agregará todos los mensajes con comas

## 🏗️ Arquitectura técnica

### Flujo de ejecución

```
1. Notificación de WhatsApp detectada (AutoNotification Intercept)
   ↓
2. Validación: ¿Es mensaje directo? (no grupo)
   ↓
3. Extracción de datos: sender (%antitle), mensaje (%antext)
   ↓
4. Generación de session_id (MD5 del sender)
   ↓
5. Agregación con debounce (3s):
   - Incrementar InFlight_%session_id
   - Acumular en Buf_%session_id (separado por comas)
   - Guardar Sender_%session_id (solo primera vez)
   ↓
6. Leader election:
   - Esperar DEBOUNCE_SECONDS
   - El último mensaje actúa como líder
   ↓
7. Construcción del payload HTTP:
   - model: gpt-4o-mini
   - store: true
   - previous_response_id: (si existe para este contacto)
   - input: "sender: [nombre]\nmessage:\n[mensajes agregados]"
   ↓
8. HTTP Request a https://api.openai.com/v1/responses
   ↓
9. Limpieza de semáforos (SIEMPRE ejecuta, incluso si falla HTTP)
   ↓
10. Extracción de respuesta:
    - Primario: Structured Output (%http_data[output.content.text])
    - Fallback: Parseo manual con JavaScript
   ↓
11. Persistencia de contexto:
    - Guardar response_id en PrevResp_%session_id
    - Plan B: Variable Set como fallback
   ↓
12. Formateo de errores:
    - Convertir mensajes técnicos de red a amigables
   ↓
13. AutoNotification Reply:
    - Enviar respuesta a WhatsApp
```

### Variables por sesión

Cada contacto (identificado por `session_id` = MD5 del nombre) tiene:

- `InFlight_%session_id`: Contador de mensajes en ventana de debounce
- `Buf_%session_id`: Buffer de mensajes acumulados
- `Sender_%session_id`: Nombre del contacto (persistido)
- `GATE_SHOULD_SEND_%session_id`: Gate para controlar envío único ("send"/"skip")
- `HttpBody_%session_id`: Payload JSON construido por el líder
- `PrevResp_%session_id`: response_id de OpenAI para contexto conversacional

### Acciones clave (42 acciones totales)

- **act0**: MD5 hash de antitle → session_id
- **act3**: JavaScriptlet de agregación y debounce (líder/seguidores)
- **act6**: JavaScriptlet de construcción de payload HTTP
- **act9-act11**: Gate de envío (verifica que sea el líder)
- **act12**: HTTP Request a OpenAI Responses API
- **act13**: Limpieza de semáforos post-HTTP (crítica, flags=16)
- **act14-act16**: Plan B - Variable Set para persistencia de response_id
- **act22-act24**: Parseo de respuesta con fallbacks
- **act33**: Persistencia de response_id en contexto (JavaScriptlet)
- **act39**: Formateo de errores de red amigables
- **act40**: AutoNotification Reply (envío a WhatsApp)

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Si encuentras un bug o tienes una mejora:

1. Abre un **Issue** describiendo el problema o la mejora propuesta
2. Si deseas contribuir código, crea un **Pull Request** con:
   - Descripción clara de los cambios
   - Pruebas realizadas
   - Capturas de pantalla si aplica

## 📄 Licencia

Este proyecto está licenciado bajo la Licencia MIT. Ver el archivo [LICENSE](LICENSE) para más detalles.

## ⚠️ Disclaimer

- Este proyecto es de código abierto y se proporciona "tal cual", sin garantías de ningún tipo
- El uso de la OpenAI API genera costos. Monitorea tu uso en [OpenAI Usage](https://platform.openai.com/usage)
- Ten cuidado con la información sensible que compartes en conversaciones
- Este proyecto no está afiliado con WhatsApp, Meta, Tasker o OpenAI

## 🙏 Agradecimientos

- [Tasker](https://tasker.joaoapps.com/) por la increíble plataforma de automatización de Android
- [AutoNotification](https://joaoapps.com/autonotification/) por permitir interactuar con notificaciones
- [OpenAI](https://openai.com/) por la potente Responses API

---

**¿Preguntas?** Abre un Issue en este repositorio.
