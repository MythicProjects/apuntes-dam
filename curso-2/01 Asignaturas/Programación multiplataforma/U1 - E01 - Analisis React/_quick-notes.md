---
tags:
  - programacion-multiplataforma
  - DAM2
---
# Resumen. Estilos, plataformas y animación

## 1. SafeAreaView

Los móviles tienen zonas que tapan el contenido: la barra de estado, el *notch* (la muesca de la cámara) y la barra de navegación inferior. Su tamaño cambia según el dispositivo, así que no conviene dejar el espacio con un `padding` fijo.

`SafeAreaView` (del paquete `react-native-safe-area-context`, incluido en la plantilla de Expo) añade automáticamente el espacio exacto que necesita cada móvil.

```tsx
<SafeAreaView style={{ flex: 1, backgroundColor: colors.background }} edges={["top", "bottom"]}>
```

| Concepto | Detalle |
|---|---|
| `edges` | Indica qué bordes se protegen: `top`, `bottom`, `left` y `right`. |
| `backgroundColor` | Da color a las franjas protegidas. |
| `StatusBar` (`expo-status-bar`) | `style="light"` pone en blanco el texto de la barra de estado, para fondos oscuros. |

> [!tip] `padding` frente a `margin`
> El `padding` crea espacio **dentro** del elemento y se pinta con su fondo. El `margin` crea espacio **fuera** y se ve el fondo del padre.

---

## 2. Configuración del tema

Un tema es un archivo (`src/constants/theme.ts`) que centraliza los valores compartidos, como colores y espaciados. Los componentes los importan en lugar de escribir el valor a mano.

```ts
export const colors = { background: "#1e1e2f", card: "#733dad", text: "#ffffff" } as const;
export const spacing = { sm: 8, md: 16, lg: 24 } as const;
```

```ts
import { colors, spacing } from "@/constants/theme";
// padding: spacing.md, backgroundColor: colors.card
```

- **Un único punto de cambio:** si modificas un valor en el tema, cambia en toda la app.
- **Nombres por uso, no por color:** `card` y no `purple`, para que el código siga teniendo sentido si cambia el color.
- **`as const`:** permite que TypeScript autocomplete y avise si escribes mal un nombre.
- **Usar un valor del tema dentro de un texto:** se hace con un *template literal*, por ejemplo `` `0 2px 4px ${colors.primary}40` ``. Los dos dígitos finales dan transparencia a un color hexadecimal; `40` equivale a un 25 %.

---

## 3. iOS y Android

React Native comparte la mayor parte del código, pero algunas cosas funcionan distinto en cada plataforma.

### 3.1. Herramientas para adaptar el código

| Herramienta | Cuándo usarla |
|---|---|
| `Platform.OS` | Para un valor puntual: `Platform.OS === "ios" ? 22 : 20`. |
| `Platform.select({ ios, android })` | Para varios estilos o valores distintos por plataforma. |
| Archivos `Nombre.ios.tsx` y `Nombre.android.tsx` | Cuando el componente entero es diferente. Al importar `./Nombre` se carga solo el de la plataforma correspondiente. |

### 3.2. Diferencias típicas y cómo resolverlas

| Tema | iOS | Android | Mejor solución |
|---|---|---|---|
| Sombras | `shadowColor`, `shadowOffset`, `shadowOpacity`, `shadowRadius` | `elevation` | `boxShadow` (sintaxis de CSS), que funciona en ambas |
| *Notch* y barras del sistema | *Notch* e indicador inferior | Barra de estado y de navegación | `SafeAreaView` |
| Barra de estado | Texto claro u oscuro | Texto y fondo | `StatusBar` de `expo-status-bar` |
| Efecto al pulsar | No tiene efecto nativo | Onda (*ripple*) | `Pressable` con `pressed`; en Android, además, `android_ripple` |
| Teclado sobre los campos | Hay que desplazar la vista | Lo suele hacer el sistema | `KeyboardAvoidingView` con `behavior` según `Platform.OS` |
| Botón "atrás" | No existe | Botón físico o gesto | Lo gestiona la navegación (Expo Router) |
| Fuente por defecto | San Francisco | Roboto | Cargar una fuente propia con `expo-font` |
| Permisos | `Info.plist` | `AndroidManifest.xml` | Declararlos en `app.json` y Expo los traslada |
| Componentes del sistema (alertas, selectores de fecha) | Aspecto de iOS | Aspecto de Android | Aceptar la diferencia o usar un componente propio |

> [!important]
> Lo que se ve bien en el emulador de Android puede verse distinto en iOS. Detectar y gestionar estas diferencias forma parte del desarrollo multiplataforma.

---

## 4. Animación del botón "me gusta"

### 4.1. Qué es Animated

`Animated` es la API de animaciones que incluye React Native. Funciona así: se crea un **valor animado** (un número especial), se conecta a una propiedad de estilo y se le dice cómo cambiar con el tiempo.

