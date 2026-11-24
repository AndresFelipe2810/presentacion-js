# Guion de exposición · Pokémon Card Battle Arena
**Campuslands · Proyecto final de JavaScript Vanilla**
Andrés Felipe · Nicolas Archila

> **Duración estimada:** 22–26 min + preguntas. La presentación tiene **19 slides** (el HUD de abajo muestra `n / 19`).
> **Navegación:** `→` / `Espacio` avanzan, `←` retroceden. Después de pulsar un botón de una demo, las flechas siguen funcionando (el botón pierde el foco solo). **No pulses `E`** durante la exposición: activa el modo edición de texto.
> **Antes de empezar:** abre `index.html` en pantalla completa (F11), con internet (las cartas de las demos cargan desde GitHub/PokéAPI), y haz una pasada rápida por las demos para que las imágenes queden en caché.
> **Convención:** 🖥️ = lo que se ve · 👆 = qué hacer en vivo · 🎤 = qué decir. Si son dos personas, la columna "quién" es solo una sugerencia: **A** = Andrés, **N** = Nicolas.

---

## Mapa de tiempos (guía)

| # | Slide | Tiempo | Quién (sugerido) |
|---|-------|--------|------------------|
| 1 | Portada | 0:30 | A |
| 2 | El juego | 1:30 | A |
| 3 | Sobres y mazo (demo) | 1:30 | A |
| 4 | Mini-arena (demo) | 2:00 | A |
| 5 | Código · daño | 2:00 | A |
| 6 | Código · especial | 2:00 | A |
| 7 | Arquitectura | 2:00 | N |
| 8 | CustomEvent | 2:00 | N |
| 9 | Hash + Salt (demo) | 2:30 | N |
| 10 | Multijugador · secuencia | 2:30 | N |
| 11 | Concurrencia · `seq` (demo) | 2:00 | N |
| 12 | Código · escritura optimista | 2:00 | N |
| 13 | Código · pasar turno | 1:30 | A |
| 14 | Código · aplicar acción | 1:30 | A |
| 15 | Ranking y admin | 1:00 | A |
| 16 | Cold start (demo) | 2:00 | A |
| 17 | Testing y CI/CD (demo) | 1:30 | N |
| 18 | Decisiones y límites | 1:30 | N |
| 19 | Cierre + preguntas | 0:30 | A y N |

---

## Slide 1 · Portada  (⏱ 0:30)

🖥️ **En pantalla:** "Pokémon Card Battle Arena — Sobres, mazos y combates en línea". Tres badges: *Web Components*, *ES Modules*, *48 pruebas*. Ash y Pikachu de fondo.

🎤 **Qué decir:**
> "Buenas, somos Andrés y Nicolas y les presentamos **Pokémon Card Battle Arena**: un juego de cartas hecho en JavaScript puro, sin frameworks, donde abres sobres, armas tu mazo y peleas contra la máquina o contra otro jugador en línea.
>
> Antes de enseñarles código, un adelanto de lo que van a ver: un motor de combate que se prueba solo, contraseñas con hash y salt, un multijugador que funciona **sin WebSockets**, y 48 pruebas automáticas que bloquean el deploy si algo falla. Vamos."

👆 **Acción:** ninguna. Señala con la mano el badge de **48 pruebas** (lo retomamos en el slide 17).

---

## Slide 2 · El juego  (⏱ 1:30)

🖥️ **En pantalla:** título "Un juego de cartas que vive en el navegador". Siete chips de stack: JavaScript Vanilla, Web Components, **Vite 5**, **JSON Server**, PokéAPI, **Web Crypto · SHA-256**, **node:test**. Debajo, tres paneles: 🎴 Colecciona · ⚔️ Combate · 🏆 Compite.

🎤 **Qué decir:**
> "El juego tiene tres ideas. **Colecciona**: abres sobres y desbloqueas cartas de Pokémon reales. **Combate**: por turnos, contra la máquina o contra otra persona en línea. **Compite**: hay un ranking global por puntos y un panel de administrador para gestionar las cartas.
>
> Del stack, JavaScript, Web Components y la PokéAPI ya los conocemos todos, así que me quedo con las dos herramientas que no son tan obvias en un juego:"

👆 **Acción:** señala el chip **Vite 5**, luego **JSON Server**.

> "**Vite** es nuestro servidor de desarrollo y empaquetador. Nos da recarga instantánea mientras programamos y, con `npm run build`, genera la carpeta `dist` lista para publicar. Usamos ES Modules nativos, así que Vite casi no 'transforma' nada: solo sirve y empaqueta. Además, en `vite.config.js` configuramos `base: "/proyecto_javascript/"` para que las rutas funcionen cuando se publica en GitHub Pages, que sirve la app desde una subcarpeta.
>
> **JSON Server** es nuestro backend. Con un archivo, `db.json`, levanta una **API REST completa**: `/cards`, `/players`, `/matches`, `/battles` y `/admins`, con GET, POST, PUT, PATCH y DELETE. No tuvimos que escribir un servidor, y eso nos dejó el tiempo para meternos en la parte difícil: el multijugador. Pero eso tiene consecuencias de diseño que vamos a ver más adelante, porque JSON Server no tiene ni WebSockets ni transacciones.
>
> Lo último: el contexto. Es un proyecto en pareja del curso, para demostrar dominio de JavaScript Vanilla sin librerías de interfaz."

---

## Slide 3 · Sobres y mazo — demo  (⏱ 1:30)

🖥️ **En pantalla:** "Abre un sobre: 5 cartas al mazo". Tres sobres: 🔥 Fuego, 💧 Agua, 🌿 Planta. Debajo un mini flujo: Registro ➜ **Sobre inicial** ➜ Ruleta · Memoria ➜ Mazo de 5. Badges: `generarSobre()`, `TAMANO_SOBRE = 5`, "prioriza cartas nuevas".

