---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Errores frecuentes en React Native + Expo + TypeScript
---
# Errores frecuentes

> [!tip] Leer la pantalla roja
> La primera línea dice **qué** falla; en *Call Stack* / *Component Stack*, el primer nombre que no está en gris es **tu** archivo.

## Mensajes

| Mensaje | Causa | Solución |
|---|---|---|
| *Text strings must be rendered within a `<Text>`* | Texto suelto en un `View` (o `""` con `&&`). | `<Text>` o `? : null`. |
| *Cannot find module '…'* | Ruta mal (`./` / `../`) o archivo inexistente. | Revisar la ruta o usar `@/`. |
| *Property 'x' does not exist on type '() => …'* | Falta llamar a la función. | `usePlaces()` |
| *…() is undefined* | No exportada o sin `return`. | `export function`, `return context;` |
| *Property 'find' does not exist* | Se guardó el objeto entero. | `const { places } = usePlaces();` |
| *Parameter implicitly has an 'any' type* | Parámetro sin tipo. | `(place: Place) => …` |
| *Unmatched route* | No existe el archivo o falta `export default`. | Crear la pantalla en `src/app/`. |
| *Invalid hook call* / error dentro de `useState` | Hook fuera del componente, condicional o tras un `return`. | Hooks arriba del todo. |
| *Too many re-renders* | `onPress={f()}` se ejecuta al dibujar. | `onPress={() => f()}` |
| *Disconnected from Metro* | Servidor parado. | `pnpm expo start` |
| `adb: no devices/emulators found` | Emulador apagado. | `emulator -avd pixel` |

## Síntomas

| Síntoma | Causa | Solución |
|---|---|---|
| Import vale `undefined` | Llaves con `export default`, o al revés. | Llaves solo para exports con nombre. |
| Props `undefined` | Parámetros sin llaves. | `function Card({ title }: CardProps)` |
| Imagen local falla | `require` con variable. | Ruta literal; `require` en el padre. |
| Imagen remota invisible | Sin tamaño. | `width` y `height`. |
| Pie de página no baja | Falta `flex: 1`. | En la raíz y en la zona central. |
| Estilos perdidos con `Link asChild` | `style` como función. | `router.push` en `onPress`. |
| Detalle "no encontrado" | Ids numéricos frente a parámetro de texto, o busca en la constante en lugar del estado. | Ids de texto; leer del Context. |
| Lista no se actualiza al volver | `useEffect([])` no se repite en un `Stack`. | Context o `useFocusEffect`. |
| La interfaz no cambia | Estado modificado directamente. | Setter con un valor nuevo. |
| Se guarda la lista anterior | Se guarda el estado tras `setX`. | Guardar la variable nueva. |
| Datos "perdidos" al reabrir | Ventana privada, otra dirección u otro `slug`. | Abrir siempre igual. |
| `Alert.alert` no aparece | Se prueba en la web. | Emulador o `window.confirm`. |
| La alerta no se ve al volver | `router.back()` a la vez que la alerta. | `router.back()` en el botón de la alerta. |
| `fetch` "funciona" con un 404 | No se comprueba `response.ok`. | `if (!response.ok) throw …` |
| Indicador de carga colgado | `setLoading(false)` solo en el `try`. | En `finally`. |
| Sombra no aparece en Android | `shadow*` es solo de iOS. | `boxShadow` o `elevation`. |
| Espacio perdido en el texto | Salto de línea en JSX. | `{" "}` |
| Plugin ignorado en `app.json` | Plugin metido dentro de otro array. | Cada plugin, su `[nombre, opciones]`. |
| `create-expo-app` se niega | Carpeta no vacía. | Crear en una subcarpeta y mover. |
| SDK cambiado | `npm audit fix --force`. | Restaurar la versión y `pnpm expo install --fix`. |
