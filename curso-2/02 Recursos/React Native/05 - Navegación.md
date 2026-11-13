---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Navegación con Expo Router
---
# 05 · Navegación (Expo Router)

## Rutas

| Archivo | Ruta | Función |
|---|---|---|
| `app/_layout.tsx` | — | Navegador que contiene las pantallas. |
| `app/index.tsx` | `/` | Pantalla inicial. |
| `app/add.tsx` | `/add` | Ruta estática. |
| `app/detail/[id].tsx` | `/detail/1`, `/detail/2`… | Ruta dinámica. |

> [!warning]
> Una ruta sin archivo, o un archivo sin `export default`, muestra *Unmatched route*.

## Stack

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `Stack` | Navegador en pila: cada pantalla se pone encima. | `<Stack />` |
| `screenOptions` | Opciones comunes. | `screenOptions={{ headerTintColor: "#234c3b" }}` |
| `Stack.Screen` | Opciones de una pantalla; `name` sin extensión y con carpeta. | `<Stack.Screen name="detail/[id]" options={{ title: "Detalle" }} />` |
| Título dinámico | `Stack.Screen` dentro de la pantalla. | `<Stack.Screen options={{ title: place.name }} />` |
| Ocultar cabecera | `headerShown: false`. | `options={{ headerShown: false }}` |

## Navegar

| Método | Efecto | Ejemplo |
|---|---|---|
| `router.push` | Abre encima. | `router.push("/add")` |
| `push` con parámetros | Ruta dinámica. | `router.push({ pathname: "/detail/[id]", params: { id: place.id } })` |
| `router.back` | Vuelve. | `router.back()` |
| `router.canGoBack` | ¿Hay pantalla anterior? | `router.canGoBack() ? router.back() : router.replace("/")` |
| `router.replace` | Sustituye sin historial. | `router.replace("/")` |
| `useLocalSearchParams` | Lee los parámetros (**siempre texto**). | `const { id } = useLocalSearchParams<{ id: string }>();` |
| `useFocusEffect` | Efecto cada vez que la pantalla se vuelve visible. | `useFocusEffect(useCallback(() => { load(); }, []));` |

> [!tip]
> Por la ruta viaja solo el `id`; la pantalla busca los datos con él. Los ids de los datos deben ser texto.

## Link frente a router

| | `Link` | `router` |
|---|---|---|
| Estilo | Declarativo | Imperativo (en `onPress`) |
| Web | Genera un `<a href>` real | Solo navega |
| Uso | Enlaces simples | Botones; navegar tras guardar o validar |

```tsx
<Link href="/add">Añadir</Link>

<Link href="/add" asChild>
  <Pressable style={styles.button}><Text>Añadir</Text></Pressable>
</Link>
```

> [!danger] `Link asChild` + `style` como función
> Con `asChild`, el `Link` mezcla estilos con `{ ...a, ...b }`; si el `Pressable` usa `style={({ pressed }) => …}`, se pierden. Solución: `router.push` en `onPress`.

## Rutas protegidas

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `Stack.Protected` | Las pantallas de dentro solo son accesibles si `guard` es `true`; si no, redirige. | `<Stack.Protected guard={isLoggedIn}><Stack.Screen name="index" /></Stack.Protected>` |
| Renderizado condicional | Alternativa sin rutas. | `isLoggedIn ? <Home /> : <Login />` |
