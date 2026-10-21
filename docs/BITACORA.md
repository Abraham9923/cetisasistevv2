# Bitácora de cambios — CetisAsiste

Aquí se anota **todo lo que cambia en el proyecto, incluso lo pequeño**, en orden: qué se
hizo, en qué archivos y cómo se comprobó. Lo más reciente va arriba.

Para el detalle de la base de datos ver [BASE_DE_DATOS.md](BASE_DE_DATOS.md); para el
*porqué* de cada decisión, los ADR en [adr/](adr/).

> **Cómo agregar una entrada:** copia el formato de abajo (fecha, número, título, *Qué*,
> *Archivos*, *Cómo se comprobó*) y ponla arriba de la más reciente.

---

## 20 de octubre de 2026

### 20-04. Perfiles de la consola actualizados y reglas nuevas publicadas

- **Pedido del usuario:** actualizar las reglas y ajustar los roles en la consola (con su
  nota `PARA_MANANA.md`).
- **Qué se hizo:**
  - `git pull` en `main` para traer el PR #1 ya fusionado (la copia local iba 4 commits atrás
    y no tenía cambios propios).
  - **Perfiles en Firestore** (consola, desde Chrome), iguales a `firebase/roles.json`:
    - `roles/alumno`: `inicioDeSesion` = `no_control`; `camposDelUsuario[0]` = `noControl`.
    - `roles/prefecto`: `descripcion` con "No. de control"; `pantallas[2]` = "Captura por No.
      de control"; `permisos[1]` = `capturar_no_control`.
    - `roles/enfermeria`: `plataformaConfirmada` = `true` (ADR-011).
  - **Reglas:** el contenido completo de `firebase/firestore.rules` (237 líneas, con
    `noControl` y los avisos de ADR-010) quedó **pegado** en el editor de reglas de la consola,
    idéntico al archivo. El sistema de permisos no dejó a Claude presionar "Publish" (cambia la
    seguridad de la base); la pestaña se recargó y se perdió lo pegado, así que se volvió a pegar
    y **el usuario lo publicó**. Después se recargó la página y se comprobó que las reglas
    publicadas son las nuevas (238 líneas, con `noControl` y `avisoValido`, sin `matricula`).
- **Un susto, sin daño:** al editar `permisos[1]` de `roles/prefecto` se abrió por error el
  diálogo "Delete Data"; se canceló y no se borró nada (se comprobó que el campo seguía).
- **Archivos:** `firebase/roles.json` (nota de que Enfermería ya está en la consola),
  `docs/PENDIENTES.md`.
- **Cómo se comprobó:** se leyó cada documento de `roles` en la consola después de guardarlo
  y coincide con `roles.json`. Las reglas pegadas se compararon con el archivo (iguales).
- **Falta:** crear el índice de `registros`
  (`fecha` ↑, `alerta` ↑, `fechaHora` ↓), instalar la app nueva y crear al alumno de prueba
  con 14 dígitos.

### 20-03. Primera versión de la web de Control Escolar y Enfermería (ADR-011)

- **Pedido del usuario:** "empieza a crear el apartado web, son 2 usuarios".
- **Qué se hizo:**
  - Carpeta nueva **`web/`**: una página con HTML, CSS y JavaScript **sin compilar**, con la
    librería de Firebase 12.19.0 desde `gstatic.com`. Colores, letra y logo de la app.
  - **Acceso:** usuario o correo y contraseña, como la app; solo entran `control_escolar` y
    `enfermeria`; los demás ven "Esta cuenta se usa en la app…". Cambio de contraseña
    obligatorio si toca. La cuenta se escucha en vivo.
  - **Enfermería:** alumnos por grupo y búsqueda, perfil, **pase de salida por salud**,
    **activar/desactivar por salud** y datos de salud.
  - **Control Escolar:** lo mismo para ver alumnos, **activar/desactivar por pago**, **nuevo
    estatus**, datos personales, entradas y salidas del alumno, **Entradas y salidas del
    día** (en vivo, "solo alertas") y **Avisos al padre** (guarda
    `configuracion/plantel.avisarAlPadreEstatus` y `avisarAlPadreAlertas`).
  - Cada acción se guarda en un lote con los **avisos a los padres** (ADR-010), firmada con el
    uid de la sesión. Notificaciones propias con contador en el menú.
  - **Hosting:** `firebase/firebase.json` publica `web/` y configura los emuladores de
    Authentication, Firestore y Hosting.
  - **Emulador:** `firebase/emulador/datos-de-prueba.mjs` carga cuentas y datos "Prueba".
  - La app ahora dice "El perfil de … se usa desde la web de CetisAsiste, en una computadora"
    (antes: "que todavía no está lista"). `firebase/roles.json`: Enfermería confirmada en web.
- **Archivos:** `web/` (nuevo: `index.html`, `css/estilos.css`, `js/*.js`,
  `js/pantallas/*.js`, `pruebas/`, `README.md`, `package.json`, fuentes y logo copiados de la
  app), `firebase/firebase.json`, `firebase/emulador/datos-de-prueba.mjs` (nuevo),
  `firebase/roles.json`, `.gitignore`, `app/src/main/res/values/strings.xml`,
  `ui/CetisApp.kt` (comentario). Documentos: ADR-011 (nuevo), BASE_DE_DATOS.md (§2, §4.9,
  §10), PENDIENTES.md (§1, §8), TRASPASO_AGENTE.md.
- **Cómo se comprobó:** `npm test` en `web/` pasa **10 pruebas** (`avisos.js` y
  `formatos.js`). Con los emuladores (reglas reales) y Chromium, `npm run recorrido` pasa los
  **18 pasos**: el padre no entra; Enfermería da un pase y desactiva por salud (en la base
  quedan el estatus firmado y 2 avisos al padre); el cambio de contraseña obligatorio
  funciona; Control Escolar activa por pago, pone un castigo (aviso al padre), ve 3 entradas
  con 2 alertas, guarda los avisos al padre y marca sus notificaciones como leídas; en un
  teléfono de 390 px no hay scroll horizontal. Se revisaron las capturas (se corrigió el
  contraste de la tarjeta guinda y el menú del teléfono). La app vuelve a compilar con sus 30
  pruebas. **No se probó con la base real** (falta registrar la web en Firebase y publicarla).

### 20-02. Revisión de la arquitectura de todo el sistema

- **Pedido del usuario:** "analiza la arquitectura del proyecto, dime qué le mejorarías".
- **Qué se hizo:** documento nuevo [ARQUITECTURA.md](ARQUITECTURA.md) con cómo está el
  sistema, lo que conviene conservar y **qué mejorar, por prioridad**: el escaneo sin señal
  se queda en "Guardando…" (A1), la hora la pone el teléfono (A2), un solo proyecto para
  pruebas y plantel (A3), el plan (A4), aviso de privacidad y Analytics (B1), custom claims
  (B2), App Check (B3), respaldos (B4), un script de administración con el Admin SDK para
  importar alumnos y crear cuentas (E1), GitHub Actions (D1) y sacar `.idea/` de git (D2).
  ADR-008 ahora enlaza a esta revisión (§3.4 dice qué avanzó). **No se cambió código**: el
  usuario decide qué se aplica.
- **Archivos:** `docs/ARQUITECTURA.md` (nuevo), `docs/adr/ADR-008-mejoras-a-la-estructura.md`,
  `docs/PENDIENTES.md` (§12).
- **Cómo se comprobó:** cada punto se revisó en el código (por ejemplo, `lote.commit().await()`
  en `CetisRepositorio.registrarEscaneo` para A1 y `Timestamp.now()` para A2).

### 20-01. Notificaciones, alertas y avisos para padres y prefectos (ADR-010)

- **Pedido del usuario:** "encárgate de la gestión del sistema de notificaciones, alertas y
  avisos, tanto para los padres como las alertas para los prefectos".
