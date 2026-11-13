---
tags:
  - programacion-multiplataforma
  - DAM2
unidad: 2
tema: Práctica MyPlaces · Documentación del procedimiento
---
# MyPlaces · Documentación del procedimiento

Aplicación móvil con **React Native + Expo + TypeScript** para consultar, añadir y guardar lugares de interés. Incluye navegación entre pantallas, formulario validado, persistencia local, consulta a una API pública, ubicación con permisos, estado de la conexión y varias ampliaciones (accesibilidad, coordenadas por lugar, acelerómetro y favoritos).

## Estructura final del proyecto

```
src/
├── api/
│   └── discover.ts          ← API query (fetch)
├── app/                     ← screens (one file = one route)
│   ├── _layout.tsx          ← Stack + PlacesProvider
│   ├── index.tsx            ← place list
│   ├── add.tsx              ← add place
│   ├── discover.tsx         ← API results + favourites
│   ├── sensors.tsx          ← accelerometer
│   └── detail/
│       └── [id].tsx         ← place detail (dynamic route)
├── components/
│   ├── PlaceCard.tsx        ← reusable card
│   ├── PlaceForm.tsx        ← form
│   ├── LocationButton.tsx   ← "Usa mi ubicación"
│   └── ConnectionStatus.tsx ← connection bar
├── context/
│   └── PlacesContext.tsx    ← shared state + AsyncStorage
├── data/
│   └── places.ts            ← Place type (+ sample data)
└── hooks/
    └── useLocation.ts       ← permission + location logic
```

> [!important] Regla de organización
> En `app/` solo van **pantallas**: Expo Router convierte cada archivo de esa carpeta en una ruta. Componentes, datos, lógica y llamadas a la API van fuera de `app/`.

---

## 0 - Preparación

El proyecto funcionará dentro de un contenedor Podman. Este paso es opcional, pero útil para evitar instalaciones innecesarias y posibles problemas de seguridad en el equipo. Para ello se crea la carpeta `.devcontainer` con el archivo `devcontainer.json` y la imagen necesaria para ejecutar pnpm:

```json
{
  "name": "ex02-my-places",
  "image": "docker.io/library/node:lts",
  "runArgs": ["--network=host"],
  "postCreateCommand": "npm install -g pnpm",
  "customizations": {
    "vscode": {
      "extensions": [
        "expo.vscode-expo-tools",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode"
      ]
    }
  }
}
```

| Opción | Función |
|---|---|
| `image` | Imagen oficial de Node.js (LTS) como base del contenedor |
| `--network=host` | El contenedor comparte la red del equipo: Metro queda accesible en `localhost:8081` |
| `postCreateCommand` | Instala pnpm la primera vez que se crea el contenedor |
| `extensions` | Extensiones de VS Code que se instalan dentro del contenedor |

> [!info] Podman en lugar de Docker
> La extensión *Dev Containers* usa Docker por defecto. Para usar Podman se configura en los ajustes de VS Code `dev.containers.dockerPath` con el valor `podman`.

Una vez creado, el contenedor se ejecuta desde VS Code con la extensión *Dev Containers*: `Ctrl + Shift + P` → **Rebuild and Reopen in Container**.

[[ IMAGEN: terminal de VS Code dentro del contenedor, con el prompt `root@...:/workspaces/...` ]]

Con el contenedor configurado, se crea el proyecto:

```bash
pnpm create expo-app .
```

> [!warning] Carpeta no vacía
> `create-expo-app` se niega a crear el proyecto en una carpeta con archivos (por ejemplo, si `.devcontainer` está dentro). En ese caso se crea en una subcarpeta y se mueve:
> ```bash
> pnpm create expo-app tmp --no-install
> cp -a tmp/. .
> rm -rf tmp
> pnpm install
> ```

Para eliminar el proyecto de ejemplo de la plantilla:

```bash
pnpm run reset-project
```

En `app.json` se ajusta el nombre del proyecto:

```json
"name": "MyPlaces",
"slug": "myplaces",
"scheme": "myplaces",
```

