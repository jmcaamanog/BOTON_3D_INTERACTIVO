# Guía Completa del Proyecto — Pulsador Musical 3D (100×100 mm · Módulo 8 MB)

Este documento reúne todo lo aprendido, diseñado y construido en este proyecto para que puedas entender cómo funciona por dentro, cómo publicarlo y compartirlo en cualquier dispositivo, y cómo replicar este mismo flujo en futuros proyectos de impresión 3D + visor web interactivo.

---

## 1. Estructura limpia del proyecto (sin archivos duplicados)

```text
boton_mercadona/
├── index.html          ← Visor 3D universal (Ordenador + iPhone + Android, 100% offline y listo para web)
├── manifest.json       ← Configuración PWA para instalar el visor como app en Android e iPhone
├── GUIA_PROYECTO.md    ← Esta guía técnica y de aprendizaje
├── stl.zip             ← Paquete comprimido con los 10 archivos .STL listos para imprimir
├── step/               ← Los 7 archivos CAD paramétricos (.STEP AP214) sin duplicar
│   ├── boton_emergencia_conjunto_completo.step
│   ├── 1_base_inferior_gris.step
│   ├── 2_tapa_superior_amarilla.step
│   ├── 3_boton_seta_rojo.step
│   ├── 4_cuello_estriado_y_accesorios.step
│   ├── 5_soporte_pulsador_interior.step
│   └── 6_muelle_imprimible.step
├── stl/                ← Los 10 archivos .STL individuales listos para laminar (PLA/PETG)
└── recursos/           ← Material original de referencia (fotos, medidas y audio)
    ├── boton.jpeg
    ├── mercadona_mercadona_cancion.mp3
    └── electronica/    ← Fotos con cotas reales del módulo de sonido 8 MB (LJY-YYH-V1)
```

> **Ventaja clave de esta estructura**: existe una **única versión universal** de `index.html`. Sirve exactamente igual si la abres con **doble clic en un ordenador** (incluso sin internet) que si arrastras toda la carpeta a **Netlify Drop** para compartir un enlace por WhatsApp compatible a la vez con **Android, iPhone y PC**.

---

## 2. Cómo se diseñó el proyecto paso a paso (de la electrónica al visor 3D)

### Fase A · Levantamiento exacto de la electrónica real
Partimos de las fotografías con cotas reales del **Módulo de Sonido Grabable 8 MB (`LJY-YYH-V1`)**:
- **Placa PCB**: `47.0 × 25.0 × 1.6 mm` con puerto **USB-C** e interruptor deslizante **ON/OFF** en el mismo borde de `25 mm` (el conector USB-C sobresale `~1.5 mm` del borde), conectores JST (`BAT`, `SP`, `PLAY`), micrófono y botones `VOL`/`REC`.
- **Altavoz**: redondo `Ø35.0 mm` (`8 Ω · 0.5 W`) con bornera verde lateral.
- **Batería LiPo**: `X502030` (`3.7 V · 200 mAh`), dimensiones `30.0 × 20.0 × 5.0 mm`.
- **Pulsador táctil**: cuadrado `12.0 × 12.0 mm` cableado al conector `PLAY`.

### Fase B · Rediseño mecánico y CAD 3D (impresión multicolor Snapmaker U1)
Para adaptar la caja de `100 × 100 × 102 mm` a la electrónica real y mejorar su ergonomía y realismo, se rediseñaron y validaron los sólidos CAD en Python:
1. **Base inferior (`1_base_inferior_gris`)**:
   - **Cámara interior optimizada** con 3 bahías independientes y holguras de impresión (`+0.5 / +0.8 mm`):
     - Cuna PCB (`47.4 × 25.6 mm`) con repisa perimetral a `Z = 5.5 mm` para librar soldaduras y tope trasero escalonado reforzado.
     - Cuna acústica circular (`Ø35.6 mm` interior) alineada con las 5 ranuras frontales de salida de sonido.
     - Cuna en U para batería LiPo (`30.8 × 20.8 mm`) con canal para cables hacia la PCB.
     - Puente central rediseñado de 3 pilares que despeja por completo las tres bahías electrónicas.
   - **Ventana lateral derecha USB-C + ON/OFF abocinada en 3 escalones** (`20.6 × 9.2 mm` interior → `23.0 × 11.3 mm` rebaje → `25.0 × 12.6 mm` bisel exterior) para enchufar cualquier cable USB-C cómodamente desde fuera.
   - **Detalles funcionales y estéticos añadidos**:
     - **Iconos exteriores en relieve** (`USB` e indicador `I / O`) encima de la ventana lateral derecha.
     - **Aleta ergonómica de clic (`Aleta_Click_ON_OFF`)** para accionar el pequeño interruptor `ON/OFF` con el dedo desde el exterior.
