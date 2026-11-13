---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Teoría de TypeScript y funciones de base
---
# 09 · TypeScript

## Variables y tipos

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `const` / `let` | No reasignable / reasignable. | `const name: string = "Rome"; let count: number = 0;` |
| `type` | Forma de un dato. | `type Coords = { latitude: number; longitude: number };` |
| `: tipo` | Anotación. | `(id: string) => …` |
| `string`, `number`, `boolean` | Tipos básicos, siempre en minúscula (`String` es otra cosa). | `let ok: boolean = true;` |
| `Tipo[]` | Array. | `places: Place[]` |
| `campo?:` | Opcional (puede ser `undefined`). | `image?: string;` |
| Unión `A \| B` | Uno u otro. | `string \| null` |
| `null` / `undefined` | Vacío a propósito / no asignado o no existe. | `useState<string \| null>(null)` |
| Tipos literales | Valor exacto. | `status === "granted"` |
| `keyof` | Nombres de los campos de un tipo. | `keyof FormData` → `"name" \| "description"` |
| `typeof` (en tipos) | Tipo de algo existente. | `typeof Accelerometer.addListener` |
| `ReturnType<F>` | Lo que devuelve una función. | `ReturnType<typeof Accelerometer.addListener>` |
| Genéricos `<T>` | Concretan el tipo que maneja una función. | `useState<Place[]>([])` |
| `Promise<T>` | Valor futuro. | `async function load(): Promise<Place[]>` |
| `as const` | Objeto literal de solo lectura. | `{ sm: 8 } as const` |
| `any` | Sin comprobación; evitarlo. | *Parameter implicitly has an 'any' type* |
| `ImageSourcePropType`, `ReactNode` | Tipos de React Native y React. | `children: ReactNode` |

## Funciones

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Declaración | Forma clásica. | `function savePlaces(places: Place[]): void { … }` |
| Flecha | Forma corta, en variable o como argumento. | `const goBack = (): void => { router.back(); };` |
| Flecha con `( )` | Devuelve la expresión directamente. | `const double = (n: number): number => n * 2;` |
| Flecha con `{ }` | Bloque: necesita `return`. | `(n: number) => { const d = n * 2; return d; }` |
| `() => void` | Función sin parámetros que no devuelve nada. | `onPress: () => void;` |
| Callback | Función que otra llamará después. | `onPress={() => router.push("/add")}` |
| `f` frente a `f()` | Pasar la función / **ejecutarla al dibujar**. | `onPress={getLocation}` |
| Flecha con argumentos | Para pasar argumentos sin ejecutar al dibujar. | `onPress={() => saveFavorite(item)}` |
| Parámetro por defecto | Si falta o es `undefined`. | `function fetchNearby(lat: number = 41.38) {}` |
| `return;` vacío | Sale de la función. | `if (!place) return;` |
| Sin `return` | Devuelve `undefined`. | Hook que olvida `return context;` |
| `export` / `export default` | Con nombre (con llaves) / principal (sin llaves). | `export type Place = {…};` |

## Asincronía

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `async` | Devuelve una promesa y permite `await`. | `const load = async (): Promise<void> => { … };` |
| `await` | Espera el resultado. | `const data = await response.json();` |
| `.then` | Alternativa a `await`. | `loadPlaces().then(setPlaces);` |
| `try / catch / finally` | Intentar / capturar / siempre. | `try { } catch { } finally { }` |
| `catch` sin `(error)` | Si no se usa el error. | `catch { return []; }` |
| `throw` | Lanza un error al `catch` más cercano. | `throw new Error("HTTP 404");` |

## Objetos y arrays

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Desestructurar objeto | Campos a variables. | `const { places, addPlace } = usePlaces();` |
| Con renombre | Otro nombre. | `const { loading: locating } = useLocation();` |
| Desestructurar array | Por posición. | `const [error, setError] = useState<string \| null>(null);` |
| En parámetros | Directamente en la firma. | `function Card({ place }: CardProps)` |
| Spread | Copia y permite añadir o sobrescribir. | `[...places, place]`, `{ ...form, name: "Rome" }` |
| Rest | Junta lo que sobra. | `const { id, ...rest } = item;` |
| Clave computada | Variable como nombre del campo. | `{ [key]: value }` |
| Propiedad abreviada | `{ name }` = `{ name: name }`. | `addPlace({ id, name, description })` |
| Inmutabilidad | Copias nuevas en lugar de modificar. | `setPlaces(places.filter((place) => place.id !== id))` |

## Operadores

| Operador | Explicación | Ejemplo |
|---|---|---|
| `===` / `!==` | Igualdad estricta (también el tipo). | `"1" === 1` → `false` |
| `!` | Negación. | `if (!place) return;` |
| `&&` / `\|\|` | Y / O. | `lat !== undefined && lon !== undefined` |
| Ternario | `if/else` en una expresión. | `saved ? "✓ Saved" : "★ Save"` |
| `?.` | Accede si existe; si no, `undefined`. | `coords?.latitude`, `item.coordinates?.[0]?.lat` |
| `??` | Por defecto solo si `null` o `undefined` (`0` y `""` se conservan). | `item.extract ?? ""` |
| `**` | Potencia. | `x ** 2` |
| Template literal | Texto con variables. | `` `Photo of ${name}` `` |
| Comparar con `undefined` | Para números que pueden ser `0`. | `lat !== undefined ? … : null` |

## Funciones de base

| Función | Qué hace | Ejemplo |
|---|---|---|
| `trim()` | Quita espacios de los extremos. | `"  Rome ".trim()` → `"Rome"` |
| `startsWith(x)` | ¿Empieza por `x`? | `url.startsWith("https://")` |
| `toString()` | A texto. | `(42).toString()` → `"42"` |
| `toFixed(n)` | Texto con `n` decimales. | `(2.12345).toFixed(2)` → `"2.12"` |
| `Number(x)` | Texto → número. | `Number("42")` → `42` |
| `map(fn)` | Transforma cada elemento. | `[1, 2].map((n: number) => n * 2)` → `[2, 4]` |
| `find(fn)` | Primero que cumple, o `undefined`. | `places.find((place) => place.id === id)` |
| `filter(fn)` | Los que cumplen. | `places.filter((place) => place.id !== id)` |
| `some(fn)` | ¿Alguno cumple? | `places.some((place) => place.id === id)` |
| `length` | Número de elementos. | `places.length` |
| `Object.keys(obj)` | Nombres de los campos. | `Object.keys({ a: 1 })` → `["a"]` |
| `JSON.stringify` / `JSON.parse` | Objeto → texto / texto → objeto. | `JSON.stringify([{ id: "1" }])` |
| `Date.now()` | Milisegundos desde 1970. | `Date.now().toString()` |
| `Math.sqrt(x)` | Raíz cuadrada. | `Math.sqrt(9)` → `3` |
| `new URLSearchParams({...})` | Parámetros de URL escapados. | `new URLSearchParams({ q: "rome" }).toString()` |
| `console.log` / `console.warn` | Mensajes de depuración (terminal de Expo o navegador). | `console.log("loaded:", text);` |
| `window.confirm` | Diálogo Aceptar/Cancelar (solo web). | `window.confirm("Delete?")` |
