---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Permisos, ubicación y sensores
---
# 07 · Dispositivo

## Permisos

| Regla | Explicación |
|---|---|
| Pedir antes de usar | Primero el permiso; solo si se concede, la función. |
| Explicar para qué | Texto del permiso en `app.json`. |
| Si se deniega | Mensaje y la app sigue funcionando. |

## Ubicación (`expo-location`)

`pnpm expo install expo-location`

| Función | Para qué | Ejemplo |
|---|---|---|
| `import * as Location` | Importa todo el módulo con un nombre. | `import * as Location from "expo-location";` |
| `requestForegroundPermissionsAsync` | Pide permiso; devuelve `{ status }`. | `const { status } = await Location.requestForegroundPermissionsAsync();` |
| `getCurrentPositionAsync` | Posición actual. | `const position = await Location.getCurrentPositionAsync({ accuracy: Location.Accuracy.Balanced });` |
| `position.coords` | Latitud y longitud. | `position.coords.latitude` |
| `Accuracy.Balanced` | Equilibrio entre precisión y batería. | — |
| Plugin en `app.json` | Texto del permiso. | `["expo-location", { "locationWhenInUsePermission": "…" }]` |
| Simular | Emulador: `⋯` → Location → Set location. | — |
| Reiniciar permiso | Ajustes → Apps → Expo Go → Permisos. | — |

```ts
type Coords = { latitude: number; longitude: number };

const getLocation = async (): Promise<void> => {
  try {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== "granted") {
      setMessage("Permission denied");
      return;
    }
    const { coords } = await Location.getCurrentPositionAsync({
      accuracy: Location.Accuracy.Balanced,
    });
    setCoords({ latitude: coords.latitude, longitude: coords.longitude });
  } catch {
    setMessage("Could not get the location");
  } finally {
    setLoading(false);
  }
};
```

## Sensores (`expo-sensors`)

`pnpm expo install expo-sensors`

| Función | Para qué | Ejemplo |
|---|---|---|
| `isAvailableAsync` | ¿Existe el sensor? (en un navegador de escritorio, no). | `await Accelerometer.isAvailableAsync()` |
| `setUpdateInterval` | Milisegundos entre lecturas. | `Accelerometer.setUpdateInterval(200);` |
| `addListener` | Recibe `{ x, y, z }` en cada lectura; devuelve una suscripción. | `const subscription = Accelerometer.addListener(setReading);` |
| `remove` | Cancela al salir (batería). | `subscription.remove();` |
| Tipo de la suscripción | `ReturnType` de `addListener`. | `ReturnType<typeof Accelerometer.addListener>` |
| Unidades | *g*; en reposo, fuerza total ≈ 1. | `Math.sqrt(x ** 2 + y ** 2 + z ** 2)` |
| Simular | Emulador: `⋯` → Virtual sensors → Device pose. | — |

```tsx
useEffect(() => {
  let subscription: ReturnType<typeof Accelerometer.addListener> | null = null;
  let cancelled = false;

  Accelerometer.isAvailableAsync().then((isAvailable: boolean) => {
    if (cancelled || !isAvailable) return;
    subscription = Accelerometer.addListener(setReading);
  });

  return () => {
    cancelled = true;
    subscription?.remove();
  };
}, []);
```

## Emulador o dispositivo real

| | Emulador | Dispositivo real |
|---|---|---|
| Ventajas | Rápido, varias configuraciones, sensores y ubicación simulados | Sensores, permisos y rendimiento reales |
| Inconvenientes | No reproduce todo el *hardware* | Más configuración |