2. **Soporte interior del pulsador (`5_soporte_pulsador_interior`)**:
   - Cajera central rebajada de `12.5 × 12.5 × 3.0 mm` con canal de salida de cables de `5.0 mm`, montada sobre dos brazos elásticos tangenciales (`15.6 mm`) para amortiguar el final de carrera.
3. **Accesorios y tornillería (`4_cuello_estriado_y_accesorios`)**:
   - Se añadieron las **4 tuercas hexagonales M4 (`Tuerca_Esquina_1..4`)** a juego con los 4 tornillos avellanados de esquina, además del cuello estriado dorado (`40` estrías), el clip retenedor en C y la tuerca hexagonal lateral.
4. **Verificación geométrica estricta (regla de oro para impresión 3D)**:
   - Todos los archivos `.STL` fueron verificados topológicamente como **mallas 2-manifold cerradas (watertight)**: `0` aristas abiertas (`boundary = 0`), `0` aristas no-manifold (`non_manifold = 0`) y `0` normales invertidas (`bad_orient = 0`).
   - Todos los archivos `.STEP` (AP214) se verificaron con **0 referencias rotas** y sólidos independientes por color para laminar en extrusores multicolor.
   - **Convención de ejes**: los `.step` y `.stl` individuales están apoyados en cama (`Z_min = 0.0 mm`), mientras que el conjunto completo y el visor 3D conservan sus coordenadas reales de ensamblaje.

### Fase C · Desarrollo del visor 3D web interactivo (`index.html`)
En lugar de depender de librerías pesadas o servidores, construimos un visor CAD a medida en **HTML + WebGL 1.0 puro + shaders GLSL + JavaScript** con todos los recursos embebidos en Base64:
- **Geometría 3D incrustada**: las 27 mallas triangulares codificadas como `Float32Array` en Base64 (`window.MODEL_PARTS`), más la electrónica modelada proceduralmente a escala real (con cable USB-C animado).
- **Audio MP3 y fotos reales incrustados**: la canción (`data:audio/mpeg;base64,...`) y las 3 fotos con cotas reales (`data:image/avif;base64,...`) viajan dentro del propio archivo.
- **Tailwind CSS embebido**: la librería de estilos está integrada inline dentro de `index.html` para que cargue al instante y funcione **100% sin internet**.

---

## 3. Cómo funciona por dentro: herramientas, controles y arquitectura

### Mapa de funciones del visor 3D

