---
title: "Primeros pasos con OpenClaw: instalación, uso básico y seguridad"
date: 2026-03-15
draft: false
tags: ["openclaw", "ai", "automation", "seguridad"]
categories: ["AI", "Desarrollo", "Productividad"]
showTableOfContents: true
---
Si quieres empezar con OpenClaw sin perderte en la configuración inicial, esta guía te lleva desde cero: instalación, uso básico diario y recomendaciones de seguridad para operar con confianza.

<!--more-->

## ¿Qué es OpenClaw?

OpenClaw es una plataforma para ejecutar agentes de IA conectados a herramientas reales (archivos, terminal, web, mensajería, etc.), con memoria persistente y reglas de seguridad.

Piensa en OpenClaw como un asistente operativo, no solo conversacional.

## Requisitos previos

Antes de instalar:

- Linux, macOS o Windows (WSL recomendado en Windows para entornos de desarrollo)
- Node.js 22+
- npm o pnpm
- Acceso a una cuenta de modelo (por ejemplo OpenAI)

## Instalación rápida

Instala OpenClaw de forma global:

```bash
npm install -g openclaw
```

Verifica la instalación:

```bash
openclaw --version
```

Ejecuta el onboarding inicial:

```bash
openclaw onboard
```

Este asistente configura elementos clave como autenticación, workspace y servicios base.

## Configuración inicial recomendada

### 1) Verificar estado general

```bash
openclaw status --deep
```

Te permite validar rápidamente:

- estado del gateway
- estado de canales
- sesión activa
- alertas de seguridad

### 2) Revisar actualización disponible

```bash
openclaw update status
```

### 3) Ejecutar auditoría de seguridad

```bash
openclaw security audit --deep
```

## Uso básico diario

### Iniciar/reiniciar gateway

```bash
openclaw gateway start
openclaw gateway restart
```

### Ver logs en vivo

```bash
openclaw logs --follow
```

### Enviar un mensaje de prueba desde CLI

```bash
openclaw message send --channel telegram --target <chat_id> --message "Hola desde OpenClaw"
```

### Trabajar con sesiones

```bash
openclaw sessions list
```

## Estructura base del workspace

OpenClaw usa un workspace por agente para mantener continuidad.

Archivos comunes:

- `SOUL.md`: estilo/voz del agente
- `USER.md`: contexto del usuario
- `MEMORY.md`: memoria de largo plazo
- `memory/YYYY-MM-DD.md`: memoria diaria
- `HEARTBEAT.md`: tareas proactivas periódicas

Recomendación práctica: mantén `USER.md` y `MEMORY.md` actualizados para respuestas más útiles y consistentes.

## Recomendaciones de seguridad (importante)

### 1) Mantén bind local por defecto

Usa `loopback` cuando no necesites exponer la consola fuera del host.

```bash
openclaw config get gateway.bind
```

Si necesitas acceso remoto, prefiere túnel SSH antes que exposición directa de puertos.

### 2) Evita ejecutar con permisos elevados sin necesidad

El modo elevated debe usarse con criterio y solo para tareas realmente necesarias.

### 3) Audita con frecuencia

Corre periódicamente:

```bash
openclaw security audit --deep
```

### 4) Mantén OpenClaw y sistema operativo actualizados

- `openclaw update status`
- actualizaciones del sistema (`apt`, `dnf`, etc.)

### 5) Usa políticas de acceso claras en canales

Define `dmPolicy`, `groupPolicy`, allowlists y pairing según tu modelo de riesgo.

### 6) No expongas secretos en texto plano

Tokens y credenciales deben guardarse de forma segura y nunca publicarse en repositorios.

## Flujo recomendado para producción personal

1. Instalar OpenClaw
2. Ejecutar `onboard`
3. Validar con `status --deep`
4. Correr `security audit --deep`
5. Configurar canales y políticas mínimas
6. Probar en entorno controlado
7. Activar automatizaciones (cron/heartbeats)

## Errores comunes al iniciar

- **"access not configured" en Telegram**: falta aprobar pairing o policy restrictiva.
- **No puedo usar herramientas del host**: probablemente sesión en sandbox (revisar `agents.defaults.sandbox.mode`).
- **Comandos no encontrados**: entorno aislado o PATH incompleto.

## Conclusión

OpenClaw tiene una curva de entrada corta si empiezas por lo esencial: instalación limpia, validaciones de estado y seguridad desde el primer día.

Si configuras bien tu workspace y tus políticas de acceso, tendrás un asistente potente y confiable para automatizar tareas reales sin sacrificar control.

## Recursos

- Documentación: https://docs.openclaw.ai
- Repositorio: https://github.com/openclaw/openclaw
- Comunidad: https://discord.com/invite/clawd
