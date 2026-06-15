---
title: "Subdominios: qué son, para qué sirven y cómo los uso en mi servidor"
date: 2026-06-14
draft: true
tags: [redes, dns, devops, homelab, caddy]
categories: [Tutorial]
description: "Aprende qué es un subdominio, cuándo usarlo en lugar de una subcarpeta y cómo está organizado el servidor de R&D."
---

Si tienes un servidor propio corriendo servicios —o estás pensando en montarlo— tarde o temprano te vas a topar con la pregunta: *¿cómo hago para que esto se vea desde mi dominio?* Este artículo responde exactamente eso, sin asumir que estudiaste redes ni gestión de servidores.

## Primero lo primero: ¿qué es un dominio?

Un dominio es la dirección legible de tu sitio en internet. En mi caso, `yoshidev22.com`. Detrás de esa dirección hay una IP real (la de mi VPS), pero el sistema DNS se encarga de traducir el nombre bonito a la dirección numérica para que los navegadores sepan a dónde conectarse.

Hasta ahí, todo normal.

## Las tres formas de exponer una app desde tu dominio

Imagina que tu dominio es un edificio. Tienes una dirección única (`yoshidev22.com`) y quieres alojar varias apps distintas ahí. Hay tres formas de hacerlo, y cada una tiene implicaciones diferentes.

### 1. Subcarpeta (o subpath)

`yoshidev22.com/portainer`

Es como poner una habitación dentro de tu piso. La app vive dentro de la misma dirección principal, simplemente en una "carpeta".

**Ventajas:** fácil de configurar si la app lo soporta, no necesitas tocar DNS.

**Desventajas:** muchas apps no están diseñadas para correr desde un subpath (esperan estar en la raíz `/`), lo cual genera errores de rutas rotas, assets que no cargan, etc. Además, si tienes muchas apps, la URL se vuelve larga y poco clara.

### 2. Subdominio

`dock.yoshidev22.com`

Es como darle un piso propio a cada app dentro del mismo edificio. El piso tiene su propia puerta, su propio número, pero sigue siendo parte del mismo inmueble.

**Ventajas:** cada app tiene su URL limpia, HTTPS independiente, y el reverse proxy puede distinguir fácilmente a cuál servicio dirigir cada petición. Es el estándar en homelab y en producción.

**Desventajas:** requiere crear un registro DNS por cada subdominio (aunque esto toma literalmente dos minutos). Si abusas, puedes acumular decenas de subdominios difíciles de mantener.

### 3. Dominio propio separado

`portainer.io`, `n8n.io`

Es como construir un edificio nuevo para cada app. Funciona, pero implica comprar y gestionar dominios adicionales. Para uso personal o de homelab, casi nunca tiene sentido.

---

### Tabla comparativa

| | Subcarpeta | Subdominio | Dominio separado |
|---|---|---|---|
| Dificultad de setup | Baja\* | Media | Alta |
| HTTPS automático | Compartido | Independiente | Independiente |
| Compatibilidad con apps | Variable | Alta | Alta |
| Orden y claridad | Baja | Alta | Alta |
| Coste adicional | Ninguno | Ninguno | Sí (dominio extra) |

\*Baja solo si la app lo soporta bien. En la práctica puede ser más frustrante que un subdominio.

---

## ¿Cómo funciona un subdominio por dentro?

Cuando escribes `dock.yoshidev22.com` en el navegador, pasan dos cosas en orden:

**Paso 1 — DNS:** Tu navegador consulta al sistema DNS: *"¿a qué IP apunta `dock.yoshidev22.com`?"*. En el panel de tu proveedor de dominio habrás creado un **registro A** apuntando ese subdominio a la IP de tu servidor. El DNS responde con esa IP.

**Paso 2 — Reverse proxy:** Tu servidor recibe la petición. El reverse proxy (en mi caso, Caddy) lee la cabecera HTTP que dice *"el cliente quiere `dock.yoshidev22.com`"* y, según su configuración, redirige esa petición al contenedor de Portainer corriendo en el puerto correspondiente.

El navegador nunca sabe nada de puertos ni contenedores. Solo ve una URL limpia con HTTPS.

## Mis subdominios en la práctica

Así está organizado actualmente el servidor de R&D:

| Subdominio | Servicio | Para qué lo uso |
|---|---|---|
| `dock.yoshidev22.com` | Portainer | Gestión visual de contenedores Docker |
| `n8n.yoshidev22.com` | n8n | Automatización de flujos y workflows |
| `iot.yoshidev22.com` | Dashboard IoT | Monitoreo de dispositivos *(próximamente)* |

Cada uno tiene su propio certificado TLS gestionado automáticamente por Caddy vía Let's Encrypt. No hay que renovar nada manualmente.

## ¿Cuándo usar cada opción?

Regla rápida:

- **Subcarpeta** → si la app lo soporta bien y quieres evitar tocar DNS (casos raros).
- **Subdominio** → siempre que puedas. Es el estándar por algo.
- **Dominio separado** → si el proyecto es lo suficientemente grande como para merecer su propia identidad y presupuesto.

Para un homelab o un servidor personal, **la respuesta casi siempre es subdominio**.

## Una nota de seguridad

Si te animas a montar tus propios subdominios, recuerda siempre mantenerlos protegidos con autenticación. Un servicio expuesto en internet sin contraseña es una invitación abierta. Da igual que "nadie sepa la URL": los scanners automatizados encuentran subdominios en segundos. **Activa la autenticación en cada servicio antes de exponerlo públicamente.**

---

En una próxima entrada veremos cómo configurar concretamente un subdominio en Caddy, paso a paso, con el Caddyfile real.