| Sección / Control | Qué hace | Cómo está implementado técnicamente |
| :--- | :--- | :--- |
| **Botón rojo `PULSAR`** (o clic en la seta / tecla `Espacio`) | Hunde el botón `8 mm`, comprime el muelle, reproduce la canción MP3 y vibra en Android. | Interpolación senoidal de traslación Z (`-8 mm`) y escala Z del muelle respecto al pivote `Z = 49 mm`, reproducción `<audio>` con desbloqueo táctil iOS/Android y `navigator.vibrate()`. |
| **Orden de montaje en 7 pasos** | 1º Inf. Abajo (Base + Electrónica + Soporte) → 2º Sup. Arriba (Tapa + Cuello + Muelle + Seta + Clip) → 3º Unión vertical → 4º 4 Tornillos → 5º 4 Tuercas → 6º Tuerca Lateral → 7º Pulsación + Canción automática. | Función cinemática `getPartKinematics(p)` basada en ventanas normalizadas `t ∈ [0, 1]` con suavizado cúbico `easeOutCubic`; cada tornillo y tuerca está separado en su propia malla (`1..4`) para girar sobre su propio eje `(±38, ±38)`. |
| **Barra superior derecha: Zoom suave + Avance/Bucle** | Zoom logarítmico (`30%` a `445%`, inicial `44%`) con `−`, barra, `+` y `⌖` encajar; debajo, con igual ancho: `↺` reiniciar, `▶ / ⏸` reproducir/pausar, barra manual y `☑ Bucle`. | Amortiguación exponencial independiente de FPS (`1 - e^(-11·dt)`), escala logarítmica para zoom perceptualmente uniforme e iconos SVG vectoriales. |
| **Corte 3D macizo (`Corte 3D`)** | Plano 3D visible y **arrastrable directamente con el ratón o dedo** en los ejes **X / Y / Z**, con inversión de lado y acceso directo `USB`. Los sólidos cortados se ven **macizos con rayado CAD a 45°**. | Técnica **WebGL Stencil Buffer Capping** (`stencil: true`): pasada de conteo de caras frontales (`INCR_WRAP`) vs traseras (`DECR_WRAP`) y dibujo del plano de tapa donde `stencil != 0` con sombreado rayado en el Fragment Shader. |
| **Interruptor `Persp \| Iso`** | Alterna entre proyección perspectiva cónica y vista isométrica/ortográfica CAD sin fuga. | Cambio en tiempo real entre matriz `mat4Persp(...)` y `mat4Ortho(...)` ajustada al mismo tamaño aparente en pantalla. |
| **Modo Rayos X (`☢ Rayos X`)** | `OFF` → **Lupa** (escáner circular que atraviesa la carcasa siguiendo el cursor o centrado en móvil) → **Global** (carcasa holográfica translúcida). | Fragment Shader con distancia en píxeles a `uMouseFrag` (`gl_FragCoord.xy`) descartando la carcasa opaca dentro del radio `uLensRadius` y redibujándola en pasada translúcida con efecto Fresnel. |
| **Cotas y Regla 3D (`📏 Cotas`)** | `OFF` → **Cotas 3D** (medidas clave proyectadas en SVG) → **Regla 2-Pts** (clic en punto A y punto B sobre cualquier pieza 3D para medir distancia real `mm` y `ΔX / ΔY / ΔZ`). | Raycasting exacto rayo-triángulo (algoritmo **Möller–Trumbore**) transformando el rayo de cámara al espacio local de cada malla con la inversa de su matriz de modelo. |
| **Multi-selección, Aislamiento y Descarga (`.STL` / `.STEP` / `.ZIP`)** | Permite marcar una o varias piezas (o subconjuntos completos), aislarlas y descargar sus archivos imprimibles. | El botón **`Descargar .STL`** construye el archivo **STL binario real en el navegador** (`ArrayBuffer` + `DataView`, 84 bytes de cabecera + 50 bytes por triángulo) aplicando el desplazamiento `printZShift` para que cada pieza caiga plana en `Z = 0`. |
| **Información Detallada + `VER` + `INFO DETALLADA`** | Muestra resumen limpio; en `Electrónica 8 MB` muestra el botón **`VER`** que abre las 3 fotos reales con medidas sobre el 3D; los datos técnicos (`STEP`, cotas, volumen, filamento...) viven en el desplegable `INFO DETALLADA` replegado por defecto. | Modal flotante (`z-40`) alimentado por `window.ELECTRONICS_PHOTOS` (Base64 AVIF) con pestañas intercambiables y cierre por clic exterior o `Esc`. |
| **Paneles flotantes replegables + Perímetro amarillo** | Todos los bloques (`Ajustes`, `Piezas`, `Despiece y Vistas`, `Información Detallada`) flotan sobre el 3D y se repliegan individualmente o de golpe; solo los elementos activos/marcados llevan borde amarillo. | Regla unificada `applyActivePerimeter()` + diseño adaptable: en PC el panel arranca abierto a la izquierda; en móvil (`< 768 px`) se convierte en menú deslizable `☰ Panel` con fondo atenuado y etiquetas ocultas al iniciar. |

### Atajos de teclado (Ordenador) y gestos táctiles (Móvil)
- **Ordenador**: arrastrar botón izq. = girar · botón der. o `Shift` + arrastrar = mover cámara · rueda = zoom suave · `Espacio` = pulsar + canción · `+` / `-` / `0` = zoom y encaje · `Esc` = cerrar ventana de fotos.
- **Móvil (Android / iPhone)**: 1 dedo = girar · 2 dedos (pellizco) = zoom suave y mover cámara · tocar seta roja o **`PULSAR`** = pulsar + canción (+ vibración háptica en Android).

---

## 4. Cómo publicar este proyecto (o cualquier futuro proyecto) en la web gratis

### Opción A · Netlify Drop (la más rápida — 30 segundos, sin comandos)
Ideal para compartir un enlace por WhatsApp que funcione al instante en **Android, iPhone y PC**:
1. Asegúrate de que tu archivo principal se llame **`index.html`** dentro de la carpeta del proyecto (aquí ya está listo así).
2. Entra en **https://app.netlify.com/drop** desde el ordenador.
3. **Arrastra la carpeta completa** (`boton_mercadona`) dentro del recuadro punteado.
   - *Límite recomendado*: hasta ~100 MB (esta carpeta completa pesa ~58 MB y sube en pocos segundos).
