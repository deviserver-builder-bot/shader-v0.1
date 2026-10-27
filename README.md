# Umami Visuals

Shader para **Minecraft Bedrock en Android** (motor RenderDragon) pensado para
celulares de gama baja como el **Infinix Note 30** (Helio G99, GPU Mali-G57 MC2).
Mantiene las texturas originales del juego: solo cambia la luz, las sombras, el cielo y el ambiente.

### Cielo
- Día degradado, **atardeceres y amaneceres** naranjas con resplandor y tono rosado del lado opuesto.
- Noche azul profunda con **estrellas que titilan** (en tiempo real) y **fases de la luna**:
  en luna nueva la noche es más oscura.
- Cada **bioma tiñe su horizonte** (desierto más cálido, pantano más verdoso…).
- Nubes teñidas por el atardecer, con base translúcida.
- **End** con cielo morado degradado que gira lentamente.

### Luz y sombras
- Colores fieles al original a pleno sol; **sombras suaves** bajo árboles y techos.
- **Sol de lado**: con el sol alto se ilumina el suelo; al amanecer y atardecer se encienden las paredes.
- **Mobs, jugadores y el objeto en la mano** con la misma luz del mundo: la cara que mira
  al sol se ilumina y la de atrás queda en sombra. El destello rojo de daño, los ojos que
  brillan y los encantamientos siguen como en el original.
- Antorchas cálidas con un **titileo** leve; de noche la luz de antorchas atraviesa la niebla.
- **Nether** iluminado con el color de cada bioma (carmesí, distorsionado, valle de almas…).
- Bajo el agua: tinte azul, reflejos de luz en el fondo (también sobre los mobs) y más oscuro al mirar abajo.

### Ambiente
- **Hojas y pasto que se mueven** con el viento (más con lluvia).
- Agua con reflejo del cielo, destello del sol y olas suaves.
- **Lluvia y nieve** teñidas por la luz del momento (naranja al atardecer, azul de noche).
- Con lluvia, el suelo al aire libre **se ve mojado** (más oscuro y con reflejo del cielo).
- Niebla lejana con el color real del cielo (bruma dorada al atardecer, gris con lluvia).

## Texturas fieles al original (v0.2)

El shader **no reemplaza texturas**: usa las de Minecraft (o las del paquete de
texturas que tengas activo). Lo que hace es respetar sus colores:

- A pleno sol la textura se ve con su **color real** (x1.0, apenas cálido).
  En v0.1 se multiplicaba por x1.6 y se veía lavada y azulada.
- Las sombras bajo árboles y techos oscurecen sin volver gris la textura.
- **Definición**: los píxeles oscuros de cada textura se oscurecen un poco y los
  claros se aclaran (tonos medios iguales), así se notan más los detalles.
  Se puede ajustar o apagar con `UM_TEX_CONTRAST` en `src/umami/config.h`.

Diferencia media contra Minecraft sin shader (terreno, 0–255): mediodía 26 → **10**,
sombra de árbol 35 → **7**.

**¿Quieres texturas distintas?** Pon cualquier paquete de texturas 16x16 debajo de
Umami Visuals en *Recursos globales*: el shader funciona con todos. Los de 16x16
no cuestan FPS; los de 32x32 o más gastan más memoria y pueden dar tirones.

## Por qué es tan ligero

| | Terreno vanilla | Umami Visuals |
|---|---|---|
| Lecturas de textura por píxel | 2 | **1** |
| Instrucciones por píxel, terreno (medidas) | 111 | **76** |
| Instrucciones por píxel, agua | — | **82** |

En v0.3 todos los efectos nuevos se calculan por vértice, así que el costo por píxel
no cambió respecto a v0.2. Los bloques que solo aplican a veces (lluvia, atardecer,
Nether, bajo el agua, luz de antorcha) usan condiciones que la GPU se salta completas
cuando no aplican. En tiempo medido, el cuadro completo quedó igual que v0.2
(Equilibrado −3 %, Ultra +2 %, dentro del ruido de la medición; Bonito +7 % por el
destello por píxel).

Todo lo caro (luz, sombras, niebla, agua) se calcula **por vértice**, nunca por
píxel. En Mali solo se ejecuta para vértices visibles. El píxel del terreno hace
lo mínimo posible: 1 textura y 2 operaciones (la niebla ya viene mezclada desde el vértice). Los cálculos
de color del cielo usan media precisión (FP16), que en Mali corre al doble de velocidad.

## Subpacks (opciones)

En Minecraft: *Recursos globales → Umami Visuals → engrane ⚙*