> [!danger] No usar `npm audit fix --force`
> En un proyecto Expo, `--force` puede cambiar la versión del SDK (por ejemplo, de la 57 a la 53) y dejar dependencias incompatibles. Las vulnerabilidades que muestra `npm audit` están en herramientas de desarrollo, no en la app instalada. Además, se usa un solo gestor de paquetes (pnpm) y se instalan los paquetes con `pnpm expo install`, que elige la versión compatible con el SDK.

Antes de empezar a construir la aplicación se crea `src/data/places.ts`, con el tipo `Place` y unos datos de ejemplo:

```ts
// src/data/places.ts
export type Place = {
  id: string; // text: route parameters always arrive as text
  name: string;
  description: string;
  image?: string;
  latitude?: number;
  longitude?: number;
};

export const samplePlaces: Place[] = [
  {
    id: "1",
    name: "Great Wall of China",
    description: "A historic fortification in China.",
    image:
      "https://upload.wikimedia.org/wikipedia/commons/thumb/f/fa/Great_Wall_of_China_July_2006.JPG/960px-Great_Wall_of_China_July_2006.JPG",
  },
  // ...
];
```

> [!tip] Identificadores de texto
> Los parámetros de ruta de Expo Router llegan siempre como `string`. Si los ids fueran números, `place.id === id` nunca se cumpliría.

Para arrancar la aplicación:

```bash
pnpm expo start
```

Y se pulsa `a` (emulador de Android) o `w` (navegador).

---

## A - Navegación y componentes

La aplicación usa **Expo Router**: cada archivo de `src/app/` es una pantalla. Se crean la lista (`index.tsx`), el detalle con ruta dinámica (`detail/[id].tsx`) y un componente reutilizable para las tarjetas.

### Componente reutilizable: `PlaceCard`

La tarjeta recibe el lugar y la acción al pulsarla por *props*. No sabe adónde lleva: eso lo decide la pantalla, y por eso puede reutilizarse en cualquier lista.

```tsx
// src/components/PlaceCard.tsx
type PlaceCardProps = {
  place: Place;
  onPress: () => void;
};

export default function PlaceCard({ place, onPress }: PlaceCardProps) {
  return (
    <Pressable onPress={onPress} accessibilityRole="button" accessibilityLabel={`Ver ${place.name}`}>
      {place.image && <Image source={{ uri: place.image }} style={styles.image} />}
      <Text style={styles.name}>{place.name}</Text>
      <Text numberOfLines={1}>{place.description}</Text>
    </Pressable>
  );
}
```

> [!warning] Imágenes remotas
> Una imagen remota (`source={{ uri: ... }}`) necesita `width` y `height` en el estilo; sin tamaño no se muestra. `place.image && ...` evita dibujar una imagen vacía cuando el lugar no la tiene.

### Lista: `index.tsx`

Un `FlatList` dibuja una `PlaceCard` por lugar. Al pulsar, se navega al detalle pasando el `id`:

```tsx
<FlatList
  data={places}
  keyExtractor={(place) => place.id}
  renderItem={({ item }) => (
    <PlaceCard
      place={item}
      onPress={() => router.push({ pathname: "/detail/[id]", params: { id: item.id } })}
    />
  )}
/>
```

> [!danger] `Link asChild` con estilo como función
> Con `<Link asChild>`, Expo Router mezcla su estilo con el del hijo haciendo `{ ...linkStyle, ...childStyle }`. Si el `Pressable` usa `style={({ pressed }) => [...]}`, la mezcla produce un objeto vacío y **se pierden los estilos**. Por eso se navega con `router.push` desde `onPress`.

[[ IMAGEN: pantalla principal con la lista de lugares, cada tarjeta con su foto, nombre y descripción ]]

### Detalle: `detail/[id].tsx`

Los corchetes del nombre de archivo indican una **ruta dinámica**: un único archivo sirve para `/detail/1`, `/detail/2`… La pantalla lee el parámetro y busca el lugar:

```tsx
const { id } = useLocalSearchParams<{ id: string }>();
const place = places.find((place) => place.id === id);
```

Si `find` no encuentra nada devuelve `undefined`, y la pantalla muestra un mensaje en lugar de fallar:

```tsx
{place ? (
  <>{/* image, name, description, delete button */}</>
) : (
  <Text>No existe ningún lugar con este identificador.</Text>
)}
```