🎤 + 👆 **Qué decir y hacer:**
> "Cuando un jugador se registra, lo primero que recibe es un **sobre inicial**. Miren."

👆 **Haz clic en el sobre de Fuego.** El sobre se sacude (~0,65 s) y salen 5 cartas con giro 3D, una cada 0,3 s.

> "Cinco cartas, todas con 250 de vida. Si ahora hago clic en el de Agua, repite con otras cinco."

👆 **Haz clic en el de Agua** (opcional, solo si vas bien de tiempo).

> "Esta demo es una simulación visual, pero la lógica real vive en `generarSobre()`, en `progression.js`. Lo que hace es: primero **descarta las cartas que el jugador ya tiene**, para que un sobre siempre dé cartas nuevas mientras queden. Después ordena el pool por vida y lo parte en **tres tercios** (débiles, medias y fuertes) y va sacando una de cada banda, para que ningún sobre sea todo lo mismo. `TAMANO_SOBRE` vale 5 porque el mazo de combate es de 5.
>
> Y para seguir ganando cartas hay dos minijuegos en el flujo: una **ruleta** y un **juego de memoria**. En la ruleta, `resolverSorteo` da un 20 % de carta nueva, un 40 % de un tiro extra y un 40 % de nada."

*(Esos porcentajes son `PROBABILIDAD_CARTA = 0.2` y `PROBABILIDAD_TIRO = 0.4`.)*

---

## Slide 4 · Mini-arena jugable — demo  (⏱ 2:00)

🖥️ **En pantalla:** "Juega el motor de combate en vivo". A la izquierda Pikachu, a la derecha Charizard, cada uno con barra de vida **250/250**, etiqueta "Especial: Turno 2" y escudo. Abajo cuatro botones: ⚔️ Atacar, 🛡️ Defender, ✨ Especial, ↺. Badges: 250 HP, Defensa −50 %, Especial: turno 2 · CD 3, ±15 % de azar.

🎤 **Qué decir:**
> "Aquí no les mostramos un video: esto es el motor real corriendo en la presentación. Las reglas son las de `battleEngine.js`. Pikachu soy yo, Charizard es la máquina."

👆 **Secuencia de clics (sigue este orden, dura ~1 min):**

1. **Mira el botón ✨ Especial**: está deshabilitado y dice "Turno 2". Señálalo: *"El especial empieza bloqueado; no se puede usar en el primer turno."*
2. **Clic en ⚔️ Atacar.** Sale un número flotante rojo sobre Charizard y la barra baja. *"Daño con azar: nunca pega igual dos veces."* Luego juega la máquina (espera ~1 s).
3. **Clic en 🛡️ Defender.** Aparece el escudo 🛡️ en Pikachu. *"Defender no hace daño, pero el siguiente golpe que reciba baja a la mitad."*
4. Cuando el botón ✨ se ilumine: **clic en ✨ Especial.** *"Mucho más fuerte, y ahora queda en enfriamiento: miren el cartel 'CD 3' que baja cada turno."*
5. Sigue hasta que alguien caiga a 0 y pulsa **↺** para reiniciar si quieres repetir.

🎤 **Mientras juegas, explica estos valores:**
- **250 HP** para todas las cartas: partidas parejas y largas.
- **Defensa −50 %**: se consume al recibir un golpe.
- **±15 %**: cada ataque se multiplica por un factor entre 0,85 y 1,15.
- **Especial desde el turno 2 y cooldown 3**: no es spameable.
- **La IA de Charizard**: si tiene el especial disponible lo usa el **60 %** de las veces; la defensa solo la elige con un 25 % y nunca dos veces seguidas. Nunca elige una acción bloqueada.

> "Ahora veamos cómo está escrito eso por dentro."

---

## Slide 5 · Código · Daño con azar y defensa  (⏱ 2:00)

🖥️ **En pantalla:** tres funciones de `src/utils/battleEngine.js` a la izquierda y cuatro bullets a la derecha: *±15 %*, *Defender reduce el golpe a la mitad…*, *…pero nunca baja de 1*, *applyDamage devuelve una carta nueva*.

🎤 **Qué decir, función por función:**

> "Tres funciones pequeñas, cada una con una sola responsabilidad."

👆 **Señala `rollDamage`:**
> "`rollDamage` recibe el daño base y lo multiplica por un factor. `Math.random()` da algo entre 0 y 1; lo multiplicamos por **0,3** y le sumamos **0,85**, así el factor queda entre **0,85 y 1,15**. Es decir, ±15 %. Un ataque de 40 puntos, por ejemplo, termina pegando entre 34 y 46. `Math.round` para que siempre sea un entero."

👆 **Señala `calculateDamage`:**
> "`calculateDamage` aplica la defensa. Si el objetivo **no** se está defendiendo, devuelve el daño tal cual. Si sí, multiplica por `1 - damageReduction`: con reducción 0,5 el golpe queda a la mitad. Siguiendo el ejemplo, un 40 pasa a 20.
>
> Ojo con el `Math.max(1, reduced)`: aunque el cálculo diera 0, **siempre entra al menos 1 punto**. Esto evita combates que no avanzan nunca, porque alguien se defiende para siempre sin recibir daño. Además la reducción es un **parámetro con valor por defecto**: cada carta puede traer la suya en su campo `defense.damageReduction`, y si no trae, se usa `DEFENSE_REDUCTION`."

👆 **Señala `applyDamage`:**
> "`applyDamage` resta la vida con `Math.max(0, …)` para que nunca quede negativa, y devuelve **una carta nueva** con el spread `...card`, no modifica la original. Fíjense que además pone `isDefending: false`: **la defensa se consume al recibir el golpe**. Defenderse protege una vez, no para siempre."