| Subpack | Qué cambia | FPS |
|---|---|---|
| **Equilibrado** (por defecto) | Todo activado. Recomendado para el Note 30. | = |
| **Ultra ligero** | Sin olas, plantas quietas, sin titileo, sin definición extra de texturas, cielo simplificado. | más |
| **Bonito** | + brillo extra al sol y destello del sol en el agua calculado por píxel. | menos |
| **Noche clara** | Igual que Equilibrado, con noches ~55 % más visibles. | = |
| **Sin niebla** | Sin bruma lejana (solo el desvanecido del borde del mapa). | = |
| **Sin movimiento** | Sin olas, plantas ni titileo (para quien se maree). | un poco más |
| **Diagnóstico** | Pinta el mundo según lo que detecta el shader. Úsalo solo para reportar fallas. | = |

**Colores del modo Diagnóstico:** gris = normal · rojo = Nether · morado = End ·
azul = bajo el agua · amarillo = lluvia. Si ves un color que no corresponde
(por ejemplo, amarillo sin que llueva), esa detección está fallando en tu dispositivo.

## Instalar en Android

Bedrock no soporta shaders oficialmente; se cargan con **MB Loader**:

1. Instala [MB Loader](https://play.google.com/store/apps/details?id=io.github.bambosan.mbloader).
2. Abre Minecraft desde MB Loader.
3. Importa `umami-visuals-...-android.mcpack` y actívalo en **Recursos globales**.

Consejos para más FPS en el Note 30: distancia de renderizado 8–10 chunks,
"Nubes bonitas" desactivado si notas tirones, y el subpack *Ultra ligero*.

## Compilar el `.mcpack`

### Opción A: automática en GitHub (sin instalar nada)
1. Sube esta carpeta a un repositorio de GitHub.
2. Pestaña **Actions → Compilar Umami Visuals → Run workflow**.
3. Al terminar, descarga el artefacto `umami-visuals` (trae el `.mcpack`).
4. Para publicar una versión descargable para todos, crea una etiqueta (por ejemplo
   `v0.3`): GitHub compila y la publica sola en **Releases** con el `.mcpack` adjunto.

### Opción B: en tu PC (Windows)
```
pip install -r requirements.txt
build.bat setup
build.bat pack -p android
```
El pack queda en `build\`.

### Opción C: en el mismo celular (Termux)
```
pkg install python git
pip install -r requirements.txt
./build.sh setup
./build.sh pack -p android
```

## Límites conocidos

- Bedrock no deja a los shaders calcular **sombras reales del sol** en celular (no hay
  mapa de sombras). Las sombras salen de la luz del cielo que da el juego: se ven bajo
  árboles, techos y cuevas, pero un poste no proyecta sombra en el suelo.
- Los bloques que brillan (lava, piedra luminosa) no reciben un brillo extra propio: para eso
  hacen falta texturas modificadas y este pack mantiene las originales. Sí iluminan a su
  alrededor, y esa luz atraviesa la niebla de noche.
- La lluvia, el Nether y el agua se detectan con el color y la distancia de la niebla. Si
  algo se ve raro, el subpack **Diagnóstico** muestra qué está detectando.
- Todo está probado compilando las 609 combinaciones en GLSL ES 3.0 y renderizando
  escenas de prueba, pero falta probarlo en un celular real.

## Ajustar colores y efectos

Todo está en `src/umami/config.h`: colores del cielo de día, atardecer y noche,
intensidad del sol (`UM_SUN_STRENGTH`) y de las antorchas, movimiento de hojas y pasto
(`UM_LEAF_WAVE`, `UM_GRASS_WAVE`), tinte por bioma (`UM_BIOME_TINT`), brillo extra de
la noche clara, olas del agua, etc.

## Herramientas de desarrollo (`devtools/`)

Permiten probar sin el juego (con texturas de prueba propias estilo vanilla), usando Mesa (GLSL ES 3.00, el mismo lenguaje del celular):

```
python3 devtools/glcheck.py                 # compila las 609 combinaciones (87 variantes x 7 subpacks)
python3 devtools/preview.py render --q DEFAULT   # imágenes de prueba (día, atardecer, noche...)
python3 devtools/preview.py compare --vanilla    # mismas tomas con y sin shader
python3 devtools/cost.py                    # instrucciones por shader (métrica de optimización)
python3 devtools/preview.py bench           # tiempos por material
```
Requiere `python3`, `numpy`, `pillow` y Mesa (`libEGL`, `libGLESv2`).

## Créditos

- Herramienta de compilación y lógica de detección de entorno basadas en
  [Newb X Legacy](https://github.com/devendrn/newb-x-mcbe) de devendrn (MIT,
  ver `LICENSE-newb-tool.txt`).
- Compilación de materiales con [Lazurite](https://github.com/veka0/lazurite).
