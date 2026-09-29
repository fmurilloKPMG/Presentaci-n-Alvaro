# PROMPT — Rediseño de "Krino · KPMG Access Intelligence" como keynote horizontal

## 0. Rol y objetivo

Actúa como un equipo senior formado por un director de arte de keynotes de producto (nivel Apple Event / Linear / Stripe Sessions), un motion designer y un ingeniero front-end creativo especialista en GSAP y WebGL.

Tu misión es transformar `C:\Users\usuariolocal\Desktop\Presentación Alvaro\Krino - KPMG Access Intelligence.html` en una **presentación de navegación horizontal tipo PowerPoint/Keynote**. Cada diapositiva entra desplazándose hacia la derecha, tiene "clics" internos (builds) como en PowerPoint y cuenta con animaciones de gran calidad. Aun así, debe seguir siendo una pieza corporativa de KPMG sobria, legible y defendible delante de un cliente.

**No hay prisa. Prima la calidad sobre la velocidad.** Trabaja por fases, verifica cada una en el navegador antes de pasar a la siguiente y no des nada por terminado hasta superar la checklist de QA de la sección 9.

Entregable: `Krino - KPMG Access Intelligence v2.html` en la misma carpeta. **No modifiques ni borres el original.**

---

## 1. Fase de preparación (obligatoria antes de escribir código)

1. **Carga estas skills y aplícalas:**
   - `frontend-design`: dirección estética y huida de lo genérico.
   - `artifact-design`: tokens, tema claro/oscuro, contrato de página y restricciones de CDN.
   - `artifact-diagramming`: diagramas SVG reales (cadena de evidencia, arco de servicio, flujo de autorización).
   - `dataviz`: bloque de cifras, contadores y cualquier visualización.
2. **Lee el HTML original completo, incluido el `<script>`.** Haz un inventario de:
   - todos los textos, sin parafrasear;
   - las cifras: 155 riesgos SoD, 81 funciones, 1.434 acciones SAP, 109 reglas de acceso crítico y base de reglas v5;
   - las interacciones: demo Ana/Luis/Marta y simulador de retirada de rol;
   - los disclaimers: caso sintético, "Concepto en diseño", maqueta sin modelo conectado, footer legal y "Document Classification: KPMG Public";
   - el logo KPMG en base64.
3. **Escribe un storyboard breve** (tabla slide → mensaje → layout → animación de entrada → builds → transición de salida) y enséñamelo antes de construir.

### Reglas de contenido (no negociables)
- Todo el texto se conserva **literal**. Puedes repartirlo entre slides y builds, pero no reescribirlo, resumirlo ni inventar frases.
- Prohibido inventar cifras, clientes, logos de terceros, testimonios o capturas "de producto".
- Los disclaimers deben ser visibles y legibles, sin esconderse en letra diminuta ni con bajo contraste.
- La maqueta del asistente sigue presentándose como **concepto**, nunca como función disponible.

---

## 2. Dirección de arte

**Concepto rector: "separar la señal del ruido".** Krino viene del griego κρίνω, "separar". Cada efecto visual tiene que expresar esa idea: cosas que se separan, se filtran, se enfocan o se desenredan. Un efecto que no sirva a esa narrativa se elimina.

- **Motivo gráfico:** la "costura" del original, una línea que se divide en dos (cyan y violeta). Úsala como hilo conductor: separadores, transiciones, subrayados y barra de progreso.
- **Ritmo cromático:** alterna slides oscuras (azul KPMG profundo con luz violeta y cyan) y claras (papel `#F4F7FB`) para dar respiración, como en una keynote.
- **Profundidad:** capas con parallax sutil, luz volumétrica suave, grano fino (noise al 3–4 %) sobre los fondos oscuros y sombras largas y difusas. Nada de sombras duras.
- **Minimalismo:** una idea por slide y como máximo tres elementos protagonistas. El espacio negativo es parte del diseño.

### Tokens (respeta los del original y amplía)
```
--kpmg #00338D   --kpmg-dark #0C233C   --cobalt #005EB8   --cyan #00B8F5
--violet #7213EA --pink #FD349C        --mtd-blue #1E49E2
--ink #0A1633    --muted #65718A       --paper #F4F7FB
--red #D9304A    --amber #E0932A       --green #12916B
```
- Tipografía: la pila del original (`KPMG Bold`, `Univers for KPMG`, fallback Arial), `Palatino` para el griego y monoespaciada para el código SAP.
- Escala tipográfica fluida con `clamp()`: display de 72–160 px, H2 de 40–72 px, cuerpo de 18–22 px (se proyecta, así que nunca menos de 16 px), eyebrow de 12 px con tracking .16em.
- Retícula de 12 columnas y márgenes de seguridad del 6 vw. Ningún texto pegado a los bordes.

