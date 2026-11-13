---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Estado, efectos, referencias, Context y hooks propios
---
# 04 · Estado y hooks

## Hooks de React

| Import (`react`) | Para qué | Ejemplo |
|---|---|---|
| `useState` | Dato que, al cambiar, vuelve a dibujar el componente. | `const [count, setCount] = useState<number>(0);` |
| `useEffect` | Código después de dibujar y cuando cambian sus dependencias. | `useEffect(() => { load(); }, []);` |
| `useRef` | Valor que se mantiene entre renders **sin** provocar renders. | `const scale = useRef(new Animated.Value(1)).current;` |
| `createContext` | Canal de datos compartidos. | `const PlacesContext = createContext<PlacesValue \| null>(null);` |
| `useContext` | Lee el Context (solo dentro de su Provider). | `const context = useContext(PlacesContext);` |
| `ReactNode` | Tipo de "cualquier cosa que React puede dibujar". | `{ children }: { children: ReactNode }` |

## useState

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Setter | La única forma de cambiar el estado. | `setCount(count + 1)` |
| Inmutabilidad | Se crea un valor nuevo; nunca se modifica el actual. | `setPlaces([...places, place])` ✅ `places.push(place)` ❌ |
| Valor antiguo | Tras `setX(nuevo)`, `x` conserva el valor anterior hasta el siguiente render. | Guardar `updated`, no `places` |
| Estado por instancia | Cada copia del componente tiene el suyo. | Cada tarjeta, su `liked` |
| Estado de objeto | Varios campos en un solo estado. | `useState<FormData>({ name: "", description: "" })` |

## useEffect

| Forma | Cuándo se ejecuta | Ejemplo |
|---|---|---|
| `useEffect(fn, [])` | Una vez, al montar. | `useEffect(() => { loadPlaces().then(setPlaces); }, []);` |
| `useEffect(fn, [a, b])` | Al montar y cuando cambie `a` o `b`. | `useEffect(() => { load(coords); }, [isConnected, coords]);` |
| `useEffect(fn)` | Tras cada render (casi siempre un error). | — |
| Limpieza | Lo que devuelve se ejecuta al salir. | `return () => subscription?.remove();` |
| Sin `async` directo | La función `async` se llama dentro. | `useEffect(() => { load(); }, []);` |
| Bandera `cancelled` | No actualizar si la pantalla se cerró antes de acabar la promesa. | `if (cancelled) return;` |

> [!warning] Stack y montaje
> En un `Stack`, la pantalla de debajo no se destruye: al volver, su `useEffect([])` no se repite. Se resuelve con Context o con `useFocusEffect` (de `expo-router`), que se ejecuta cada vez que la pantalla vuelve a verse.

## useState frente a useRef

| | `useState` | `useRef` |
|---|---|---|
| Al cambiar | Vuelve a dibujar | No dibuja |
| Uso | Lo que se ve en pantalla | Objetos que deben mantenerse (valores animados, temporizadores) |
| Ejemplo | `const [liked, setLiked] = useState(false);` | `const scale = useRef(new Animated.Value(1)).current;` |

> [!warning]
> `const scale = new Animated.Value(1)` sin `useRef` crea un valor nuevo en cada render y corta la animación.

## Context

Comparte datos entre pantallas sin pasarlos por props.

```tsx
type PlacesValue = { places: Place[]; addPlace: (place: Place) => void };

const PlacesContext = createContext<PlacesValue | null>(null);

export function PlacesProvider({ children }: { children: ReactNode }) {
  const [places, setPlaces] = useState<Place[]>([]);
  const addPlace = (place: Place) => setPlaces([...places, place]);
  return <PlacesContext.Provider value={{ places, addPlace }}>{children}</PlacesContext.Provider>;
}

export function usePlaces(): PlacesValue {
  const context = useContext(PlacesContext);
  if (!context) throw new Error("usePlaces must be used inside PlacesProvider");
  return context;
}
```

| Pieza | Explicación | Ejemplo |
|---|---|---|
| Provider | Envuelve la app en `_layout.tsx`. | `<PlacesProvider><Stack /></PlacesProvider>` |
| Hook del Context | Se llama con `()` y se desestructura. | `const { places } = usePlaces();` |
| Fuera del Provider | `useContext` devuelve `null`. | `RootStack` hijo del Provider si el layout lo necesita |

## Hooks propios

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Hook propio | Función `useAlgo` que agrupa estados y lógica. | `export function useLocation() { …; return { coords, getLocation }; }` |
| Uso | Igual que cualquier hook. | `const { coords, getLocation } = useLocation();` |
| Renombrar al usarlo | Evita choques de nombres. | `const { loading: locating } = useLocation();` |

## Reglas de los hooks

| Regla | ❌ | ✅ |
|---|---|---|
| Dentro de un componente u otro hook | Junto a los imports | En el cuerpo del componente |
| Nivel superior | Dentro de `if`, bucles o funciones | Arriba del todo |
| Antes de cualquier `return` | Después de `if (loading) return …` | Antes de los returns tempranos |
| Siempre en el mismo orden | Condicionales | Incondicionales |