> "¿Por qué tanto cuidado con no mutar? Porque estas mismas funciones las usan el modo contra la máquina y el modo online, y eso solo funciona si son **funciones puras**: mismo input, mismo output, sin efectos secundarios. Eso, además, es lo que nos permitió probarlas con `node:test`."

---

## Slide 6 · Código · Especial: bloqueo y enfriamiento  (⏱ 2:00)

🖥️ **En pantalla:** `beginTurn` y un fragmento de `getActions` con `nextTurn`, `reason`, `unlockTurn` y `specialCooldown`. A la derecha tres bullets.

🎤 **Qué decir:**
> "El especial es la habilidad fuerte de cada carta. En `db.json`, el de Pikachu, 'Trueno Total', tiene `baseDamage: 65`, `unlockTurn: 2` y `cooldown: 3`. Veamos cómo se hacen cumplir esos dos números."

👆 **Señala `beginTurn`:**
> "`beginTurn` se llama **al inicio de cada turno propio**. Si el cooldown ya es 0, devuelve la misma carta; si no, devuelve una copia con el cooldown reducido en 1. Insisto: el cooldown no baja con el tiempo ni con cada acción del rival, baja **una vez por turno propio**."

👆 **Señala el fragmento de `getActions`:**
> "En `getActions` armamos la lista de acciones disponibles. Para el especial calculamos `nextTurn = turnsPlayed + 1`, el turno que la carta está a punto de jugar. Hay dos razones para bloquearlo:
> 1. **Todavía no está desbloqueado**: `nextTurn < unlockTurn`. Con `unlockTurn = 2`, en el turno 1 está bloqueado.
> 2. **Está en enfriamiento**: `specialCooldown > 0`.
>
> Fíjense que no devolvemos solo un `true/false`: guardamos un **`reason`**, un texto como 'Disponible en el turno 2' o 'Cooldown: 2'. Así la interfaz no tiene que adivinar nada: pinta el botón deshabilitado y muestra el motivo. La UI no decide, solo muestra."

👆 **Traza en voz alta (puedes escribirla en el aire):**
> "Turno 1: bloqueado, 'disponible en el turno 2'. Turno 2: disponible; lo uso y el cooldown pasa a 3. Turno 3: `beginTurn` lo baja a 2, bloqueado. Turno 4: 1, bloqueado. Turno 5: 0, disponible otra vez. O sea, **un especial cada tres turnos**."

*(La misma traza está en la mini-arena del slide 4: si quieren, vuelven un segundo a ella.)*

---

## Slide 7 · Arquitectura y modularización  (⏱ 2:00)

🖥️ **En pantalla:** diagrama con tres columnas de colores: **UI · Componentes** (amarillo), **Reglas · Motor** (verde), **API · Estado** (azul). Flechas: UI → Reglas ("llama"), API → UI por fetch REST, y una barra inferior "window · CustomEvent" que une API y UI.

🎤 **Qué decir:**
> "La arquitectura son tres capas con una regla: **cada capa sabe lo menos posible de las otras**."

👆 **Señala la columna amarilla (UI):**
> "**UI**: los Web Components en `src/components`: `gameApp`, `battle`, `online`, el ranking, el panel de cartas, `serverStatus`... Su trabajo es **pintar y escuchar**. No calculan daño, no deciden turnos."

👆 **Señala la columna verde (Reglas), luego la flecha 'llama':**
> "**Reglas**: `src/utils`, donde viven `battleEngine`, `onlineMatch`, `progression`, `crypto`, `validators`. Son **funciones puras**: no tocan el DOM y no hacen `fetch`. Por eso se pueden probar en Node sin navegador. La UI las llama, y ellas nunca llaman a la UI."

👆 **Señala la columna azul (API) y la flecha de arriba ('fetch · REST'):**
> "**API**: un módulo por recurso, `cardsApi`, `playersApi`, `matchesApi`..., más `http.js`, que centraliza el `fetch`, y `apiConfig.js`, que elige la URL según el entorno con variables de Vite: desarrollo o producción."

👆 **Señala la barra inferior `window · CustomEvent`:**
> "¿Y cómo se avisan entre capas sin acoplarse? Por esta barra de abajo: **eventos del navegador**. La API publica 'el servidor se cayó' y la UI, si le interesa, escucha. Es el siguiente slide."

---

## Slide 8 · CustomEvent: hablar sin conocerse  (⏱ 2:00)

🖥️ **En pantalla:** a la izquierda tres fragmentos numerados (1: `http.js`, 2: `starterPacks.js`, 3: `gameApp.js`); a la derecha una lista de 6 eventos con el emisor y el receptor.

🎤 **Qué decir, fragmento por fragmento:**

👆 **Señala el fragmento 1:**
> "**Uno**: la capa de API avisa sin conocer a la interfaz. En `http.js`, cada vez que un `fetch` funciona o falla, hacemos `window.dispatchEvent` de un `CustomEvent` llamado `api-status`, con el estado en `detail`. `http.js` no tiene ni idea de quién escucha."

👆 **Señala el fragmento 2:**
> "**Dos**: un componente avisa hacia arriba. `starterPacks`, cuando termina de abrir un sobre, dispara `progress-updated` con el jugador actualizado. Fíjense en **`bubbles: true`**: eso hace que el evento **suba por el árbol del DOM**. El componente solo dice 'pasó esto' y no sabe quién se entera."

👆 **Señala el fragmento 3:**
> "**Tres**: quien necesita saberlo, escucha. `gameApp`, que es el padre de todo, hace `addEventListener('player-registered', …)` y con el `detail` inicia sesión."