> [!tip] Comprobar el caso "no encontrado"
> Se cambia temporalmente el `id` del `router.push` por `"999"` y se comprueba que aparece el mensaje.

El botón **Volver** contempla que la pantalla se haya abierto sin una anterior (por un enlace directo):

```tsx
const goBack = () => {
  if (router.canGoBack()) router.back();
  else router.replace("/");
};
```

**Extra:** el nombre del lugar se muestra en la cabecera colocando un `Stack.Screen` dentro de la propia pantalla:

```tsx
<Stack.Screen options={{ title: place ? place.name : "No encontrado" }} />
```

[[ IMAGEN: detalle de un lugar con la foto grande, el nombre en la cabecera y los botones Eliminar y Volver ]]

[[ IMAGEN: pantalla "No encontrado" al abrir un id inexistente ]]

### Layout: `_layout.tsx`

Define el navegador (`Stack`) y las opciones de cada pantalla. Las rutas se escriben sin extensión y con su carpeta:

```tsx
<Stack>
  <Stack.Screen name="index" options={{ title: "MyPlaces" }} />
  <Stack.Screen name="detail/[id]" options={{ title: "Detalle" }} />
</Stack>
```

> [!warning]
> `name="detail[id]"` (sin la barra) no corresponde a ninguna ruta y sus opciones no se aplican.

---

## B - Formulario y validación

Para que un lugar nuevo aparezca en la lista, la lista no puede ser una constante: tiene que ser **estado compartido** entre pantallas. Se crea un **Context**.

### Estado compartido: `PlacesContext`

```tsx
// src/context/PlacesContext.tsx
const PlacesContext = createContext<PlacesContextValue | null>(null);

export function PlacesProvider({ children }: { children: ReactNode }) {
  const [places, setPlaces] = useState<Place[]>([]);
  const addPlace = (place: Place) => setPlaces([...places, place]);
  // ...
  return (
    <PlacesContext.Provider value={{ places, addPlace }}>{children}</PlacesContext.Provider>
  );
}

export function usePlaces() {
  const context = useContext(PlacesContext);
  if (!context) throw new Error("usePlaces must be used inside PlacesProvider");
  return context;
}
```

El *Provider* envuelve el `Stack` en `_layout.tsx`, y cualquier pantalla accede a los datos con `usePlaces()`.

> [!danger] Errores habituales con el hook
> - Olvidar el `return context` en `usePlaces`: la función devuelve `undefined` y falla `const { places } = usePlaces()`.
> - Olvidar los paréntesis: `usePlaces` es la función; `usePlaces()` es su resultado. El error de TypeScript lo delata: `Property 'places' does not exist on type '() => PlacesContextValue'`.
> - Olvidar las llaves: `const places = usePlaces()` guarda el objeto entero `{ places, addPlace }`, que no tiene `find`.

### Formulario: `PlaceForm`

El formulario **solo recoge datos**: no valida ni guarda. Los entrega a la pantalla con `onSubmit`, y recibe el error ya calculado por *props*.

```tsx
export type PlaceFormData = { name: string; description: string; image: string };

const [form, setForm] = useState<PlaceFormData>({ name: "", description: "", image: "" });

// One state for every field; [key] updates only that field
const change = (key: keyof PlaceFormData, value: string) =>
  setForm({ ...form, [key]: value });

<TextInput
  placeholder="Nombre"
  value={form.name}
  onChangeText={(text) => change("name", text)}
/>
```

> [!note] TextInput controlado
> El campo muestra lo que hay en el estado (`value`) y cada cambio actualiza el estado (`onChangeText`). El estado es la única fuente de verdad.

> [!warning] `error ? ... : null` en lugar de `error && ...`
> Si `error` fuera una cadena vacía, `&&` devolvería ese texto suelto dentro de un `View` y React Native fallaría con *Text strings must be rendered within a `<Text>` component*.

### Pantalla: `add.tsx`

La pantalla decide qué hacer con los datos: valida, guarda, avisa y vuelve.

```tsx
const save = (data: PlaceFormData, coords: Coords | null) => {
  const name = data.name.trim();

  if (name === "") {
    setError("El nombre es obligatorio");
    return; // do not save
  }

  addPlace({ id: Date.now().toString(), name, description: data.description.trim(), /* ... */ });

  Alert.alert("Lugar guardado", `«${name}» se ha añadido a la lista`, [
    { text: "Aceptar", onPress: () => router.back() },
  ]);
};
```