- **Qué se hizo:**
  - **Un solo buzón** por cuenta (`usuarios/{uid}/notificaciones`) para todos los avisos.
    Nuevo `datos/Avisos.kt` (a quién se avisa y cómo se arma el aviso, sin Firebase) y
    `TipoAviso` en `Modelos.kt`. Cada aviso trae ahora `creadaPor` y `venceEn` (+60 días).
  - **Padre:** además de cada entrada y salida, recibe un aviso cuando el profesor pone falta,
    reporte, llamada de atención o castigo (con la descripción), y cuando al escanear a su
    hijo hay alerta de "inactivo" o "salida fuera de horario". El plantel lo cambia con
    `configuracion/plantel.avisarAlPadreEstatus` y `avisarAlPadreAlertas` (si no existen, se
    usa lo de por omisión). Antes, con alerta no se le avisaba (ADR-009 punto 6).
  - **Campana en vivo (los cuatro roles):** el punto rojo sale **solo** si hay avisos sin leer
    (antes salía siempre; nuevo ícono `ic_bell` sin punto); el menú ☰ muestra cuántos ("3",
    "9+") con el nuevo estilo `Cetis.Contador`. La pantalla Notificaciones se escucha en
    tiempo real; "Marcar todas como leídas" solo sale si hay algo sin leer. Íconos por tipo
    (alerta en rojo, estatus, estado, grupo que salió).
  - **Prefecto:** en el Inicio, "Últimos escaneos" (8, en vivo, en rojo los que tuvieron
    alerta) y el renglón "Alertas de hoy" con cuántas van; pantalla nueva **Alertas de hoy**
    (`prefecto_alertas.xml`, también en el menú ☰); el resultado del escaneo dice **por qué**
    saltó la alerta; en Notificaciones ve los grupos que salieron hoy.
  - **Profesor:** "Cambiar estatus" avisa si el padre recibirá el aviso; el estatus y los
    avisos se guardan en un mismo lote.
  - **Reglas:** el buzón acepta solo avisos con la forma correcta y firmados; prefecto y
    Enfermería solo avisan a los padres del alumno, el profesor solo a los de sus grupos.
    Estatus (`asignadoPor`) y cambios de estado (`por`) también deben ir firmados.
  - **Pruebas de las reglas** con el emulador: `firebase/pruebas-reglas/` (21 pruebas; 6 fallan
    con las reglas anteriores). `firebase/firebase.json` ahora configura el emulador.
  - **Índice nuevo:** `registros` (`fecha`, `alerta`, `fechaHora`) para "Alertas de hoy".
  - **Menos lecturas:** `configuracion/plantel` se guarda 5 minutos en memoria (antes se leía
    dos veces en cada escaneo).
- **Archivos:** `datos/Avisos.kt` (nuevo), `datos/Modelos.kt`, `datos/CetisRepositorio.kt`,
  `datos/Escaneo.kt` (`castigoVigente`), `ui/comun/Carga.kt` (`sinLeerEnVivo`),
  `ui/comun/PantallasComunes.kt`, `ui/comun/VistasComunes.kt` (`marcarCampana`, contador del
  menú), `ui/comun/VariablesComunes.kt`, `ui/alumno/AlumnoApp.kt`, `ui/padres/PadresApp.kt`,
  `ui/prefecto/PrefectoApp.kt`, `ui/prefecto/PrefectoVistas.kt`,
  `ui/prefecto/EscaneoPrefecto.kt`, `ui/profesor/ProfesorApp.kt`,
  `ui/profesor/ProfesorVistas.kt`; `res/layout/prefecto_alertas.xml` (nuevo),
  `prefecto_inicio.xml`, `prefecto_resultado.xml`, `profesor_cambiar_estatus.xml`,
  `comun_menu.xml`; `res/drawable/ic_bell.xml` y `shape_contador.xml` (nuevos);
  `strings_prefecto.xml`, `strings_profesor.xml`, `strings_comun.xml`, `styles_cetis.xml`;
  `app/src/test/.../AvisosTest.kt` (nuevo); `firebase/firestore.rules`,
  `firebase/firestore.indexes.json`, `firebase/firebase.json`, `firebase/pruebas-reglas/`
  (nuevo). Documentos: ADR-010 (nuevo), BASE_DE_DATOS.md (§4.1, §4.8, §6, §7, §10),
  MANUAL_TECNICO.md, VISTAS_POR_ROL.md, PENDIENTES.md.
- **Cómo se comprobó:** en la sesión en la nube (Linux, con el SDK de Android 37):
  `gradlew :app:assembleDebug` compila; `:app:testDebugUnitTest` pasa las **30 pruebas** (20
  de `EscaneoTest`, 9 de `AvisosTest` y la de ejemplo); `:app:lintDebug` sin errores. Las
  reglas pasan sus **21 pruebas** en el emulador de Firestore. **No se probó en el teléfono**
  (no hay teléfono en la nube): lo prueba el usuario (PENDIENTES §10, "Avisos").
- **Falta (lo hace el usuario en la consola):** publicar las reglas, crear el índice e
  instalar la app nueva al mismo tiempo (PENDIENTES §1, punto 4). Los `.docx` no se generaron
  (en la nube no hay Word): correr `md_a_word.ps1` con cada `.md` cambiado.

---

## 19 de octubre de 2026

### 19-35. No. de control de 14 dígitos, QR con el No. de control y escaneo del prefecto (ADR-009)

- **Pedido del usuario:** cambiar todo lo de "matrícula" por "No. de control" (mostró una
  credencial con 14 dígitos; **sus datos no se guardaron**, solo el formato), exigir 14
  dígitos, que el QR lleve el No. de control y empezar el escaneo.
- **Qué se hizo:**
  - **"No. de control" en todo:** textos, documentos, código, reglas y perfiles. En Firestore
    `matricula` → `noControl` (`alumnos` y `usuarios`), `medio` `matricula` → `no_control`,
    `roles/alumno.inicioDeSesion` → `no_control`. Ids de vistas y textos renombrados
    (`txt_no_control`, `field_no_control`, `prefecto_acceso_no_control`…). No se tocaron el
    código viejo de `Main.kt`, ni las entradas anteriores de esta bitácora, ni la copia del
    documento original de roles.
  - **14 dígitos:** `NoControl.esValido`; al iniciar sesión con números que no sean 14 sale
    "El No. de control tiene 14 dígitos." (nuevo `MensajeLogin.NoControlInvalido`).
  - **QR:** `NoControl.contenidoQr` = `CETIS61:` + No. de control (alumno y padre lo ven igual).
    Se quitó `codigoQr` del modelo (`AlumnoDoc` ignora campos viejos).
  - **Escaneo:** `datos/Escaneo.kt` (qué alerta toca), `CetisRepositorio.datosDelEscaneo` y
    `registrarEscaneo` (un lote: registro siempre; sin alerta, ubicación y aviso a los padres),
    `ui/prefecto/EscaneoPrefecto.kt` (escáner, captura y resultado),
    `prefecto_escaner.xml`, `prefecto_captura.xml`, `prefecto_resultado.xml`, la opción
    "Escanear" en el menú ☰ del prefecto y las tarjetas del Inicio ya funcionan.
  - **Librería nueva:** `com.journeyapps:zxing-android-embedded:4.3.0` (cámara) y permiso de
    cámara en el manifiesto (no obligatoria).
  - **Pruebas:** `app/src/test/.../EscaneoTest.kt`, 20 pruebas de alertas y No. de control.
  - Recursos nuevos: `ic_x_circle`, `img_marco_escaner`, `shape_segmento`,
    `shape_boton_borde_claro`, `shape_input_activo`, `shape_resultado_exito`,
    `shape_resultado_alerta`, `shape_circulo_peligro`, color `cetis_escaner_fondo`, estilos
    `Cetis.Segmento`, `Cetis.Tecla`, `Cetis.FilaDeTeclas` y textos en `strings_prefecto.xml`.