👆 **Señala la lista de eventos (de arriba hacia abajo, rápido):**
> "Seis eventos conectan todo el juego: `api-status`, de `http.js` a `serverStatus`; `server-state`, que le avisa al formulario de registro si el servidor ya está listo; `player-registered`; `progress-updated`; `battle-finished`, cuando termina una batalla; y `match-ready`, cuando el lobby online ya tiene rival.
>
> ¿La ventaja? Podemos cambiar o borrar un componente y los demás ni se enteran. Y el flujo se lee fácil: **el hijo notifica, el padre decide**."

---

## Slide 9 · Seguridad · Hash + Salt — demo  (⏱ 2:30)

🖥️ **En pantalla:** "Nunca guardamos la contraseña". A la izquierda el código de `crypto.js`; a la derecha una demo con tres zonas: un campo **Contraseña** (`pikachu123`), un **Salt aleatorio** con botón "🎲 nuevo", y el **SHA-256(salt + contraseña)** en un bloque grande. Badges: SHA-256, 16 bytes de salt, `crypto.getRandomValues`, sin texto plano.

🎤 **Qué decir, empezando por el problema:**
> "Cuando alguien se registra, tenemos que guardar su contraseña, pero guardar el texto tal cual es un riesgo: si la base de datos se filtra, se filtran todas las claves. La solución es guardar solo una **huella irreversible**, un *hash*. Y como dos personas con la misma clave tendrían el mismo hash (y existen tablas gigantes de hashes ya calculados), le añadimos un **salt**: un texto aleatorio único por usuario."

👆 **Señala `generarSalt`:**
> "`generarSalt` pide 16 bytes aleatorios a `crypto.getRandomValues`, que es el generador **criptográficamente seguro** del navegador (no `Math.random`, que es predecible), y los convierte a hexadecimal con `aHex`: cada byte pasa a `toString(16)` con `padStart(2, '0')` para que siempre sean dos caracteres."

👆 **Señala `hashPassword`:**
> "`hashPassword` concatena **salt + contraseña**, lo convierte a bytes con `TextEncoder` y lo pasa por `crypto.subtle.digest('SHA-256', …)`. Es la **Web Crypto API**: viene nativa, sin instalar ninguna librería. Es asíncrona, por eso el `await`."

👆 **Señala `verificarPassword`:**
> "Para el login hacemos lo mismo: leemos el `salt` guardado del jugador, volvemos a calcular el hash con la clave que escribió y **comparamos con el `passwordHash` guardado**. Si el jugador no tiene hash o salt (cuentas antiguas), devuelve `false`. En ningún momento la contraseña se guarda ni viaja a la base de datos: solo guardamos `salt` y `passwordHash`."

👆 **Demo en vivo (haz esto con calma):**
1. *"Esta es la contraseña `pikachu123`."* Señala el hash de abajo: 64 caracteres hexadecimales.
2. **Escribe una letra más** en el campo (por ejemplo `pikachu1234`). *"Cambio un solo carácter y el hash cambia **por completo**."*
3. **Borra esa letra** para volver a `pikachu123`: el hash vuelve al anterior. *"La misma entrada con el mismo salt siempre da el mismo hash, por eso podemos verificar."*
4. **Clic en 🎲 nuevo.** *"Misma contraseña, otro salt, otro hash totalmente distinto. Aunque dos jugadores usen la misma clave, en la base de datos se ven diferentes."*

> "Y siendo honestos: SHA-256 es muy rápido, y para contraseñas reales lo ideal es un algoritmo **deliberadamente lento** como PBKDF2 o Argon2. Para este proyecto elegimos Web Crypto porque es nativo, y lo dejamos como mejora; lo veremos en el slide de límites."

---

## Slide 10 · Multijugador · secuencia  (⏱ 2:30)

🖥️ **En pantalla:** diagrama de secuencia con tres participantes: **Jugador 1** (amarillo), **/matches (Render)** (azul, el servidor) y **Jugador 2** (verde). Los 8 mensajes aparecen animados uno a uno. Hay píldoras de tiempo: **60 s** y **25 s**.

🎤 **Qué decir:**
> "Aquí está el corazón del proyecto. **No hay WebSockets**: el servidor es un JSON Server que solo sabe responder peticiones. Así que los dos jugadores se coordinan escribiendo y leyendo **el mismo documento**, un registro en `/matches`. Un reto tiene **cuatro fases**."

👆 **Recorre el diagrama de arriba abajo, señalando cada flecha cuando aparezca:**

1. **Flecha 1, `POST · status: waiting` (con la píldora de 60 s):**
   > "**Fase 1, invitación.** El jugador 1 crea el match con estado `waiting`. La invitación vive **60 segundos** (`INVITACION_MS`); pasado ese tiempo, deja de ser válida."
2. **Flecha 2, `GET ?p2Id&status=waiting · 4 s`:**
   > "El jugador 2 consulta cada **4 segundos** si alguien lo retó. Es un *polling* de invitaciones."
3. **Flecha 3, `PATCH · selecting (20 s)`:**
   > "Cuando acepta, pasa el match a `selecting`. **Fase 2, selección de mazo**: cada uno tiene **20 segundos**. Si alguien no termina, `completarMazo` rellena el mazo con cartas al azar de su colección, sin repetir."
4. **Flecha 4, `PATCH mazo · polling 1.5 s`:**
   > "Cada jugador guarda su mazo con un PATCH y consulta cada **1,5 s** si el rival ya lo hizo."
5. **Flecha 5, `PATCH battleState · seq 1` (con la píldora de 25 s):**
   > "**Fase 3, batalla.** El jugador 1 es quien crea el estado inicial, `iniciarBatalla`: sortea quién empieza, deja `seq: 1` y fija un tiempo límite de **25 segundos por turno**. Si p1 no responde, p2 lo inicia pasado el periodo de gracia de 5 s."
