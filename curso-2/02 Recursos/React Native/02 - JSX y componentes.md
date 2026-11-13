---
tags:
  - programacion-multiplataforma
  - DAM2
tema: JSX, componentes, props, listas e imágenes
---
# 02 · JSX y componentes

## De HTML a React Native

| Web | React Native | Nota |
|---|---|---|
| `<div>` | `<View>` | Contenedor. |
| `<p>`, `<span>`, `<h1>` | `<Text>` | **Todo** texto va dentro de `<Text>`. |
| `<img>` | `<Image>` | Remota: necesita `width` y `height`. |
| `<button>` | `<Pressable>` | `onClick` → `onPress`. |
| `<input>` | `<TextInput>` | `onChange` → `onChangeText`. |
| Scroll de página | `<ScrollView>` / `<FlatList>` | No hay scroll automático. |
| `<ul>` + `<li>` | `<FlatList>` | Solo dibuja lo visible. |
| `<a href>` | `router.push()` / `<Link>` | Expo Router. |
| CSS | `StyleSheet` + `style` | Véase [[03 - Estilos y maquetación]]. |
| `:hover` | Estado `pressed` de `Pressable` | No hay ratón. |

Todos se importan de `react-native`:

```tsx
import { FlatList, Image, Pressable, Text, TextInput, View } from "react-native";
```

## JSX

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Componente | Función en mayúscula que devuelve JSX. | `export default function Header() { return <Text>Hi</Text>; }` |
| `{ }` | Inserta una expresión. | `<Text>{place.name}</Text>` |
| `{{ }}` | Llaves de JSX con un objeto dentro. | `source={{ uri: url }}` |
| Un elemento raíz | El `return` devuelve un solo elemento. | `<View>…</View>` |
| Fragment | Agrupa sin contenedor real. | `<><Text>A</Text><Text>B</Text></>` |
| Etiquetas cerradas | Todo elemento se cierra. | `<Image … />` |
| Saltos de línea | Se eliminan entre texto y `{…}`; `{" "}` conserva el espacio. | `Lon:{" "}{lon}` |
| Comentarios | Entre llaves. | `{/* comment */}` |
| `cond && <A />` | Dibuja `A` si se cumple. | `{place.image && <Image … />}` |
| `cond ? <A /> : null` | Más seguro con textos o números (`""` o `0` con `&&` se dibujan sueltos). | `{error ? <Text>{error}</Text> : null}` |
| Return temprano | Distinto JSX según el estado. | `if (loading) return <ActivityIndicator />;` |

## Exportar e importar

| Forma | Importación | Ejemplo |
|---|---|---|
| `export default` | Sin llaves (una por archivo). | `import Header from "@/components/Header";` |
| `export` con nombre | Con llaves. | `import { usePlaces } from "@/context/PlacesContext";` |
| Ruta relativa | `./` carpeta actual, `../` sube un nivel; sin extensión. | `import Header from "../components/Header";` |

## Props

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Props | Datos del padre al hijo; llegan en **un único objeto**. | `<GameCard title="Hades" year={2020} />` |
| Tipar | `type` con la forma de las props. | `type GameCardProps = { title: string; year: number };` |
| Desestructurar | Llaves en el parámetro. | `function GameCard({ title, year }: GameCardProps)` |
| Prop opcional | `?` en el tipo. | `image?: string;` |
| Función como prop | El padre decide qué hacer; el hijo la ejecuta. | `onPress: () => void;` |
| `children` | Lo que va entre las etiquetas. | `{ children }: { children: ReactNode }` |
| Composición | Pantallas hechas de componentes pequeños. | `<Header /><GameList /><Footer />` |

## Listas

| Concepto | Explicación | Ejemplo |
|---|---|---|
| `.map()` | Array de datos → componentes. | `{games.map((game) => <GameCard key={game.id} {...game} />)}` |
| `key` | Id único por elemento; no llega como prop. | `key={game.id}` |
| Rest `...` | Al recibir: **junta** lo que sobra. | `({ id, ...game }) => …` |
| Spread `...` | Al pasar: **reparte** las propiedades. | `<GameCard {...game} />` |
| `FlatList` | Lista eficiente. | `<FlatList data={places} renderItem={…} keyExtractor={(place) => place.id} />` |
| `renderItem` | Recibe `{ item }` por cada elemento. | `renderItem={({ item }) => <PlaceCard place={item} />}` |
| `renderItem` con `{ }` | Bloque para calcular antes de devolver (necesita `return`). | `({ item }) => { const saved = isSaved(item); return (…); }` |
| `keyExtractor` | Devuelve el id como texto. | `(item) => item.id.toString()` |
| `ListHeaderComponent` | Elemento encima de la lista (uno solo: varios, en un `View`). | `ListHeaderComponent={<View>…</View>}` |
| `ListEmptyComponent` | Elemento si la lista está vacía. | `ListEmptyComponent={<Text>Vacío</Text>}` |

## Imágenes

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Local | `require` con la ruta escrita literalmente. | `source={require("../../assets/images/icon.png")}` |
| `require` sin variables | Metro empaqueta las imágenes antes de ejecutar. | `require(path)` ❌ |
| Remota | Objeto con `uri` y tamaño obligatorio. | `source={{ uri: "https://…" }}` |
| `ImageSourcePropType` | Tipo para una imagen local o remota. | `image: ImageSourcePropType;` |
| `resizeMode` | Cómo se ajusta a su caja. | `resizeMode="cover"` |

| `resizeMode` | Resultado |
|---|---|
| `contain` | Entera, sin deformar; puede sobrar espacio (iconos). |
| `cover` | Rellena y recorta lo que sobra (fotos). |
| `stretch` | Rellena deformando. |
| `center` | Centrada sin escalar. |
