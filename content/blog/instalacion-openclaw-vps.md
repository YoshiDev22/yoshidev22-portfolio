---
title: "Cómo instalar OpenClaw en un VPS con Docker y Telegram"
date: 2026-06-15
draft: false
tags: [openclaw, docker, telegram, vps, gemini, ia-agente]
categories: [Tutorial]
description: "Guía real para instalar OpenClaw en un VPS Hostinger con Docker y Telegram, corrigiendo errores de la doc oficial y problemas de RAM."
---

OpenClaw es un agente de IA personal de código abierto que vive en tu propio servidor y que puedes controlar desde Telegram (u otros canales). No es otro chatbot de suscripción: tú eliges el modelo de lenguaje y qué herramientas instalar. Si bien el agente y el contexto local se mantienen en tu servidor, es importante mencionar que los modelos de lenguaje (como Gemini, ChatGPT o Claude) procesan las peticiones en sus propios servidores al usar sus APIs.

¿Completamente gratis? Casi. La arquitectura no tiene costo, pero los modelos de lenguaje (como Gemini, ChatGPT o Claude) que uses sí pueden tenerlo dependiendo del proveedor que elijas. Existen opciones con tier gratuito (como Google AI Studio con Gemini Flash, que es la que se usa en este tutorial) que permiten hacer pruebas sin costo.

## ¿Para qué sirve?

OpenClaw puede adaptarse a muchos flujos de trabajo distintos. Algunos ejemplos:

- **Documentar experimentos**: extrae texto de PDFs y genera borradores en Markdown listos para revisar
- **Gestión de tareas**: recordatorios y listas de pendientes directamente desde Telegram, sincronizados con Google Calendar
- **Despliegues rápidos**: aplica cambios menores en tu código o sitio web sin estar frente a tu PC
- **Investigación móvil**: consulta y resume información con conexión limitada
- **Automatización en casa**: integra con sensores, cámaras o sistemas domóticos

> **Lo que construiremos en este tutorial**
> Un asistente que lee PDFs técnicos, genera borradores en Markdown para un blog en Hugo y hace commits a GitHub — pero solo cuando tú lo autorices explícitamente.

## ¿Necesito un VPS para usarlo?

No es totalmente necesario. Puedes correr OpenClaw en tu propia computadora. El problema es que si la apagas o pierdes conexión, el bot deja de funcionar. Un VPS garantiza disponibilidad 24/7 sin que tengas que mantener tu máquina encendida.

Si solo quieres probar OpenClaw o lo usarás ocasionalmente, es mejor instalarlo en local. Si quieres que tu asistente esté siempre disponible, un VPS será la mejor opción.

## Requisitos

Este tutorial se realizó en un VPS Hostinger KVM 2 con las siguientes especificaciones:

| Recurso | KVM 2 (usado) | Mínimo recomendado |
|---------|--------------|-------------------|
| RAM | 8 GB | 2 GB (solo para ejecutar; ver nota) |
| CPU | 2 vCores | 1 vCore |
| Disco | 100 GB NVMe | 10 GB |
| SO | Ubuntu 24.04 | Ubuntu 22.04 / 24.04 |

> **Nota importante sobre la RAM:** compilar la imagen de Docker desde el código fuente exige 8 GB de RAM. Ejecutar la imagen pre-compilada (la opción que se usa en este tutorial) consume entre 1 GB y 2 GB, lo que hace viable correrlo en planes más modestos. Se explica en detalle en la Fase 3.

Además necesitas:

- Acceso SSH al servidor con privilegios sudo
- Docker y Docker Compose instalados (se verifica en la Fase 1)
- Una cuenta de Google AI Studio para obtener una API Key de Gemini (tier gratuito)
- Un bot de Telegram creado con @BotFather

> **¿Por qué Docker?** Este tutorial despliega OpenClaw en un contenedor Docker para aislarlo del resto del sistema. Esto evita conflictos de dependencias, facilita actualizaciones y —lo más importante— protege tus archivos de comportamientos inesperados del agente, como escrituras accidentales fuera del workspace o riesgos de prompt injection que pudieran afectar otros servicios del servidor.

## Fase 1: Verificar Docker en el servidor

*(Si vas a instalar OpenClaw en local, omite esta fase.)*

Conéctate por SSH a tu VPS y comprueba que Docker y Docker Compose estén disponibles:

```bash
docker --version
docker compose version
```