6. **Flechas 6 y 7, el GET y el PATCH `seq 2` (píldora de 25 s en el jugador 2):**
   > "El jugador 2 hace *polling*, ve que cambió a `seq 1`, juega, y escribe `seq 2`."
7. **Flecha 8 y texto final:**
   > "Y así sigue, alternando. **Fase 4, fin**: cuando a alguien se le acaban las cartas, el estado pasa a `finished` y se guarda el `winnerId`."

> "Lo importante: **todo el estado de la partida está en un solo objeto, `battleState`**, y cada jugada lo reemplaza por una versión nueva con un número de secuencia, `seq`. Ese número es lo que resuelve el siguiente problema."

---

## Slide 11 · Concurrencia · `seq` — demo  (⏱ 2:00)

🖥️ **En pantalla:** "Evitar condiciones de carrera". Dos carriles (**Jugador 1** y **Jugador 2**) a los lados y en el centro un anillo con **seq = 4**. Al entrar al slide, la animación arranca sola tras ~1 s. Abajo, el botón **▶ Simular carrera** y tres chips: *Polling cada 1.5 s*, *Cada acción hace seq += 1*, *Escritura optimista*. Badges: HTTP polling, optimistic locking, sin WebSockets.

🎤 **Primero plantea el problema:**
> "Con polling hay un problema clásico: la **condición de carrera**. Imaginen que los dos jugadores leen el estado al mismo tiempo, y los dos creen que es su turno, o los dos mandan una jugada. Si ambos escriben, **el último pisa al primero** y se pierde una jugada."

👆 **La animación arranca sola; si ya terminó, pulsa ▶ Simular carrera.** Narra mientras aparece cada paso:

1. **Paso 1 de ambos carriles (aparecen a la vez):** *"Los dos leen `seq = 4`."*
2. **Paso 2 del Jugador 1:** *"El jugador 1 escribe primero: su jugada entra y el servidor pasa a `seq = 5`."* (Señala el anillo central: el número cambia y 'late'.)
3. **Paso 2 del Jugador 2 (en rojo, ✖):** *"El jugador 2 va a escribir, pero antes **relee** y ve `seq = 5`, distinto de los 4 que él tenía. Su información está vieja."*
4. **Paso 3 del Jugador 2 (en verde):** *"Entonces **descarta su acción y aplica el estado remoto**: ve lo que hizo el rival y ahora decide con el estado correcto."*

🎤 **Cierra con el concepto:**
> "Esto se llama **control de concurrencia optimista**, o *optimistic locking*: no bloqueamos nada, asumimos que normalmente no habrá conflicto y, antes de escribir, **verificamos que nadie haya cambiado la versión**. Cada acción hace `seq += 1`, así que `seq` funciona como número de versión. Y como no tenemos WebSockets, el estado se sincroniza con *polling* cada 1,5 segundos."

---

## Slide 12 · Código · Escritura optimista  (⏱ 2:00)

🖥️ **En pantalla:** el método `escribir(transformar)` de `src/components/online/onlineBattle.js`, en un bloque grande, y cuatro bullets a la derecha.

🎤 **Qué decir, recorriendo el código de arriba hacia abajo:**

👆 **Señala la firma `async escribir(transformar)`:**
> "Este método es **la única puerta de escritura** de la batalla online. Recibe una función `transformar` que sabe cómo convertir el estado viejo en el nuevo (atacar, defender, pasar por timeout, abandonar...). Así todas las acciones pasan por el mismo control."

👆 **Señala `if (this.enviando) return; this.enviando = true;`:**
> "Primera defensa: la bandera **`enviando`**. Si ya hay una petición viajando, ignoramos el clic. Esto evita el doble clic accidental del *mismo* jugador."

👆 **Señala `const actual = await getMatch(...)`:**
> "Segunda: **antes de escribir, volvemos a leer** el estado actual del servidor. No confiamos en lo que tenemos en pantalla."

👆 **Señala el `if` con `seq` y `estaTerminada`:**
> "Tercera, la clave: comparamos `actual.battleState.seq` con el `seq` que nosotros teníamos. Si es distinto, **el rival ya movió**, y entonces no escribimos: llamamos a `aplicarRemoto(actual)` y nos ponemos al día. Lo mismo si la partida ya terminó."

👆 **Señala `transformar` y `patchMatch`:**
> "Si todo coincide, calculamos el nuevo estado con `transformar`. Si devuelve `null` (por ejemplo, una acción no permitida), no hacemos nada. Si no, mandamos un **PATCH** con `parcheBatalla`, que no solo guarda el `battleState` sino que también actualiza el `status`: pasa a `finished` y fija el `winnerId` si alguien ganó. Con la respuesta del servidor aplicamos el estado."

👆 **Señala `catch` y `finally`:**
> "Si algo falla, mostramos el error. Y el **`finally`** libera la bandera `enviando` pase lo que pase, para que el juego nunca quede bloqueado."

🎤 **Y la salvedad honesta** *(importante, pregunta típica)*:
> "Ojo: entre el GET y el PATCH sigue habiendo una ventana de milisegundos. Un servidor con transacciones permitiría un 'compare-and-set' atómico. JSON Server no lo tiene. Nosotros reducimos mucho el riesgo con la relectura justo antes de escribir y con que cada turno pertenece a **un solo jugador**: `aplicarAccion` rechaza cualquier jugada fuera de turno, así que en la práctica solo uno escribe a la vez."

---

## Slide 13 · Código · Cambio de turno  (⏱ 1:30)