- **Un tropiezo, corregido:** al actualizar documentos, un script dañó `referencias/FIGMA.md`,
  `ADR-002` y `ADR-004` (cambió caracteres sueltos). Se restauraron desde la copia de respaldo
  de la sesión y se volvieron a aplicar los cambios. También se corrigió "mandal No. de
  control" (debía ser "manda el No. de control") en ADR-002.
- **Archivos:** los de arriba; `datos/Modelos.kt` (`NoControl`, `Medio`, `TipoRegistro`,
  `Ubicacion`, `Alerta`), `datos/CetisRepositorio.kt`, `ui/CetisApp.kt`,
  `ui/alumno/AlumnoApp.kt`, `ui/padres/PadresApp.kt`, `ui/prefecto/PrefectoApp.kt`,
  `ui/prefecto/PrefectoVistas.kt`, `firebase/firestore.rules`, `firebase/roles.json`,
  `gradle/libs.versions.toml`, `app/build.gradle.kts`, `AndroidManifest.xml`. Documentos:
  ADR-009 (nuevo), `DECISIONES.md` (sección M; J6 y K6 reemplazadas), `BASE_DE_DATOS.md`,
  `MANUAL_TECNICO.md`, `VISTAS_POR_ROL.md`, `VISTAS_ALUMNO.md`, `TRASPASO_AGENTE.md`,
  `PENDIENTES.md`, `FIGMA.md` y notas en ADR-002, 004 y 005.
- **Cómo se comprobó:** `gradlew :app:assembleDebug` compila sin advertencias,
  `:app:lintDebug` sin errores y `:app:testDebugUnitTest` pasa las 20 pruebas. **No se probó
  en el teléfono**: el usuario lo prueba. **Faltan** (los hace el usuario en la consola):
  publicar `firebase/firestore.rules`, actualizar `roles/alumno` y `roles/prefecto` y volver a
  crear al alumno de prueba con 14 dígitos (BASE_DE_DATOS §10.1).

### 19-34. Notificaciones también en el menú ☰

- **Pedido del usuario:** poner la campana de notificaciones en el menú desplegable, sin
  quitarla de la pantalla.
- **Qué se hizo:**
  - `comun_menu.xml` tiene la opción **"Notificaciones"** (ícono de campana), debajo de las
    pantallas principales y antes de "Cerrar sesión". Abre la misma pantalla que la campana de
    Inicio (`comun_notificaciones.xml`); se apila, así que "atrás" regresa a donde estabas.
  - `VistasComunes.conectarMenu` recibe `onNotificaciones`; los 4 `*App.kt` le pasan
    "abrir Notificaciones".
  - **Alumno:** su campana de Inicio no abría nada y no tenía pantalla de notificaciones.
    Ahora `AlumnoPantalla.Notificaciones` usa `NotificacionesXml` (la de todos los roles) y la
    campana y el menú la abren.
- **Archivos:** `res/layout/comun_menu.xml`, `ui/comun/VistasComunes.kt`,
  `ui/comun/PantallasComunes.kt` (comentario), `ui/alumno/AlumnoApp.kt`,
  `ui/padres/PadresApp.kt`, `ui/prefecto/PrefectoApp.kt`, `ui/profesor/ProfesorApp.kt`.
  Documentos: ADR-007, `DECISIONES.md` (L4), `VISTAS_POR_ROL.md`, `VISTAS_ALUMNO.md`,
  `MANUAL_TECNICO.md`.
- **Cómo se comprobó:** `gradlew :app:assembleDebug` compila sin advertencias y `:app:lintDebug`
  termina sin errores; se instaló en el teléfono y **el usuario lo probó: funciona**.

### 19-33. Prueba del menú ☰ en el teléfono y encabezado corregido

- **Qué se probó** (teléfono del usuario, cuenta del alumno de prueba, con su permiso): el
  Inicio ya no tiene barra inferior y el contenido termina justo arriba de la barra de gestos;
  el ☰ abre el menú con nombre, "Alumno", las 4 opciones (la de la pantalla abierta marcada)
  y "Cerrar sesión" hasta abajo; elegir "Boletín", "Horario" y "Perfil" abre esa pantalla, y
  las tres tienen ☰; el menú se cierra tocando fuera y con "atrás" (y "atrás" no cambia de
  pantalla); "Cerrar sesión" muestra "¿Cerrar sesión?" y "Cancelar" lo quita sin salir. No se
  tocó "Cerrar sesión" del diálogo, para no cerrar la sesión de prueba.
- **Qué se corrigió:** la foto del encabezado del menú quedaba pegada a la hora y la batería.
  Ahora el encabezado mide lo que necesita y el código le suma arriba el alto de la barra de
  estado de cada teléfono.
- **Archivos:** `res/layout/comun_menu.xml` (encabezado `encabezado_menu`, alto según su
  contenido), `ui/comun/VistasComunes.kt` (`mostrarMenu`).
- **Cómo se comprobó:** compila, `lint` sin errores; se instaló otra vez y en el teléfono el
  encabezado ya empieza debajo de la barra de estado. **Falta** probar con padre, prefecto y
  profesor (hacen falta sus contraseñas).

### 19-32. Revisión de la estructura del proyecto (ADR-008, propuesta)

- **Qué se hizo:** a pedido del usuario, se revisó cómo está organizado el código y se
  escribió una propuesta de mejoras en orden: borrar código muerto, ordenar carpetas, una sola
  navegación, pruebas, un `ViewModel` por rol (antes del escaneo), paquete propio (antes de
  publicar), actualizar librerías y decidir `minSdk`. **No se aplicó nada**: lo decide el
  usuario.
- **Archivos:** `docs/adr/ADR-008-mejoras-a-la-estructura.md` (nuevo); `docs/PENDIENTES.md`
  (sección 12 y paso 10 del orden sugerido); `docs/TRASPASO_AGENTE.md`.
- **Cómo se comprobó:** cada punto se revisó en el código (tamaño de los archivos, carpetas,
  dependencias, pruebas existentes y avisos de lint).

### 19-31. Menú ☰ en lugar de la barra inferior (ADR-007)

- **Pedido del usuario:** "darle uso al menú hamburguesa" y que las opciones de la barra
  inferior se agrupen en el menú desplegable; quitar la barra; el ☰ en todas las pantallas que
  la tenían, en los 4 roles; y un botón de **cerrar sesión hasta abajo del menú**.
- **Qué se hizo:**
  - **Se quitó la barra inferior** de alumno, padre, prefecto y profesor (con sus estilos
    `Cetis.Nav.*`, `Cetis.Pantalla.Columna.ConBarra`, `Cetis.Texto.Navegacion`, las medidas
    `cetis_bottom_nav_*` y `cetis_content_bottom_space`, y las formas `shape_nav`,
    `shape_nav_indicador` e `img_corner_accent_left`).
  - **Botón ☰** (estilo nuevo `Cetis.BarraSuperior.Menu`) en 12 pantallas: `alumno_inicio`,
    `alumno_boletin`, `alumno_horario`, `alumno_perfil`, `padre_inicio`, `padre_anuncios`,
    `padre_perfil`, `prefecto_inicio`, `prefecto_horarios`, `profesor_inicio`,
    `profesor_boletin`, `comun_calendario` y `comun_perfil_personal`.
  - **Menú común:** `comun_menu.xml` (nombre y rol, opciones, "Cerrar sesión" hasta abajo y
    "CETis 61 · Cd. Juárez"), `comun_item_menu.xml` (una opción), `shape_menu.xml` y estilos
    `Cetis.Menu.*`. Entra desde la izquierda sobre la capa oscura; se cierra al tocar fuera, al
    elegir una opción o con "atrás". "Cerrar sesión" abre el diálogo de confirmación de Perfil.
  - **Código:** `VistasComunes.conectarMenu` y la clase `Pestana` (reemplazan a
    `marcarPestana`); una lista `PESTANAS` en cada `*App.kt`; `CalendarioXml` y
    `PerfilPersonalXml` reciben `conectarMenu` en lugar de la barra. `AlumnoApp` pasó a usar
    `PantallaXml` como los demás.
  - **Barra de gestos:** `PantallaXml` deja abajo de cada pantalla el alto de la barra de
    gestos de Android (antes lo cubría el espacio de la barra inferior).
  - Textos nuevos: `comun_rol_alumno` y `comun_rol_padre`.
