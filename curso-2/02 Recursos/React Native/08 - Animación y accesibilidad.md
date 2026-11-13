---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Animaciones con Animated y accesibilidad
---
# 08 · Animación y accesibilidad

## Animated

Se crea un **valor animado**, se conecta a un estilo y se indica cómo cambia con el tiempo.

| Pieza | Para qué | Ejemplo |
|---|---|---|
| `Animated.Value` | Número animable (guardado con `useRef`). | `const scale = useRef(new Animated.Value(1)).current;` |
| `Animated.timing` | Lleva el valor a otro en un tiempo. | `Animated.timing(scale, { toValue: 1.35, duration: 150, useNativeDriver: true })` |
| `Animated.spring` | Igual, con rebote. | `Animated.spring(scale, { toValue: 1, useNativeDriver: true })` |
| `Animated.sequence` | Una tras otra. | `Animated.sequence([grow, shrink])` |
| `Animated.parallel` | A la vez. | `Animated.parallel([fade, move])` |
| `Animated.loop` | Repetir. | `Animated.loop(pulse)` |
| `.start()` | Arranca (sin él no ocurre nada). | `Animated.sequence([…]).start();` |
| `setValue` | Cambia sin animar. | `scale.setValue(1.35);` |
| `Animated.View` / `Text` / `Image` | Componentes que aceptan valores animados. | `<Animated.Image style={{ transform: [{ scale }] }} />` |
| `useNativeDriver: true` | Animación en el lado nativo; solo `transform` y `opacity`. | — |
| Propiedades animables | `scale`, `translateX`, `translateY`, `rotate`, `opacity`… | — |
| Reanimated | Biblioteca para gestos y animaciones complejas. | — |

### Ejemplo: botón "me gusta" con "pop"

```tsx
const [liked, setLiked] = useState<boolean>(false);
const scale = useRef(new Animated.Value(1)).current;

const toggleLike = (): void => {
  const willLike = !liked; // liked does not change until the next render
  setLiked(willLike);
  if (!willLike) scale.setValue(1.35);
  Animated.sequence([
    Animated.timing(scale, { toValue: willLike ? 1.35 : 0.75, duration: 150, useNativeDriver: true }),
    Animated.timing(scale, { toValue: 1, duration: 150, useNativeDriver: true }),
  ]).start();
};

<Pressable
  onPress={toggleLike}
  style={({ pressed }) => [styles.likeButton, pressed && { transform: [{ scale: 1.1 }] }]}
>
  <Animated.Image
    source={liked ? require("../../assets/images/heart_full.png") : require("../../assets/images/heart_empty.png")}
    style={[styles.heart, { transform: [{ scale }] }]}
    resizeMode="contain"
  />
</Pressable>
```

| Pieza | Por qué |
|---|---|
| `liked` con `useState` | Al cambiar, la imagen debe cambiar (render). |
| `scale` con `useRef` | Debe ser el mismo objeto siempre; `Animated` lo cambia sin render. |
| `willLike` | `liked` no cambia hasta el siguiente render. |
| `pressed` | Sustituye al `:hover`: crece mientras se mantiene pulsado. |
| `Animated.Image` | Un `Image` normal no acepta valores animados. |
| `resizeMode="contain"` | El icono nunca se recorta ni se deforma al crecer. |
| Dos `require` | Las rutas deben ser literales. |

## Accesibilidad

Que un lector de pantalla (TalkBack, VoiceOver) anuncie qué es cada elemento y qué hace.

| Prop | Para qué | Ejemplo |
|---|---|---|
| `accessibilityRole` | Qué es: `button`, `header`, `image`, `alert`… | `accessibilityRole="button"` |
| `accessibilityLabel` | Lo que se lee cuando el texto visible no basta. | `` accessibilityLabel={`Photo of ${place.name}`} `` |
| `accessibilityHint` | Qué pasará al pulsar. | `accessibilityHint="Opens the detail"` |
| `accessibilityState` | Estado. | `accessibilityState={{ disabled: saved, busy: loading }}` |
| `accessibilityLiveRegion` | Anuncia los cambios automáticamente (Android). | `accessibilityLiveRegion="polite"` |

| Dónde aplicarlo | Prop |
|---|---|
| Todos los `Pressable` | `accessibilityRole="button"` + `accessibilityLabel` |
| Imágenes | `accessibilityLabel` |
| Títulos | `accessibilityRole="header"` |
| `TextInput` | `accessibilityLabel` (el `placeholder` desaparece al escribir) |
| Errores | `accessibilityRole="alert"` |
| Contenido que cambia solo | `accessibilityLiveRegion="polite"` |
| Indicador de carga | `accessibilityLabel="Loading"` |

| Comprobar | Cómo |
|---|---|
| Emulador | Ajustes → Accesibilidad → TalkBack (un toque lee, doble toque pulsa). |
| Navegador | `F12` → pestaña Accesibilidad (`aria-label` y `role`). |
