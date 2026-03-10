---
title: "Taskfile: instalación y primeros pasos"
date: 2026-03-09
draft: false
tags: ["taskfile", "task", "automatizacion", "devtools"]
categories: ["DevOps", "Productividad"]
showTableOfContents: true
---

Si haces tareas repetitivas en desarrollo (instalar dependencias, correr tests, build, deploy), **Taskfile** te ayuda a automatizarlas de forma simple y legible.

<!--more-->

En este artículo te dejo una guía práctica para empezar desde cero: instalación, primeros pasos y cómo estructurar tus tareas.

## ¿Qué es Taskfile?

[Task](https://taskfile.dev/) es un _task runner_ moderno. Su configuración vive en un archivo `Taskfile.yml`, donde defines comandos reutilizables para tu proyecto.

Es similar a `Make`, pero con sintaxis YAML más amigable y enfocada en DX.

## Instalación

### macOS

Con Homebrew:

```bash
brew install go-task/tap/go-task
```

### Linux

Con script oficial:

```bash
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d
```

También puedes usar gestores como `snap`, `apt` o descargar el binario desde releases.

### Windows

Con Scoop:

```powershell
scoop install task
```

Con Chocolatey:

```powershell
choco install go-task
```

### Verificación

```bash
task --version
```

## Primeros pasos

1. Ve a la raíz del proyecto.
2. Crea un `Taskfile.yml`.
3. Define tareas comunes (`install`, `dev`, `test`, `build`).
4. Ejecuta tareas con `task <nombre>`.

Ejemplo básico:

```yaml
version: "3"

tasks:
  install:
    desc: Instalar dependencias
    cmds:
      - npm install

  dev:
    desc: Levantar entorno de desarrollo
    deps: [install]
    cmds:
      - npm run dev

  test:
    desc: Ejecutar pruebas
    cmds:
      - npm run test
```

Comandos útiles:

```bash
task           # ejecuta la tarea por defecto (si existe)
task --list    # lista tareas disponibles
task test      # ejecuta la tarea test
```

## Estructura recomendada del Taskfile

Un `Taskfile.yml` suele tener estos bloques:

- `version`: versión del esquema.
- `tasks`: mapa con todas las tareas.
- `desc`: descripción para la ayuda.
- `cmds`: comandos a ejecutar.
- `deps`: dependencias entre tareas.
- `vars`: variables reutilizables.
- `env`: variables de entorno.
- `sources` / `generates`: control de _up-to-date_ (evita ejecuciones innecesarias).

Ejemplo más completo:

```yaml
version: "3"

vars:
  APP_NAME: loan-calculator

env:
  NODE_ENV: development

tasks:
  default:
    desc: Mostrar tareas
    cmds:
      - task --list

  install:
    desc: Instalar dependencias
    cmds:
      - npm ci

  lint:
    desc: Revisar estilo y calidad
    cmds:
      - npm run lint

  test:
    desc: Ejecutar pruebas unitarias
    deps: [install]
    cmds:
      - npm run test:run

  build:
    desc: Compilar proyecto
    deps: [install]
    cmds:
      - npm run build

  ci:
    desc: Flujo de integración continua local
    deps: [lint, test, build]
```

## Buenas prácticas para tareas

- Usa nombres cortos y claros (`build`, `test`, `deploy`).
- Agrega `desc` a todas las tareas.
- Crea una tarea `ci` que ejecute el flujo completo.
- Evita duplicar comandos largos: encapsúlalos en tareas.
- Reutiliza variables para rutas, nombres y flags.

## Conclusión

Taskfile es una forma rápida de estandarizar cómo ejecutas procesos en tus proyectos.

Si trabajas en equipo, te permite que todos usen los mismos comandos y reduce errores por diferencias de entorno.

Si estás empezando, define primero 4 tareas base: `install`, `dev`, `test` y `build`. Desde ahí puedes crecer hacia flujos más avanzados como `release` o `deploy`.

## Recursos

- Sitio oficial: https://taskfile.dev/
- Documentación: https://taskfile.dev/docs/guide
- Repositorio: https://github.com/go-task/task