---

## 3. Arquitectura técnica

- Un único `.html` **autocontenido**: CSS y JS inline y el logo en base64. Solo se permiten librerías externas de `cdn.jsdelivr.net/npm/` o `cdnjs.cloudflare.com`, con versión fijada. **Verifica cada URL antes de usarla** y no inventes rutas.
  - **GSAP 3.x** con `ScrollTrigger`, `SplitText`, `Flip`, `DrawSVGPlugin`, `MorphSVGPlugin`, `MotionPathPlugin` y `CustomEase`. Todos son gratuitos desde GSAP 3.13.
  - **Lenis** para la inercia suave del rueda/trackpad.
  - **Three.js** (u OGL si pesa menos) solo para los fondos WebGL de portada y cierre.
- **Motor de deck propio**, no scroll nativo:
  - Un `track` con las slides en fila (`display:flex`, cada una `100vw × 100dvh`) desplazado con `transform: translate3d`.
  - Máquina de estados `{slide, build}`. `→`, espacio, `PageDown`, rueda y swipe primero avanzan el siguiente *build* de la slide actual y, cuando no quedan, pasan a la siguiente slide. Es exactamente como los clics de PowerPoint.
  - Cada slide expone `enter(dir)`, `exit(dir)`, `build(n)` y `reset()` como timelines de GSAP. Al volver atrás, la slide aparece ya "construida".
  - La rueda se agrupa con un umbral y un cooldown para que un gesto de trackpad no salte cinco slides.
- **Controles:**
  - `← →`, `↑ ↓`, espacio, `PageUp`/`PageDown`, `Home`/`End`.
  - `G` abre la vista de miniaturas en rejilla (grid overview con Flip) para saltar a cualquier slide.
  - `F` activa la pantalla completa.
  - `P` abre el **modo presentador** en otra ventana, sincronizado con `BroadcastChannel`: slide actual, siguiente, notas del orador y cronómetro.
  - `B` pone la pantalla en negro.
  - `?` muestra la ayuda de atajos.
  - Deep link con `#/5/2` (slide/build) y el historial del navegador funcionando.
- **Interfaz de deck:** barra de progreso superior con la "costura" cyan/violeta, contador `05 / 16`, puntos de navegación inferiores y flechas discretas que se ocultan tras 2 s de inactividad del ratón.
- **Notas del orador:** una `<aside class="notes">` por slide, escritas a partir del texto original (qué decir y qué pregunta anticipar). Solo se ven en modo presentador.

---

## 4. Principios de motion

- **Curvas:**
  - entradas `expo.out`;
  - salidas `power3.in`;
  - elementos que "encajan" `back.out(1.4)` con moderación;
  - una `CustomEase` propia llamada "krino" para las transiciones de slide.
- **Duraciones:**
  - micro 150–250 ms;
  - elementos 500–800 ms;
  - transición de slide 900–1200 ms;
  - stagger de 40–80 ms.
- **Coreografía:** fondo, después titular y después detalle. Nunca animes todo a la vez.
- **Rendimiento:** anima solo `transform`, `opacity`, `clip-path` y `filter` ligero. Nada de `top`, `left`, `width` ni `box-shadow` animados. El WebGL se pausa (`renderer.setAnimationLoop(null)`) cuando su slide no está activa. Asigna `will-change` solo durante la animación.
- **`prefers-reduced-motion`:** sin WebGL, sin parallax ni SplitText. Las transiciones pasan a fundidos de 200 ms y el contenido queda completo y usable.

### Transiciones entre slides (variadas y con intención)
| Tipo | Uso |
|---|---|
| Push horizontal con parallax por capas | Transición por defecto, la del "desplazar hacia la derecha" |
| Reveal con `clip-path` circular desde el cursor o el botón pulsado | Paso a la demo |
| Cortina que se divide en dos por la costura cyan/violeta | Cambio de claro a oscuro |
| Zoom-through: un elemento de la slide actual se convierte en el fondo de la siguiente (Flip) | Cifras → punto de partida |

---

## 5. Guion slide a slide

> El texto entre comillas es literal del original. Los *builds* (B1, B2…) son los clics.

