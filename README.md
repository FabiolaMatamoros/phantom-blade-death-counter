<a id="english"></a>

# Automatic Death Counter for OBS  ( Phantom Blade Zero )

**🇺🇸 English** · [🇪🇸 Español](#espanol)

An OBS overlay that **counts your deaths on its own**. It detects the game's death screen, knows when you're fighting a boss, and resets the boss counter when you win. No buttons needed.

![Preview](preview.jpg)

- **DEATHS**: your total for the playthrough. Goes up every time the **DEAD** screen appears.
- **CURRENT BOSS**: shows up when a boss health bar appears, counts your deaths against that boss, and goes back to 0 (and hides) when you defeat it.

It's a single HTML file — nothing to install. Made by [MistyWaifu](https://twitch.tv/mistywaifu).

---

## Requirements

- **OBS Studio 28 or newer** (WebSocket is built in).
- **Phantom Blade Zero** captured in OBS **full screen at 16:9** (1920×1080, 2560×1440, etc.).
- Optional: a **Stream Deck** with Elgato's official **OBS Studio** plugin, for manual corrections.

---

## Setup

### 1. Download the file

Download `contador-muertes.html` from this repo (**Code → Download ZIP**, or open the file and click the download icon). Save it in a folder you won't move, for example `Documents\stream`. If you move it later, you'll need to select it again in OBS.

### 2. Turn on OBS WebSocket and copy your password ⚠️

The counter **will not work** without this step.

1. In OBS, go to **Tools → WebSocket Server Settings**.
2. Check **Enable WebSocket server**.
3. Leave the port at **4455**.
4. Keep **Enable Authentication** checked. If the password is empty, click **Generate Password**.
5. Click **Show Connect Info** → **Copy** next to *Server Password*.
6. Click **Apply**, then **OK**.

> 🔒 **Never show the *Connect Info* window on stream.** Anyone on your network with that password could control your OBS. If it ever shows up on stream, generate a new one.

### 3. Add your password to the file

Open `contador-muertes.html` in **Notepad** (right-click → *Open with* → *Notepad*) and find this line near the top of the code:

```js
const OBS_PASSWORD = 'YOUR_PASSWORD_HERE';
```

Replace `YOUR_PASSWORD_HERE` with the password you copied, **keeping the quotes**:

```js
const OBS_PASSWORD = 'aB3xYz9QwErTy123';
```

Save with **Ctrl + S**.

### 4. Name your game capture `GAME`

In **Sources**, right-click the source that captures the game (*Game Capture* or *Window Capture*) → **Rename** → type `GAME` in capital letters.

The counter looks for the death screen **inside that source**, not your whole scene, so your webcam or alerts on top of the game won't confuse it. (Want a different name? See [Customize](#customize).)

### 5. Add the counter to your scene

1. **Sources → + → Browser**.
2. Check **Local file** → **Browse** → select `contador-muertes.html`.
3. **Width:** `440`, **Height:** `140`.
4. Click **OK** and place it wherever you like (top-left works well).

### 6. Check that it works

With OBS open, double-click `contador-muertes.html` to open it in your browser. The status is shown in the bottom-left corner:

| Message | What it means |
|---|---|
| 🟢 **OBS connected · Stream Deck ready** | All good |
| 🟢 **Detection active · DEAD 0.00 …** | It can see your game |
| 🔴 **add your password to the file** | Redo step 3 |
| 🔴 **wrong password** | The password doesn't match — copy it again (step 2) |
| 🔴 **OBS not connected** | WebSocket isn't on (step 2) |
| 🔴 **can't find the source "GAME"** | Check the name (step 4) |

Once everything is green, **close that browser tab**. It's only for checking, and leaving it open uses extra resources.

---

## Manual buttons (optional)

In case detection misses a death or counts one too many.

**In OBS**, add these **Text (GDI+)** sources to your game scene with **empty** text (they're invisible):

| Source name | What it does |
|---|---|
| `ADD_DEATH` | +1 to the total |
| `REMOVE_DEATH` | −1 from the total |
| `NEW_BOSS` | Resets *Current boss* to 0 |
| `RESET_DEATHS` | Resets everything to 0 |

Every time you show or hide one of these sources (click its 👁 eye icon), the counter runs that action.

Manual buttons only change the **total**. *Current boss* is only changed by automatic detection and by `NEW_BOSS`.

---

## 🎛️ Stream Deck setup (optional)

If you have a Stream Deck, you can turn those sources into physical buttons. You **don't need** a Twitch plugin or any extra counter plugin — just Elgato's OBS plugin.

### 1. Install the OBS Studio plugin

1. Make sure your **Stream Deck app is version 6.9 or newer** (update it if the plugin won't install).
2. Install Elgato's official **OBS Studio** plugin, either:
   - from the [Elgato Marketplace](https://marketplace.elgato.com/product/obs-studio-35615969-830f-45c9-ba0a-1a295bba7fec), or
   - inside the Stream Deck app: click the **Plugins** tab on the right, search **OBS Studio**, and install the one **by Elgato**.

> Avoid third-party "death counter" or "win/lose counter" plugins for this — they keep their own number and won't talk to this overlay.

### 2. Create the sources in OBS first

Before configuring the keys, create the empty **Text (GDI+)** sources from the table above (`ADD_DEATH`, `REMOVE_DEATH`, `NEW_BOSS`, `RESET_DEATHS`) in the **same scene** where you play. If they don't exist yet, they won't show up in the Stream Deck list.

### 3. Add the keys

1. In the Stream Deck app, click the **Keys** tab and search **obs**.
2. Under **OBS Studio**, find **Source Visibility**.
3. Drag it onto an empty key (tip: use a free page, like page 2, so you don't overwrite your other buttons).
4. Click the key and configure it at the bottom:
   - **Scene:** the scene where you play (for example `GAMING`)
   - **Source:** `ADD_DEATH`
   - **Title:** something like `+1`
5. The first time, it may ask you to connect to OBS: OBS must be **open**, and the password is the same one from step 2.
6. Repeat for the other buttons:

| Key title | Source |
|---|---|
| `+1` | `ADD_DEATH` |
| `-1` | `REMOVE_DEATH` |
| `BOSS` | `NEW_BOSS` |
| `RESET` | `RESET_DEATHS` |

### 4. Test it

Press the `+1` key with OBS open. The eye icon next to `ADD_DEATH` should toggle in OBS, and the counter should go up by 1.

- **The eye toggles but the counter doesn't change** → the overlay isn't connected to OBS. Open the HTML in your browser and check the status message (see **Setup → step 6**).
- **The eye doesn't toggle** → check that the key has the right **Scene** and **Source** selected.

> Put `RESET` in a corner of your Stream Deck — it wipes everything and is easy to hit by accident.

---

## How it works

About twice per second, the counter asks OBS for a small screenshot of the `GAME` source and checks it **in memory**. Nothing is saved to your disk, and it uses very little CPU or RAM.

| What it looks for | Where |
|---|---|
| The red word **DEAD** | Bottom center |
| The frame of the **boss health bar** | Top center (works even when the boss is almost dead) |
| The white **kanji** shown when a boss is defeated | Center-right |

To avoid false positives, each one has to appear in 2 screenshots in a row, and each death is only counted once.

Deaths count toward **CURRENT BOSS** only if the boss bar was on screen (or seen in the last 12 seconds). Regular deaths only add to the total.

---

## FAQ

**Does it reset when I close OBS?**
No. The numbers are saved even if you close OBS or refresh the source. To start from zero, use `RESET_DEATHS`.

**I clicked *Refresh* and it didn't go back to 0.**
That's expected — *Refresh* doesn't clear the numbers. Use `RESET_DEATHS`.

**I want to start with the deaths I already had.**
Press `ADD_DEATH` as many times as you need (outside a boss fight).

**It doesn't detect deaths.**
Make sure the game fills the whole `GAME` source at 16:9, with no black bars. Open the HTML in your browser and check the *DEAD* number when you die: it should go above 0.70.

**I opened the HTML in my browser and it shows different numbers.**
The browser and OBS each keep their own numbers. The one on your stream is the OBS one.

**I get *HTTP ERROR 426* when I open `http://127.0.0.1:4455`.**
That's a good sign — WebSocket is on. If you get *ERR_CONNECTION_REFUSED* instead, uncheck and recheck *Enable WebSocket server*, click *Apply*, and restart OBS.

---

## Customize

In the same settings block at the top of the HTML you can change:

- `LABEL_TOTAL` and `LABEL_BOSS`: the overlay text. For Spanish, use `'MUERTES :'` and `'JEFE ACTUAL :'`.
- `SOURCE_GAME`: the name of your game source, if you don't want to call it `GAME`.
- `SOURCE_ADD`, `SOURCE_REMOVE`, `SOURCE_RESET`, `SOURCE_NEW_BOSS`: the source names used for the manual buttons.

To test without OBS, open the HTML in your browser and use the keyboard: `+` / `-` add and subtract, `J` resets the boss, `B` simulates a boss bar, and `0` resets everything.

---

## License

MIT — use it, change it, and share it freely. If you use it on stream, a shoutout to [MistyWaifu](https://twitch.tv/mistywaifu) is always appreciated! 💜

---
---

<a id="espanol"></a>

# ⚔️ Contador de muertes automático para OBS — Phantom Blade Zero

[🇺🇸 English](#english) · **🇪🇸 Español**

Overlay para OBS que **cuenta tus muertes solo**. Detecta la pantalla de muerte del juego, sabe cuándo estás peleando con un jefe y reinicia el contador del jefe cuando lo derrotas. No hace falta apretar nada.

- **MUERTES**: el total de tu partida. Suma cada vez que aparece la pantalla **DEAD**.
- **JEFE ACTUAL**: aparece cuando sale la barra de vida de un jefe, cuenta tus muertes contra ese jefe y vuelve a 0 (y se esconde) cuando lo derrotas.

Es un solo archivo HTML, sin instalar nada. **Es el mismo archivo para inglés y español**: solo cambias dos líneas para que el overlay diga *MUERTES* y *JEFE ACTUAL* (paso 3). Hecho por [MistyWaifu](https://twitch.tv/mistywaifu).

---

## Requisitos

- **OBS Studio 28 o más nuevo** (ya trae el WebSocket incluido).
- **Phantom Blade Zero** capturado en OBS **a pantalla completa en 16:9** (1920×1080, 2560×1440, etc.).
- Opcional: **Stream Deck** con el plugin oficial **OBS Studio** de Elgato, para correcciones manuales.

---

## Instalación

### 1. Descarga el archivo

Descarga `contador-muertes.html` de este repo (**Code → Download ZIP**, o abre el archivo y dale al ícono de descarga). Guárdalo en una carpeta fija, por ejemplo `Documentos\stream`. Si lo mueves después, tienes que volver a elegirlo en OBS.

### 2. Activa el WebSocket de OBS y copia tu contraseña ⚠️

Sin este paso el contador **no funciona**.

1. En OBS: **Tools → WebSocket Server Settings** (*Herramientas → Ajustes del servidor WebSocket*).
2. Marca **Enable WebSocket server**.
3. Deja el puerto en **4455**.
4. Deja marcado **Enable Authentication**. Si la contraseña está vacía, dale a **Generate Password**.
5. Clic en **Show Connect Info** → **Copy** al lado de *Server Password*.
6. Dale **Apply** y **OK**.

> 🔒 **Nunca muestres la ventana de *Connect Info* en directo.** Con esa contraseña alguien en tu red podría controlar tu OBS. Si se llega a ver, genera una nueva.

### 3. Configura el archivo (contraseña e idioma)

Abre `contador-muertes.html` con el **Bloc de notas** (clic derecho → *Abrir con* → *Bloc de notas*). Casi al principio del código está el bloque de configuración:

```js
const OBS_PASSWORD = 'YOUR_PASSWORD_HERE';

const LABEL_TOTAL = 'DEATHS :';
const LABEL_BOSS = 'CURRENT BOSS :';
```

Cámbialo así, **dejando las comillas**:

```js
const OBS_PASSWORD = 'aB3xYz9QwErTy123';   // ← tu contraseña

const LABEL_TOTAL = 'MUERTES :';
const LABEL_BOSS = 'JEFE ACTUAL :';
```

Guarda con **Ctrl + S**.

### 4. Nombra la captura del juego `GAME`

En **Sources**, clic derecho en la fuente que captura el juego (*Game Capture* o *Window Capture*) → **Rename** → escribe `GAME` en mayúsculas.

El contador busca la pantalla de muerte **dentro de esa fuente**, no en tu escena completa, así que tu cámara o tus alertas encima del juego no lo confunden. (¿Quieres otro nombre? Mira [Personalizar](#personalizar).)

### 5. Agrega el contador a tu escena

1. **Sources → + → Browser**.
2. Marca **Local file** → **Browse** → elige `contador-muertes.html`.
3. **Width:** `440`, **Height:** `140`.
4. **OK** y colócalo donde quieras (arriba a la izquierda queda bien).

### 6. Comprueba que funciona

Con OBS abierto, abre `contador-muertes.html` con doble clic en el navegador. Abajo a la izquierda verás el estado (los mensajes salen en inglés):

| Mensaje | Qué significa |
|---|---|
| 🟢 **OBS connected · Stream Deck ready** | Todo bien |
| 🟢 **Detection active · DEAD 0.00 …** | Ya está viendo tu juego |
| 🔴 **add your password to the file** | Falta la contraseña: repite el paso 3 |
| 🔴 **wrong password** | La contraseña no coincide; cópiala de nuevo (paso 2) |
| 🔴 **OBS not connected** | El WebSocket no está activo (paso 2) |
| 🔴 **can't find the source "GAME"** | Revisa el nombre (paso 4) |

Cuando todo esté en verde, **cierra esa pestaña**. Solo era para revisar, y si la dejas abierta gasta recursos.

---

## Botones manuales (opcional)

Por si la detección no agarra una muerte o cuenta de más.

**En OBS**, crea en tu escena del juego estas fuentes **Text (GDI+)** con el texto **vacío** (no se ven en pantalla):

| Nombre de la fuente | Qué hace |
|---|---|
| `ADD_DEATH` | +1 al total |
| `REMOVE_DEATH` | −1 al total |
| `NEW_BOSS` | Pone *Jefe actual* en 0 |
| `RESET_DEATHS` | Pone todo en 0 |

Cada vez que muestras u ocultas una de esas fuentes (clic en su ojito 👁), el contador hace la acción.

Los botones manuales solo cambian el **total**. *Jefe actual* solo cambia con la detección automática y con `NEW_BOSS`.

---

## 🎛️ Configurar el Stream Deck (opcional)

Si tienes Stream Deck, puedes convertir esas fuentes en botones físicos. **No necesitas** el plugin de Twitch ni otros plugins de contador: solo el plugin de OBS de Elgato.

### 1. Instala el plugin OBS Studio

1. Revisa que tu **app de Stream Deck sea la versión 6.9 o más nueva** (actualízala si el plugin no se instala).
2. Instala el plugin oficial **OBS Studio** de Elgato, de cualquiera de estas formas:
   - desde el [Elgato Marketplace](https://marketplace.elgato.com/product/obs-studio-35615969-830f-45c9-ba0a-1a295bba7fec), o
   - dentro de la app de Stream Deck: pestaña **Plugins** a la derecha → busca **OBS Studio** → instala el que dice **by Elgato**.

> No uses plugins de terceros tipo "death counter" o "win/lose counter": llevan su propio número y no se comunican con este overlay.

### 2. Crea primero las fuentes en OBS

Antes de configurar los botones, crea las fuentes **Text (GDI+)** vacías de la tabla de arriba (`ADD_DEATH`, `REMOVE_DEATH`, `NEW_BOSS`, `RESET_DEATHS`) en la **misma escena** donde juegas. Si no existen, no aparecen en la lista del Stream Deck.

### 3. Agrega los botones

1. En la app de Stream Deck, pestaña **Keys**, busca **obs**.
2. Dentro de **OBS Studio**, busca **Source Visibility**.
3. Arrástrala a un botón vacío (tip: usa una página libre, como la 2, para no pisar tus otros botones).
4. Haz clic en el botón y configúralo abajo:
   - **Scene:** la escena donde juegas (por ejemplo `GAMING`)
   - **Source:** `ADD_DEATH`
   - **Title:** algo como `+1`
5. La primera vez puede pedirte conectarse a OBS: OBS tiene que estar **abierto**, y la contraseña es la misma del paso 2.
6. Repite con los otros botones:

| Título del botón | Fuente |
|---|---|
| `+1` | `ADD_DEATH` |
| `-1` | `REMOVE_DEATH` |
| `JEFE` | `NEW_BOSS` |
| `RESET` | `RESET_DEATHS` |

### 4. Pruébalo

Aprieta el botón `+1` con OBS abierto. El ojito de `ADD_DEATH` debería cambiar en OBS y el contador subir 1.

- **El ojito cambia pero el contador no** → el overlay no está conectado a OBS. Abre el HTML en el navegador y revisa el mensaje de estado (**Instalación → paso 6**).
- **El ojito no cambia** → revisa que el botón tenga bien elegidas la **Scene** y la **Source**.

> Pon `RESET` en una esquina del Stream Deck: borra todo y es fácil apretarlo sin querer.

---

## Cómo funciona

Unas dos veces por segundo, el contador le pide a OBS una captura pequeña de la fuente `GAME` y la revisa **en memoria**. No se guarda nada en tu disco y casi no usa CPU ni RAM.

| Qué busca | Dónde |
|---|---|
| La palabra roja **DEAD** | Abajo al centro |
| El marco de la **barra de vida del jefe** | Arriba al centro (funciona aunque al jefe le quede poca vida) |
| El **kanji blanco** de jefe derrotado | Centro-derecha |

Para evitar falsos positivos, cada cosa tiene que verse en 2 capturas seguidas, y cada muerte se cuenta una sola vez.

Las muertes cuentan para **JEFE ACTUAL** solo si la barra del jefe estaba en pantalla (o se vio en los últimos 12 segundos). Las muertes normales solo suman al total.

---

## Preguntas frecuentes

**¿Se reinicia al cerrar OBS?**
No. Los números se guardan aunque cierres OBS o recargues la fuente. Para empezar de cero usa `RESET_DEATHS`.

**Le di *Refresh* y no volvió a 0.**
Es normal: el *Refresh* no borra los números. Usa `RESET_DEATHS`.

**Quiero empezar con las muertes que ya llevaba.**
Aprieta `ADD_DEATH` las veces que necesites (fuera de una pelea de jefe).

**No detecta las muertes.**
Revisa que el juego llene toda la fuente `GAME` en 16:9, sin bordes negros. Abre el HTML en el navegador y fíjate qué número marca *DEAD* cuando mueres: debería pasar de 0.70.

**Abrí el HTML en el navegador y tiene otros números.**
El navegador y OBS guardan cada uno sus propios números. El que sale en tu stream es el de OBS.

**Me sale *HTTP ERROR 426* al abrir `http://127.0.0.1:4455`.**
Es buena señal: el WebSocket está encendido. Si en cambio sale *ERR_CONNECTION_REFUSED*, desmarca y vuelve a marcar *Enable WebSocket server*, dale *Apply* y reinicia OBS.

---

<a id="personalizar"></a>

## Personalizar

En el bloque de configuración del principio del HTML puedes cambiar:

- `LABEL_TOTAL` y `LABEL_BOSS`: el texto del overlay (paso 3).
- `SOURCE_GAME`: el nombre de la fuente del juego, si no quieres llamarla `GAME`.
- `SOURCE_ADD`, `SOURCE_REMOVE`, `SOURCE_RESET`, `SOURCE_NEW_BOSS`: los nombres de las fuentes de los botones. Por ejemplo, si prefieres nombres en español, cambia `'ADD_DEATH'` por `'MUERTE'` y crea la fuente con ese mismo nombre en OBS.

Para probar sin OBS, abre el HTML en el navegador y usa el teclado: `+` / `-` suman y restan, `J` reinicia el jefe, `B` simula la barra de un jefe y `0` reinicia todo.

---

## Licencia

MIT: úsalo, modifícalo y compártelo libremente. Si lo usas en tu stream, ¡un shoutout a [MistyWaifu](https://twitch.tv/mistywaifu) se agradece! 💜