| Detalle | Motivo |
|---|---|
| `trim()` antes de comprobar | Un nombre de solo espacios cuenta como vacío |
| `return` dentro del `if` | Detiene la función: no se guarda nada |
| `Date.now().toString()` | Id único y de texto, coherente con las rutas |
| `router.back()` en el botón de la alerta | Primero se lee el aviso y después se vuelve; si se ejecutan a la vez, la alerta puede no llegar a verse |

> [!info] `Alert.alert` en la web
> En el navegador, `Alert.alert` no muestra nada. Las alertas se prueban en el emulador.

El acceso al formulario es un botón en la cabecera de la lista (`ListHeaderComponent`) que navega con `router.push("/add")`. La ruta funciona solo si existe el archivo `src/app/add.tsx` con un `export default`.

[[ IMAGEN: formulario "Añadir lugar" con el mensaje "El nombre es obligatorio" tras intentar guardar con el nombre vacío ]]

[[ IMAGEN: alerta "Lugar guardado" en el emulador ]]

---

## C - Persistencia local

Los datos de `useState` viven en memoria y se pierden al cerrar la app. Se guardan con **AsyncStorage**, un almacén clave/valor que solo guarda texto.

```bash
pnpm expo install @react-native-async-storage/async-storage
```

Toda la persistencia se concentra en `PlacesContext.tsx`; las pantallas no saben que AsyncStorage existe.

```tsx
const STORAGE_KEY = "places";

// Read the saved list; empty list if there is nothing or it is corrupt
async function loadPlaces(): Promise<Place[]> {
  try {
    const text = await AsyncStorage.getItem(STORAGE_KEY);
    return text ? JSON.parse(text) : [];
  } catch {
    return [];
  }
}

async function savePlaces(places: Place[]) {
  try {
    await AsyncStorage.setItem(STORAGE_KEY, JSON.stringify(places));
  } catch (error) {
    console.warn("Could not save places", error);
  }
}
```

Dentro del *Provider*: se carga una vez al arrancar y se guarda la lista completa en cada cambio.

```tsx
const [places, setPlaces] = useState<Place[]>([]); // starts empty

useEffect(() => {
  loadPlaces().then(setPlaces);
}, []);

const update = (updated: Place[]) => {
  setPlaces(updated);
  savePlaces(updated);
};

const addPlace = (place: Place) => update([...places, place]);
const removePlace = (id: string) => update(places.filter((place) => place.id !== id));
```

| Detalle | Motivo |
|---|---|
| `JSON.stringify` / `JSON.parse` | AsyncStorage solo guarda texto |
| `text ? JSON.parse(text) : []` | La primera vez `getItem` devuelve `null` |
| Se guarda `updated` y no `places` | Tras `setPlaces`, `places` conserva el valor anterior hasta el siguiente render |
| Lista inicial vacía | El enunciado pide gestionar una lista inicialmente vacía |
| Carga en el Context | La lista se actualiza al volver del formulario sin `useFocusEffect` |

> [!danger] `useEffect` no admite una función `async` directamente
> `useEffect(async () => ...)` es incorrecto. Se usa `loadPlaces().then(setPlaces)` o se declara la función `async` dentro del efecto.

### Eliminar

En el detalle, el botón **Eliminar** pide confirmación antes de llamar a `removePlace`. En la web se usa `window.confirm`, porque `Alert.alert` no funciona allí:

```tsx
if (Platform.OS === "web") {
  if (window.confirm(`¿Eliminar «${place.name}»?`)) { removePlace(place.id); goBack(); }
  return;
}
Alert.alert("Eliminar lugar", `¿Eliminar «${place.name}»?`, [
  { text: "Cancelar", style: "cancel" },
  { text: "Eliminar", style: "destructive", onPress: () => { removePlace(place.id); goBack(); } },
]);
```

> [!warning]
> El botón **Eliminar** va dentro de la rama `place ? (...)`: fuera de ella, `place` puede ser `undefined` y TypeScript marca `place.name` como error.

### Lista vacía

