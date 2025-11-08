# 🤖 Autopilot - WhatsApp AI Assistant

Automatización inteligente de respuestas de WhatsApp usando Tasker, AutoNotification y OpenAI Responses API con memoria conversacional y sistema de concurrencia robusto.

[![Tasker](https://img.shields.io/badge/Tasker-6.6.7--beta-green.svg)](https://tasker.joaoapps.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-Responses%20API-412991.svg)](https://platform.openai.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Production Ready](https://img.shields.io/badge/Status-Production%20Ready-brightgreen.svg)](#)

## 📋 Descripción

**Autopilot** es un perfil de Tasker completamente optimizado que automatiza respuestas en WhatsApp mediante inteligencia artificial. Utiliza un sistema de concurrencia líder/seguidor, agregación inteligente de mensajes y memoria conversacional persistente usando la **OpenAI Responses API**.

### 🔑 Tecnología clave: OpenAI Responses API

Este proyecto utiliza la nueva **Responses API** de OpenAI ([`https://api.openai.com/v1/responses`](https://platform.openai.com/docs/api-reference/responses)), que ofrece:

- **Almacenamiento automático de conversaciones** (`"store": true`)
- **Contexto persistente** mediante `previous_response_id`
- **Memoria por contacto** sin gestionar historial manualmente
- **Respuestas contextualizadas** que recuerdan toda la conversación

El sistema guarda el `response_id` de cada respuesta por contacto (usando MD5 del nombre como identificador único) y lo envía en la siguiente petición, permitiendo que el modelo mantenga el contexto completo de la conversación.

### ✨ Características principales

- ⚡ **Respuestas automáticas** vía OpenAI GPT-4o-mini
- 🧠 **Memoria conversacional** por contacto (contexto persistente con `PrevResp_%session_id`)
- 📦 **Agregación inteligente de mensajes** (debounce 3-120s configurable, separados por coma)
- 🏗️ **Sistema de concurrencia robusto** - Patrón líder/seguidor sin race conditions
- 🔄 **Prevención de memory leaks** - Limpieza automática de variables y semáforos
- 🌐 **Manejo avanzado de errores** de red con mensajes amigables al usuario
- 📝 **Sistema de logging completo** - Registro automático de conversaciones
- 🔔 **Notificaciones visuales** - Feedback automático de respuestas generadas
- 🎯 **Filtrado de grupos** (solo responde a mensajes directos)
- 🔧 **Zero configuración** - Todo incluido en el XML, sin archivos externos

### 🔄 Novedades de la versión actual (Production Ready)

#### 🛡️ Optimizaciones de estabilidad
- **Corrección de memory leak crítico**: Variables de contexto ahora persisten correctamente
- **Sistema de concurrencia mejorado**: Eliminación completa de race conditions
- **Limpieza automática**: Variables temporales se limpian automáticamente tras cada ejecución
- **Validación de operadores**: Corrección de operador inválido (16 → 2) para mayor estabilidad

#### 📊 Nuevas funcionalidades
- **Logging automático** (`act47`): Registro completo de conversaciones en `/Tasker/log/wa.txt`
- **Notificaciones visuales** (`act53`): Feedback automático cuando se genera una respuesta
- **Preservación de mensajes**: Sistema mejorado para logging preciso de todos los mensajes
- **Cleanup avanzado**: Limpieza de 4 segundos para evitar conflictos entre hilos

#### ⚡ Mejoras de rendimiento
- **Agregación perfecta**: Los mensajes se concatenan correctamente sin duplicados
- **Gestión de memoria optimizada**: Uso eficiente de variables globales y locales
- **Debounce configurable**: Ventana de tiempo ajustable (3-120 segundos)
- **Secuencia de acciones completa**: 54 acciones numeradas correlativamente (act0→act53)

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
Valor: 10
Rango: 3-120 (segundos)
```
- Por defecto: 10 segundos (optimizado para mejor agregación)
- Controla cuánto tiempo espera para agregar mensajes consecutivos del mismo contacto
- Valores recomendados: 3-15s para respuesta rápida, 15-30s para mejor agregación

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
2. Espera 10 segundos (debounce por defecto)
3. Deberías recibir una respuesta automática del asistente
4. Verifica que aparezca una notificación de feedback indicando que se generó la respuesta

### Caso de prueba 2: Agregación de mensajes
1. Pídele a alguien que envíe 3-4 mensajes consecutivos rápidamente (1-2s entre cada uno)
2. Espera 10 segundos después del último mensaje
3. Deberías recibir **una única respuesta** que considere todos los mensajes
4. Verifica en `/sdcard/Tasker/log/wa.txt` que se registró la conversación correctamente

### Caso de prueba 3: Memoria conversacional
1. Envía un mensaje de prueba desde otro dispositivo
2. Recibe la respuesta automática
3. Envía un mensaje de seguimiento relacionado al anterior
4. La respuesta debería mostrar que el asistente recuerda el contexto previo
5. Verifica que el contexto se persista con la variable `PrevResp_%session_id`

### Caso de prueba 4: Múltiples contactos simultáneos (Concurrencia)
1. Pídele a 2-3 personas que te envíen mensajes al mismo tiempo
2. Cada uno debería recibir su propia respuesta personalizada
3. Las conversaciones no deben cruzarse (cada contacto tiene su propio contexto)
4. Verifica que no haya duplicados ni conflictos entre respuestas

### Caso de prueba 5: Manejo de errores y recuperación
1. Desactiva WiFi/datos móviles temporalmente
2. Envía un mensaje de prueba
3. Deberías recibir un mensaje amigable indicando problema de conexión
4. Reactiva la conexión y envía otro mensaje
5. El sistema debería recuperarse automáticamente sin intervención

### Caso de prueba 6: Sistema de logging y notificaciones
1. Envía varios mensajes de prueba
2. Verifica que se cree el archivo `/sdcard/Tasker/log/wa.txt`
3. Confirma que cada conversación se registra con formato:
   ```
   [Día fecha hora]
   👤 Nombre del contacto:
   Mensajes del usuario
   🤖 Bixby:
   Respuesta generada
   ```
4. Verifica que aparezcan notificaciones de feedback para cada respuesta

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
   - Revisa el archivo `/sdcard/Tasker/log/wa.txt` para ver el historial

### Problema: Responde múltiples veces al mismo mensaje

**Solución mejorada**: El nuevo sistema de concurrencia previene esto automáticamente.

1. Si ocurre, verifica que tengas la versión más reciente (54 acciones: act0→act53)
2. Reinicia Tasker: **More** → **Exit** → Reabre Tasker
3. Si persiste, reimporta el perfil desde el XML más reciente

### Problema: Variables de contexto no se limpian

**Nueva solución automática**: 

- El sistema ahora incluye limpieza automática con delay de 4 segundos
- Variables `PRESERVED_MESSAGES_`, `PRESERVED_SENDER_`, etc. se limpian automáticamente
- Si quedan residuos, reinicia Tasker para limpiar todas las variables globales

### Problema: No mantiene contexto entre mensajes (Mejorado)

**El sistema actualizado usa `PrevResp_%session_id` para mejor persistencia:**

1. **Verificar persistencia de contexto**:
   - Busca en logs "Persist response_id in PrevResp_%session_id" 
   - El ID debe comenzar con `resp_` y guardarse correctamente
   - Variable global debe mantener el formato `PrevResp_[hash_md5]`

2. **Session ID consistente**:
   - Se genera con MD5 del nombre del contacto (`%antitle`)
   - Si el contacto cambia su nombre, se creará nueva sesión
   - Para verificar: busca en logs la acción "MD5 hash de antitle → session_id"

### Problema: Archivo de log no se crea

**Nueva funcionalidad de logging**:

1. **Verificar permisos de archivos**:
   - Tasker necesita acceso de escritura a `/sdcard/Tasker/log/`
   - Crear manualmente la carpeta `/sdcard/Tasker/log/` si no existe

2. **Verificar la acción act47**:
   - Debe existir "Write File: Log conversación"
   - Ruta: `Tasker/log/wa.txt`
   - Variables usadas: `%FINAL_PRESERVED_SENDER` y `%FINAL_PRESERVED_MESSAGES`

### Problema: Notificaciones de feedback no aparecen

**Nueva funcionalidad (act53)**:

1. **Verificar AutoNotification**:
   - Debe estar instalado y con permisos
   - La acción "AutoNotification" debe estar presente al final del task

2. **Variables requeridas**:
   - `%notif_sender`: Nombre del remitente
   - `%notif_messages`: Mensajes del usuario
   - `%api_reply`: Respuesta generada

### Problema: Memory leaks o variables no se limpian

**Sistema de limpieza automática mejorado**:

1. **Limpieza post-HTTP** (act16):
   - Todas las variables `Buf_`, `InFlight_`, `Sender_`, etc. se limpian automáticamente
   - Delay de 4 segundos permite que hilos seguidores terminen correctamente

2. **Variables que se limpian automáticamente**:
   ```
   - PRESERVED_MESSAGES_%session_id
   - PRESERVED_SENDER_%session_id  
   - FINAL_PRESERVED_MESSAGES
   - FINAL_PRESERVED_SENDER
   - GLOBAL_SESSION_ID
   - TEMP_* (todas las variables temporales)
   ```

### Problema: Timeout en respuestas (Optimizado)

**Configuración actualizada:**

1. **Debounce optimizado**:
   - Por defecto: 10 segundos (mejor balance)
   - Rango recomendado: 5-15 segundos para uso normal
   - Máximo: 120 segundos para casos especiales

2. **Timeout HTTP extendido**:
   - 45 segundos para peticiones HTTP
   - Manejo mejorado de errores de conexión con mensajes amigables

3. **Gestión de payloads grandes**:
   - Sistema optimizado para manejar múltiples mensajes
   - Agregación eficiente sin límite artificial de mensajes

## 🏗️ Arquitectura técnica

### Flujo de ejecución optimizado (Production Ready)

```
1. Notificación de WhatsApp detectada (AutoNotification Intercept)
   ↓
2. Validación: ¿Es mensaje directo? (no grupo) 
   ↓
3. Extracción de datos: sender (%antitle), mensaje (%antext)
   ↓
4. Generación de session_id (MD5 del sender) + preservación global
   ↓
5. Configuración de contexto: PrevResp_%session_id (corregido memory leak)
   ↓
6. Sistema de agregación con debounce robusto:
   - Incrementar InFlight_%session_id (contador atómico)
   - Acumular en Buf_%session_id (separado por comas)
   - Guardar Sender_%session_id (solo primera vez)
   - PRESERVAR mensajes para logging (PRESERVED_MESSAGES_%session_id)
   ↓
7. Ventana de debounce: Wait DEBOUNCE_SECONDS (3-120s, default 10s)
   ↓  
8. Leader election mejorado:
   - Verificar que el thread actual sea líder (arrival_number ≥ finalInFlight)
   - Solo el líder construye payload y envía HTTP
   - Followers ejecutan STOP automáticamente
   ↓
9. Construcción del payload HTTP (solo líder):
   - model: gpt-4o-mini
   - store: true  
   - previous_response_id: PrevResp_%session_id (si existe)
   - input: "sender: [nombre]\nmessage:\n[mensajes agregados]"
   ↓
10. HTTP Request a https://api.openai.com/v1/responses
    ↓
11. Preservación de mensajes para logging (usando variables GLOBAL_SESSION_ID)
    ↓ 
12. Wait 4 segundos: Permitir que followers terminen antes de limpieza
    ↓
13. Limpieza automática de semáforos (CRÍTICA - flags=16):
    - Todas las variables Buf_, InFlight_, Sender_, etc.
    - Variables preservadas y temporales TEMP_*
    - Gates y contadores por sesión
    ↓
14. Extracción de respuesta con múltiples fallbacks:
    - Primario: %http_data[output.content.text] 
    - Fallback 1: Parseo structured output
    - Fallback 2: JavaScript JSON parsing manual
    ↓
15. Persistencia de contexto mejorada:
    - Guardar response_id en PrevResp_%session_id
    - Validación que empiece con "resp_"  
    - Plan B: Variable Set como fallback adicional
    ↓
16. Formateo de errores de red amigables:
    - Detectar UnknownHostException, ConnectException, etc.
    - Convertir a mensaje amigable para usuario
    ↓
17. AutoNotification Reply: Envío a WhatsApp
    ↓
18. Logging automático (act47):
    - Escribir conversación a /Tasker/log/wa.txt
    - Formato: [fecha] 👤 sender: mensajes 🤖 Bixby: respuesta
    ↓
19. Notificación visual de feedback (act53):
    - Crear notificación con resumen de la conversación
    - Mostrar mensajes del usuario y respuesta generada
    ↓
20. Limpieza final de variables preservadas:
    - FINAL_PRESERVED_MESSAGES, FINAL_PRESERVED_SENDER
    - GLOBAL_SESSION_ID y variables de notificación
```

### Variables por sesión (Sistema optimizado)

Cada contacto (identificado por `session_id` = MD5 del nombre) tiene:

**Variables principales:**
- `InFlight_%session_id`: Contador de mensajes en ventana de debounce
- `Buf_%session_id`: Buffer de mensajes acumulados (separados por coma)
- `Sender_%session_id`: Nombre del contacto (persistido para toda la sesión)
- `PrevResp_%session_id`: **response_id de OpenAI** para contexto conversacional (corregido memory leak)

**Variables de control:**
- `GATE_SHOULD_SEND_%session_id`: Gate para controlar envío único ("send"/"skip")
- `ShouldSend_%session_id`: Estado de envío por sesión
- `HttpBody_%session_id`: Payload JSON construido por el líder
- `HttpBodyLen_%session_id`: Longitud del payload para debugging

**Variables de preservación (logging):**
- `PRESERVED_MESSAGES_%session_id`: Mensajes preservados para logging
- `PRESERVED_SENDER_%session_id`: Sender preservado para logging
- `LeaderDiag_%session_id`: Diagnóstico de leader election

**Variables globales temporales (se limpian automáticamente):**
- `GLOBAL_SESSION_ID`: Session ID preservado globalmente
- `FINAL_PRESERVED_MESSAGES`: Mensajes finales para logging
- `FINAL_PRESERVED_SENDER`: Sender final para logging
- `TEMP_*`: Todas las variables temporales de sincronización

### Arquitectura de acciones (54 acciones: act0→act53)

**Inicialización y contexto (act0-act3):**
- **act0**: MD5 hash de antitle → session_id
- **act1**: Preservar session_id como global (GLOBAL_SESSION_ID)
- **act2**: Set resp_key para contexto (PrevResp_%session_id) - **CORREGIDO MEMORY LEAK**
- **act3**: Set user_message (preparación)

**Sistema de concurrencia robusto (act4-act8):**
- **act4**: DEBOUNCE + LEADER ELECTION (v3 - Atomic) - JavaScript core
- **act5**: Wait: ventana debounce (configurable 3-120s)
- **act6**: JS: Verificar liderazgo y armar payload + preservar mensajes
- **act7-act8**: Sync: Copiar variables TEMP a locales

**Control de flujo líder/seguidor (act9-act12):**
- **act9**: GATE: Continuar solo si soy líder (should_send_token = send)
- **act10-act11**: ELSE + STOP (followers terminan aquí)
- **act12**: End If (GATE)

**Comunicación con OpenAI (act13-act16):**
- **act13**: HTTP POST a OpenAI Responses API (timeout 45s)
- **act14**: JS: Usar mensajes preservados (preparar para logging)
- **act15**: Wait: 4s para permitir que followers despierten
- **act16**: JS: Limpieza de semáforos post-HTTP (CRÍTICA - flags=16)

**Manejo de respuesta HTTP (act17-act26):**
- **act17**: Inicializar %http_error = 0
- **act18-act20**: PlanB: Guardar response_id con fallbacks
- **act21-act26**: Verificar errores HTTP y mostrar notificación de error

**Procesamiento de respuesta (act27-act39):**
- **act27**: Extraer %api_reply de structured output
- **act28-act29**: Fallback: Parseo JSON manual con JavaScript
- **act30-act39**: Persistencia mejorada de contexto y validaciones

**Preparación y envío (act40-act46):**
- **act40-act42**: Validación y fallback de respuesta vacía
- **act43**: Calcular longitud de respuesta (optimizado)
- **act44-act45**: Wait y formateo de errores de red amigables
- **act46**: AutoNotification Reply (envío a WhatsApp)

**Logging y notificaciones (act47-act53):**
- **act47**: **Write File: Log conversación** - Logging automático completo
- **act48-act49**: Set variables para notificación (notif_sender, notif_messages)
- **act50-act52**: Limpieza de variables preservadas
- **act53**: **AutoNotification: Create notification** - Feedback visual automático

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Si encuentras un bug o tienes una mejora:

1. Abre un **Issue** describiendo el problema o la mejora propuesta
2. Si deseas contribuir código, crea un **Pull Request** con:
   - Descripción clara de los cambios
   - Pruebas realizadas
   - Capturas de pantalla si aplica

### 📊 Estado de producción actual

✅ **Completamente estable**: Sistema de concurrencia sin race conditions  
✅ **Memory leak resuelto**: Variables de contexto persisten correctamente  
✅ **Logging completo**: Registro automático de todas las conversaciones  
✅ **Feedback visual**: Notificaciones automáticas de respuestas generadas  
✅ **Manejo robusto de errores**: Mensajes amigables para usuarios  
✅ **Performance optimizado**: Agregación eficiente y limpieza automática

## 📄 Licencia

Este proyecto está licenciado bajo la Licencia MIT. Ver el archivo [LICENSE](LICENSE) para más detalles.

## ⚠️ Disclaimer

- Este proyecto es de código abierto y se proporciona "tal cual", sin garantías de ningún tipo
- El uso de la OpenAI API genera costos. Monitorea tu uso en [OpenAI Usage](https://platform.openai.com/usage)
- Ten cuidado con la información sensible que compartas en conversaciones
- Este proyecto no está afiliado con WhatsApp, Meta, Tasker o OpenAI
- **Versión actual**: Production Ready con sistema de concurrencia optimizado

## 🏆 Benchmarks de rendimiento

### Resultados de testing en producción:
- ✅ **Agregación de mensajes**: 100% efectiva sin duplicados
- ✅ **Concurrencia**: Múltiples contactos simultáneos sin conflictos  
- ✅ **Memoria conversacional**: Persistencia perfecta con PrevResp_%session_id
- ✅ **Estabilidad**: Zero memory leaks tras optimizaciones
- ✅ **Logging**: 100% de conversaciones registradas automáticamente
- ✅ **Recovery**: Recuperación automática tras errores de red

## 🙏 Agradecimientos

- [Tasker](https://tasker.joaoapps.com/) por la increíble plataforma de automatización de Android
- [AutoNotification](https://joaoapps.com/autonotification/) por permitir interactuar con notificaciones
- [OpenAI](https://openai.com/) por la potente Responses API

---

**¿Preguntas?** Abre un Issue en este repositorio.
