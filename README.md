# ZIntercept

Proxy de interceptación de escritorio para **pruebas de seguridad web**. Solo ciberseguridad, nada más.

> **Aviso legal.** ZIntercept está pensada para auditar sistemas **propios** o con **autorización por escrito** del titular. Interceptar tráfico de terceros sin permiso es ilegal. El autor no se responsabiliza del mal uso.

## Estado: Fase 2 completa, Fase 3 en marcha

| Fase | Contenido | Estado |
|------|-----------|--------|
| 1 | Proxy MITM + CA propia + historial + visor de petición/respuesta | ✅ |
| 2 | Repetidor ✅ · Fuzzer ✅ · Match & Replace ✅ · Intercept (pausar en vivo) pendiente | parcial |
| 3 | Hallazgos pasivos ✅ · Comparador ✅ · **Scope seguro** pendiente | parcial |
| 4 | Informe exportable en español | pendiente |

Pensada para ser **más fácil de empezar a usar que Burp** y sin los límites artificiales de la edición gratuita: el Fuzzer no está capado de fábrica (solo por la concurrencia que tú fijes), y el navegador de pruebas no exige tocar el almacén de certificados del sistema.

## Descargar e instalar (usuario final)

ZIntercept se distribuye como instalador normal, igual que cualquier programa de escritorio (al estilo VirtualBox): se descarga, se ejecuta el instalador y listo. **No hace falta tener Python instalado**: el intérprete y el motor (mitmproxy) van dentro del propio instalador.

- **Windows (x64):** `ZIntercept-Setup-x64.exe`. También funciona en Windows en ARM64 gracias a la emulación x64 integrada en Windows 11 — no hay (todavía) una versión nativa ARM64, ver nota abajo.
- **Linux (Debian, Ubuntu y derivadas):** `zintercept-amd64.deb`. Se instala con `sudo apt install ./zintercept-amd64.deb`.