```tsx
ListEmptyComponent={<Text style={styles.empty}>Todavía no hay lugares guardados.</Text>}
```

[[ IMAGEN: lista vacía con el mensaje "Todavía no hay lugares guardados." ]]

[[ IMAGEN: prueba de persistencia: lista con un lugar antes de cerrar la app y la misma lista tras reabrirla ]]

> [!tip] Dónde se guardan los datos
> - **Web:** en el `localStorage` del navegador (Firefox: `F12` → **Almacenamiento** → **Almacenamiento local**).
> - **Android:** en una base de datos interna de la app. Para revisarlo, un `console.log` temporal en `loadPlaces` muestra en la terminal de Expo lo que se ha leído.

> [!warning] Datos que "desaparecen"
> Si los datos no se conservan, el código suele estar bien y la causa es el entorno: ventana privada o borrado de datos al cerrar en Firefox, abrir la app desde direcciones distintas (`localhost` frente a una IP) o cambiar el `slug` en `app.json`.

---

## D - Consulta HTTP/HTTPS

Pantalla **Descubre** que consulta una API pública sin claves: **JSONPlaceholder** (`https://jsonplaceholder.typicode.com/posts`).

La petición va en una función auxiliar, separada de la pantalla:

```ts
// src/api/discover.ts
export type Post = { id: number; title: string; body: string };

export async function fetchPosts(): Promise<Post[]> {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  if (!response.ok) throw new Error(`HTTP error ${response.status}`);
  return response.json();
}
```

> [!danger] `fetch` no falla con los errores HTTP
> Con un `404` o un `500`, `fetch` se resuelve con normalidad y `response.ok` vale `false`. Hay que comprobarlo y lanzar el error a mano.

La pantalla gestiona los tres estados de toda consulta remota:

```tsx
const [posts, setPosts] = useState<Post[]>([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState<string | null>(null);

const load = async () => {
  setLoading(true);
  setError(null);
  try {
    setPosts(await fetchPosts());
  } catch {
    setError("No se han podido cargar los resultados. Comprueba la conexión.");
  } finally {
    setLoading(false); // always, with or without error
  }
};
```

| Estado | Qué se muestra |
|---|---|
| Cargando | `ActivityIndicator` y "Cargando…" |
| Error | Mensaje y botón **Volver a intentar** (que vuelve a llamar a `load`) |
| Correcto | Lista de resultados |
| Vacío | `ListEmptyComponent`: "No hay resultados" |

> [!tip] Forzar cada estado
> - Error: cambiar temporalmente la URL a `.../postsx` (devuelve 404).
> - Vacío: cambiar temporalmente la URL a `.../posts?userId=999`.

> [!info]
> El acceso a **Descubre** es un segundo botón en la cabecera de la lista. Como `ListHeaderComponent` admite un único elemento, los botones se envuelven en un `View` con `gap`.

[[ IMAGEN: pantalla Descubre cargando ]]

[[ IMAGEN: pantalla Descubre con la lista de resultados ]]

[[ IMAGEN: pantalla Descubre con el mensaje de error y el botón Volver a intentar ]]

---

## E - Ubicación y permisos

```bash
pnpm expo install expo-location
```

El permiso se declara en `app.json`. Cada plugin con opciones es un array propio `[nombre, opciones]`:

```json
"plugins": [
  "expo-router",
  ["expo-splash-screen", { "...": "..." }],
  [
    "expo-location",
    { "locationWhenInUsePermission": "MyPlaces usa tu ubicación para mostrar tus coordenadas." }
  ]
]
```

La lógica de permiso y lectura está en un **hook propio**, para reutilizarla en varios componentes:

```ts
// src/hooks/useLocation.ts
const getLocation = async () => {
  setLoading(true);
  setMessage(null);
  try {
    // 1. Ask for permission first
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== "granted") {
      setMessage("Permiso de ubicación denegado. Puedes seguir usando la app con normalidad.");
      return;
    }
    // 2. Only then read the position
    const position = await Location.getCurrentPositionAsync({
      accuracy: Location.Accuracy.Balanced,
    });
    setCoords({ latitude: position.coords.latitude, longitude: position.coords.longitude });
  } catch {
    setMessage("No se ha podido obtener la ubicación.");
  } finally {
    setLoading(false);
  }
};

return { coords, message, loading, getLocation };
```

