# Portada SGS · estilo y movimiento

Cómo está hecha la portada de `tableros.pages.dev` y cómo llevarla a otro tablero.
Es un complemento del README de **SGS UI v1**: aquel describe el sistema completo,
éste documenta la portada tal como quedó en producción y —sobre todo— las trampas
que costaron varias vueltas encontrar. Si vas a copiar el movimiento a otro
proyecto, la sección [Trampas](#trampas) es la que importa.

HTML estático, un solo archivo, sin build, sin dependencias, sin red externa.

---

## Llevarlo a otro proyecto

Hay dos caminos y conviene elegir a conciencia.

**Autocontenido (el que usa este hub).** Todo el CSS, el JS y los logos viven
embebidos en el `index.html`: 124 KB en una sola request, cero dependencias.
Copiás el archivo, cambiás el array `PERFILES` y listo. Es lo correcto cuando el
tablero tiene que funcionar sí o sí, sin importar qué más esté caído.

**Por CDN.** El kit está pensado para servirse desde `siegfried-ui.pages.dev`:

```html
<link rel="stylesheet" href="https://siegfried-ui.pages.dev/ui/v1/sgs.css">
<link rel="stylesheet" href="https://siegfried-ui.pages.dev/ui/v1/sgs-portada.css">
<link rel="stylesheet" href="https://siegfried-ui.pages.dev/ui/v1/sgs-entrada.css">
```

Es mejor a largo plazo —un arreglo se propaga a todos los tableros— pero
**a la fecha de este documento ese dominio no está publicado**. Verificalo antes
de apoyarte en él: si no existe, el tablero se ve sin estilos.

```bash
curl -o /dev/null -w "%{http_code}\n" https://siegfried-ui.pages.dev/ui/v1/sgs.css
```

Cuando uses el CDN, el `<body>` necesita `class="sgs"`. La versión autocontenida
**no** la lleva, y eso es deliberado: sin esa clase, la regla global del kit que
achica todas las animaciones a 1 ms bajo `prefers-reduced-motion` no aplica.

---

## Tokens

Ningún color se escribe a mano. Si falta uno, se agrega al kit.

| Token | Valor | Uso |
|---|---|---|
| `--sgs-bordo-700` | `#95232C` | El de la marca. Aguja indicadora, nunca relleno de áreas grandes |
| `--sgs-bordo-800` / `900` | `#711920` / `#4C1015` | Profundidad del telón |
| `--sgs-bordo-400` | `#D26069` | Franjas diagonales, destello de la chapa |
| `--sgs-p-crema` | `#FBF8F3` | La hoja y la chapa del logo. **No** blanco |
| `--sgs-p-angulo` | `-78deg` | Inclinación de las franjas |
| `--sgs-ease-std` | `cubic-bezier(.2,0,0,1)` | El easing por defecto |
| `--sgs-dur-1..4` | 120 / 180 / 280 / 420 ms | Escala de duraciones de interfaz |

Tipografía: serif (`Iowan Old Style`, `Palatino`, Georgia) para títulos y numerales
en itálica; `Inter` para etiquetas, versalitas y datos.

---

## La regla que ordena todo

> **El fondo actúa, el contenido no espera.**

El panel de contenido **no tiene animación de entrada**: está desde el frame 0,
legible y clickeable. Lo único que se mueve al cargar es el fondo y la cabecera.

La consecuencia práctica es la que importa: si la animación falla, si el navegador
no la soporta o si el usuario pidió menos movimiento, la página sigue siendo
perfectamente usable. **La decoración es prescindible por construcción, no por
configuración.**

De ahí se derivan las demás:

1. Se anima solo `transform` y `opacity`. Nunca `width`, `height`, `top`, `left`.
2. Todo lo que anima entra con `fill: backwards` o equivalente, para que el
   contenido exista aunque la animación no corra.
3. Nada loopea en el contenido. Un elemento que se mueve para siempre se lee como
   estado de carga.

---

## La coreografía

Dos regímenes de tiempo, y mezclarlos es el error fácil:

| | Portada | Tablero |
|---|---|---|
| Entrada | 1,1 – 1,4 s | 280 – 420 ms |
| Ambiente | campo de partículas a la deriva | ninguno |
| Cuándo | no hay nada que leer todavía | hay dato que leer |

`sgs-portada.css` va **solo** en el hub o un login. Nunca en un tablero: quien
abre un tablero para decidir algo en 30 segundos no puede esperar 1,4 s de
coreografía.

### Orden de entrada

| ms | Qué |
|---|---|
| 0 | La hoja de contenido ya está, completa y clickeable |
| 0 – 1100 | El telón entra desde arriba a la izquierda (`sgs-p-masa`) |
| 150 / 250 / 350 | Las tres franjas barren en secuencia (`sgs-p-franja`, 1400 ms) |
| 70 – 690 | La chapa del logo se acomoda (`sgs-p-placa`, 620 ms) |
| 420 | El título se desliza dentro de su marco recortado |
| 700 / 820 | La bajada y el año asoman |
| 140 + 24·n | Las tarjetas de acceso, escalonadas |

El orden lo pone `--t` inline, así que **la coreografía se lee mirando el HTML**:

```html
<h1 data-sgs-corre style="--t:420ms"><span>Tableros de gestión</span></h1>
<span data-sgs-asoma style="--t:700ms">Argentina · Inteligencia de Mercados</span>
```

| Atributo | Gesto |
|---|---|
| `data-sgs-corre` | Deslizado dentro de un marco recortado. Necesita `<span>` interno |
| `data-sgs-asoma` | Solo opacidad. Para lo que ya está en su lugar |
| `data-sgs-sube` | Fade + elevación |
| `data-sgs-abre` | `scaleX` desde la izquierda. Filetes y divisores |

### El campo de partículas

Veinticuatro curvas blancas a la deriva. Es **la S del isotipo como geometría
propia**, no el arte oficial repetido: la serpiente es el elemento central de la
marca, no un motivo de trama, y a 14 px una serpiente reconocible se lee como un
pelo en la pantalla. Una curva se lee como curva a cualquier tamaño.

```css
.sgs-sierpe {
  position: absolute;
  width: var(--p-tam, 24px); height: var(--p-tam, 24px);
  animation: sgs-p-deriva calc(var(--p-dur, 18s) * .38) linear infinite;
  animation-delay: var(--p-delay, 0s);   /* negativo: desfasa el arranque */
  transform: translateZ(0);
}
.sgs-sierpe::before {
  content: ""; position: absolute; inset: 0;
  background-image: url("data:image/svg+xml,…");  /* curva blanca */
  background-size: contain;
  transform: rotate(var(--p-rot, 0deg));
  opacity: 1;                            /* ← la opacidad la maneja el keyframe */
}

@keyframes sgs-p-deriva {
  0%   { transform: translateZ(0) scale(.92); opacity: 0; }
  14%  { opacity: var(--p-op, .72); }
  78%  { opacity: var(--p-op, .72); }
  100% { transform: translate3d(calc(var(--p-x,20px) * 2.2),
                                calc(var(--p-y,-200px) * 1.5), 0)
                    scale(1.07); opacity: 0; }
}
```

Cada partícula lleva su tamaño, duración, rotación, recorrido y opacidad en
custom properties, y un **delay negativo** para que cada una arranque en un punto
distinto de su ciclo: así el campo nunca late sincronizado.

Valores que funcionan: tamaños 24–66 px, opacidad 0,58–0,88, ciclos de 4,6 a 9,5 s,
recorridos de 215–290 px. Eso da **~42 px/s**, que es el número que importa.

### El logo

El isotipo es monocromo `#95232C` con transparencia y mezcla tinta con huecos:
**necesita fondo claro**, sobre el telón se fundiría. Por eso va sobre una chapa
crema —el mismo papel que la hoja, no blanco— que la primera lámina del telón
empuja a su lugar. El logo no tiene animación propia: viaja quieto dentro de la
chapa, como corresponde a un logo.

Al hover la chapa se levanta 2 px y un destello la cruza. **El destello va en la
chapa, nunca en el arte**: la chapa es material y puede atrapar luz, la tinta no
brilla. Sobre claro no se puede aclarar, así que es bordó al 14 % con un hueco al
medio — el rebote real que recoge una chapa clara al lado de una pared bordó.

### El escalonado de tarjetas

```css
.acceso {
  animation: acceso-entra 380ms cubic-bezier(.2,0,0,1) backwards;
  animation-delay: calc(var(--i,0) * 24ms + 140ms);
}
@keyframes acceso-entra { from { opacity: 0; transform: translateY(6px); } }
```

El índice lo emite el JS al construir cada tarjeta (`style="--i:3"`). `backwards`
en vez de `forwards` es lo que hace que la tarjeta exista y sea clickeable aunque
la animación no llegue a correr.

---

## Trampas

Todo lo de abajo se descubrió en producción, no en la teoría.

### 1. Las opacidades se multiplican

Un `::before` con `opacity: .34` dentro de un elemento cuya animación oscila entre
`.35` y `.62` da un **14 % efectivo**. El campo de partículas era literalmente
invisible por esto.

> **Regla:** la opacidad la maneja **una sola capa**. Si el keyframe la anima, el
> `::before` va en `opacity: 1`.

### 2. `prefers-reduced-motion` apaga la página entera en equipos corporativos

Windows 11 trae *Configuración → Accesibilidad → Efectos visuales → Efectos de
animación* **desactivado** en muchos equipos de empresa. Cuando lo está, el
navegador declara `prefers-reduced-motion: reduce` sin que el usuario lo haya
pedido nunca. Si el CSS responde con `animation: none !important`, la portada
queda completamente muerta —logo incluido— y el síntoma es desconcertante:
*"no se ve ninguna animación, en ningún lado"*.

> **Regla:** distinguir **movimiento perpetuo** de **entrada de una sola vez**.
> Las entradas duran menos de 1,5 s y terminan: se dejan. Lo que loopea, se
> modera. El apagado total se ofrece aparte, por clase manual
> (`html.sgs-no-motion`), para quien de verdad lo necesite — que existe: hay
> gente con sensibilidad vestibular para quien esto no es cosmético.

### 3. Hay una velocidad mínima para que algo se lea como movimiento

Los ciclos originales del kit (12–25 s) daban **~10 px/s**. Técnicamente se
movían; a la vista estaban congeladas. Por debajo de unos 15 px/s el ojo no
registra deriva sin fijar la vista a propósito.

> **Regla:** medir **píxeles por segundo**, no "¿la animación está corriendo?".
> Que `getAnimations()` devuelva 24 no dice nada sobre si se ve algo. Para deriva
> ambiente, apuntar a 30–50 px/s.

### 4. Ralentizar no es atenuar

El intento de "respetar" `prefers-reduced-motion` multiplicando la duración por
2,5 produjo el peor resultado posible: recorridos de 30–60 s, o sea ~4 px/s. Una
partícula tan lenta no molesta menos — obliga a fijar la vista para saber si se
mueve, que es exactamente lo contrario de lo buscado.

> **Regla:** para atenuar, bajar **opacidad** o **cantidad**. La velocidad se deja.

### 5. `ease-in-out` arruina los loops de deriva

Frena la partícula justo en las dos puntas del ciclo, que es cuando el ojo la está
seguiendo. El campo se lee como si titilara en vez de derivar.

> **Regla:** deriva continua → `linear`. `ease-in-out` es para gestos con
> principio y fin.

### 6. Un loop de posición tiene que fundir a cero en las dos puntas

Si el keyframe termina en `opacity: .42` y vuelve a empezar en otra posición, el
salto se ve. Con partículas tenues pasa desapercibido; apenas se las hace
visibles, aparece.

> **Regla:** `0%` y `100%` en `opacity: 0`, con mesetas al 14 % y 78 %.

### 7. Un contenedor opaco ancho se come el fondo

Al ensanchar la hoja de 1120 a 1500 px para compactar, pasó a cubrir del 12 % al
88 % del ancho — y las partículas estaban repartidas entre el 6 % y el 92 %. De 18,
quedaban visibles unas 5.

> **Regla:** ubicar el campo en las bandas que el contenido deja libres (cabecera,
> márgenes, pie) y verificarlo por geometría, comparando el rectángulo de cada
> partícula contra el del contenedor.

### 8. Los glifos Unicode geométricos no son confiables

La plantilla original usaba `▤ ◫ ⌗ ◹` como iconos. Dependen de las fuentes del
sistema y pueden salir como cuadrito vacío en otra máquina.

> **Regla:** SVG inline. Se ve igual en todos lados y pesa lo mismo.

### 9. Verificar lo que la plantilla no trae

La plantilla del kit no incluía `target="_blank"` ni `rel="noopener"` en los
accesos. Si el hub anterior los tenía, migrar sin revisarlo es una regresión
silenciosa: los tableros pasan a abrirse encima del hub.

---

## Agregar un tablero

Todo el contenido sale de un array. Los badges y las cuentas se calculan solos.

```js
var PERFILES = [
  { n: "01", titulo: "Promoción", rol: "Gerente regional y jefes de equipo",
    abierto: true,
    accesos: [
      // [ nombre, origen, SVG del icono, URL ]
      ["ABC Dashboard", "Siegfried · Interno", "<svg …></svg>",
       "https://abc-dashboard-6p1.pages.dev/"]
    ] }
];
```

El origen es sólo texto: se muestra en versalitas debajo del nombre y no cambia
ningún estilo — la chapa del icono es siempre la misma (26 px, fondo `#F2E7E9`,
trazo en `--sgs-bordo-700`). Agregar un acceso es **una línea**.

Los campos se inyectan como HTML, no como texto plano — así entra el SVG del
icono. Eso permite acentuar parte de un nombre, como el `+` de ABC:

```js
["ABC <span class=\"acceso__mas\">+</span>", "Siegfried · Interno", "<svg …>", "https://…"]
```

Con la contrapartida de siempre: los nombres vienen de este archivo y de nadie
más. Si algún día el array se llenara desde una fuente externa, hay que escapar.

---

## Verificar antes de publicar

Lo que conviene mirar, en orden de cuánto duele que falle:

1. **Los datos.** Comparar el DOM renderizado contra la versión anterior —nombre,
   origen y URL de cada acceso— en vez de leerlo a ojo.
2. **Los links.** Que ninguno quede en `#` y que todos conserven `target` y `rel`.
3. **El movimiento, en px/s.** Adelantando `animation.currentTime` a mano y
   midiendo el `transform` en varios puntos del ciclo. Ojo: algunos entornos de
   preview congelan el reloj de animación (`currentTime` no avanza) y ahí es
   imposible observar deriva real — hay que forzarla.
4. **La geometría del fondo.** Cuántas partículas caen efectivamente fuera del
   contenedor opaco.
5. **El alto total** contra el viewport, si la meta es que entre sin scroll.
