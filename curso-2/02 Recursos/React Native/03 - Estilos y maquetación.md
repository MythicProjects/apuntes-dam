---
tags:
  - programacion-multiplataforma
  - DAM2
tema: Estilos, maquetación, tema y diferencias entre iOS y Android
---
# 03 · Estilos y maquetación

## StyleSheet

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `StyleSheet.create` | Estilos como objeto, al final del archivo. | `const styles = StyleSheet.create({ title: { fontSize: 24 } });` |
| Aplicar | Prop `style`. | `<Text style={styles.title}>` |
| Colocation | Los estilos propios van en el mismo archivo que el componente. | `Card.tsx` con su `styles` |
| Array de estilos | Combina; los `false` se ignoran y los últimos mandan. | `style={[styles.button, pressed && styles.pressed]}` |
| camelCase sin unidades | `font-size: 16px` → `fontSize: 16`. | `backgroundColor: "red"` |
| Unidades dp | Píxeles independientes de la densidad. | 1080 px a 420 dpi ≈ 411 dp |

## Maquetación

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Flexbox | Activo por defecto, en columna. | `flexDirection: "row"` |
| `flex: 1` | Ocupa el espacio que sobra (solo en lo que debe estirarse). | `screen: { flex: 1 }` |
| Pie de página abajo | `flex: 1` en la raíz y en la zona central. | `Header` + lista `flex: 1` + `Footer` |
| `justifyContent` / `alignItems` | Alineación en el eje principal / secundario. | `alignItems: "center"` |
| `gap` | Espacio entre hijos. | `gap: 12` |
| `padding` / `margin` | Espacio dentro (con el fondo propio) / fuera (se ve el padre). | `paddingVertical: 12` |
| `textAlign` | Solo en `<Text>`. | `textAlign: "center"` |
| `numberOfLines` | Corta el texto con "…". | `<Text numberOfLines={2}>` |
| `ScrollView` → `style` | Contenedor: tamaño, fondo. | `style={{ flex: 1 }}` |
| `ScrollView` → `contentContainerStyle` | Contenido: padding, alineación. | `contentContainerStyle={{ padding: 16 }}` |

> [!warning]
> `alignItems` o `justifyContent` en el `style` de un `ScrollView` dan error: van en `contentContainerStyle`.

## SafeAreaView

`SafeAreaView` (de `react-native-safe-area-context`) añade el espacio justo para el *notch* y las barras del sistema.

```tsx
<SafeAreaView style={{ flex: 1, backgroundColor: colors.background }} edges={["top", "bottom"]}>
```

| Concepto | Detalle |
|---|---|
| `edges` | Bordes protegidos: `top`, `bottom`, `left`, `right`. |
| `backgroundColor` | Color de las franjas protegidas. |
| `StatusBar` (`expo-status-bar`) | `style="light"` pone en blanco el texto de la barra de estado. |

## Tema

Un archivo `src/constants/theme.ts` centraliza colores y espaciados.

```ts
export const colors = { background: "#1e1e2f", card: "#733dad", text: "#ffffff" } as const;
export const spacing = { sm: 8, md: 16, lg: 24 } as const;
```

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Un único punto de cambio | Cambiar el tema cambia toda la app. | `backgroundColor: colors.card` |
| Nombres por uso | `card`, no `purple`. | `colors.card` |
| `as const` | Solo lectura y autocompletado. | `{ sm: 8 } as const` |
| Transparencia hexadecimal | Dos dígitos más al final. | `` `${colors.primary}40` `` (≈ 25 %) |

## Efectos

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Sombra multiplataforma | `boxShadow`, con sintaxis CSS; el color va dentro del texto. | `` boxShadow: `0 2px 4px ${colors.primary}40` `` |
| Sombra iOS | `shadowColor`, `shadowOffset`, `shadowOpacity`, `shadowRadius`. | `shadowOpacity: 0.25` |
| Sombra Android | `elevation`. | `elevation: 4` |
| Degradado | `LinearGradient` de `expo-linear-gradient`. | `<LinearGradient colors={["#733dad", "#9898ff"]} start={{ x: 0, y: 0 }} end={{ x: 1, y: 1 }} />` |

| CSS | `start` → `end` |
|---|---|
| `to bottom` | `{x:0,y:0}` → `{x:0,y:1}` |
| `to right` | `{x:0,y:0}` → `{x:1,y:0}` |
| `to bottom right` | `{x:0,y:0}` → `{x:1,y:1}` |

## iOS y Android

| Herramienta | Cuándo | Ejemplo |
|---|---|---|
| `Platform.OS` | Un valor puntual. | `Platform.OS === "ios" ? 22 : 20` |
| `Platform.select` | Varios valores por plataforma. | `Platform.select({ ios: { color: "blue" }, android: { color: "green" } })` |
| Archivos `.ios.tsx` / `.android.tsx` | El componente entero es distinto. | `Button.ios.tsx` |

| Tema | iOS | Android | Solución |
|---|---|---|---|
| Sombras | `shadow*` | `elevation` | `boxShadow` |
| *Notch* y barras | *Notch* e indicador inferior | Barra de estado y navegación | `SafeAreaView` |
| Barra de estado | Texto claro u oscuro | Texto y fondo | `StatusBar` |
| Efecto al pulsar | Sin efecto nativo | Onda (*ripple*) | `pressed`; en Android, `android_ripple` |
| Teclado sobre campos | Hay que desplazar la vista | Lo suele hacer el sistema | `KeyboardAvoidingView` con `behavior` según `Platform.OS` |
| Botón atrás | No existe | Físico o gesto | Lo gestiona Expo Router |
| Fuente por defecto | San Francisco | Roboto | Fuente propia con `expo-font` |
| Permisos | `Info.plist` | `AndroidManifest.xml` | Declararlos en `app.json` |
| Diálogos del sistema | Aspecto iOS | Aspecto Android | Aceptarlo o componente propio |
| Web | — | — | `Alert.alert` no se muestra; sin sensores; tipo de red `unknown` |