🖥️ **En pantalla:** la función `pasarTurno(state, role, now)` de `src/utils/onlineMatch.js`, en letra grande, y cuatro bullets.

🎤 **Qué decir, línea por línea:**

> "`pasarTurno` es una función corta, pero resume cómo avanza una partida."

👆 **Señala cada línea mientras la explicas:**
- `const siguiente = rival(role);` → *"`rival` devuelve 'p2' si le paso 'p1' y al revés. Estos son los **roles** de la partida."*
- `if (role !== state.firstTurn) state.round += 1;` → *"Una **ronda** son los dos turnos. La ronda sube cuando termina de actuar el jugador que **no** empezó; así cuenta rondas completas."*
- `state.turn = siguiente;` → *"Le pasa el turno al rival."*
- `state.turnDeadline = now + TURNO_MS;` → *"Fija el nuevo plazo: `TURNO_MS` son **25 000 ms = 25 segundos**. Para el tiempo que le queda, la interfaz mira ese `turnDeadline` cada 250 ms y mueve la barra."*
- las dos últimas líneas → *"Toma la carta activa del siguiente y le aplica **`beginTurn`**: acá se descuenta un punto de cooldown del especial. Ven cómo se conectan los slides 6 y 13."*

> "Fíjense también en que muta `state`, pero **solo el clon** que le pasan: quien la llama (`aplicarAccion`, en el siguiente slide) ya hizo un `structuredClone`. Dentro del archivo hay otra pieza relacionada, `aplicarTimeout`: si un jugador deja correr los 25 s, pierde el turno; y con **3 timeouts seguidos (`MAX_TIMEOUTS`) pierde la partida por inactividad**. El rival espera 5 segundos extra (`GRACIA_MS`) antes de aplicar el timeout, por si el otro tuvo un microcorte."

---

## Slide 14 · Código · Aplicar una acción  (⏱ 1:30)

🖥️ **En pantalla:** `aplicarAccion(state, role, actionIndex, now)` de `onlineMatch.js`, con comentarios `// ... aplica el resultado ...`, y cuatro bullets.

🎤 **Qué decir, paso a paso:**

👆 **Señala la primera línea (`if (estaTerminada(state) || state.turn !== role) return null;`):**
> "Primero, las **guardas**: si la partida terminó o **no es mi turno**, devuelve `null` y no pasa nada. Esto es lo que evita que alguien juegue fuera de turno aunque manipule el cliente."

👆 **Señala `getActions(...)[actionIndex]` y `!action?.enabled`:**
> "Busca la acción elegida con el mismo `getActions` del modo contra la máquina. Si está **bloqueada** (especial en cooldown, por ejemplo), también devuelve `null`. El servidor de reglas es el mismo que usamos arriba: **un solo motor para CPU y online**."

👆 **Señala `structuredClone(state)`:**
> "Aquí viene lo importante: `structuredClone` hace una **copia profunda** del estado. Todo lo que sigue trabaja sobre la copia, así que **el estado anterior nunca se muta**. Si algo falla a la mitad, el estado original sigue intacto. Y gracias a eso podemos comparar versiones."

👆 **Señala `resolveAction(...)`:**
> "Llama a `resolveAction` del motor, que calcula el daño con `rollDamage`, `calculateDamage` y `applyDamage`, los del slide 5. Los comentarios `// ... aplica el resultado ...` recortan cosas que no caben en pantalla: guardar actor y objetivo, registrar el `lastEvent` y, si la carta enemiga cayó, avanzar a la siguiente de su mazo."

👆 **Señala `next.seq += 1`, `timeouts[role] = 0`:**
> "`seq += 1` es la nueva versión: el número que vimos en el slide 11. Y reiniciamos los timeouts del jugador, porque si actuó, no está inactivo."

👆 **Señala `if ... terminar(next, role)` y `pasarTurno`:**
> "Si derrotó la **última** carta del rival, `terminar` fija el ganador; si no, `pasarTurno`, que ya vimos. Y se devuelve el estado nuevo. Cada partida online acaba otorgando **+50 puntos al ganador y +10 al perdedor**, constantes `POINTS_WIN` y `POINTS_LOSS`."

---

## Slide 15 · Ranking y administrador  (⏱ 1:00)

🖥️ **En pantalla:** dos columnas. Izquierda: "Ranking · puntos → victorias → menos partidas" con un `sort` y un podio (2.º, 1.er, 3.er). Derecha: "Admin · verbos HTTP correctos" con `editarCartaCompleta` (PUT) y `cambiarEstadoCarta` (PATCH), chips GET/POST/PUT/PATCH/DELETE y una nota.

🎤 **Qué decir:**

👆 **Señala el `sort` del ranking:**
> "El ranking es un solo `sort` encadenado con `||`: primero por **puntos** (descendente); si hay empate, por **victorias**; y si siguen iguales, gana quien tiene **menos partidas jugadas**, porque llegó al mismo resultado con menos intentos. El `?? 0` evita errores con jugadores nuevos sin estadísticas. Los tres primeros se destacan en el podio."

👆 **Señala el bloque de admin (PUT y PATCH):**
> "En el panel de administrador hacemos el CRUD completo de cartas, y nos importó usar **el verbo HTTP correcto**. **PUT** reemplaza el recurso **completo**: lo usamos al editar una carta con todos sus campos. **PATCH** actualiza **solo el campo que le mandamos**: lo usamos para activar o desactivar una carta, enviando únicamente `{ active }`. Y DELETE para borrar."

👆 **Señala la nota inferior:**
> "Dos reglas de negocio: siempre deben quedar **mínimo 10 cartas activas** (`MIN_ACTIVE_CARDS`), porque hay que poder armar mazos; y si creas una carta sin imagen, se busca automáticamente en la PokéAPI."