- **Archivos:** los XML de arriba; `res/values/styles_cetis.xml`, `dimens_cetis.xml`,
  `strings_comun.xml`; `ui/comun/VistasComunes.kt`, `ui/comun/PantallasComunes.kt`,
  `ui/alumno/AlumnoApp.kt`, `ui/padres/PadresApp.kt`, `ui/prefecto/PrefectoApp.kt`,
  `ui/profesor/ProfesorApp.kt`. **Borrados:** `padre_barra_inferior.xml`,
  `prefecto_barra_inferior.xml`, `profesor_barra_inferior.xml`, `shape_nav.xml`,
  `shape_nav_indicador.xml`, `img_corner_accent_left.xml` (copia de respaldo en la carpeta
  temporal de la sesión). Documentos: ADR-007 (nuevo), ADR-001 y ADR-006 (marcados como
  reemplazados), `VISTAS_POR_ROL.md`, `VISTAS_ALUMNO.md`, `MANUAL_TECNICO.md`,
  `DECISIONES.md` (sección L), `TRASPASO_AGENTE.md` y `PENDIENTES.md`.
- **Cómo se comprobó:** `gradlew :app:assembleDebug` compila sin advertencias y
  `:app:lintDebug` termina sin errores. Se instaló en el teléfono. **No se probó en pantalla**:
  el teléfono estaba en uso con otra app, así que no se tocó. Falta abrir el menú en los 4
  roles (PENDIENTES §10).

### 19-30. Los pendientes viven solo en PENDIENTES.md

- **Qué se hizo:** a pedido del usuario, las listas de pendientes de `BASE_DE_DATOS.md`
  (§11), `MANUAL_TECNICO.md` (§11) y `TRASPASO_AGENTE.md` (§8) se cambiaron por un enlace a
  `PENDIENTES.md`, para que no haya tres listas que se desactualicen. `PENDIENTES.md` lo
  explica al principio, y el traspaso lo agrega a su tabla de documentos.
- **Archivos:** `docs/BASE_DE_DATOS.md` (§11 y la receta §9.8, que enlazaba a §11),
  `docs/MANUAL_TECNICO.md`, `docs/TRASPASO_AGENTE.md`, `docs/PENDIENTES.md`,
  `docs/herramientas/unir_documentacion.ps1` (ya no reemplaza esas secciones: las fuentes
  ya enlazan a Pendientes) y los `.docx` de cada uno, más `DOCUMENTACION_COMPLETA.docx`.
- **Cómo se comprobó:** se regeneraron los Word y se revisaron en PDF las secciones
  cambiadas.

### 19-29. Documentación unida en un Word y pendientes en un documento aparte

- **Qué se hizo:** a pedido del usuario, la documentación (menos el manual técnico y Figma)
  se juntó en un solo Word, `docs/DOCUMENTACION_COMPLETA.docx`, con portada, índice, un
  capítulo por documento y números de página. Los pendientes de todos los documentos se
  juntaron, sin repetir, en `docs/PENDIENTES.md`, ordenados por tema y con quién tiene que
  actuar (plantel, usuario o desarrollo).