Descárgalos desde la web (https://ivanzv-z.github.io/#descargar) o desde *Releases*. Los nombres de archivo no llevan versión a propósito: así el enlace `releases/latest/download/...` siempre apunta a la última.

El código de la aplicación va empaquetado (`.asar`), no como una carpeta de archivos suelta — esto es un empaquetado **básico**: evita que alguien copie o modifique la app con un simple copiar-pegar de la carpeta, pero no es un sistema de licencias ni DRM, así que no impide una copia deliberada por alguien con conocimientos.

**Nota sobre Windows ARM64:** mitmproxy depende de `aioquic` (para HTTP/3), y la versión que mitmproxy exige actualmente no tiene build para `win_arm64` en PyPI (sí la tiene a partir de una versión más reciente de aioquic que mitmproxy aún no admite). Por eso no se genera un instalador nativo ARM64 con Python embebido; en su lugar, el instalador x64 corre perfectamente en Windows ARM64 por la emulación x64 (Prism) que trae el propio Windows. En cuanto mitmproxy actualice ese límite de versión, añadir el build ARM64 nativo es sencillo.

## Compilar los instaladores (para desarrolladores)

```bash
npm install
npm run build          # compila la UI (Vite)
npm run dist:linux      # .deb (se puede generar desde Linux)
npm run dist:win        # instalador NSIS para Windows x64 (en Linux necesita wine64 + wine32)
```

El runtime de Python ya recortado y con mitmproxy instalado vive en `engine/runtime/<linux|win>-x64/` y se empaqueta automáticamente dentro del instalador (`extraResources` en `package.json`); el usuario final nunca lo ve ni lo toca. Para regenerarlo desde cero (por ejemplo al subir de versión mitmproxy o Python): `scripts/bundle-engine.sh linux` y `scripts/bundle-engine.sh win`.

Para builds reproducibles sin depender de wine (necesario para el instalador de Windows desde Linux), usa el flujo de GitHub Actions en `.github/workflows/build.yml`: se lanza desde la pestaña "Actions" del repositorio (o al crear una etiqueta `vX.Y.Z`) y genera los tres instaladores en máquinas nativas de cada sistema.

## Desarrollo

- Node.js 20+
- Para trabajar sobre el motor sin usar el runtime embebido: Python 3.10+

```bash
npm install
npm run engine:setup   # crea engine/.venv e instala mitmproxy + websockets (solo para desarrollo)
npm run dev            # abre ZIntercept en modo desarrollo
```

Para una ejecución sin Vite en caliente: `npm start`.

## Primer uso

1. Arranca ZIntercept. El estado arriba a la derecha debe decir **motor listo**. La escucha arranca **detenida**: no se registra nada hasta que tú enciendes el interruptor.
2. Pulsa **abrir navegador de pruebas** (o la tecla `B`). ZIntercept lanza Chrome, Edge, Brave o Chromium ya configurado: proxy puesto y certificado aceptado, **sin tocar nada del sistema**. La primera pestaña es la web de Iván Z (va directa, sin pasar por el proxy, así que no ensucia el historial); navega desde ahí a la web que quieras auditar y **enciende la escucha** (interruptor *detenido / escuchando* o tecla `P`).
3. Navega. Cada petición, también las HTTPS, aparece en el historial; haz clic para ver cabeceras y cuerpo.

**Pausar la escucha:** el botón **escuchando / en pausa** (junto al estado, también con la tecla `P`) deja el proxy funcionando pero deja de registrar peticiones mientras está en pausa — útil para no llenar el historial de ruido mientras navegas hasta la web que quieres auditar, y activarlo justo ahí.

**Cómo funciona el navegador de pruebas:** usa un perfil aislado (`userData/browser-profile`), no el de tu navegador personal. La CA de ZIntercept se acepta solo en esa instancia, por su huella SPKI (`--ignore-certificate-errors-spki-list`), y el tráfico a `localhost` también pasa por el proxy. Chromium puede mostrar una barra amarilla de "opción de línea de comandos no compatible": es normal.

**Otras herramientas** (curl, apps móviles, otro navegador): configuración manual. El botón de la barra copia la dirección del proxy y **certificado CA** abre la carpeta con `mitmproxy-ca-cert.pem`. La CA es propia de ZIntercept (`userData/mitmproxy`), no la compartida de mitmproxy. No la instales en el almacén de confianza de tu sistema si no es imprescindible: quien tenga su clave privada podría interceptar tu tráfico.

**Modo oscuro / claro:** arranca siempre en oscuro (marca de ZIntercept). El icono de sol/luna de la barra superior cambia a claro y se recuerda entre sesiones (se guarda en el propio navegador de la app, no en la nube).

## Atajos

Pulsa el icono de ayuda (`?`) en la barra superior para ver esta lista dentro de la propia app.

| Tecla | Acción |
|-------|--------|
| `b` | Abrir el navegador de pruebas |
| `p` | Pausar/reanudar la escucha (el tráfico sigue pasando, solo deja de registrarse) |
| `?` | Mostrar/ocultar la ayuda de atajos |
| `/` | Enfocar el filtro |
| `j` / `k` (o ↓ / ↑) | Siguiente / anterior petición |
| `Esc` | Limpiar el filtro / cerrar la ayuda |
| `Ctrl+L` | Limpiar el historial al instante |
| `Ctrl+Enter` | Enviar la petición del Repetidor o del Fuzzer |

El botón **limpiar** (clic de ratón) pide confirmación antes de borrar nada; `Ctrl+L` es el atajo deliberado y borra al instante.

Además: **copiar cURL** reconstruye la petición seleccionada como comando.

## Estructura: historial fijo + pestañas de trabajo

El historial queda siempre visible en una columna fija a la izquierda (con su propio buscador), trabajes en lo que trabajes. A la derecha, el espacio de trabajo funciona con **pestañas al estilo del navegador**: la pestaña **Detalle** es fija (muestra la petición que tengas seleccionada en el historial) y puedes tener a la vez, cada una en su propia pestaña cerrable, varias peticiones del **Repetidor**, del **Fuzzer** y del **Comparador** abiertas — sin perder nunca el historial de vista.

- Los botones **repetidor** / **fuzzer** del detalle abren una pestaña nueva con esa petición ya cargada.
- Los botones **+ repetidor** / **+ fuzzer** / **+ comparador** de la barra de pestañas abren una pestaña en blanco de cada tipo.
- Doble clic en una pestaña para renombrarla (por ejemplo, "login admin"); la `×` la cierra.
- El icono de duplicar (en Repetidor) clona la pestaña (destino y petición) sin la respuesta anterior.

## Repetidor

Cada pestaña de Repetidor trae el destino (host, puerto, TLS) y la petición en formato crudo (editable).

- Edita el método, la ruta, las cabeceras o el cuerpo directamente en el editor; los botones **GET/POST/PUT/PATCH/DELETE** cambian el método sin tocar el resto.
- **Enviar** (o `Ctrl+Enter`) la manda **directamente al objetivo, sin pasar por el proxy** — igual que el Repeater de Burp.
- El `Content-Length` se recalcula siempre al enviar, para que coincida con el cuerpo que has editado.
- El certificado TLS del objetivo no se valida (uso de pentesting contra objetivos propios).
- El icono de copiar, en la petición y en la respuesta, copia ese texto al portapapeles.

## Fuzzer

Cada pestaña de Fuzzer es independiente: puedes tener varios lanzamientos corriendo o terminados a la vez, cada uno en su pestaña. Marca uno o varios puntos de la petición con `§§` (en la ruta, una cabecera o el cuerpo) y escribe o pega una lista de valores, uno por línea.

- Sin límite artificial de velocidad: el único tope es la **concurrencia** que tú fijes (de 1 a 30 a la vez). La edición gratuita de Burp capa el Intruder a propósito; aquí no hay ese capado, solo el sentido común de no tumbar el objetivo.
- Trae listas incorporadas (usuarios, rutas y contraseñas comunes) para no depender de otra herramienta solo para tener un diccionario con el que empezar.
- Los resultados se pueden ordenar por estado, tamaño o tiempo con un clic en la cabecera de la columna.
- Doble clic en un resultado lo abre en una pestaña nueva del **Repetidor** con ese payload concreto ya puesto, para inspeccionar la respuesta completa.
- Tope práctico de 1000 payloads por lanzamiento (no por plan: evita tumbar el objetivo por error con una lista pegada por accidente).
- Igual que el Repetidor, va directo al objetivo sin pasar por el proxy ni validar su certificado TLS.

## Comparador

Elige dos peticiones ya capturadas en el historial y compara su petición o su respuesta línea a línea. Las líneas añadidas y quitadas se marcan con `+`/`−` (texto, no solo color) para que sean legibles también si no distingues bien los colores.

## Hallazgos pasivos

En el detalle de cualquier petición con respuesta, la pestaña **Hallazgos** aplica unas heurísticas sencillas: cabeceras de seguridad que faltan (HSTS, CSP, X-Content-Type-Options…), cookies sin `Secure`/`HttpOnly`/`SameSite`, CORS mal configurado, versiones de servidor expuestas, y patrones de credenciales o errores de servidor en el cuerpo (JWT, claves de AWS, trazas de Python/PHP, errores SQL…).

Son pistas por patrones conocidos, **no un escáner activo ni un análisis certero**: no se manda ninguna petición nueva, solo se mira lo que ya pasó por el proxy. Verifica cada hallazgo a mano antes de darlo por bueno o de reportarlo.

## Match & Replace

El icono de reglas en la barra superior abre un panel para definir sustituciones que se aplican **en vivo a todo el tráfico que pasa por el proxy** (no al Repetidor ni al Fuzzer, que van directos al objetivo). Por ejemplo: mantener siempre puesta una cabecera de autorización, o reescribir un valor en cada respuesta. Cada regla actúa sobre la petición o la respuesta, sobre una cabecera o el cuerpo, con texto literal o expresión regular.

## Arquitectura

```
Electron (main.js)  ──arranca──►  mitmdump + engine/zintercept_addon.py
      │                                   │  proxy en 127.0.0.1:8080
      └─ preload.js ─► UI React           │  WebSocket en 127.0.0.1:<puerto aleatorio>
                         └────── ws + token ───────┘
```

- El motor reutiliza mitmproxy (TLS, HTTP/2, WebSockets resueltos); ZIntercept aporta la interfaz y el flujo de trabajo.
- El WebSocket solo escucha en `127.0.0.1` y exige un **token aleatorio de 256 bits** generado en cada arranque; sin él se cierra la conexión (código 4401).
- La UI corre con `contextIsolation`, `sandbox` y una CSP estricta; el contenido capturado se muestra siempre como texto, nunca como HTML.
- El historial vive **solo en memoria** (máx. 5000 peticiones, cuerpos de hasta 1 MB). Puede contener credenciales del objetivo: trátalo como dato sensible.

## Protocolo del motor (para ampliar)

Mensajes JSON por WebSocket. Primer mensaje obligatorio: `{"type":"auth","token":"..."}`.

- `get {id}` → `detail` con petición y respuesta completas
- `clear` → vacía el historial (`cleared`)
- `repeat` → `repeated` (Repetidor, directo al objetivo)
- `fuzz_start`/`fuzz_cancel` → `fuzz_result` por payload + `fuzz_done` al terminar (Fuzzer, directo al objetivo)
- `rules_set` → `rules_ack` (Match & Replace, aplicado al tráfico que pasa por el proxy)
- el motor emite `ready` (incluye las reglas activas), `flow` (resumen de cada petición) y `error`

Detalle completo en la cabecera de `engine/zintercept_addon.py`.

## Estructura

```
engine/zintercept_addon.py     motor: addon de mitmproxy + WebSocket
engine/requirements.txt
engine/runtime/           Python portable + mitmproxy ya instalado (uno por plataforma); se genera con scripts/bundle-engine.sh y se empaqueta dentro del instalador
electron/main.js          ventana, arranque/parada del motor
electron/browser.js       detección del navegador, huella SPKI y argumentos
electron/preload.js       puente mínimo UI ↔ main
electron/promo.html       página de bienvenida (primera pestaña del navegador de pruebas)
src/                      UI React (App, engine hook, repetidor, fuzzer, comparador (diff.js), hallazgos (findings.js), formato, estilos)
scripts/setup-engine.js   instalador del motor para desarrollo (venv local)
scripts/bundle-engine.sh  genera el runtime portable para los instaladores
.github/workflows/build.yml  CI: genera los tres instaladores en máquinas nativas
```

## Sugerencias y fallos

- **Sugerir una mejora** (funciones o interfaz): botón con el bocadillo en la barra superior de la app, o pestaña *Issues* → *Sugerir una mejora*. Es **público**: no pegues tráfico, cookies ni datos de clientes.
- **Fallos de seguridad**: aviso **privado** en *Security* → *Report a vulnerability*. Ver `SECURITY.md`.

ZIntercept no envía nada por su cuenta: esos botones solo abren tu navegador.

---

Hecho por **Iván Z** · https://ivanzv-z.github.io
