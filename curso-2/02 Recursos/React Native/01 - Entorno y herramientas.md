---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Entorno, herramientas y estructura de un proyecto Expo
---
# 01 · Entorno y herramientas

## Herramientas

| Herramienta | Qué es | Ejemplo |
|---|---|---|
| Node.js | Ejecuta JavaScript/TypeScript fuera del navegador; lo necesitan las demás herramientas. | `node -v` |
| pnpm | Gestor de paquetes; en un proyecto se usa **un solo** gestor (pnpm o npm, no ambos). | `pnpm install` |
| Expo | Plataforma que crea y ejecuta proyectos React Native (móvil y web). | `pnpm create expo-app .` |
| Metro | Servidor de desarrollo de Expo: empaqueta el código y lo envía a la app. | `pnpm expo start` |
| Expo Go | App que ejecuta el proyecto en el móvil o emulador sin compilarlo. | Tecla `a` |
| SDK | Conjunto de herramientas y bibliotecas de una plataforma. | Expo SDK 57, SDK de Android |
| Android Emulator | Móvil Android virtual; no necesita el IDE Android Studio. | `emulator -avd pixel` |
| `adb` | Comunica el ordenador con el emulador o el móvil. | `adb devices` |
| Dev Container | Carpeta `.devcontainer` para trabajar dentro de un contenedor (Podman o Docker). | `Ctrl+Shift+P` → *Reopen in Container* |
| Vite | Crea proyectos de React **para la web**; no sirve para móvil. | `pnpm create vite` |

## Comandos

| Comando | Qué hace |
|---|---|
| `pnpm create expo-app .` | Crea el proyecto en la carpeta actual (debe estar vacía). |
| `pnpm run reset-project` | Borra el ejemplo de la plantilla. |
| `pnpm expo install <paquete>` | Instala la versión compatible con el SDK (mejor que `pnpm add`). |
| `pnpm expo install --fix` | Ajusta todas las dependencias al SDK. |
| `pnpm expo start` | Arranca Metro: `a` Android, `w` web, `r` recargar. |
| `pnpm expo start -c` | Arranca limpiando la caché (tras cambiar `app.json` o instalar paquetes). |
| `emulator -list-avds` | Lista los emuladores creados. |
| `adb shell getprop ro.build.version.release` | Versión de Android del emulador. |

> [!danger] Nunca `npm audit fix --force` en Expo
> Puede cambiar el SDK y romper dependencias; los avisos de `audit` suelen estar en herramientas de desarrollo, no en la app.

## Estructura del proyecto

| Elemento | Para qué | Ejemplo |
|---|---|---|
| `src/app/` | Solo pantallas: cada archivo es una ruta. | `add.tsx` → `/add` |
| `_layout.tsx` | Envuelve las pantallas y define la navegación; no es pantalla. | `<Stack />` |
| `index.tsx` | Pantalla inicial de su carpeta. | `/` |
| `[id].tsx` | Ruta dinámica. | `detail/[id].tsx` → `/detail/42` |
| `src/components/` | Piezas de interfaz reutilizables. | `PlaceCard.tsx` |
| `src/context/` | Estado compartido. | `PlacesContext.tsx` |
| `src/hooks/` | Lógica reutilizable (hooks propios). | `useLocation.ts` |
| `src/api/` | Llamadas a la red. | `discover.ts` |
| `src/data/` | Tipos y datos de ejemplo. | `places.ts` |
| `src/constants/` | Valores fijos (tema). | `theme.ts` |
| `assets/` | Imágenes y fuentes. | `assets/images/icon.png` |
| `.tsx` / `.ts` | Archivos con JSX / sin JSX (tipos, lógica, datos). | `Card.tsx`, `places.ts` |
| Alias `@/` | Apunta a `src/`; evita `../../`. | `import { usePlaces } from "@/context/PlacesContext";` |

> [!warning]
> Componentes, datos o hooks dentro de `app/` se convierten en rutas.

## Configuración

| Elemento | Para qué | Ejemplo |
|---|---|---|
| `app.json` | Nombre, iconos, permisos y plugins. | `"name": "MyPlaces", "slug": "myplaces"` |
| Plugins con opciones | Cada uno es un array propio `[nombre, opciones]`. | `["expo-location", { "locationWhenInUsePermission": "..." }]` |
| `typedRoutes` | TypeScript solo acepta rutas que existen (se regeneran con Metro en marcha). | `router.push("/add")` exige `add.tsx` |
| `tsconfig.json` → `strict` | Activa todas las comprobaciones de TypeScript. | `"strict": true` |
| Archivos por plataforma | `.web.tsx`, `.android.tsx` o `.ios.tsx` se eligen solos al importar. | `Button.android.tsx` |

## Dev Container (Podman)

```json
{
  "name": "my-project",
  "image": "docker.io/library/node:lts",
  "runArgs": ["--network=host"],
  "postCreateCommand": "npm install -g pnpm"
}
```

| Clave | Para qué |
|---|---|
| `image` | Imagen base con Node.js. |
| `--network=host` | Metro accesible en `localhost:8081`. |
| `postCreateCommand` | Instala pnpm al crear el contenedor. |
| `dev.containers.dockerPath = podman` | Ajuste de VS Code para usar Podman. |