| Pieza | Para qué sirve |
|---|---|
| `new Animated.Value(1)` | Valor numérico que se puede animar. Aquí, la escala del corazón (1 = tamaño normal). |
| `Animated.timing(valor, { toValue, duration })` | Lleva el valor hasta `toValue` en un tiempo dado. |
| `Animated.spring(...)` | Igual, pero con efecto muelle (rebote). |
| `Animated.sequence([...])` | Ejecuta varias animaciones **una tras otra**. |
| `Animated.parallel([...])` | Ejecuta varias animaciones **a la vez**. |
| `Animated.loop(...)` | Repite una animación. |
| `.start()` | Arranca la animación. Sin él, no ocurre nada. |
| `valor.setValue(n)` | Cambia el valor de golpe, sin animación. |
| `Animated.View`, `Animated.Text`, `Animated.Image` | Versiones de los componentes que aceptan valores animados en `style`. |
| `useNativeDriver: true` | La animación se ejecuta en el lado nativo, más fluida. Solo sirve para `transform` y `opacity`. |

Con `Animated` se pueden animar el tamaño (`scale`), la posición (`translateX` y `translateY`), la rotación (`rotate`) y la opacidad (`opacity`), entre otras propiedades.

> [!info]
> Para animaciones más complejas, como gestos o arrastrar elementos, existe la biblioteca **Reanimated**, que ya viene en la plantilla de Expo.

### 4.2. Por qué `useState` y por qué `useRef`

```tsx
const [liked, setLiked] = useState(false);
const scale = useRef(new Animated.Value(1)).current;
```

| Variable | Hook | Motivo |
|---|---|---|
| `liked` | `useState` | Al cambiar, la **interfaz debe cambiar**: el corazón pasa de vacío a lleno. Solo el estado provoca un nuevo render. |
| `scale` | `useRef` | Necesitas **el mismo objeto** durante toda la vida del componente. `Animated` lo modifica directamente, sin pasar por React, así que no hace falta un render. |

> [!warning]
> Si escribieras `const scale = new Animated.Value(1)` sin `useRef`, se crearía un valor nuevo en cada render y la animación se reiniciaría o se cortaría. `useRef` guarda el objeto y `.current` lo devuelve.

### 4.3. Qué hace `toggleLike`

Es la función que se ejecuta al pulsar el botón:

1. **Calcula el nuevo valor:** `const willLike = !liked;`. Se guarda en una variable porque `liked` no cambia hasta el siguiente render.
2. **Actualiza el estado:** `setLiked(willLike)`. React vuelve a dibujar la tarjeta con el corazón nuevo.
3. **Elige la animación** según si se marca o se desmarca:
   - **Marcar** (`heartPop`): `sequence` de dos `timing`, que van de 1 a 1,35 y de 1,35 a 1.
   - **Desmarcar** (`heartPopOut`): primero `setValue(1.35)` salta al tamaño grande sin animar, y después una `sequence` va de 1,35 a 0,75 y de 0,75 a 1.
4. **Arranca la animación** con `.start()`.

Las dos fases de 150 ms suman los 0,3 s del `@keyframes` original.

### 4.4. Cómo se construye el Pressable

```tsx
<Pressable
  onPress={toggleLike}
  style={({ pressed }) => [styles.likeButton, pressed && { transform: [{ scale: 1.1 }] }]}
>
```

**`onPress`**
En el móvil no hay ratón, así que no existe `onClick`. Los eventos táctiles son `onPress` (pulsación completa), `onPressIn` (al tocar), `onPressOut` (al soltar) y `onLongPress` (pulsación larga). Se pasa la función **sin paréntesis**, `toggleLike`, para que se ejecute al pulsar y no al dibujar.

**`style` como función**
`style` puede recibir un objeto normal o **una función**. Si recibe una función, el propio `Pressable` la llama cada vez que cambia su estado de pulsación y le pasa un objeto con información: `{ pressed: true }` mientras se mantiene pulsado y `{ pressed: false }` cuando no.

- **`({ pressed })`:** desestructura ese objeto, igual que se hace con las props. El dato `pressed` lo proporciona `Pressable`.
- **`=> [ ... ]`:** la función devuelve un **array de estilos**. React Native los combina en orden, y los posteriores sobrescriben a los anteriores.
- **`pressed && {...}`:** si `pressed` es `true`, añade el estilo de escala; si es `false`, el array contiene `false` y React Native lo ignora.

Así se sustituye el `:hover` del CSS, que no existe en el móvil: el botón crece mientras se mantiene pulsado.

**`Animated.Image`**
Es el corazón. Usa `Animated.Image` porque un `Image` normal no entiende valores animados. Su estilo combina el tamaño fijo con `transform: [{ scale }]`, que es el valor animado. La imagen se elige con `liked ? require(lleno) : require(vacío)`, con las dos rutas escritas literalmente.

### 4.5. Por qué `resizeMode`

`resizeMode` decide cómo se ajusta la imagen a su caja (por ejemplo, 40×40) cuando sus proporciones no coinciden. Equivale a `object-fit` en CSS.

| Valor | Resultado |
|---|---|
| `contain` | La imagen se ve **entera**, sin deformarse; puede sobrar espacio. Es el adecuado para iconos. |
| `cover` | Rellena toda la caja y recorta lo que sobra. Es el adecuado para portadas y fotos. |
| `stretch` | Rellena la caja deformando la imagen. |
| `center` | Centra la imagen sin escalarla. |

Se usa `contain` para que el corazón nunca se recorte ni se deforme, ni siquiera mientras crece con la animación.