---

## Slide 16 · Resiliencia · cold start — demo  (⏱ 2:00)

🖥️ **En pantalla:** "Mientras Render despierta". A la izquierda una **máquina de estados** con cuatro círculos: `checking`, `waking`, `online` (verde) y `offline` (rojo), con las flechas "ping falla", "reintento · 2-10 s", "ping ok", "5 intentos" y una flecha punteada "botón «Conectar» ↩ checking". Debajo un botón **▶ Simular arranque** con un mensaje de estado. A la derecha, el código de `serverStatus.js`.

🎤 **Primero el problema real:**
> "Esto es algo que no esperábamos y que nos obligó a diseñar una solución. Nuestro JSON Server está desplegado en **Render, en el plan gratuito**, y ese plan **apaga el servidor tras un rato de inactividad**. Cuando entra alguien después, el primer arranque puede tardar **cerca de un minuto**. Sin hacer nada, el jugador vería errores de red, un formulario que no responde... y creería que la app está rota."

> "La solución: mientras el servidor despierta, **bloqueamos toda la pantalla** con una pantalla de carga a pantalla completa, y no dejamos jugar hasta que responda."

👆 **Muestra el diagrama y pulsa ▶ Simular arranque** (también arranca solo ~1 s después de llegar al slide). Narra cada estado según se ilumina:

1. **`checking`**: *"Al abrir la app, hacemos un `ping`. Si responde, perfecto."*
2. **`waking`** (el texto cambia a 'Despertando servidor… reintento en 2 s, 4 s, 6 s…'): *"Si no responde, pasamos a 'despertando' y **reintentamos con esperas crecientes**."*
3. **`online`** (se ilumina en verde, mensaje '✔ En línea'): *"Cuando el ping funciona, se cierra la pantalla de carga y el juego se desbloquea."*

👆 **Señala el código de la derecha:**
> "`ESPERAS_MS` es `[2000, 4000, 6000, 8000, 10000]`: un **backoff lineal**. Esperamos 2 s, luego 4, 6, 8 y 10. No martillamos al servidor mientras arranca y tampoco hacemos esperar de más cuando está casi listo.
>
> `conectar()` primero pone `checking` y hace `pingServer()`. Después un `for…of`: si el ping ya funcionó, `break`. Si no, estado `waking`, **`render()`** para que la pantalla se vea, espera con `setTimeout` envuelto en una promesa, y vuelve a hacer ping. Al final, si `ok` es verdadero queda `online`, y si no, `offline`.
>
> En total son **un ping inicial y 5 reintentos: 6 intentos**; por eso la pantalla de carga muestra 'intento 1/6'. Sumando las esperas, damos unos 30 segundos de margen antes de rendirnos."

> "Dos detalles más. El `pingServer` usa un **`AbortController` con un timeout de 15 s**, para que una petición colgada no bloquee la espera. Y la pantalla de carga es un **overlay `position: fixed` a pantalla completa con `z-index: 9999`**, con `role='alert'`, así que ni se puede hacer clic en el formulario de atrás. Si agotamos los 5 reintentos, pasa a `offline` y aparece el botón **'Conectar / Despertar servidor'**: la flecha punteada, que nos devuelve a `checking`.
>
> Y como todo está conectado por eventos: `serverStatus` publica el estado con `server-state`, el formulario de registro lo escucha para habilitar el botón solo cuando el servidor está listo, y al reconectar se dispara `api-reconnected` para que `gameApp` refresque."

*(Si quieres hacer el momento más visual: si el servidor real está dormido en la demo, **ábrela y que se vea el overlay real**; si no, usa la simulación.)*

---

## Slide 17 · Testing y CI/CD — demo  (⏱ 1:30)

🖥️ **En pantalla:** "**48** pruebas en cada deploy". Arriba un pipeline: `git push` ➜ `npm ci` ➜ `npm test` ➜ `build` ➜ `GitHub Pages`. Abajo a la izquierda cuatro barras: **progression 20**, **onlineMatch 17**, **battleEngine 6**, **auth 5** y un botón **▶ Ejecutar pruebas**; a la derecha el fragmento de `deploy.yml`. Badges: `node --test`, GitHub Actions, CI → CD, "falla = no hay deploy".

🎤 + 👆 **Qué decir y hacer:**
> "Las pruebas son lo que nos dio confianza para tocar el multijugador sin romper el motor. Usamos **`node:test`**, el ejecutor de pruebas **nativo de Node**: sin Jest, sin instalar nada. El script es `node --test tests/*.test.js`."

👆 **Pulsa ▶ Ejecutar pruebas.** El pipeline se va iluminando etapa por etapa y el número grande del título cuenta hasta 48 cuando llega a `npm test`.

> "Miren cómo cuenta: **48**. Se reparten así: 20 en `progression` (sobres, ruleta, memoria, bonos), 17 en `onlineMatch` (turnos, timeouts, abandono, `seq`, mazos), 6 en `battleEngine` (daño, defensa, cooldown) y 5 en `auth` (hash y verificación)."

👆 **Señala el pipeline y luego el `deploy.yml`:**
> "Y esto va conectado al despliegue. En el workflow de GitHub Actions, cuando hacemos `git push` a `main`: `npm ci` instala exactamente lo del lockfile, **`npm test` corre las 48 pruebas** y solo si pasan se hace el `build` y se publica en **GitHub Pages**. Fíjense: es una sola línea, `run: npm test`, pero si **una** prueba falla, el workflow se detiene y **no hay deploy**. Es la red de seguridad.
>
> Nota sobre el `build`: le pasamos `VITE_API_MODE: production` y la URL de la API como variable, para que el mismo código apunte a Render y no a `localhost`."