Deberías ver números de versión en ambos casos. Si obtienes `command not found`, instala Docker primero siguiendo la [guía oficial de instalación en Ubuntu](https://docs.docker.com/engine/install/ubuntu/) antes de continuar.

## Fase 2: Clonar el repositorio y ejecutar el script de instalación

Crea un directorio para el proyecto y clona el repositorio:

```bash
mkdir -p ~/Proyectos && cd ~/Proyectos
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

El repositorio es activo y en desarrollo continuo. Los comandos de este tutorial corresponden a la versión disponible en junio de 2026. Si el comportamiento difiere, revisa el `README.md` del repositorio para ver si hubo cambios.

Ahora ejecuta el script de setup. Atención aquí: la documentación oficial de Hostinger indica el comando:

```bash
# ⚠️ Este comando puede no funcionar en versiones recientes
./docker-setup.sh
```

Si el repositorio que clonaste corresponde a la versión actual, ese archivo no existe en la raíz y obtendrás:

```
-bash: ./docker-setup.sh: No such file or directory
```

El script en versiones recientes se encuentra en una ruta distinta. El comando correcto para mi versión fue:

```bash
bash ./scripts/docker/setup.sh
```

## Fase 3: El problema de RAM y la solución

Al ejecutar el script por primera vez, este intenta compilar la imagen de Docker localmente desde el código fuente. En un VPS con poca RAM (como el KVM 2 con 8 GB o menos), el proceso puede congelar el servidor y hacer que falle la compilación.

**¿Por qué pasa esto?**

OpenClaw es un monorepo en TypeScript que compila simultáneamente el frontend, backend y extensiones. Esto requiere explícitamente `NODE_OPTIONS=--max-old-space-size=8192`, es decir, 8 GB de RAM dedicados solo a la compilación. Un servidor con 8 GB como el mío, que además corre el sistema operativo y otros servicios, simplemente no tiene margen suficiente.

**¿"Compilar" significa que la imagen se adapta a mi hardware?**

Yo también lo asumí así al principio — es una confusión natural si estás empezando con Docker. Pero no. "Construir la imagen" no significa que el software se optimice para tu procesador o arquitectura; es simplemente el proceso de empaquetar el código fuente en un contenedor ejecutable. El resultado es el mismo independientemente del servidor donde se compile. Solo cambia cuánta RAM consume hacerlo.

**La solución: usar la imagen pre-compilada**

En lugar de compilar, descarga la imagen ya construida desde el registro oficial. Antes de ejecutar el script de setup, define la variable de entorno que le indica qué imagen usar:

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
bash ./scripts/docker/setup.sh
```

Con esto, el script omite la compilación local y hace un `pull` de la imagen directamente desde `ghcr.io` (GitHub Container Registry). El proceso es mucho más ligero y el servidor no se congelará.

Una vez descargada, la imagen en ejecución consume entre 1 GB y 2 GB de RAM, perfectamente viable en planes modestos.

## Fase 4: El asistente de configuración (Setup Wizard)

Una vez ejecutado el script, se lanza un asistente interactivo. A continuación se documentan las decisiones tomadas y el razonamiento detrás de cada una:

| Opción | Elección | Razón |
|--------|----------|-------|
| Setup Mode | QuickStart (recommended) | Usa SQLite integrado; evita levantar PostgreSQL o Redis y ahorra RAM |
| LLM Provider | Google AI Studio (Gemini) | Tiene tier gratuito para pruebas; no requiere tarjeta de crédito inmediata |
| Modelo LLM | [VERIFICAR: nombre exacto del modelo Flash mostrado en el wizard] | Las versiones Flash tienen límites de tasa más permisivos en el tier gratuito que las versiones Pro |
| Canal | Telegram (Bot API) | Gratuito, manejo nativo de archivos e imágenes, sin fricción |
| Web Search | Parallel Search (Free) | No requiere API key; permite buscar en internet y extraer contenido de páginas web directamente desde el chat |
| Skills instaladas | Solo nano-pdf | Esencial para el caso de uso: leer y extraer texto de PDFs técnicos |
| Skills ignoradas | Las 37 dependencias externas restantes | Mantiene el servidor ligero; las integraciones de pago no son necesarias |
| Hooks | Solo session-memory | Permite reiniciar el contexto con /new liberando RAM, pero guarda un resumen permanente en disco |

> **Sobre el LLM:** se puede usar cualquier proveedor compatible, incluyendo opciones de pago como Anthropic (Claude) u OpenAI, o incluso un modelo local corriendo en tu propio equipo si tienes el hardware. Para este tutorial se eligió Gemini Flash en el tier gratuito exclusivamente para la fase de pruebas.

El workspace de sesiones se guarda en `~/.openclaw/workspace`. Ahí es donde el agente conserva el resumen de contexto entre sesiones.

## Fase 5: Conectar Telegram y autorizar el bot

### Crear el bot en Telegram

1. Abre Telegram y busca **@BotFather**
2. Envía `/newbot`
3. Elige un nombre visible y un nombre de usuario (debe terminar en `bot`)
4. BotFather te entregará un token. Guárdalo; lo necesitarás en el siguiente paso

> El nombre de usuario del bot es público por diseño. El token sí es sensible: trátalo como una contraseña y nunca lo compartas ni lo subas a un repositorio.

### Verificar que el contenedor esté corriendo

```bash
cd ~/Proyectos/openclaw
docker compose ps
```

Deberías ver el contenedor con estado `Running`.

### La capa de seguridad de OpenClaw

Al enviar el primer mensaje al bot desde Telegram, no recibirás una respuesta inmediata. En su lugar, el bot emite un código de seguridad en la terminal. Este es el mecanismo de emparejamiento personal de OpenClaw: el agente solo acepta mensajes de usuarios explícitamente autorizados por el administrador del servidor.

Para aprobar tu ID de Telegram, ejecuta en el servidor:

```bash
openclaw pairing approve telegram [CÓDIGO]
```

Donde `[CÓDIGO]` es el código que apareció en los logs del contenedor. Puedes verlo con:

```bash
docker compose logs openclaw-gateway
```

Una vez aprobado, el bot responderá normalmente. Ningún otro usuario puede interactuar con él a menos que repitas este proceso de aprobación explícita.

### Primeros mensajes: configurar el rol del asistente

OpenClaw no tiene un rol definido por defecto más allá de ser un agente general. Los primeros mensajes que envíes al bot configuran su comportamiento para la sesión (y, si `session-memory` está activo, de forma persistente).

El asistente pedirá o esperará:

- Tu nombre o cómo quieres que te llame
- Qué rol debe tomar (asistente general, especializado, con nombre propio)
- El flujo de trabajo específico que debe seguir

A continuación un ejemplo de prompt inicial adaptado al caso de uso de este tutorial:

```
Hola. Prefiero que me llames [TU_NOMBRE] y que tus respuestas sean siempre breves y concisas.
Soy [TU_PROFESIÓN]. A partir de hoy eres [NOMBRE_DEL_ASISTENTE], el asistente principal
de mi laboratorio y emprendimiento [NOMBRE_DEL_PROYECTO].

Tu objetivo principal es ayudarme a digitalizar manuales técnicos.
El flujo de trabajo será: tú extraes el texto, generas un archivo Markdown
en la carpeta de mi blog en Hugo con el estado 'draft: true' y esperas mi revisión.
Solo cuando yo te lo autorice explícitamente por este chat, harás el commit
y el push a GitHub en mi rama developer.
```

## Preguntas frecuentes

**¿Qué diferencia hay entre construir la imagen desde cero y usar la pre-compilada?**

Construir desde cero toma el código fuente del repositorio y lo empaqueta en una imagen ejecutable en tu servidor. Usar la imagen pre-compilada descarga ese resultado ya listo desde el registro oficial. El software que corre es el mismo; la diferencia está en dónde ocurre el trabajo de empaquetado. Construir localmente consume 8 GB de RAM y varios minutos. Descargar la imagen pre-compilada es rápido y ligero. Para la mayoría de los casos, la imagen oficial es suficiente.

**¿Necesito abrir puertos en el firewall?**

En el modo QuickStart (CLI + Telegram), no. No hay interfaz web activa, así que no es necesario exponer ningún puerto. Si en el futuro habilitas la interfaz gráfica, necesitarás abrir el puerto 18789.

**¿Puedo usarlo con Claude o GPT-4 en lugar de Gemini?**

Sí. Durante el setup wizard puedes configurar cualquier proveedor compatible: Anthropic (Claude), OpenAI, o un modelo local a través de una API compatible con OpenAI. El tier gratuito de Gemini es útil para pruebas; para producción, evalúa el proveedor según tu caso de uso y presupuesto.

**¿Qué pasa si reinicio el servidor?**

El contenedor Docker no arranca automáticamente a menos que lo configures. Para que OpenClaw inicie con el servidor:

```bash
docker update --restart unless-stopped openclaw-gateway
```

> [VERIFICAR: confirmar si este comando aplica a tu setup o si el `restart: unless-stopped` en el Docker Compose file ya lo cubre]

**¿Puedo correr el modelo de IA en mi propia GPU y evitar pagar una suscripción?**

Sí, si tienes una GPU con suficiente VRAM. OpenClaw acepta cualquier proveedor con una API compatible con OpenAI, lo que incluye servidores de modelos locales como LM Studio u Ollama. Puedes correr un modelo como Llama, Mistral o Qwen en tu propia máquina y apuntar OpenClaw a tu endpoint local. El resultado: cero costo por token y sin dependencia de servicios externos. La contrapartida es que necesitas hardware capaz y que tu máquina esté encendida mientras uses el agente.

**¿Puedo cambiar el modelo de IA que configuré inicialmente?**

Sí. No estás atado al proveedor que elegiste durante el setup wizard. Puedes modificar la configuración de OpenClaw editando el archivo de variables de entorno del contenedor y reiniciándolo. Esto te permite migrar, por ejemplo, de Gemini Flash a Claude o a un modelo local sin reinstalar nada.

> [VERIFICAR: ruta exacta del archivo de configuración donde se define el proveedor/modelo en tu instalación]

## Referencias

- [Repositorio oficial de OpenClaw](https://github.com/openclaw/openclaw)
- [Documentación de instalación — Hostinger](https://www.hostinger.com/tutorials/openclaw)
- [Google AI Studio (Gemini API)](https://aistudio.google.com)