`LocationButton` lo usa y muestra el resultado:

```tsx
const { coords, message, loading, getLocation } = useLocation();
```

| Caso | Resultado |
|---|---|
| Permiso concedido | Latitud y longitud con 5 decimales |
| Permiso denegado | Mensaje; el resto de la app sigue funcionando |

> [!warning] JSX y saltos de línea
> En JSX, un salto de línea entre un texto y una `{expresión}` se elimina. Si Prettier parte `Lon: {...}` en dos líneas, se añade `{" "}` para conservar el espacio.

> [!tip] Simular la ubicación en el emulador
> `⋯` (*Extended controls*) → **Location** → **Set location**. Para volver a probar tras denegar: Ajustes → Apps → Expo Go → Permisos → Ubicación.

[[ IMAGEN: coordenadas mostradas tras conceder el permiso ]]

[[ IMAGEN: mensaje de permiso denegado ]]

---

## F - Conectividad

```bash
pnpm expo install @react-native-community/netinfo
```

`useNetInfo()` es un hook que se actualiza solo cuando cambia la red. `isConnected` vale `true`, `false` o `null` (todavía desconocido).

```tsx
// src/components/ConnectionStatus.tsx
const { isConnected, type } = useNetInfo();
if (isConnected === null) return null;

<View style={[styles.bar, isConnected ? styles.online : styles.offline]}>
  <Text>{isConnected ? `Conectado (${type})` : "Sin conexión"}</Text>
</View>
```

En **Descubre**, la consulta no se lanza sin conexión y se repite sola al recuperarla:

```tsx
const load = async () => {
  if (isConnected === false) {
    setError("Sin conexión: no se puede consultar la API.");
    setLoading(false);
    return;
  }
  // ...
};

useEffect(() => {
  if (isConnected !== null) load();
}, [isConnected]);
```

> [!info]
> Se compara con `null` y `false` de forma explícita porque `!isConnected` también sería cierto mientras el estado es desconocido.

> [!tip] Probar sin red
> - **Emulador:** desactivar Wi-Fi y datos móviles desde los ajustes rápidos.
> - **Firefox:** `Alt` → **Archivo** → **Trabajar sin conexión**, con la app ya cargada.

[[ IMAGEN: barra verde "Conectado (wifi)" en la cabecera de la lista ]]

[[ IMAGEN: barra roja "Sin conexión" y aviso en Descubre ]]

---

## Ampliaciones

### Accesibilidad

Se añaden roles y etiquetas para que un lector de pantalla (TalkBack) anuncie cada elemento con sentido:

| Prop | Uso en MyPlaces |
|---|---|
| `accessibilityRole="button"` | Todos los `Pressable` |
| `accessibilityLabel` | Botones, imágenes (`Foto de ...`) y campos del formulario |
| `accessibilityHint` | Tarjetas y botón Eliminar |
| `accessibilityRole="header"` | Nombre del lugar en el detalle |
| `accessibilityRole="alert"` | Mensajes de error |
| `accessibilityLiveRegion="polite"` | Coordenadas, estado de la conexión y lecturas del sensor: se anuncian al cambiar |
| `accessibilityState` | `{ busy, disabled }` en los botones mientras cargan |

> [!tip] Comprobación
> Emulador: Ajustes → Accesibilidad → **TalkBack**. Firefox: `F12` → pestaña **Accesibilidad**.

### Coordenadas por lugar

El tipo `Place` incluye `latitude?` y `longitude?`. El formulario reutiliza el hook `useLocation` con un botón **Añadir mi ubicación (opcional)** y entrega las coordenadas a la pantalla:

```tsx
onPress={() => onSubmit(form, coords)}
```

```tsx
latitude: coords?.latitude,   // undefined if there is no location
longitude: coords?.longitude,
```

El detalle las muestra si existen. Se compara con `undefined` y no con `&&`, porque una latitud de `0` es válida pero se evaluaría como falsa:

```tsx
{place.latitude !== undefined && place.longitude !== undefined ? (
  <Text>Ubicación: {place.latitude.toFixed(5)}, {place.longitude.toFixed(5)}</Text>
) : null}
```

