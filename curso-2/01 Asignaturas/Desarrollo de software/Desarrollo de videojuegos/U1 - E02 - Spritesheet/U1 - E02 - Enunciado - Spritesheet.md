# Animación de sprites con MVP e hilos

## Contexto y objetivos

Hay que programar en Java, con AWT y Swing, una ventana que reproduzca a la vez al menos 10 animaciones de sprites, cada una en su propio hilo. Los fotogramas se recortan de hojas de sprites y se guardan una sola vez en un pool compartido. Parte de la demo [SpriteAnimation](https://github.com/jumibot/SpriteAnimation) y de la estructura MVP del ejercicio del fuego.

Al terminar, se debe saber:

- Recortar una hoja de sprites en fotogramas calculando cada celda a partir de su índice.
- Compartir datos ya cargados entre muchas instancias con un pool, sin volver a leer el disco.
- Separar el programa en modelo, vista y presentador, unidos por interfaces e inyección por constructor.
- Ejecutar varias animaciones en paralelo, cada una en su hilo, sin que los hilos se pisen.

## Requisitos funcionales

**RF1. Una hoja por animación.** Cada animación sale de su propio archivo PNG: la hoja de idle da la animación idle, la de run da run, y así con todas. Se usan al menos 4 hojas distintas. Los fotogramas de una hoja tienen todos el mismo tamaño y se ordenan en rejilla, de izquierda a derecha y de arriba abajo. La última fila puede quedar incompleta.

**RF2. El constructor carga la hoja.** `SpriteSheet` recibe la ruta del recurso, el número de columnas, el de filas y el de fotogramas. En su constructor lee la imagen y recorta todos los fotogramas. Si el archivo no existe o no se puede leer, lanza una excepción con un mensaje claro.

**RF3. Cálculo del fotograma por índice.** El recorte usa un solo bucle sobre el índice `n` del fotograma, no dos bucles anidados:

```latex
\begin{aligned}
w &= \frac{\text{sheetWidth}}{\text{columns}} & h &= \frac{\text{sheetHeight}}{\text{rows}} \\
row &= n \;\text{div}\; \text{columns} & col &= n \bmod \text{columns} \\
x &= col \cdot w & y &= row \cdot h
\end{aligned}
```

Ejemplo de la pizarra: una hoja de 400 × 75 px con 4 columnas y 3 filas da fotogramas de 100 × 25 px. Con 10 fotogramas, el número 5 está en la fila 5 div 4 = 1 y la columna 5 mod 4 = 1, en el píxel (100, 25).

**RF4. Pool de secuencias.** Un pool guarda las secuencias de fotogramas por nombre ("idle", "run"…). Cada hoja se carga una sola vez. Las instancias piden su secuencia al pool por nombre y comparten los mismos fotogramas; nadie vuelve a leer el disco. Pedir un nombre que no existe lanza una excepción.

**RF5. Entidades.** Hay dos tipos. Las estáticas muestran un solo fotograma de una secuencia del pool y no necesitan hilo. Las animadas recorren su secuencia, guardan su propio fotograma actual, su velocidad y su posición, y avanzan en su propio hilo. Hay 10 instancias animadas a la vez, el jugador incluido, y varias pueden usar la misma secuencia. Las 4 hojas mínimas se reparten entre las entidades estáticas, las animadas y el jugador; la tilesheet no cuenta.

**RF6. Velocidad independiente y regulable.** Cada instancia tiene sus fotogramas por segundo, independientes de las demás y del ritmo de pintado. Se pueden cambiar mientras la animación corre. El avance depende del tiempo transcurrido, no de cuántas veces se pinte.

**RF7. Pintado.** La ventana pinta todas las instancias a su ritmo propio, que también se puede regular.

**RF8. Posición por código.** `main` indica la posición en píxeles de cada entidad al crearla, y la entidad aparece ahí.

## Extras

Todo lo que va más allá de RF1–RF8 cuenta como extra. Se implementan en este orden, porque cada uno se apoya en el anterior.

**E1. Escena por tiles.** Una tilesheet se recorta con el mismo cálculo de RF3, reutilizando `SpriteSheet`. La escena es una matriz `int[][]` en la que cada celda guarda el índice del tile que va ahí. Como el fondo no cambia, se pinta una sola vez en una imagen y se reutiliza en cada vuelta de pintado. Las entidades se dibujan encima.

**E2. Jugador movido por teclado.** `Player` se mueve con las flechas a una velocidad en píxeles por segundo, por tiempo y no por pulsación. Tiene 8 secuencias: idle y walk, cada una hacia arriba, abajo, izquierda y derecha. Una secuencia idle puede ser un solo fotograma. La izquierda puede salir de la derecha en espejo: se voltea al cargar y se guarda en el pool como secuencia propia. La vista guarda qué flechas están pulsadas (pressed y released) en vez de mover en cada `keyPressed`, que depende de la repetición del teclado.

**E3. Colisión.** Cada tile se marca como sólido o transitable, por ejemplo con un conjunto de índices sólidos. Antes de mover, el modelo comprueba si la caja del jugador en la nueva posición pisa algún tile sólido o sale de la escena. Si es así, no avanza en ese eje. X e Y se comprueban por separado, para que el jugador se deslice por la pared en vez de pararse en seco.

**E4. Audio.** Música de fondo en bucle y un efecto al chocar, con `javax.sound.sampled` (`Clip`) y archivos WAV. El efecto suena una vez por choque, no en cada vuelta mientras el jugador empuja la pared. El audio es salida, como el pintado, así que vive en la vista. El modelo no conoce el audio: avisa del choque al presentador por su interfaz, y el presentador se lo pide a la vista.

Orden de pintado: fondo de tiles, entidades y, encima, el jugador.

## Arquitectura MVP

El programa se divide en modelo, vista y presentador. La vista y el modelo no se hablan nunca: todo pasa por el presentador.

- **Paquetes:** `main`, `model`, `view`, `presenter` y `contract`. Ni `model` importa nada de `view`, ni `view` nada de `model`.
- **Contrato:** tres interfaces en `contract`, una por archivo: `ModelContract`, `ViewContract` y `PresenterContract`. Cada método es algo que la otra parte necesita de verdad.
- **Inyección por constructor:** el presentador recibe `ModelContract` y `ViewContract` en su constructor, como interfaces y no como clases. Ninguna capa hace `new` de otra.
- **Montaje en `main`:** es el único sitio que crea las capas. Primero construye y conecta: crea el modelo, carga el pool, crea las instancias y la vista, y después el presentador. Al final arranca todo con `presenter.start()`.
- **Datos entre capas:** solo viajan datos inmutables, como un record `AnimationView` con la imagen, la posición, el nombre y los fps. La imagen viaja en copia. Ningún objeto del modelo llega a la vista.
- **La vista pide, nadie empuja:** en cada vuelta de pintado, la vista pide al presentador lo que tiene que dibujar. El modelo no avisa a nadie.

## Hilos

Cada hilo vive en una sola capa. Los hilos de las animaciones nunca tocan Swing, y el hilo de pintado nunca toca una instancia del modelo.

|   |   |   |   |
|---|---|---|---|
|Hilo|Capa|Qué hace|Qué toca|
|Uno por entidad animada|Modelo|Avanza su fotograma según el tiempo pasado y sus fps|Su propio índice de fotograma|
|Pintado|Vista|Pide al presentador qué dibujar y lo pinta, a su ritmo|Solo lee los datos que recibe|
|Eventos de Swing|Vista|Atiende los gestos del usuario y avisa al presentador|Cambia fps o posición a través del contrato|
|Jugador|Modelo|Extra E2. Mueve al jugador según las flechas pulsadas, comprueba la colisión y avanza su fotograma|Su posición, dirección y fotograma|

- Los campos que escribe un hilo y lee otro (fotograma actual, fps, posición) son `volatile`.
- La lista de instancias es segura para hilos, por ejemplo una `CopyOnWriteArrayList`, si se pueden añadir instancias mientras se pinta.
- El pool solo se llena en `main`, antes de arrancar los hilos. Después solo se lee, así que no necesita sincronización.
- Cada hilo lleva un nombre descriptivo, como `"Animation run #3"`, para reconocerlo al depurar.

## Clases mínimas

Los nombres se pueden cambiar, pero cada responsabilidad debe existir y estar en su paquete.

|   |   |   |
|---|---|---|
|Paquete|Clase|Responsabilidad|
|`main`|`Main`|Crea y configura las capas, llena el pool, crea las instancias y arranca la aplicación|
|`contract`|`ModelContract`|Lo que el presentador puede pedir al modelo: arrancar, contar instancias, dar sus datos por índice, cambiar fps y posición|
|`contract`|`ViewContract`|Lo que el presentador puede pedir a la vista: guardar el presentador, arrancar, leer y cambiar el ritmo de pintado|
|`contract`|`PresenterContract`|Lo que la vista puede pedir al presentador: qué pintar y los gestos del usuario|
|`contract`|`AnimationView`|Record inmutable con lo que se pinta de cada instancia|
|`model`|`SpriteSheet`|Carga una hoja en su constructor y recorta los fotogramas con div y mod|
|`model`|`SpritePool`|Guarda las secuencias por nombre y las entrega a las instancias|
|`model`|`SpriteAnimation`|Una entidad animada: secuencia compartida, fotograma actual, fps, posición y su hilo (`Runnable`)|
|`model`|`Model`|Lista de instancias; implementa `ModelContract`|
|`presenter`|`Presenter`|Aplica las reglas (límites de fps) y traduce el modelo a `AnimationView`|
|`view`|`View`|La ventana (`JFrame`) con su hilo de pintado; implementa `ViewContract`|
|`view`|`Viewer`|El lienzo donde se dibujan las animaciones|
|`model`|`StaticEntity`|Una entidad estática: un fotograma fijo de una secuencia del pool y su posición, sin hilo|
|`model`|`TileMap`|Extra E1 y E3. La matriz de índices de tiles, sus fotogramas y qué tiles son sólidos; responde si una caja choca|
|`model`|`Player`|Extra E2. Dirección, estado (idle o walk), posición y su hilo, que mueve y anima; comprueba la colisión con TileMap|
|`view`|`KeyHandler`|Extra E2. Escucha las flechas y avisa al presentador al pulsar y al soltar|
|`view`|`AudioPlayer`|Extra E4. Carga los WAV, reproduce la música en bucle y el efecto de choque|

## Restricciones y entregables

- Java 21, solo con la biblioteca estándar: AWT, Swing, `javax.imageio` y `javax.sound.sampled`. Sin librerías externas.
- Se compila y ejecuta con `javac` y `java`, sin Gradle ni Maven, con un script de ejecución.
- Las hojas, la tilesheet y los sonidos van en `src/main/resources/` y se cargan como recursos del classpath, no por ruta absoluta.
- El audio va en WAV: Java no lee MP3 ni OGG sin librerías externas.
- Al cerrar la ventana termina la aplicación con todos sus hilos (`EXIT_ON_CLOSE`).

Se entrega:

1. El código fuente, organizado en los paquetes anteriores.
2. Las hojas de sprites, la tilesheet y los sonidos, con un archivo que indique su origen y su licencia.
3. Un `README.md` con cómo ejecutarlo, qué animaciones hay y los controles.
4. La documentación del proyecto en Markdown, en `docs/`: arquitectura, contrato, hilos y decisiones.