> "Y por eso la lógica está en funciones puras: las probamos sin navegador, sin servidor y en milisegundos."

---

## Slide 18 · Decisiones técnicas y sus límites  (⏱ 1:30)

🖥️ **En pantalla:** seis paneles. Dos verdes con ✔: *Lógica pura y compartida*, *Cero frameworks*. Cuatro amarillos con ⚠: *Polling, no tiempo real*, *JSON Server en Render*, *Seguridad básica*, *Pruebas solo de lógica*.

🎤 **Qué decir:**
> "Cerramos la parte técnica con lo que decidimos y lo que **sabemos que no está perfecto**, porque saber los límites de tu propio proyecto es parte de la ingeniería."

👆 **Señala los dos paneles verdes:**
> "Lo que salió bien: **un solo motor para CPU y online**, fácil de probar; y **cero frameworks**, todo con Web Components y ES Modules nativos."

👆 **Señala los cuatro amarillos, uno por uno:**
> "**Polling, no tiempo real.** Es simple y barato, pero hay una latencia de 1,5 a 2 segundos entre que el rival juega y tú lo ves. Con más tiempo, usaríamos WebSockets o Server-Sent Events.
>
> **JSON Server en Render.** El disco del plan gratuito es **efímero**: cuando el servidor se reinicia, los datos que no estén en `db.json` pueden perderse. Para producción usaríamos una base de datos real.
>
> **Seguridad básica.** Guardamos hash y salt, pero `GET /players` **los expone** públicamente, y el usuario administrador está en `db.json`. Lo correcto sería un backend que nunca devuelva esos campos y use un hash lento como PBKDF2 o Argon2, además de tokens de sesión.
>
> **Pruebas solo de lógica.** Las 48 pruebas cubren las funciones puras; no hay pruebas de interfaz, de la capa API ni del ranking. Es lo siguiente que agregaríamos."

---

## Slide 19 · Cierre  (⏱ 0:30 + preguntas)

🖥️ **En pantalla:** "¿Listos para el combate?" con Ash y Pikachu. Nombres: Andrés Felipe · Nicolas Archila.

🎤 **Qué decir:**
> "Resumiendo: un juego completo en JavaScript puro, con un motor de reglas **puro y compartido**, un multijugador que funciona sin WebSockets gracias a **`seq` y el control optimista**, contraseñas con **hash + salt**, una pantalla de carga que protege al jugador cuando el servidor **despierta**, y **48 pruebas** que bloquean el deploy. Gracias por escucharnos. ¿Preguntas?"

👆 **Si quieres cerrar con algo de show:** vuelve al slide 4 (`←` o escribe `#4` en la URL) y juega una partida rápida contra Charizard mientras responden preguntas.

---

## Banco de preguntas probables (respuestas cortas)

**1. ¿Por qué no usaron WebSockets?**
JSON Server no los soporta, y el proyecto pedía usar una API REST con Fetch. El *polling* con `seq` es simple, y el costo es una latencia de 1,5 a 2 s. Con más tiempo usaríamos WebSockets o SSE.

**2. ¿Qué pasa si dos jugadores escriben exactamente al mismo tiempo?**
Antes de escribir, cada cliente relee el match y compara `seq`. Si cambió, descarta su acción y aplica el estado remoto. Queda una ventana de milisegundos entre el GET y el PATCH que solo cerraría un servidor con transacciones; la reducimos porque **solo el jugador con el turno puede escribir** (`aplicarAccion` rechaza lo demás).

**3. ¿Por qué SHA-256 y no bcrypt?**
Porque Web Crypto lo trae nativo en el navegador y no queríamos dependencias. Es suficiente para la demostración (con salt único), pero para producción usaríamos un algoritmo lento y con *cost factor*: PBKDF2 (también en Web Crypto), bcrypt o Argon2, y mover la verificación al servidor.

**4. ¿Qué pasa si un jugador se desconecta en medio de la partida?**
El turno tiene un límite de **25 s**; el rival espera 5 s de gracia y aplica el `aplicarTimeout`. A los **3 timeouts** pierde por inactividad (`MAX_TIMEOUTS = 3`). También existe `aplicarAbandono` si sale voluntariamente.

**5. ¿Por qué usaron JSON Server y no un backend propio?**
Para centrar el esfuerzo en JavaScript del cliente y en el diseño del multijugador. Las consecuencias (disco efímero, sin transacciones, `/players` expuesto) las dejamos explícitas en el slide 18.

**6. ¿Cómo evitan que alguien haga trampa manipulando el cliente?**
Parcialmente: las reglas se validan en el cliente (`aplicarAccion` rechaza jugadas fuera de turno o bloqueadas), pero como no hay backend propio, un usuario técnico podría hacer PATCH directo. En un sistema real las reglas correrían en el servidor.

**7. ¿Qué mejorarían primero?**
(1) Ocultar `passwordHash` y `salt` en `/players` y usar PBKDF2/Argon2; (2) pruebas de API y de interfaz; (3) tiempo real con WebSockets/SSE; (4) una base de datos persistente.

---

## Tips finales de exposición
- **No leas el código línea por línea de corrido:** señálalo con el cursor y di *qué problema resuelve*, después el detalle.
- Si se te va el tiempo, **recorta** los slides 13 y 14 (ya cubiertos por el 12) y el 15 (ranking); los de **más valor técnico** son 9, 11–12, 16 y 17.
- Si falla internet o la PokéAPI, las demos de sobres y mini-arena muestran cartas sin imagen, pero **siguen funcionando**; los de hash, `seq`, estados y pruebas son 100 % locales.
- Practica el slide 16 con calma: es el que más pregunta genera ("¿y qué pasa si el servidor está dormido?").
