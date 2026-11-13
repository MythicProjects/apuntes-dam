---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Formularios, persistencia local, HTTP y conectividad
---
# 06 · Datos

## Formularios

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `TextInput` controlado | `value` del estado y `onChangeText` lo actualiza. | `<TextInput value={name} onChangeText={setName} />` |
| Estado de objeto | Un estado para todos los campos. | `useState<FormData>({ name: "", description: "" })` |
| Cambiar un campo | Clave computada + spread. | `const change = (key: keyof FormData, value: string) => setForm({ ...form, [key]: value });` |
| `placeholder` | Texto de ayuda. | `placeholder="Nombre"` |
| `multiline` | Varias líneas. | `multiline` |
| `keyboardType` | Tipo de teclado. | `keyboardType="url"` |
| `autoCapitalize` / `autoCorrect` | Desactivar en URL o correos. | `autoCapitalize="none" autoCorrect={false}` |
| `textAlignVertical` | En Android, texto arriba en un campo multilínea. | `textAlignVertical: "top"` |
| Formulario con `onSubmit` | El componente entrega los datos; la pantalla decide qué hacer. | `<PlaceForm onSubmit={save} error={error} />` |
| `Switch` | Interruptor. | `<Switch value={on} onValueChange={setOn} />` |

## Validación

| Paso | Explicación | Ejemplo |
|---|---|---|
| Limpiar | Quitar espacios. | `const name = data.name.trim();` |
| Comprobar y cortar | Error en el estado y `return`. | `if (name === "") { setError("Required"); return; }` |
| Confirmar | Diálogo nativo con botones. | `Alert.alert("Saved", "Done", [{ text: "OK", onPress: () => router.back() }]);` |
| Confirmar en la web | `Alert.alert` no funciona allí. | `if (window.confirm("Delete?")) remove(id);` |
| Id único | Texto a partir de la fecha. | `id: Date.now().toString()` |

## Eventos

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `onPress` | Pulsación completa (también `onPressIn`, `onPressOut`, `onLongPress`). | `onPress={save}` |
| `style` como función | `Pressable` pasa `{ pressed }`. | `style={({ pressed }) => [styles.button, pressed && styles.pressed]}` |
| `disabled` | Desactiva. | `disabled={loading}` |

## Persistencia (AsyncStorage)

`pnpm expo install @react-native-async-storage/async-storage`

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Memoria frente a disco | `useState` se pierde al cerrar; AsyncStorage se conserva. | — |
| AsyncStorage | Clave/valor de **texto**, sin cifrar, para datos pequeños. | `import AsyncStorage from "@react-native-async-storage/async-storage";` |
| `setItem` | Guarda. | `await AsyncStorage.setItem("places", JSON.stringify(places));` |
| `getItem` | Lee (`null` si no existe). | `const text = await AsyncStorage.getItem("places");` |
| `removeItem` | Borra una clave. | `await AsyncStorage.removeItem("places");` |
| Caso `null` | Primera ejecución. | `text ? JSON.parse(text) : []` |
| Clave constante | Igual al leer y al escribir. | `const STORAGE_KEY = "places";` |
| Guardar en cada cambio | Estado y disco con la misma lista. | `const update = (list: Place[]) => { setPlaces(list); savePlaces(list); };` |
| Secretos | Van en `expo-secure-store`. | — |
| En la web | `localStorage` del navegador. | Firefox: `F12` → Almacenamiento |

```ts
async function loadPlaces(): Promise<Place[]> {
  try {
    const text = await AsyncStorage.getItem(STORAGE_KEY);
    return text ? JSON.parse(text) : [];
  } catch {
    return [];
  }
}
```

## HTTP (fetch)

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `fetch` | Petición HTTP; devuelve una promesa. | `const response = await fetch(url);` |
| `response.ok` | `true` si 200-299; `fetch` **no** falla con 404 o 500. | `` if (!response.ok) throw new Error(`HTTP ${response.status}`); `` |
| `response.json()` | Cuerpo → objeto (también con `await`). | `const data: Post[] = await response.json();` |
| Tres estados | Cargando, error, correcto (+ vacío). | `loading`, `error`, `data` |
| `finally` | Quita el indicador siempre. | `finally { setLoading(false); }` |
| Reintentar | La función de carga se reutiliza en un botón. | `onPress={() => load()}` |
| `URLSearchParams` | Monta la URL escapando caracteres. | `` `${API}?${new URLSearchParams({ action: "query" })}` `` |
| `POST` | Método, cabecera y cuerpo JSON. | `fetch(url, { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify(data) })` |
| HTTPS | En Android, `http://` se bloquea en la versión final. | `https://…` |
| `localhost` en el emulador | Es el emulador; el ordenador es `10.0.2.2`. | `http://10.0.2.2:3000` |
| Claves privadas | Nunca en la app. | — |
| Prefijo de ids externos | Evita choques con los propios. | `` `wiki-${item.pageid}` `` |
| Evitar duplicados | Comprobar antes de añadir. | `if (places.some((place) => place.id === newId)) return;` |

```tsx
const load = async (): Promise<void> => {
  setLoading(true);
  setError(null);
  try {
    setResults(await fetchResults());
  } catch {
    setError("Could not load the results");
  } finally {
    setLoading(false);
  }
};
```

### API de Wikipedia (lugares cercanos)

| Parámetro | Aporta |
|---|---|
| `generator=geosearch`, `ggscoord`, `ggsradius` | Artículos en un radio (máx. 10 km) de unas coordenadas. |
| `prop=coordinates\|pageimages\|extracts` | Coordenadas, foto y resumen. |
| `pithumbsize`, `exintro`, `explaintext`, `exsentences` | Tamaño de foto y resumen corto en texto plano. |
| `formatversion=2` | `pages` como array. |
| `origin=*` | Permite la petición desde el navegador (CORS). |
| Sin resultados | No hay `query`: `data.query?.pages ?? []`. |

## Conectividad (NetInfo)

`pnpm expo install @react-native-community/netinfo`

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `useNetInfo` | Hook con el estado de la red, actualizado solo. | `const { isConnected, type } = useNetInfo();` |
| `isConnected` | `true`, `false` o `null` (todavía desconocido). | `if (isConnected === false) return;` |
| `type` | `wifi`, `cellular`, `ethernet`… (`unknown` en el navegador). | `` `Connected (${type})` `` |
| Recargar al volver la red | Efecto que depende de la conexión. | `useEffect(() => { if (isConnected !== null) load(); }, [isConnected]);` |