4. **Reclama el sitio con una cuenta gratuita** (*Claim site* con Google o correo):
   - Sin cuenta el enlace caduca en **1 hora**.
   - Con cuenta gratuita **no caduca nunca** y puedes personalizar el nombre en *Site configuration → Change site name* (ej. `mi-proyecto-3d.netlify.app`).
5. Comparte el enlace `https://...netlify.app` por WhatsApp o Telegram (ejemplo real de este proyecto ya publicado y funcionando: **https://boton-mercadona-3d.netlify.app/**).

> **Para actualizar el proyecto en el futuro**: entra en tu panel de Netlify → tu sitio → pestaña **Deploys** y vuelve a arrastrar la carpeta actualizada. El enlace sigue siendo el mismo.

### Opción B · GitHub Pages (gratis para siempre con control de versiones)
1. Crea un repositorio público en GitHub y sube la carpeta del proyecto (con `index.html` en la raíz).
2. En el repositorio ve a **Settings → Pages → Source: Deploy from a branch (`main` / `/root`)**.
3. En 1 minuto tendrás un enlace fijo `https://tu-usuario.github.io/nombre-del-repo/`.

### Opción C · Envío directo por archivo para Ordenador (sin subir a ninguna web)
- Si solo quieres pasárselo a alguien en **Windows o Mac**, basta con enviarle **`index.html`** (4,67 MB). Al hacer **doble clic** funciona todo al 100% incluso sin internet (3D, canción, fotos y exportación `.STL`). Si también necesita los `.STEP` y `stl.zip`, envíale la carpeta entera en un `.zip`.
- **Importante para móviles**: evita enviar un `.html` adjunto por WhatsApp para abrir en el teléfono, ya que el visor interno de Android/iOS suele bloquear WebGL y el audio. En móvil usa siempre un enlace `https://` (Netlify Drop, GitHub Pages, Vercel o Cloudflare Pages).

---

## 5. Detalles técnicos clave para recordar en futuros proyectos (lecciones de QA)

Si mañana haces otro diseño 3D + visor web parecido, ten presente esta lista de trucos que marcan la diferencia entre un prototipo y una app pulida:

1. **Altura de pantalla móvil (`100dvh`) y Safe Areas**: nunca uses solo `100vh` en móvil porque la barra inferior de Safari (iPhone) tapa los botones. Usa `height: 100dvh` junto con `viewport-fit=cover` y `env(safe-area-inset-top / bottom)`.
2. **Bloquear *pull-to-refresh* y zoom accidental**: añade `overscroll-behavior: none` en `html, body`, `touch-action: none` en el `<canvas>` y `e.preventDefault()` en `gesturestart / gesturechange` para que girar el 3D en Android/iPhone no recargue ni haga zoom a la web.
3. **Política de audio en móviles**: Chrome y Safari bloquean `audio.play()` hasta la primera interacción del usuario. Añade siempre un desbloqueo en el primer `touchstart` / `pointerdown` con `playsinline` y muestra un pequeño aviso interactivo si el autoplay inicial es bloqueado.
4. **Limite de resolución Retina (DPR)**: los móviles tienen pantallas `3×` (`devicePixelRatio = 3`), lo que multiplica por 9 los píxeles a renderizar. Limitar a `Math.min(window.devicePixelRatio || 1, 2)` mantiene nitidez máxima a 60 fps sin recalentar el teléfono ni perder el contexto WebGL.
5. **WebGL robusto en Windows (ANGLE/D3D11)**: en shaders GLSL ES 1.00 evita `pow(0.0, exp)` y `normalize(vec3(0.0))` protegiéndolos con un mínimo (`max(x, 0.0001)`), así nunca aparecen píxeles negros/NaN en tarjetas gráficas estrictas.
6. **Un solo archivo autocontenido**: incrustar las mallas, el audio, las fotos y el CSS (Base64 / inline) evita rutas rotas, problemas CORS en `file://` al hacer doble clic y elimina latencia en móvil.
7. **Verificación automatizada antes de publicar**:
   - Comprobar topología 3D (`boundary = 0`, `non_manifold = 0`, `bad_orient = 0`).
   - Pasar `node --check` sobre cada bloque `<script>` para garantizar 0 errores de sintaxis.
   - Comprobar el encaje visual en anchos reales de móvil (`360 px` Android, `375 px` iPhone SE, `390–430 px` modernos y horizontal) además de escritorio.