- **Archivos:** `docs/PENDIENTES.md` y su `.docx` (nuevos);
  `docs/herramientas/unir_documentacion.ps1` (nuevo: arma el Word unido desde los `.md`; se
  vuelve a correr cuando cambien); `docs/herramientas/md_a_word.ps1` (ahora entiende
  imágenes y las guarda dentro del Word; los títulos ya no quedan solos al final de una hoja;
  una **negrita** partida en dos renglones de una lista ya sale bien; los enlaces
  `<https://…>` salen como enlace; y otro script puede usar sus funciones). Por esos
  arreglos se volvieron a generar todos los `.docx` de `docs/` y `docs/adr/`;
  `docs/adr/ADR-003-inicio-de-sesion-con-correo-interno.md` (la acción 4, pantalla "Cambiar
  contraseña", se marcó como hecha: existe desde 19-23).
- **Cómo se comprobó:** se generaron los dos Word y se revisaron en PDF, página por página
  (portada, índice, capítulos, tablas y los 6 diagramas del documento de roles).

### 19-28. Sí hay Figma de padre, prefecto y profesor (corrección)

- **Qué pasó:** el usuario pasó los enlaces de las páginas **Prefecto** (`75:508`),
  **Profesor** (`75:509`) y **Padre** (`0:1`) del mismo archivo de Figma. Al pedir la lista de
  páginas, la herramienta de Figma solo devolvía "Alumno" y "Componentes", y por eso se
  escribió (en 19-23, 19-26 y 19-27) que no existían. **Era un error.**
- **Qué se hizo:** `docs/referencias/FIGMA.md` reescrito con el enlace de cada página, la lista
  de pantallas con su `node-id` y dos tablas con las **diferencias** entre ese diseño y las
  vistas actuales de prefecto y profesor. Se corrigió la frase "no hay Figma" en
  `VISTAS_POR_ROL.md`, `DECISIONES.md` (K5), `ADR-006`, `TRASPASO_AGENTE.md`,
  `BASE_DE_DATOS.md` §11, `MANUAL_TECNICO.md` §11 y `referencias/PANTALLAS_POR_ROL.md`.
- **Lo que no se hizo (a pedido del usuario):** ajustar las vistas al Figma. Se habían
  empezado a cambiar algunas (pestañas, Inicio, Horarios, textos, íconos y colores nuevos) y
  el usuario pidió **solo guardar la información**; esos cambios se **regresaron** a como
  estaban. Queda como pendiente en el traspaso.
- **Cómo se comprobó:** `gradlew :app:assembleDebug` y `:app:lintDebug` terminan bien
  después de regresar los cambios (la app quedó igual que en 19-25).

### 19-27. Enlace de Figma guardado en la documentación

- **Qué:** `docs/referencias/FIGMA.md` (nuevo) con el enlace que dio el usuario como el Figma
  "CETis 61 - Inicio (Android)", qué páginas se encontraron en él (solo "Alumno" y
  "Componentes") y qué hacer si faltan las de los otros roles. Enlazado desde
  `TRASPASO_AGENTE.md` y `referencias/PANTALLAS_POR_ROL.md`.
- **No se volvió a revisar el archivo de Figma** en esta entrada: el usuario pidió solo
  guardar el enlace. Lo descrito es lo que se vio en la revisión del mismo día (19-23).

### 19-26. El documento "Pantallas por rol" ya está en el proyecto

- **Qué:** el usuario compartió su documento "CetisAsiste — Pantallas por rol" (7 oct, 13
  páginas, 12 tablas, 6 diagramas). Se guardó en `docs/referencias/`:
  - `PANTALLAS_POR_ROL_original.docx`: el Word tal cual (no se edita).
  - `PANTALLAS_POR_ROL.md`: copia en texto con las tablas, las preguntas abiertas y los
    diagramas, más una sección final que lo **compara con la app**.
  - `pantallas_por_rol/`: los 6 diagramas (casos de uso Android y web, clases, secuencia del
    escaneo, estados del alumno, arquitectura).
- **Hallazgos:** el documento dice que **sí hay wireframes de los seis roles** en el Figma
  "CETis 61 - Inicio (Android)" (8 del prefecto y el tablero de alertas, 5 del profesor, 5 web),
  que no está enlazado. Diferencias para que decida el usuario: el profesor puede poner castigo
  (el documento lo deja "por definir"); la contraseña vence a los 90 días para cualquier rol
  (el documento pregunta si solo el personal); los textos del padre siguen hablando como
  alumno; el profesor tiene notificaciones y el prefecto calendario (preguntas abiertas).
- **Archivos:** `docs/referencias/*`, `docs/TRASPASO_AGENTE.md`, `docs/BASE_DE_DATOS.md`,
  `firebase/roles.json` (la nota de origen ahora apunta al documento).
- **Cómo se comprobó:** el texto y las 12 tablas se sacaron celda por celda con Word (solo
  lectura) y se compararon con el original; las 6 imágenes se revisaron una por una.
- **Sin `.docx` para esta copia:** su versión Word es el original (si se generara, lo
  sobrescribiría; por eso el original se llama `_original`).

### 19-25. Prefecto de prueba

- **Qué:** a pedido del usuario, para probar las vistas del prefecto.
  - **Cuenta en Authentication:** la creó **el usuario** (`prefecto.prueba@cetis61.mx`, UID
    `QpWWIb6e1Ze2zlzhmwygFnoRJss1`; la contraseña solo la tiene él).
  - **En Firestore (desde la consola):** `usuarios/QpWWIb6e1Ze2zlzhmwygFnoRJss1` con
    `nombre` = "Prefecto de Prueba", `usuario` = `prefecto.prueba`, `correo`,
    `rol` = `prefecto`, `activo` = true y `debeCambiarContrasena` = false (para entrar directo;
    si se pone en true, la app pide cambiar la contraseña al entrar).
- **Archivos:** `docs/BASE_DE_DATOS.md` §10.1, `docs/TRASPASO_AGENTE.md`.
- **Cómo se comprobó:** la cuenta aparece en *Authentication → Users* con ese UID; el
  documento se leyó de vuelta en la consola después de recargar (6 campos, tipos correctos).
  Entrar con la cuenta en el teléfono lo prueba el usuario.

### 19-24. Documentación de las vistas por rol

- **Qué:** `docs/VISTAS_POR_ROL.md` (nuevo), `docs/adr/ADR-006-vistas-xml-para-todos-los-roles.md`
  (nuevo) y actualización de `VISTAS_ALUMNO.md`, `BASE_DE_DATOS.md` (§1, §4.1, §5.1, §10,
  §11), `MANUAL_TECNICO.md` (ahora describe las vistas XML), `DECISIONES.md` (sección K) y
  `TRASPASO_AGENTE.md`. Versiones en Word de cada uno.

### 19-23. Vistas XML para padre, prefecto y profesor

- **Pedido del usuario:** "seguir con el apartado del XML para las pantallas de padres,
  prefectos y profesores"; dejar pendientes el escaneo, las alertas, los avisos push y la web.
  Respuestas: pasar al padre de Compose a XML; que el profesor sí guarde en Firestore; hacer
  "Cambiar contraseña" para todos los roles. Por qué así: ADR-006.
- **Figma:** el archivo enlazado solo tiene las páginas "Alumno" y "Componentes"; **no hay
  pantallas de prefecto ni de profesor**. Se usaron el estilo del alumno y las piezas de
  "Componentes" (`StudentRow`, `StatusChip`, `AlertBanner`). Se exportaron 5 íconos nuevos
  (`ic_users`, `ic_edit`, `ic_flag`, `ic_check_circle`, `ic_scan`) y los colores de estatus.
- **Padre (9 pantallas, ahora XML):** Inicio, Anuncios, Calendario, Perfil, Entradas y
  salidas, Horario, Notificaciones; Código QR y Credencial usan las vistas del alumno.
  Novedades: el Horario ya muestra las clases del hijo, las flechas del calendario cambian de
  mes, "Cerrar sesión" pregunta antes y "Cambiar contraseña" ya funciona.
- **Prefecto:** Inicio (con los grupos que salieron hoy, en tiempo real), Horarios de todos
  los grupos por turno, horario de cada grupo, Calendario, Perfil, Notificaciones.
  "Escanear QR" y "Capturar matrícula" se ven como "Próximamente" (pendientes a propósito).
- **Profesor:** Inicio (sus grupos), Grupo (lista de alumnos con su estado), Perfil del
  alumno (con sus estatus), Cambiar estatus, Dar salida al grupo, Boletín, Perfil,
  Notificaciones. **Escribe en Firestore:** `alumnos/{m}/estatus` y
  `periodos/{semestre}/salidasDeGrupo` (las reglas ya lo permitían; no se cambiaron).
- **Cambiar contraseña (todos, también el alumno):** pide la actual, la nueva (8 o más) y la
  repetida; Firebase confirma la actual. Luego anota `debeCambiarContrasena = false` y
  `contrasenaVence` = hoy + 90 días. Si una cuenta tiene `debeCambiarContrasena = true` o su
  contraseña ya venció, la app la muestra sola al entrar.
- **Archivos nuevos:**
  - Vistas: `padre_inicio`, `padre_anuncios`, `padre_perfil`, `padre_entradas_salidas`,
    `prefecto_inicio`, `prefecto_horarios`, `profesor_inicio`, `profesor_grupo`,
    `profesor_alumno`, `profesor_cambiar_estatus`, `profesor_dar_salida`, `profesor_boletin`,
    `comun_calendario`, `comun_notificaciones`, `comun_horario`, `comun_perfil_personal`,
    `comun_cambiar_contrasena`, las 3 barras inferiores (`*_barra_inferior`) y 5 renglones
    (`comun_item_lista`, `comun_item_encabezado`, `comun_item_dia`,
    `comun_item_notificacion`, `profesor_item_alumno`).
  - Recursos: `strings_comun.xml`, `strings_prefecto.xml`, `strings_profesor.xml`, 5 íconos
    y 9 formas (`shape_estatus_*`, `shape_punto_*`, `shape_aviso_*`, `shape_circulo_exito`,
    `shape_circulo_iniciales`, `shape_tarjeta_brand`, `shape_tarjeta_sin_leer`).
  - Código: `ui/comun/` (`Carga.kt`, `VariablesComunes.kt`, `VistasComunes.kt`,
    `PantallasComunes.kt`), `ui/padres/PadresVistas.kt`, `ui/prefecto/PrefectoApp.kt` y
    `PrefectoVistas.kt`, `ui/profesor/ProfesorApp.kt` y `ProfesorVistas.kt`.
- **Archivos cambiados:** `ui/padres/PadresApp.kt` (reescrito para XML), `ui/CetisApp.kt`
  (prefecto y profesor ya tienen pantallas; cambio de contraseña obligatorio),
  `ui/alumno/AlumnoApp.kt` (opción "Cambiar contraseña"), `datos/CetisRepositorio.kt`
  (consultas de grupos, alumnos del grupo, estatus y salidas; guardar estatus y salidas;
  cambiar contraseña), `datos/Formatos.kt`, `datos/Modelos.kt` (`TipoEstatus`, `Turno`),
  `colors_cetis.xml`, `styles_cetis.xml` (estilos compartidos nuevos).
- **Archivos borrados** (copia en la carpeta temporal de la sesión): las 9 pantallas Compose
  del padre (`InicioScreen`, `AnunciosScreen`, `CalendarioScreen`, `PerfilScreen`,
  `CodigoQrScreen`, `CredencialDigitalScreen`, `EntradasSalidasScreen`, `HorarioScreen`,
  `NotificacionesScreen`), 7 piezas (`AppBar`, `BottomNav`, `PadresScaffold`, `IconCircle`,
  `ListItemCard`, `InfoRow`, `TextoVacio`) y `PadresModelos.kt` (sus clases pasaron a
  `ui/comun/VariablesComunes.kt`). Se quedan Bienvenida, Iniciar sesión, `AppIcon`,
  `PrimaryButton` y `SchoolLogo`.
- **Detalles corregidos de paso:** `Locale("es", "MX")` (forma vieja) → `Locale.forLanguageTag`
  en `Formatos.kt`; en `styles_cetis.xml` los atributos de Android nuevo (`lineHeight`,
  `clipToOutline`, `accessibilityHeading`) se marcaron con `tools:targetApi` para que lint no
  los cuente como error (en teléfonos viejos simplemente se ignoran).
- **Cómo se comprobó:** `gradlew :app:assembleDebug` compila sin advertencias y
  `gradlew :app:lintDebug` termina sin errores. Se instaló en el teléfono: la sesión guardada
  del alumno de prueba abrió Inicio; Perfil → "Cambiar contraseña" abre la pantalla nueva,
  con los campos vacíos muestra "Llena los tres campos." y la flecha regresa a Perfil.
  **No se probó:** entrar como padre, prefecto o profesor (hacen falta sus contraseñas; lo
  prueba el usuario) ni guardar un estatus o una salida.

### 19-22. Documento de traspaso para el siguiente agente

- **Qué:** `docs/TRASPASO_AGENTE.md` (y su `.docx`): qué es la app y qué se quiere lograr,
  cómo está hoy, lo que se le hizo, mapa del código y de la base, cuentas existentes, reglas
  de trabajo con el usuario, cómo trabajar en esta computadora y qué sigue.
- **Para qué:** que quien continúe el proyecto no tenga que reconstruir el contexto.

### 19-21. Se aplicó la estructura por semestres (ADR-005)

- **Respuestas del plantel:** 2,500 alumnos; grupos matutinos desde el 100 y vespertinos
  desde el 150; especialidad solo como campo vacío; matrículas de prueba desde 25000000;
  todos los datos personales (ficticios por ahora); entradas y salidas en una lista nueva
  por semestre.
- **Reglas (publicadas, 8,276 caracteres):** nuevas `periodos/{p}` con sus `registros` y
  `salidasDeGrupo`; `alumnos/{m}/inscripciones` y `alumnos/{m}/privado/{datos|salud}`;
  `especialidades`. Se quitaron `registros` y `salidasDeGrupo` de la raíz.
- **Índices:** 4 nuevos (3 de `alumnos` para listas y búsqueda, 1 de `grupos`). Total: 10.
- **Datos en Firestore (consola):**
  - `periodos/2026-2027-1` y `configuracion/plantel.periodoActual = "2026-2027-1"`.
  - `grupos/2026-2027-1_102` con horario lunes a viernes (ahora con `profesorId`).
  - `alumnos/25311754`: `grupoId`, `grupo` y `nombre` actualizados ("Jesusito Pérez Prueba")
    y 9 campos nuevos; `inscripciones/2026-2027-1`; `privado/datos` y `privado/salud` con
    datos ficticios.
  - `usuarios/{profesor}.grupos` → `2026-2027-1_102`.
  - Se borró `grupos/prueba-matutino` (y su horario), reemplazado por el nuevo.
- **App:** `Modelos.kt` (campos y clases nuevas: `PeriodoDoc`, `InscripcionDoc`,
  `DatosPrivadosDoc`, `DatosSaludDoc`, `EspecialidadDoc`, `EstadoAlumno`);
  `CetisRepositorio.periodoActual()` y `registros()` leen la lista del semestre de 20 en 20;
  `EntradasSalidasScreen` con "Ver más"; texto nuevo `ver_mas`.
- **Archivos:** `firebase/firestore.rules`, `firebase/firestore.indexes.json`,
  `datos/Modelos.kt`, `datos/CetisRepositorio.kt`, `ui/padres/PadresApp.kt`,
  `ui/padres/screens/EntradasSalidasScreen.kt`, `res/values/strings.xml`,
  `docs/BASE_DE_DATOS.md`, `docs/adr/ADR-005-estructura-para-muchos-alumnos.md`.
- **Cómo se comprobó:** reglas idénticas al archivo tras recargar la consola; cada documento
  leído en la consola; la app compila sin advertencias. **No se instaló:** el teléfono no
  estaba conectado.

### 19-20. Propuesta de estructura para todo el plantel (ADR-005)

- **Qué:** a pedido del usuario, se analizó cómo ordenar la base para miles de alumnos. Se
  escribió `docs/adr/ADR-005-estructura-para-muchos-alumnos.md` (**propuesta**, sin cambios en
  la base ni en la app).
- **Recomendación:** organizar por **periodos escolares** (grupos con el semestre en su id),
  historial por alumno (`inscripciones`), nombre separado en apellidos con campo de búsqueda,
  datos personales en un documento aparte (`privado`), estado del alumno (inscrito, baja,
  egresado), registros paginados y una herramienta de importación masiva.
- **Hallazgo importante:** con ~2,500 alumnos, el plan gratuito (Spark) se queda corto en
  lecturas diarias; hay que decidir el plan antes de cargar a todo el plantel.
- **Pendiente:** 6 preguntas para el plantel (número de alumnos, nombres de grupos,
  especialidades, formato de matrícula, datos personales, años de historial).

### 19-19. El grupo se toma del documento del grupo, en tiempo real

- **Problema reportado:** "el grupo tampoco cambia" (el nombre ya cambiaba). En la consola,
  el usuario había cambiado `grupos/prueba-matutino.nombre` a `"102 "` (con un espacio al
  final), pero la app mostraba la copia `alumnos/25311754.grupo` ("Prueba").
- **Qué se cambió:**
  - `CetisRepositorio.escucharGrupo()` y `nombreDeGrupoEnVivo()` (en `PadresApp.kt`): la app
    escucha `grupos/{grupoId}` y muestra **su** `nombre`; la copia de la ficha solo se usa si
    el grupo no se puede leer.
  - `Formatos.alumno()` recibe ese nombre y **quita espacios** al inicio y al final (por eso
    `"102 "` se ve como "102").
  - `PadresApp` y `AlumnoApp` usan el nombre del grupo en vivo.
  - `BASE_DE_DATOS.md`: el campo `alumnos.grupo` queda descrito como copia de respaldo.
- **Archivos:** `datos/CetisRepositorio.kt`, `datos/Formatos.kt`, `ui/padres/PadresApp.kt`,
  `ui/alumno/AlumnoApp.kt`, `docs/BASE_DE_DATOS.md`.
- **Cómo se comprobó:** compila sin advertencias; instalado en el teléfono. La prueba con
  sesión iniciada la hace el usuario.

### 19-18. Los cambios en la base ya se ven al instante en la app

- **Problema reportado:** "al cambiar los nombres desde la base de datos, no se actualizan en
  la app". Se revisó la consola: se habían cambiado `usuarios/{uid}.nombre` del alumno
  ("jesusito"), del padre ("juan") y del profesor ("pozoya"), pero
  `alumnos/25311754.nombre` seguía en "Alumno de Prueba".
- **Dos causas:**
  1. **La app leía los datos una sola vez**, al iniciar sesión. Un cambio hecho con la app
     abierta no se veía hasta volver a entrar.
  2. **Las pantallas del alumno (y las del padre, que hablan del hijo) toman el nombre de la
     ficha `alumnos/{matrícula}`**, no de `usuarios/{uid}`. El nombre del padre no aparece en
     ninguna pantalla.
- **Qué se cambió:**
  - `CetisRepositorio.escucharUsuario()` y `escucharAlumno()`: escuchan el documento con
    `addSnapshotListener` y avisan cada cambio. Se quitó `alumnoDeLaSesion()` (lectura única).
  - `PadresApp` y `AlumnoApp` usan `alumnoDeLaSesionEnVivo()`: la ficha se actualiza sola.
  - `CetisApp` escucha `usuarios/{uid}`: actualiza nombre, hijos y grupos al instante, y
    **cierra la sesión** si la cuenta se desactiva o se borra.
  - `BASE_DE_DATOS.md` §10: tabla "¿De dónde sale cada nombre?".
- **Datos:** no se cambió ningún nombre en la base; el usuario decide si pone "jesusito" en
  `alumnos/25311754.nombre`.
- **Archivos:** `datos/CetisRepositorio.kt`, `ui/CetisApp.kt`, `ui/padres/PadresApp.kt`,
  `ui/alumno/AlumnoApp.kt`, `docs/BASE_DE_DATOS.md`.
- **Cómo se comprobó:** compila sin advertencias y se instaló en el teléfono. La prueba con la
  app abierta la hace el usuario (requiere iniciar sesión).

### 19-17. Primera cuenta de Control Escolar

- **Qué:** a pedido del usuario, la cuenta `abrahamemiliomorales@gmail.com` (creada por él en
  Authentication) recibió el rol `control_escolar`: documento
  `usuarios/QUgw5x0tkvRx6Dz3Nb0S2zYlnmG2` con `nombre`, `usuario` y `correo` (el correo
  completo), `activo = true`.
- **Para qué sirve:** es la cuenta que las reglas reconocen como administradora; la usará la
  web de Control Escolar. En Android solo ve el aviso de "se usa desde la web".
- **Archivos:** `docs/BASE_DE_DATOS.md` §10.1.
- **Cómo se comprobó:** el documento se leyó en la consola (8 campos, rol y activo correctos).

### 19-16. Alumno, padre y profesor de prueba para revisar la app

- **Qué:** a pedido del usuario se registraron 3 usuarios de prueba (detalle y cómo
  borrarlos en `BASE_DE_DATOS.md` §10.1):
  - **Cuentas en Authentication:** las creó **el usuario** (Claude no crea cuentas ni pone
    contraseñas). Se le dieron los pasos; usó `25311754@cetis61.mx` para el alumno en lugar
    del `26000000000001` sugerido, y se respetó su elección.
  - **En Firestore (desde la consola):** `grupos/prueba-matutino` con horario de lunes a
    viernes (5 clases cada día), `alumnos/25311754`, y los perfiles `usuarios/{uid}` del
    alumno, el padre (hijo 25311754) y el profesor (grupo prueba-matutino). Se enlazaron
    `alumnos/25311754.usuarioId`, `.padres` y `grupos/prueba-matutino.profesores`.
- **Tropiezos y cómo se corrigieron:**
  - El primer intento de crear el grupo quedó **dentro** de la colección vieja
    (`CetisAsiste/Usuarios/grupos/prueba-matutino`) porque se usó el botón "Start collection"
    del documento abierto. Se borró ese documento (solo él; la consola confirmó la ruta) y se
    creó en la raíz.
  - Se había creado `alumnos/26000000000001` antes de conocer la matrícula real de la cuenta;
    se reemplazó por `alumnos/25311754` y se borró el anterior.
- **Cuenta extra:** había una 4.ª cuenta, `abrahamemiliomorales@gmail.com`; su rol se
  decidió en 19-17.
- **Cómo se comprobó:** se leyó cada documento en la consola (campos, tipos y enlaces) y el
  horario de viernes (5 mapas, solo la última clase con `ultimaDelTurno = true`). El inicio de
  sesión en el teléfono lo prueba el usuario con sus contraseñas.

### 19-15. Perfiles de usuario en la base de datos (colección `roles`)

- **Qué:** se creó la colección `roles` con un documento por tipo de usuario: `alumno`,
  `padre`, `prefecto`, `profesor`, `enfermeria` y `control_escolar`. Cada uno tiene nombre,
  etiqueta, descripción, plataformas, con qué inicia sesión, campos extra de su cuenta,
  si debe cambiar la contraseña, sus pantallas y sus permisos (tomados del documento
  "Pantallas por rol"). No son personas ni datos de ejemplo: son los perfiles que Control
  Escolar asigna en `usuarios.rol`.
- **Reglas:** nuevo bloque `match /roles/{rol}`: lee cualquiera con sesión; escribe solo
  Control Escolar. Publicado en la consola (6,776 caracteres, igual al archivo).
- **Cómo se cargaron:** a mano en la consola de Firebase (que escribe con permisos de
  administrador), llenando el formulario con ayuda de JavaScript en la página.
- **Lo que no se hizo:** primero se intentó abrir las reglas un par de minutos para cargar
  los perfiles con un script; **el sistema lo bloqueó por debilitar la seguridad** y se
  cambió al método de la consola. Ese script (`firebase/cargar_roles.ps1`) se borró.
- **App:** `RolDoc` en `datos/Modelos.kt`, `CetisRepositorio.rol()`, y el aviso de los roles
  sin pantallas ahora usa el nombre del perfil; si el perfil es solo web dice "se usa desde
  la web" (texto nuevo `rol_solo_web`).
- **Archivos:** `firebase/roles.json` (copia de los 6 perfiles), `firebase/firestore.rules`,
  `datos/Modelos.kt`, `datos/CetisRepositorio.kt`, `ui/CetisApp.kt`,
  `res/values/strings.xml`, `docs/BASE_DE_DATOS.md` (§3, §4.9, §5.1, §6, §10),
  `docs/adr/ADR-004-modelo-de-datos.md`.
- **Cómo se comprobó:** cada documento se leyó en la consola (campos, tipos y número de
  elementos de cada lista); leer `roles/alumno` sin sesión responde 403; la app compila.

### 19-14. Documentación de todo lo de este día

- **Qué:** documento nuevo de la base de datos, esta bitácora, tres ADR nuevos y
  actualización del manual técnico, las decisiones y las notas de vistas de alumno. Versiones
  en Word de cada documento, generadas con un script nuevo.
- **Archivos:** `docs/BASE_DE_DATOS.md`, `docs/BITACORA.md`,
  `docs/adr/ADR-002-reglas-sin-cloud-functions.md`,
  `docs/adr/ADR-003-inicio-de-sesion-con-correo-interno.md`,
  `docs/adr/ADR-004-modelo-de-datos.md`, `docs/MANUAL_TECNICO.md`, `docs/DECISIONES.md`,
  `docs/VISTAS_ALUMNO.md`, `docs/herramientas/md_a_word.ps1` y los `.docx` de cada uno.
- **Ojo:** `MANUAL_TECNICO.docx` y `DECISIONES.docx` se **regeneraron** desde sus `.md` (los
  del 5 de octubre se reemplazaron). De ahora en adelante se edita el `.md` y se vuelve a
  correr el script; no se edita el `.docx` a mano.
- **Cómo se comprobó:** se exportó cada documento a PDF y se revisaron páginas como imagen
  (títulos, tablas, listas y diagramas).

### 19-13. Prueba en el teléfono

- **Qué:** se instaló la app en el Samsung (Android 16) por USB. Abre la Bienvenida y luego
  Iniciar sesión con la etiqueta nueva "Matrícula o usuario". Al tocar "Iniciar sesión" con
  los campos vacíos aparece "Escribe tu matrícula o usuario y tu contraseña."
- **No se probó:** entrar con una cuenta real, porque todavía no hay cuentas (las crea Control
  Escolar; ver `BASE_DE_DATOS.md` §9.1). Las pantallas con datos reales quedan por probar.
- **Detalle corregido después de la prueba:** el texto de ayuda del campo decía "Ingresa tu
  matrícula"; ahora dice "Ingresa tu matrícula o usuario" (`strings.xml`).

### 19-12. La app inicia sesión de verdad y muestra los datos de cada quien

- **Qué:**
  - Inicio de sesión con Firebase Authentication (matrícula o usuario → correo interno).
  - Después de entrar, la app lee `usuarios/{uid}` y según el `rol` muestra el apartado de
    padres, las vistas de alumno, o un aviso para los roles que aún no tienen pantallas.
  - Las pantallas de padres dejaron de usar datos de ejemplo: ahora reciben los datos de
    Firestore como parámetros. Si no hay nada, muestran un texto como "Todavía no hay
    anuncios.".
  - Las vistas XML del alumno ya se usan en la app (antes solo eran vistas sueltas) y se
    llenan con sus datos; los chips de días del horario cambian de día.
  - El código QR se dibuja con el campo `codigoQr` del alumno (librería ZXing).
  - El calendario muestra el mes actual con los eventos reales; el banner "Información
    importante" de Inicio solo aparece si hay un anuncio marcado como `destacado`.
  - "Marcar todas como leídas" ya funciona.
  - Si la persona no cerró sesión, la próxima vez entra directo.
- **Archivos nuevos:** `datos/Modelos.kt`, `datos/CetisRepositorio.kt`, `datos/Formatos.kt`,
  `ui/CetisApp.kt`, `ui/alumno/AlumnoApp.kt`, `ui/padres/PadresModelos.kt`,
  `ui/padres/components/TextoVacio.kt`.
- **Archivos cambiados:** `Main.kt` (ahora abre `CetisApp()`), `ui/padres/PadresApp.kt`
  (recibe la sesión y pide los datos), las pantallas `InicioScreen`, `PerfilScreen`,
  `CredencialDigitalScreen`, `CodigoQrScreen`, `EntradasSalidasScreen`, `AnunciosScreen`,
  `NotificacionesScreen`, `CalendarioScreen`, `IniciarSesionScreen`; `res/values/strings.xml`
  (19 textos nuevos); `gradle/libs.versions.toml` y `app/build.gradle.kts` (ZXing 3.5.3).
- **Archivo borrado:** `ui/padres/SampleData.kt` (los datos de ejemplo). Sus clases sin datos
  pasaron a `PadresModelos.kt`.
- **Sin cambios a propósito:** el código viejo `MainAppContainer` de `Main.kt` se conserva
  como referencia, pero ya no se usa (y las reglas nuevas lo bloquean).
- **Cómo se comprobó:** `gradlew :app:assembleDebug` compila sin errores ni advertencias en
  los archivos nuevos; prueba en teléfono (entrada 19-13).

### 19-11. Seis índices compuestos en Firestore

- **Qué:** se crearon desde la consola (Chrome) los índices de `registros` (3),
  `salidasDeGrupo`, `anuncios` y `alumnos`. La lista está en `BASE_DE_DATOS.md` §7.
- **Archivos:** `firebase/firestore.indexes.json` (copia).
- **Cómo se comprobó:** la consola los muestra los 6 en estado *Building…* (tardan unos
  minutos en quedar *Enabled*).

### 19-10. Reglas de seguridad publicadas

- **Qué:** se reemplazó el "modo prueba" (todo abierto hasta el 4 nov 2026) por reglas por
  rol. Se pegaron en la consola por medio del editor y se publicaron.
- **Archivos:** `firebase/firestore.rules`.
- **Cómo se comprobó:** el texto publicado es idéntico al archivo (6,495 caracteres) y una
  lectura sin sesión ahora responde **403**.
- **Por qué:** ADR-002.

### 19-09. Authentication activado (Correo/Contraseña)

- **Qué:** en la consola, *Authentication → Get started → Email/Password → Enable → Save*.
  No se creó ninguna cuenta.
- **Por qué:** ADR-003.

### 19-08. Se quitaron los datos de ejemplo de la base

- **Qué:** a pedido del usuario ("no quiero datos de ejemplo"), se borraron los 31
  documentos de ejemplo cargados en 19-07, uno por uno por su id. Se dejó
  `configuracion/plantel` porque son ajustes del plantel, no datos de prueba. Se borró el
  script `firebase/semilla.ps1`.
- **No se tocó:** las colecciones viejas (`CetisAsiste/Usuarios` y 2 documentos de `usuarios`
  creados por el inicio de sesión anterior). Se recomienda que alguien del equipo las borre
  desde la consola porque tienen contraseñas a la vista.
- **Cómo se comprobó:** listado de cada colección por la API: todas vacías salvo
  `configuracion` (1) y los 2 documentos viejos de `usuarios`.

### 19-07. Datos de ejemplo cargados (después retirados)

- **Qué:** con un script de PowerShell se cargaron 32 documentos con los datos de Figma (Juan
  Perez Rodriguez, grupo 102, horario del miércoles, 4 anuncios, 3 eventos, 7 registros,
  5 notificaciones) para que cada colección existiera.
- **Tropiezo:** la primera vez falló a la mitad porque PowerShell "aplana" las listas de
  listas; se corrigió con comas (`,@(...)`) y se volvió a correr sin duplicar nada.
- **Cómo se comprobó:** se leyeron de vuelta dos documentos: acentos, horas (UTC) y valores
  vacíos correctos.
- **Resultado final:** se retiraron en 19-08.

### 19-06. Diseño del modelo de datos

- **Qué:** se definieron 9 colecciones y 4 subcolecciones a partir del documento "Pantallas
  por rol" (10 clases UML, alertas, estatus, roles), el "Proceso 8" del manual técnico y las
  pantallas de Figma. Se escribieron las reglas y los índices.
- **Archivos:** `firebase/firestore.rules`, `firebase/firestore.indexes.json`,
  `firebase/firebase.json`, `firebase/.firebaserc`.
- **Por qué:** ADR-004.

### 19-05. Revisión de lo que había en Firebase

- **Qué se encontró:**
  - Proyecto `appcetis-7c79d`, plan Spark (gratuito, sin Cloud Functions).
  - Firestore ya creado, en **modo prueba**: cualquiera podía leer y escribir hasta el 4 de
    noviembre de 2026.
  - Colecciones viejas: `CetisAsiste/Usuarios` (un documento con `contrasenia` y `matricula`
    vacías) y `usuarios` (2 documentos del inicio de sesión anterior).
  - Authentication sin activar.
  - El Figma enlazado solo tiene la página "Alumno" (8 frames) y "Componentes"; los
    wireframes de los otros roles están en otro archivo ("CETis 61 - Inicio (Android)").

### 19-04. ADR-001 actualizado

- **Qué:** se anotó que la barra inferior quedó a 12 dp y que la app ya no coincide con
  Figma en eso (Figma usa 6 dp).
- **Archivos:** `docs/adr/ADR-001-barra-inferior.md`.

### 19-03. Barra inferior a 12 dp de los lados

- **Qué:** la barra inferior de las vistas de alumno quedó con el mismo margen que las
  tarjetas (`@dimen/cetis_screen_margin` = 12 dp) en lugar de 6 dp.
- **Archivos:** `alumno_inicio.xml`, `alumno_horario.xml`, `alumno_perfil.xml`,
  `alumno_boletin.xml`.
- **Cómo se comprobó:** compila; falta revisarlo en la vista previa.
- **Pendiente:** corregir la capa `BottomNav` en Figma (x = 12, ancho = 336).

### 19-02. ADR-001 (por qué el contenido termina arriba de la barra)

- **Qué:** primer registro de decisión del proyecto, con las opciones A, B y C.
- **Archivos:** `docs/adr/ADR-001-barra-inferior.md`; enlace desde `docs/VISTAS_ALUMNO.md`.

### 19-01. El contenido ya no pasa por debajo de la barra inferior

- **Qué:** en Horario (y las otras 3 vistas con barra) las tarjetas se veían por debajo y
  detrás de la barra. Ahora el contenido termina 100 dp antes del borde (68 de la barra + 32
  de margen) y la lista deja 16 dp de aire al final.
- **Archivos:** `res/values/dimens_cetis.xml` (`cetis_content_bottom_space` de 116 a 100 dp),
  `alumno_horario.xml`, `alumno_inicio.xml`, `alumno_perfil.xml`, `alumno_boletin.xml`,
  `docs/VISTAS_ALUMNO.md`.
- **Cómo se comprobó:** `gradlew :app:assembleDebug` sin errores.

---

## Antes del 19 de octubre de 2026 (resumen)

| Fecha | Qué |
|---|---|
| Inicio del proyecto | Proyecto de Android Studio conectado a Firebase; inicio de sesión con matrícula y contraseña guardada en `usuarios` (`Main.kt`) |
| 5 oct 2026 | 11 pantallas del apartado de padres en Compose, con datos de ejemplo de Figma; `MANUAL_TECNICO.md` y `DECISIONES.md` |
| 7 oct 2026 | Documento "CetisAsiste — Pantallas por rol" (6 roles, alertas, UML, arquitectura). Desde el 19 oct está en `docs/referencias/` |
| Antes del 19 oct | Vistas XML del alumno (`alumno_*.xml`), variables (`AlumnoVariables.kt`) y `AlumnoVistas.kt`; `VISTAS_ALUMNO.md` |