**01 · Portada (oscura, WebGL)**
- Fondo: campo de miles de partículas azul/violeta/cyan desordenadas (el ruido) que, al entrar, se ordenan en dos corrientes paralelas separadas por la costura (la señal). Reacciona suavemente al ratón.
- La κ gigante se dibuja con `DrawSVG` y queda al 5 % de opacidad.
- Eyebrow "KPMG · Governance, Risk & Compliance" y logo.
- Titular "Separar lo que importa *de lo que solo hace ruido.*" con `SplitText` por palabras. La segunda mitad, en cyan, se **desliza y se separa** físicamente de la primera.
- B1: lede. B2: las tres "facts" como píldoras con stagger.

**02 · κρίνω (oscura)**
- La palabra griega en Palatino a tamaño display.
- B1: las letras se transforman (`Flip` + `MorphSVG`) en "criterio" y "crítico".
- B2: la frase completa de la etimología, terminando en "…lo que hace una segregación de funciones".

**03 · La base en cifras (oscura)**
- Cuatro tarjetas glass (`backdrop-filter`) con tilt 3D sutil al pasar el ratón.
- Contadores 0 → 155 · 81 · 1.434 · 109 con separador de miles español y `tabular-nums`.
- B1: la nota "Base de reglas global, versión 5…" junto a una huella hexadecimal que se "escribe" con efecto *scramble*. Es decorativa: márcala `aria-hidden` y no la presentes como hash real.

**04 · El punto de partida (clara)**
- H2 "Una matriz dice que hay conflicto. El cliente pregunta qué hace ahora."
- Visual: una rejilla densa de cientos de celdas rojas que representa las alertas teóricas.
- B1: el lede.
- B2: las celdas se desvanecen y solo quedan unas pocas encendidas: ruido frente a señal. Aclara visualmente que es una ilustración, no datos.

**05 · Cadena de evidencia (clara)**
- Los cinco eslabones (01 Extracción → 05 Decisión) en un diagrama SVG horizontal.
- Cada build dibuja el siguiente tramo de línea con `DrawSVG`, enciende el nodo y muestra su texto literal. Cinco builds en total.

**06 · La misma transacción, tres respuestas (oscura) — DEMO**
- Transición con reveal circular.
- Conserva **exactamente** la lógica y los textos de Ana, Luis y Marta (objeto `cases` del script original).
- Panel izquierdo tipo terminal:
  - las líneas aparecen con efecto de tecleo;
  - al cambiar de persona, el valor `KOART` (K / D / "sin valor") hace un *glitch* breve y el resultado se reescribe.
- Panel derecho: el badge cambia con un *morph* de color (rojo "Hallazgo", verde "Descartado", ámbar "Por validar") y el veredicto entra con máscara.
- Los botones de persona funcionan con ratón, teclado (`1`, `2`, `3`) y como builds (B1 Ana, B2 Luis, B3 Marta), de modo que el presentador pueda recorrerlos solo con `→`.
- Pie literal "Tres estados posibles…". `aria-live="polite"` en el panel.

**07 · El cambio se ensaya primero (oscura) — SIMULADOR**
- Los dos roles como tarjetas.
- Al pulsar "Simular retirada del rol" (o con el build B1):
  - el rol `Z_..._PAGO_K` se tacha con una línea que se dibuja;
  - la tarjeta pasa a borde discontinuo y se desplaza;
  - el contador anima `1 → 1` a `1 → 0`;
  - el aviso ámbar entra con un *shake* de 2 px.
- Es reversible ("Restaurar el rol") y mantiene los textos literales.

**08 · Diez capacidades: Diagnóstico (clara)**
**09 · Análisis y priorización (clara)**
**10 · Acción y optimización (clara)**
- Una slide por bloque para no saturar.
- Cabecera con el nombre del bloque, la pregunta ("¿Cómo de grave es la situación hoy?"…) y la etiqueta monoespaciada ("VER Y DIMENSIONAR"…).
- Las capacidades entran como cartas que se voltean en 3D (`rotateY`), una por build.
- Un indicador "4/10, 7/10, 10/10" muestra el avance hacia las diez.

**11 · Dónde está el valor (clara)**
- Los cuatro pasos (Diagnóstico → Validación → Remediación → Gobierno) sobre un camino curvo SVG.
- Un punto luminoso lo recorre con `MotionPath`, un paso por build.
- "Validación" se ilumina más y aparece la anotación "aquí entra el criterio del equipo".

