# 🤖 Taller: Piedra, Papel o Tijera con Machine Learning

> **Nivel:** Secundaria / Hobbistas / Entusiastas de tecnología  
> **Conocimientos previos:** Programación básica (variables, funciones, condicionales)  
> **Duración estimada:** 2 – 3 horas  
> **Herramientas:** Navegador web, [p5.js Web Editor](https://editor.p5js.org/)

---

## 📋 Tabla de contenidos

1. [¿Qué vamos a construir?](#1-qué-vamos-a-construir)
2. [Conceptos clave](#2-conceptos-clave)
3. [Configurar el editor](#3-configurar-el-editor)
4. [Cámara y HandPose básico](#4-cámara-y-handpose-básico)
5. [Detectar gestos con la mano](#5-detectar-gestos-con-la-mano)
6. [Lógica del juego](#6-lógica-del-juego)
7. [La función draw() completa](#7-la-función-draw-completa)
8. [Código final completo](#8-código-final-completo)
9. [Retos y siguientes pasos](#9-retos-y-siguientes-pasos)
10. [Recursos adicionales](#10-recursos-adicionales)

---

## 1. ¿Qué vamos a construir?

Un juego de **Piedra, Papel o Tijera** que usa la cámara web para detectar la posición de tu mano en tiempo real, usando **Machine Learning** directamente en el navegador.

**¿Cómo funciona el juego?**

1. El jugador presiona **Espacio** → arranca la cuenta regresiva "1 – 2 – 3".
2. Al terminar el 3, el programa captura el gesto de la mano del jugador **y al mismo tiempo** la computadora elige al azar entre piedra, papel o tijera.
3. Se comparan ambas elecciones y se muestra el resultado: ganó, perdió o empate, con el marcador acumulado.

> La PC **no ve** la mano del jugador — elige completamente al azar.

---

## 2. Conceptos clave

### ¿Qué es Machine Learning?

Es enseñarle a una computadora a "aprender" reconociendo patrones en datos, en lugar de programar reglas explícitas una por una. En nuestro caso, el modelo ya viene **pre-entrenado** con millones de imágenes de manos — nosotros solo usamos sus resultados.

### Herramientas que usaremos

| Herramienta | ¿Para qué? |
|-------------|------------|
| **p5.js** | Dibujar y animar en el navegador con JavaScript sencillo |
| **ml5.js** | Biblioteca que pone modelos de ML listos para usar, construida sobre TensorFlow.js |
| **HandPose** | Modelo dentro de ml5.js que detecta 21 puntos clave de la mano en tiempo real |

> **¿Por qué ml5.js?** Porque no necesitamos saber matemáticas de ML — el modelo ya fue entrenado. Nosotros solo leemos sus resultados en pocas líneas de código.

### Los 21 keypoints de HandPose

HandPose detecta 21 puntos numerados sobre la mano. Cada punto tiene coordenadas `x`, `y` y un `name`.

```
Índices más importantes para este ejercicio:

[0]  wrist             — muñeca
[4]  thumb_tip         — punta del pulgar
[6]  index_finger_mcp  — nudillo del índice
[8]  index_finger_tip  — punta del índice
[10] middle_finger_mcp — nudillo del medio
[12] middle_finger_tip — punta del medio
[14] ring_finger_mcp   — nudillo del anular
[16] ring_finger_tip   — punta del anular
[18] pinky_mcp         — nudillo del meñique
[20] pinky_tip         — punta del meñique
```

### La idea clave para detectar gestos

En p5.js, `y = 0` está **arriba** y aumenta hacia abajo. Por lo tanto:

- Si la **punta del dedo** (`tip.y`) está **más arriba** que su **nudillo** (`mcp.y`), significa que `tip.y < mcp.y` → el dedo está **extendido**.
- Si la punta está más abajo que el nudillo → el dedo está **cerrado**.

---

## 3. Configurar el editor

### Paso 3.1 — Abrir el editor

1. Ve a **[editor.p5js.org](https://editor.p5js.org/)** en tu navegador.
2. Crea una cuenta gratuita (o inicia sesión) para poder guardar tu trabajo.
3. Haz clic en **File → New** para empezar un proyecto limpio.
4. Nómbralo: `Piedra Papel Tijera ML`.

### Paso 3.2 — Editar el archivo `index.html`

En el panel izquierdo verás los archivos del proyecto. Abre `index.html` y reemplaza su contenido con:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Piedra Papel Tijera ML</title>
    <link rel="stylesheet" type="text/css" href="style.css">
  
    <!-- Biblioteca p5.js -->    
    <script src="https://cdn.jsdelivr.net/npm/p5@2.2.3/lib/p5.js"></script>
  
    <!-- Biblioteca ml5.js -->
    <script src="https://unpkg.com/ml5@1/dist/ml5.min.js">  
  </script>
  </head>
  <body>
    <main>
    </main>
    <script src="sketch.js"></script>  
  </body>
</html>
```

> ⚠️ **Importante:** Si el editor ya tiene p5.js por defecto en el HTML, elimina esa línea y usa solo las que están arriba para garantizar la versión correcta.

### Paso 3.3 — Limpiar `sketch.js`

Abre `sketch.js` y borra todo su contenido. Empezaremos desde cero en los siguientes pasos.

> 💡 Al guardar con **Ctrl + S** el editor muestra el resultado en el panel derecho. Hazlo frecuentemente.

---

## 4. Cámara y HandPose básico

En este paso aprendemos a cargar el modelo HandPose y a mostrar la imagen de la cámara. Es la base de todo lo demás.

Copia este código en tu `sketch.js`:

```javascript
// Variables Globales
let handPose;   // instancia del modelo HandPose de ml5.js
let video;      // captura de video de la cámara web
let hands = []; // array con las manos detectadas en cada frame

// Setup: configura el sketch — canvas, cámara y detección.
// También se ejecuta una sola vez al inicio.
async function setup() {
  createCanvas(640, 480);

  // Cargamos el modelo handPose
  handPose = await ml5.handPose({ flipped: true });

  // Creamos la captura de la webcam y la ocultamos
  video = createCapture({ video: { width: 640, height: 480}, flipped: true  });
  video.hide();

  // Iniciamos la detección de manos desde el video de la webcam
  handPose.detectStart(video, gotHands);
}

// Draw: bucle principal (~60 fps).
// Por ahora solo dibuja la cámara y los keypoints.
function draw() {
  // Dibujamos el frame de la cámara como fondo
  image(video, 0, 0, width, height);

  // Dibujamos un círculo en cada keypoint detectado
  hands.forEach(hand => {
    hand.keypoints.forEach(kp => {
      // Color en RGB (rojo, verde, azul)
      fill(0, 255, 0);
      // Sin borde
      noStroke();
      // Dibujar círculo
      circle(kp.x, kp.y, 10);
    });
  });
}

// Callback De Detección: HandPose llama a esta función
// cada vez que actualiza sus predicciones.
// Guardamos el resultado en el array global "hands".
function gotHands(results) {
  hands = results;
}
```

> ✅ **¡Pruébalo!** Corre el sketch y pon tu mano frente a la cámara. Deberías ver **puntos verdes** sobre tus dedos. El navegador pedirá permiso para usar la cámara — acéptalo.

---

## 5. Detectar gestos con la mano

Ahora escribimos la función más importante: interpretar los keypoints para saber qué gesto está haciendo el usuario.

Agrega estas funciones a tu `sketch.js`:

```javascript
// Detección De Dedo Extendido
// Un dedo está extendido si su punta (tip) tiene
// coordenada Y menor que su nudillo base (mcp).
// En p5.js, Y=0 es arriba y crece hacia abajo,
// por eso "punta arriba" significa Y más pequeña.
//
// Parámetros:
//   hand   — objeto de mano devuelto por HandPose
//   tipIdx — índice del keypoint de la punta
//   mcpIdx — índice del keypoint del nudillo base
function dedoExtendido(hand, tipIdx, mcpIdx) {
  let tip = hand.keypoints[tipIdx];
  let mcp = hand.keypoints[mcpIdx];
  return tip.y < mcp.y; // punta más arriba = dedo extendido
}

// Reconocimiento De Gesto
// Evalúa qué dedos están extendidos y clasifica
// el gesto como "piedra", "papel", "tijera" o
// "..." si la mano está en transición.
//
// Índices usados (ml5 HandPose v1):
//   Índice : punta=8,  nudillo=6
//   Medio  : punta=12, nudillo=10
//   Anular : punta=16, nudillo=14
//   Meñique: punta=20, nudillo=18
function detectarGesto(hand) {
  // Evaluamos cada dedo (sin contar el pulgar)
  let indice = dedoExtendido(hand, 8,  6);
  let medio  = dedoExtendido(hand, 12, 10);
  let anular = dedoExtendido(hand, 16, 14);
  let meni   = dedoExtendido(hand, 20, 18);

  // Contamos cuántos de los cuatro dedos están abiertos
  let abiertos = [indice, medio, anular, meni]
    .filter(d => d).length;

  // Tijera: solo índice y medio extendidos
  if (indice && medio && !anular && !meni) return "Tijera";

  // Papel: los cuatro dedos extendidos
  if (abiertos === 4) return "Papel";

  // Piedra: puño cerrado, ningún dedo extendido
  if (abiertos === 0) return "Piedra";

  // Ninguna regla coincide: la mano está en transición
  return "...";
}
```

**Para probarlo**, agrega esto temporalmente al final de `draw()`:

```javascript
  // Determinar el gesto
  if (hands.length > 0) {
    let gesto = detectarGesto(hands[0]);
    fill(255);
    textSize(40);
    text(gesto, 20, 50);
  }
```

Guarda y corre el sketch. Prueba hacer piedra ✊, papel 🖐️ y tijera ✌️ — el texto debe cambiar en pantalla.

---

## 6. Lógica del juego

Ahora armamos los estados del juego, la elección aleatoria de la PC y quién gana.

### Variables de estado

Agrega estas variables al **inicio** de `sketch.js`, antes de `setup()`. Verifica las ya existentes:

```javascript
// Variables Globales
let handPose;          // instancia del modelo HandPose de ml5.js
let video;             // captura de video de la cámara web
let hands = [];        // array con las manos detectadas en cada frame

// Estado del juego: "esperar" | "cuenta" | "resultado"
let modoJuego   = "esperar";
let cuentaTimer = 0;   // marca de tiempo cuando inicia la cuenta regresiva

// Elecciones y resultado de la ronda actual
let gestoUsuario = "";
let gestoPC      = "";
let resultado    = "";

// Marcador acumulado de la sesión
let puntosUsuario = 0;
let puntosPC      = 0;

// Opciones válidas para la elección aleatoria de la PC
const GESTOS = ["Piedra", "Papel", "Tijera"];
```

### Elección aleatoria de la PC

Agrega la siguiente función al **final** de `sketch.js`.

```javascript
// Elección aleatoria de la computadora
// La computadora elige uno de los tres gestos al azar,
// sin ninguna relación con el gesto del jugador.
function pcElige() {
  return GESTOS[floor(random(GESTOS.length))];
}
```

### Lógica de victoria

Agrega la siguiente función al **final** de `sketch.js`.

```javascript
// Lógica de victoria
// Compara el gesto del jugador con el de la computadora
// y devuelve el texto del resultado.
// También actualiza el marcador acumulado.
function quienGana(usuario, pc) {
  // Caso empate
  if (usuario === pc) return "Empate";

  // Casos en que el jugador gana:
  // piedra vence tijera, tijera vence papel, papel vence piedra
  if (
    (usuario === "Piedra" && pc === "Tijera") ||
    (usuario === "Tijera" && pc === "Papel")  ||
    (usuario === "Papel"  && pc === "Piedra")
  ) {
    puntosUsuario++;
    return "¡Ganaste!";
  }

  // En cualquier otro caso, gana la PC
  puntosPC++;
  return "Ganó la computadora";
}
```

### Control de teclado

Agrega la siguiente función al **final** de `sketch.js`.

```javascript
// Control de teclado
// ESPACIO inicia una ronda (desde "esperar")
// o descarta el resultado y vuelve a esperar.
function keyPressed() {
  if (key === ' ' && modoJuego === "esperar") {
    // Inicia la cuenta regresiva
    modoJuego   = "cuenta";
    cuentaTimer = millis();
  }
  if (key === ' ' && modoJuego === "resultado") {
    // Vuelve a la pantalla de espera para una nueva ronda
    modoJuego = "esperar";
  }
}
```

---

## 7. La función draw() completa

Reemplaza la función `draw()` con esta versión que maneja los tres estados del juego, e incluye las funciones auxiliares de dibujo:

```javascript
// Dibujar Keypoints
// Dibuja un círculo verde en cada uno de los
// 21 puntos de la mano detectada por HandPose.
function dibujarKeypoints(hand) {
  fill(0, 255, 100, 200);
  noStroke();

  // forEach recorre cada keypoint sin necesitar índice manual
  hand.keypoints.forEach(kp => {
    circle(kp.x, kp.y, 8);
  });
}

// Dibujar Hud (estado "esperar")
// Muestra el gesto detectado en tiempo real
// y el mensaje para iniciar la partida.
function dibujarHUD(gesto) {
  // Franja semitransparente en la parte inferior
  fill(0, 0, 0, 140);
  noStroke();
  rect(0, height - 60, width, 60);

  // Texto con el gesto actual y la instrucción
  fill(255);
  textAlign(CENTER);
  textSize(20);
  text("Gesto: " + gesto + "   |   ESPACIO = jugar", width / 2, height - 25);
}

// Dibujar Resultado (estado "resultado")
// Muestra el veredicto de la ronda, el gesto
// del jugador vs el de la PC, y el prompt
// para continuar.
function dibujarResultado() {
  // Overlay oscuro sobre la imagen de la cámara
  fill(0, 0, 0, 170);
  noStroke();
  rect(0, 0, width, height);

  textAlign(CENTER);

  // Resultado principal: "¡Ganaste!", "Ganó la PC" o "Empate"
  fill(255, 220, 0);
  textSize(52);
  text(resultado, width / 2, height / 2 - 40);

  // Comparación: gesto del jugador vs gesto de la PC
  fill(255);
  textSize(22);
  text(
    "Tú: " + gestoUsuario + "  vs  PC: " + gestoPC,
    width / 2, height / 2 + 20
  );

  // Instrucción para continuar
  fill(200);
  textSize(16);
  text("ESPACIO = nueva ronda", width / 2, height / 2 + 70);
}

// Draw: bucle principal (~60 fps)
// Gestiona los tres estados del juego:
//   1. "esperar"   — muestra la cámara y el gesto en vivo
//   2. "cuenta"    — muestra la cuenta regresiva 1-2-3
//   3. "resultado" — muestra quién ganó la ronda
function draw() {
  // Dibujamos el frame de la cámara como fondo
  image(video, 0, 0, width, height);

  // Detectamos el gesto actual del jugador (si hay mano visible)
  let gestoActual = "...";
  if (hands.length > 0) {
    gestoActual = detectarGesto(hands[0]);
    dibujarKeypoints(hands[0]);
  }

  // Estado: Esperar
  // El jugador prepara su gesto. Mostramos el
  // gesto detectado en tiempo real.
  if (modoJuego === "esperar") {
    dibujarHUD(gestoActual);
  }

  // Estado: Cuenta Regresiva
  // Mostramos 1, 2 y 3 con un segundo de pausa
  // entre cada número. Al llegar a los 3 segundos,
  // capturamos el gesto del jugador, la PC elige
  // al azar y calculamos el resultado.
  if (modoJuego === "cuenta") {
    let elapsed = millis() - cuentaTimer;

    if (elapsed < 3000) {
      // Calculamos qué número mostrar (1, 2 o 3)
      let numero = floor(elapsed / 1000) + 1;
      fill(255, 220, 0);
      textAlign(CENTER);
      textSize(120);
      text(numero, width / 2, height / 2 + 40);
    } else {
      // La cuenta terminó: capturamos gestos y evaluamos
      gestoUsuario = gestoActual;
      gestoPC      = pcElige();        // la PC elige al azar
      resultado    = quienGana(gestoUsuario, gestoPC);
      modoJuego    = "resultado";
    }
  }

  // Estado: Resultado
  // Mostramos el veredicto de la ronda.
  if (modoJuego === "resultado") {
    dibujarResultado();
  }

  // Marcador siempre visible en la esquina superior izquierda
  fill(255);
  textAlign(LEFT);
  textSize(16);
  text("Tú: " + puntosUsuario + "  |  PC: " + puntosPC, 10, 25);
}
```

---

## 8. Código final completo

Este es el `sketch.js` completo listo para copiar y pegar. Incluye todos los pasos anteriores integrados, con comentarios en todos los bloques.

```javascript
// Variables Globales
let handPose;   // instancia del modelo HandPose de ml5.js
let video;      // captura de video de la cámara web
let hands = []; // array con las manos detectadas en cada frame

// Estado del juego: "esperar" | "cuenta" | "resultado"
let modoJuego   = "esperar";
let cuentaTimer = 0;   // marca de tiempo cuando inicia la cuenta regresiva

// Elecciones y resultado de la ronda actual
let gestoUsuario = "";
let gestoPC      = "";
let resultado    = "";

// Marcador acumulado de la sesión
let puntosUsuario = 0;
let puntosPC      = 0;

// Opciones válidas para la elección aleatoria de la PC
const GESTOS = ["Piedra", "Papel", "Tijera"];

// Setup: configura el sketch — canvas, cámara y detección.
// También se ejecuta una sola vez al inicio.
async function setup() {
  createCanvas(640, 480);

  // Cargamos el modelo handPose
  handPose = await ml5.handPose({ flipped: true });

  // Creamos la captura de la webcam y la ocultamos
  video = createCapture({ video: { width: 640, height: 480}, flipped: true  });
  video.hide();

  // Iniciamos la detección de manos desde el video de la webcam
  handPose.detectStart(video, gotHands);
}

// Dibujar Keypoints
// Dibuja un círculo verde en cada uno de los
// 21 puntos de la mano detectada por HandPose.
function dibujarKeypoints(hand) {
  fill(0, 255, 100, 200);
  noStroke();

  // forEach recorre cada keypoint sin necesitar índice manual
  hand.keypoints.forEach(kp => {
    circle(kp.x, kp.y, 8);
  });
}

// Dibujar Hud (estado "esperar")
// Muestra el gesto detectado en tiempo real
// y el mensaje para iniciar la partida.
function dibujarHUD(gesto) {
  // Franja semitransparente en la parte inferior
  fill(0, 0, 0, 140);
  noStroke();
  rect(0, height - 60, width, 60);

  // Texto con el gesto actual y la instrucción
  fill(255);
  textAlign(CENTER);
  textSize(20);
  text("Gesto: " + gesto + "   |   ESPACIO = jugar", width / 2, height - 25);
}

// Dibujar Resultado (estado "resultado")
// Muestra el veredicto de la ronda, el gesto
// del jugador vs el de la PC, y el prompt
// para continuar.
function dibujarResultado() {
  // Overlay oscuro sobre la imagen de la cámara
  fill(0, 0, 0, 170);
  noStroke();
  rect(0, 0, width, height);

  textAlign(CENTER);

  // Resultado principal: "¡Ganaste!", "Ganó la PC" o "Empate"
  fill(255, 220, 0);
  textSize(52);
  text(resultado, width / 2, height / 2 - 40);

  // Comparación: gesto del jugador vs gesto de la PC
  fill(255);
  textSize(22);
  text(
    "Tú: " + gestoUsuario + "  vs  PC: " + gestoPC,
    width / 2, height / 2 + 20
  );

  // Instrucción para continuar
  fill(200);
  textSize(16);
  text("ESPACIO = nueva ronda", width / 2, height / 2 + 70);
}

// Draw: bucle principal (~60 fps)
// Gestiona los tres estados del juego:
//   1. "esperar"   — muestra la cámara y el gesto en vivo
//   2. "cuenta"    — muestra la cuenta regresiva 1-2-3
//   3. "resultado" — muestra quién ganó la ronda
function draw() {
  // Dibujamos el frame de la cámara como fondo
  image(video, 0, 0, width, height);

  // Detectamos el gesto actual del jugador (si hay mano visible)
  let gestoActual = "...";
  if (hands.length > 0) {
    gestoActual = detectarGesto(hands[0]);
    dibujarKeypoints(hands[0]);
  }

  // Estado: Esperar
  // El jugador prepara su gesto. Mostramos el
  // gesto detectado en tiempo real.
  if (modoJuego === "esperar") {
    dibujarHUD(gestoActual);
  }

  // Estado: Cuenta Regresiva
  // Mostramos 1, 2 y 3 con un segundo de pausa
  // entre cada número. Al llegar a los 3 segundos,
  // capturamos el gesto del jugador, la PC elige
  // al azar y calculamos el resultado.
  if (modoJuego === "cuenta") {
    let elapsed = millis() - cuentaTimer;

    if (elapsed < 3000) {
      // Calculamos qué número mostrar (1, 2 o 3)
      let numero = floor(elapsed / 1000) + 1;
      fill(255, 220, 0);
      textAlign(CENTER);
      textSize(120);
      text(numero, width / 2, height / 2 + 40);
    } else {
      // La cuenta terminó: capturamos gestos y evaluamos
      gestoUsuario = gestoActual;
      gestoPC      = pcElige();        // la PC elige al azar
      resultado    = quienGana(gestoUsuario, gestoPC);
      modoJuego    = "resultado";
    }
  }

  // Estado: Resultado
  // Mostramos el veredicto de la ronda.
  if (modoJuego === "resultado") {
    dibujarResultado();
  }

  // Marcador siempre visible en la esquina superior izquierda
  fill(255);
  textAlign(LEFT);
  textSize(16);
  text("Tú: " + puntosUsuario + "  |  PC: " + puntosPC, 10, 25);
} 

// Callback De Detección: HandPose llama a esta función
// cada vez que actualiza sus predicciones.
// Guardamos el resultado en el array global "hands".
function gotHands(results) {
  hands = results;
}

// Detección De Dedo Extendido
// Un dedo está extendido si su punta (tip) tiene
// coordenada y menor que su nudillo base (mcp).
// En p5.js, y=0 es arriba y crece hacia abajo,
// por eso "punta arriba" significa Y más pequeña.
//
// Parámetros:
//   hand   — objeto de mano devuelto por HandPose
//   tipIdx — índice del keypoint de la punta
//   mcpIdx — índice del keypoint del nudillo base
function dedoExtendido(hand, tipIdx, mcpIdx) {
  let tip = hand.keypoints[tipIdx];
  let mcp = hand.keypoints[mcpIdx];
  return tip.y < mcp.y; // punta más arriba = dedo extendido
}

// Reconocimiento De Gesto
// Evalúa qué dedos están extendidos y clasifica
// el gesto como "piedra", "papel", "tijera" o
// "..." si la mano está en transición.
//
// Índices usados (ml5 HandPose v1):
//   Índice : punta=8,  nudillo=6
//   Medio  : punta=12, nudillo=10
//   Anular : punta=16, nudillo=14
//   Meñique: punta=20, nudillo=18
function detectarGesto(hand) {
  // Evaluamos cada dedo (sin contar el pulgar)
  let indice = dedoExtendido(hand, 8,  6);
  let medio  = dedoExtendido(hand, 12, 10);
  let anular = dedoExtendido(hand, 16, 14);
  let meni   = dedoExtendido(hand, 20, 18);

  // Contamos cuántos de los cuatro dedos están abiertos
  let abiertos = [indice, medio, anular, meni]
    .filter(d => d).length;

  // Tijera: solo índice y medio extendidos
  if (indice && medio && !anular && !meni) return "Tijera";

  // Papel: los cuatro dedos extendidos
  if (abiertos === 4) return "Papel";

  // Piedra: puño cerrado, ningún dedo extendido
  if (abiertos === 0) return "Piedra";

  // Ninguna regla coincide: la mano está en transición
  return "...";
}

// Elección aleatoria de la computadora
// La computadora elige uno de los tres gestos al azar,
// sin ninguna relación con el gesto del jugador.
function pcElige() {
  return GESTOS[floor(random(GESTOS.length))];
}

// Lógica de victoria
// Compara el gesto del jugador con el de la computadora
// y devuelve el texto del resultado.
// También actualiza el marcador acumulado.
function quienGana(usuario, pc) {
  // Caso empate
  if (usuario === pc) return "Empate";

  // Casos en que el jugador gana:
  // piedra vence tijera, tijera vence papel, papel vence piedra
  if (
    (usuario === "Piedra" && pc === "Tijera") ||
    (usuario === "Tijera" && pc === "Papel")  ||
    (usuario === "Papel"  && pc === "Piedra")
  ) {
    puntosUsuario++;
    return "¡Ganaste!";
  }

  // En cualquier otro caso, gana la PC
  puntosPC++;
  return "Ganó la computadora";
}

// Control de teclado
// ESPACIO inicia una ronda (desde "esperar")
// o descarta el resultado y vuelve a esperar.
function keyPressed() {
  if (key === ' ' && modoJuego === "esperar") {
    // Inicia la cuenta regresiva
    modoJuego   = "cuenta";
    cuentaTimer = millis();
  }
  if (key === ' ' && modoJuego === "resultado") {
    // Vuelve a la pantalla de espera para una nueva ronda
    modoJuego = "esperar";
  }
}
```

---

## 9. Retos y siguientes pasos

El juego básico funciona. Aquí tienes retos para extender tu versión:

### 🟢 Nivel básico

1. **Emojis en vez de texto** — Muestra ✊ para piedra, 🖐️ para papel y ✌️ para tijera usando `text()` con un `textSize()` grande.
2. **Colores por resultado** — Si ganás, el fondo del overlay se pone verde; si perdés, rojo; si es empate, amarillo.
3. **Sonido** — Usa `new Audio()` para reproducir un sonido al ganar o perder.

### 🟡 Nivel intermedio

1. **Detectar el pulgar** — El pulgar se mueve en eje horizontal, no vertical. Investiga los keypoints 2, 3 y 4 para agregarlo a la detección y mejorar la precisión.
2. **Detección por votación** — Detecta el gesto durante 10 frames seguidos antes de tomarlo como definitivo. Esto elimina falsas lecturas durante la transición.
3. **Cuenta hablada** — Usa la Web Speech API (`speechSynthesis`) para que el browser diga "uno, dos, tres" en voz alta.

### 🔴 Nivel avanzado

1. **Dos jugadores** — ml5 HandPose v1 puede detectar múltiples manos. Modifica el sketch para que dos personas jueguen frente a la misma cámara, cada una con una mano.
2. **Guardar estadísticas** — Usa `localStorage` del navegador para guardar el historial de victorias entre sesiones.
3. **Pantalla de instrucciones** — Agrega un estado `"instrucciones"` con dibujos en p5.js que muestren cómo hacer cada gesto correctamente.

---