[[ IMAGEN: detalle de un lugar con sus coordenadas guardadas ]]

### Acelerómetro

```bash
pnpm expo install expo-sensors
```

Pantalla `sensors.tsx` que comprueba si el sensor existe, se suscribe a sus lecturas y **cancela la suscripción al salir**:

```tsx
useEffect(() => {
  let subscription: ReturnType<typeof Accelerometer.addListener> | null = null;
  let cancelled = false;

  Accelerometer.isAvailableAsync().then((isAvailable) => {
    if (cancelled) return;
    setAvailable(isAvailable);
    if (!isAvailable) return;
    Accelerometer.setUpdateInterval(200);
    subscription = Accelerometer.addListener(setReading);
  });

  return () => {
    cancelled = true;
    subscription?.remove(); // stop reading when leaving the screen
  };
}, []);
```

Muestra `x`, `y`, `z` y la fuerza total en *g* (≈ 1 en reposo); por encima de 1,8 indica "¡Movimiento fuerte!".

> [!warning]
> Sin la función de limpieza, el sensor seguiría leyendo al salir de la pantalla: gasta batería y actualiza un componente que ya no existe.

[[ IMAGEN: pantalla Sensores en el emulador con los valores x, y, z (Virtual sensors → Device pose) ]]

[[ IMAGEN: pantalla Sensores en Firefox con el mensaje "El acelerómetro no está disponible" ]]

### Favoritos sin duplicados

Cada resultado de **Descubre** tiene un botón **★ Guardar** que lo añade como lugar. Reutiliza el Context y la persistencia existentes.

```tsx
// API ids (1, 2, 3...) could collide with ours: prefix them
const favoriteId = (post: Post) => `api-${post.id}`;

const isSaved = (post: Post) => places.some((place) => place.id === favoriteId(post));

const saveFavorite = (post: Post) => {
  if (isSaved(post)) return; // avoid duplicates
  addPlace({ id: favoriteId(post), name: post.title, description: post.body });
};
```

El botón cambia a **✓ Guardado** y se desactiva cuando el resultado ya está en la lista. Si se elimina el lugar, vuelve a aparecer **★ Guardar**.

> [!info]
> El `renderItem` pasa de `=> ( ... )` a `=> { ... return (...) }` para poder calcular `saved` antes de devolver el JSX.

[[ IMAGEN: Descubre con resultados marcados como "✓ Guardado" y otros con "★ Guardar" ]]

---

## G - Pruebas y documentación

### Entorno de prueba

| Dato | Valor |
|---|---|
| Sistema operativo | CachyOS (Linux) |
| Contenedor | Podman + Dev Containers (`node:lts`) |
| Emulador | Android Emulator, AVD `pixel`, Android 15 (API 35) |
| Navegador | Firefox |
| Expo | SDK 57, Expo Router |

### Casos de prueba

| # | Prueba | Pasos | Resultado esperado | Resultado obtenido | Observaciones |
|---|---|---|---|---|---|
| 1 | Crear un lugar | **+ Añadir lugar** → nombre y descripción → **Guardar** | Alerta de confirmación y el lugar aparece en la lista | | |
| 2 | Reinicio | Añadir un lugar → cerrar la app desde recientes → reabrir | El lugar sigue en la lista | | |
| 3 | Sin datos | Eliminar todos los lugares | Mensaje "Todavía no hay lugares guardados." | | |
| 4 | Error de red | Desactivar Wi-Fi y datos → abrir **Descubre** | Barra "Sin conexión" y aviso, sin fallar | | |
| 5 | Permiso denegado | **Usa mi ubicación** → No permitir | Mensaje de permiso denegado; el resto de la app funciona | | |

[[ IMAGEN: captura de cada prueba ]]

### Permisos

| Permiso | Para qué | Si se deniega |
|---|---|---|
| Ubicación en primer plano | Mostrar las coordenadas actuales y guardarlas con un lugar | Se muestra un mensaje; el resto de la app funciona con normalidad |

### Limitaciones