**12 · Lo que Krino no hace (clara)**
- H2 y lede "Con las cartas boca arriba".
- Seis cartas **boca abajo** que se giran una a una, literalmente "boca arriba", cada una con su etiqueta roja "NO".
- Seis builds, o un único build en cascada si el presentador pulsa `Shift+→`.

**13 · Hacia dónde va (oscura)**
- Chat mock: indicador "escribiendo…" (tres puntos) y después la burbuja con los textos literales de las cuatro intervenciones, una por build.
- Ribbon "Concepto en diseño" y el párrafo de disclaimer completo, siempre visibles desde la entrada de la slide.

**14 · La tesis (clara)**
- "Menos alertas. Mejores decisiones." a pantalla completa, con gradiente cyan→violeta animado muy lento en el texto.
- La costura se dibuja debajo.
- B1: el párrafo "El riesgo se detecta con datos…".
- B2: "Make it defensible." en negrita.

**15 · Siguiente paso (clara)**
- El párrafo literal del "Siguiente paso", con las tres salidas del piloto destacadas como tres opciones visuales: seguir, ajustar o parar.

**16 · Endframe "Make the difference" (`--mtd-blue`)**
- Logo KPMG. "MAKE / THE DIFFERENCE" en tipografía gigante con reveal por máscara de abajo arriba.
- Firma "KPMG. Make the Difference." y "Krino · Access Intelligence".
- Fondo WebGL: las dos corrientes de la portada que convergen en una sola línea.
- Al pulsar `→` en la última slide se abre un panel inferior con el footer legal completo, el disclaimer del caso sintético y "Document Classification: KPMG Public".

---

## 6. Responsive y otros modos

- **Proyector 16:9 (1920×1080 y 1280×720):** es el caso principal y debe verse perfecto.
- **Portátil 1440×900 y 1366×768:** sin cortes ni solapamientos.
- **Móvil (360–430 px):** las slides se navegan con swipe horizontal (umbral de 50 px más velocidad), los layouts pasan a una columna, sin WebGL pesado (fallback con gradiente), sin scroll horizontal accidental y con gutter de 16 px.
- **Impresión/PDF (`@media print`):** una slide por página A4 apaisado, todos los builds visibles, sin animaciones, con los fondos oscuros impresos (`print-color-adjust: exact`) y las notas del orador ocultas.

---

## 7. Accesibilidad

- Cada slide es un `<section role="group" aria-roledescription="diapositiva" aria-label="5 de 16: …">` y las slides inactivas llevan `inert`.
- Foco visible con el cyan del original, orden de tabulación lógico y botones reales (`<button>`).
- Contraste AA en todos los textos. Revísalo especialmente en los disclaimers sobre fondo oscuro.
- Una región `aria-live` anuncia el cambio de slide.

---

## 8. Proceso de trabajo

1. Storyboard (sección 1.3) y espera mi visto bueno.
2. Sistema base: tokens, tipografía, motor de deck, navegación y modo presentador. Verifícalo con slides vacías.
3. Construye las slides en orden y **verifica cada una en el navegador** (entrada, builds, salida hacia delante y hacia atrás) antes de pasar a la siguiente.
4. Transiciones y WebGL al final, cuando la estructura sea sólida.
5. QA completo (sección 9), corrige y repite hasta que todo pase.
6. Dame un resumen breve de lo hecho, cualquier limitación y cómo presentar (atajos).

---

## 9. Checklist de QA (todo debe cumplirse)

- [ ] Ningún error ni aviso en la consola.
- [ ] Todo el texto original está presente y literal. Compáralo con el HTML original, cadena por cadena.
- [ ] Demo Ana/Luis/Marta y simulador idénticos en lógica y textos al original.
- [ ] Navegación hacia delante y hacia atrás por todas las slides y builds sin estados rotos, incluso pulsando `→` muy rápido.
- [ ] Deep link `#/n/b`, recarga y botón "atrás" del navegador funcionan.
- [ ] Vista de miniaturas, pantalla completa, modo presentador sincronizado y pantalla en negro funcionan.
- [ ] 60 fps estables en transiciones (Performance panel) y WebGL pausado fuera de su slide.
- [ ] `prefers-reduced-motion` probado: todo es legible y navegable.
- [ ] Probado a 1920×1080, 1366×768 y 390×844, y en impresión a PDF.
- [ ] Contraste AA y navegación completa solo con teclado.
- [ ] Peso total del HTML razonable (< 1,5 MB sin contar CDNs).
- [ ] Ningún dato inventado. Disclaimers visibles. "Concepto en diseño" intacto.