- En la web, `Alert.alert` no muestra nada: se sustituye por `window.confirm` al eliminar, pero las alertas de confirmación de guardado solo se ven en el emulador.
- Las imágenes se guardan como URL: sin conexión no se muestran, y dejan de verse si la web las retira.
- En el navegador, el tipo de red puede aparecer como `unknown`.
- El acelerómetro no está disponible en un navegador de escritorio.
- JSONPlaceholder es una API de prueba con texto de relleno en pseudolatín.
- No se ha probado en un dispositivo físico ni en iOS.

### Entrega

Se eliminan los `console.log` de depuración y se comprime el proyecto sin dependencias:

```bash
zip -r myplaces.zip . -x "node_modules/*" ".expo/*"
```

---

## Checkpoints

### 0. Preparación
- [x] Proyecto Expo en TypeScript arrancando en el emulador (`pnpm expo start` → `a`)
- [x] Estructura de carpetas clara: `app/` (pantallas), `components/`, `context/`, `data/`, `api/`, `hooks/`
- [x] Datos iniciales de prueba en `data/` con ids de **texto**

### A · Navegación y componentes
- [x] Pantalla principal con la lista de lugares (`FlatList`)
- [x] Pantalla de detalle con ruta dinámica (`detail/[id].tsx`) que lee el `id`
- [x] Desde cada tarjeta se abre el detalle correcto
- [x] Botón para volver a la lista
- [x] Mensaje si el `id` no corresponde a ningún lugar
- [x] Componente reutilizable `PlaceCard`
- [x] *Extra:* el nombre del lugar en la cabecera del detalle

### B · Formulario y validación
- [x] Pantalla «Añadir lugar» con `TextInput` controlados
- [x] Botón para acceder al formulario desde la lista
- [x] Nombre vacío o con solo espacios → mensaje de error y no se guarda
- [x] Al guardar: aviso (`Alert.alert`) y vuelta a la lista
- [x] El formulario es un componente y la pantalla decide qué hacer con los datos (`onSubmit`)

### C · Persistencia local
- [x] Instalado `@react-native-async-storage/async-storage`
- [x] Se guarda la **lista completa** con `JSON.stringify`
- [x] Se carga con `getItem` + `JSON.parse` dentro de `useEffect`
- [x] Si `getItem` devuelve `null`, la app no falla
- [x] La lista se actualiza al volver del formulario (Context)
- [x] Eliminar un lugar (`filter`) actualiza el almacenamiento
- [x] Mensaje de lista vacía (`ListEmptyComponent`)
- [ ] Comprobado en el emulador: añadir → cerrar → reabrir → el lugar sigue ahí

### D · Consulta HTTP/HTTPS
- [x] Pantalla «Descubre» accesible desde la app
- [x] API pública sin claves: JSONPlaceholder
- [x] `fetch` con comprobación de `response.ok`
- [x] Estados cargando, error y lista vacía
- [x] *Extra:* botón «Volver a intentar»

### E · Ubicación y permisos
- [x] Instalado `expo-location` y declarado el permiso en `app.json`
- [x] Botón «Usa mi ubicación»
- [x] Se pide el permiso antes de leer la ubicación
- [x] Concedido → latitud y longitud
- [x] Denegado → mensaje y la app sigue funcionando
- [ ] Probado en el emulador con ubicación simulada

### F · Conectividad
- [x] Instalado `@react-native-community/netinfo`
- [x] Se muestra el estado de conexión y el tipo de red
- [x] Sin conexión, «Descubre» no consulta y muestra un aviso
- [ ] Probado desactivando la red en el emulador

### Ampliaciones
- [x] Accesibilidad (`accessibilityRole`, `accessibilityLabel`, regiones vivas)
- [x] Coordenadas guardadas con cada lugar
- [x] Pantalla del acelerómetro (`expo-sensors`)
- [x] Favoritos sin duplicados

### G · Pruebas y documentación
- [ ] Las 5 pruebas hechas en el emulador, con captura
- [ ] Tabla de pruebas completada
- [ ] README con instalación, ejecución, estructura, API, permisos y limitaciones
- [ ] Eliminados los `console.log` de depuración
- [ ] `.zip` sin `node_modules`

> [!info] Apuntes relacionados
> [[02.1 - Navegación y formularios|A y B]] · [[02.2 - Persistencia local|C]] · [[02.3 - HTTP y API REST|D]]
