# Pendientes — CetisAsiste

**Fecha:** 20 de octubre de 2026 (actualizado con ADR-010 y la web)
**Qué es:** todos los pendientes del proyecto en un solo lugar. Se juntaron de la base de
datos, el manual técnico, el traspaso, los ADR, el documento "Pantallas por rol", el
documento de Figma y la bitácora. Cada pendiente dice **quién tiene que actuar** y **de
dónde salió**.

**Este es el único lugar de los pendientes.** Las listas que tenían la base de datos (§11),
el manual técnico (§11) y el traspaso (§8) se pasaron aquí el 19 de octubre de 2026 y ahora
solo enlazan a este documento; en la columna *Origen* se siguen nombrando así. Al agregar o
resolver un pendiente, cámbialo aquí.

**Quién actúa:**

- **Plantel:** una decisión o un dato que tiene que dar la escuela.
- **Usuario:** algo que se decide o se hace del lado del dueño del proyecto (probar, revisar,
  borrar en la consola).
- **Desarrollo:** trabajo de programación o de diseño.

---

## 1. Lo más urgente

| # | Pendiente | Quién | Por qué es urgente |
|---|---|---|---|
| 1 | **Borrar las colecciones viejas** `CetisAsiste/Usuarios` y los 2 documentos viejos de `usuarios` que tienen contraseñas a la vista | Usuario, desde la consola de Firebase | Son contraseñas guardadas sin cifrar. Claude no las borró a propósito: borrar datos lo decide el usuario |
| 2 | **Probar en el teléfono** las vistas de padre, profesor y prefecto | Usuario | Ya están instaladas, pero solo se probó con la cuenta del alumno. Detalle en la sección 10 |
| 3 | **Decidir el plan de Firebase** (Spark, gratis, o Blaze, de pago) | Usuario y plantel | Con 2,500 alumnos el plan Spark se queda corto en lecturas diarias; hay que decidir antes de cargar a todo el plantel. Sin Blaze tampoco hay validación del escaneo en el servidor ni avisos push. Con los avisos al padre (ADR-010) se llega a unas 17,500 escrituras al día, cerca del límite gratis (20,000) |
| 4 | **Crear el índice nuevo e instalar la app nueva** en los teléfonos de prueba. Las reglas nuevas **ya están publicadas** (20 oct, las publicó el usuario). Los perfiles `roles/alumno`, `roles/prefecto` y `roles/enfermeria` **ya se actualizaron** (bitácora 20-04) | Usuario | Las reglas nuevas (ADR-009 y ADR-010) piden `noControl` y avisos firmados (`creadaPor`): con la app anterior el escaneo ya no se guardaría. El índice es `registros` (`fecha` ↑, `alerta` ↑, `fechaHora` ↓); sin él, "Alertas de hoy" se ve vacía |
| 5 | **Registrar la web en Firebase, llenar `web/js/config.js` y publicarla** con Firebase Hosting | Usuario | Sin esto Control Escolar y Enfermería siguen usando la consola. Pasos en [web/README.md](../web/README.md) |

Origen: base de datos §11, traspaso §8, ADR-002 (acción 3) y ADR-005 (acción 8).

---

## 2. Orden sugerido de trabajo

1. Probar en el teléfono las vistas de padre, profesor y prefecto, y el menú ☰ de los cuatro
   roles, y revisar lo que salga.
2. Decidir el plan de Firebase.
3. ~~Escaneo del prefecto: escáner QR, captura por No. de control, resultado y alertas.~~
   Hecho (ADR-009); avisos y alertas del día, hechos (ADR-010).
4. Validación del escaneo en el servidor y avisos push al padre (si se aprueba Blaze).
5. Importación masiva de alumnos y cambio de semestre.
6. Web de Control Escolar y Enfermería. **Empezada el 20 de octubre** (ADR-011, carpeta
   `web/`): falta publicarla y lo que dice la sección 8.
7. Recuperar contraseña olvidada, padre con varios hijos y vigencia de los castigos.
8. Ajustar las vistas de prefecto y profesor a su Figma, cuando el usuario lo pida.
9. Decidir las diferencias entre el documento de roles y la app.
10. Elegir qué mejoras de estructura se aplican (sección 12); el `ViewModel`, antes del paso 3.

Origen: traspaso §8.

---

## 3. Preguntas para el plantel

| Pregunta | De qué depende | Cómo está hoy | Origen |
|---|---|---|---|
| ¿Qué hace el prefecto ante cada alerta: deja pasar, retiene o avisa a alguien? | Si la pantalla de alerta necesita botones (dejar pasar, retener) | El escaneo ya existe: muestra la alerta y la guarda en el registro; con alerta, la ubicación del alumno no cambia (ADR-009) | Pantallas por rol |
| ¿Quién pone un castigo disciplinario y cuánto dura? | Si el profesor puede poner castigos y el campo `vigenteHasta` | El profesor puede poner castigos y se guardan sin fecha de fin. Al escanear, un castigo sin fecha de fin cuenta **solo el día en que se puso** (ADR-009) | Base de datos §11, manual §11, pantallas por rol |
| ¿Qué diferencia hay entre reporte y llamada de atención? ¿Los ven el alumno y el padre? | Qué estatus se muestran en las pantallas de alumno y padre | Desde ADR-010 el **padre recibe un aviso** de cada falta, reporte, llamada de atención y castigo (con la descripción); el alumno no | Pantallas por rol, ADR-010 |
| ¿De qué se le avisa al padre? | `configuracion/plantel.avisarAlPadreEstatus` y `avisarAlPadreAlertas` | Por omisión: los cuatro estatus del profesor y las alertas "inactivo" y "salida fuera de horario" (además de cada entrada y salida). Se cambia en la consola sin programar | ADR-010 |
| ¿Qué pasa al escanear a un alumno con falta notificada, reporte, llamada de atención o alta para prácticas? | Las alertas del escaneo | Hoy no generan alerta; "en prácticas" puede salir sin alerta (ADR-009) | Pantallas por rol (tabla de estatus) |
| ¿Cada cuánto vence la contraseña y a quién aplica: solo al prefecto o a todo el personal? | El valor `diasVenceContrasena` en `configuracion/plantel` | Vence a los 90 días **para cualquier rol**, incluidos alumno y padre (0 = no vence) | Base de datos §11, ADR-006, pantallas por rol |
| ¿Qué especialidades hay y desde qué semestre se separan? | El catálogo `especialidades` | El campo `especialidadId` existe pero vacío; las reglas ya están, sin documentos | ADR-005 (acción 10), base de datos §11 |
| ~~¿Enfermería trabaja solo en web?~~ **Sí** (el usuario, 20 oct) | Si hace falta una vista Android mínima para dar el pase de salud | Enfermería usa la web (ADR-011); en la app ve un aviso | Pantallas por rol, base de datos §5.1 |
| ¿El padre puede mostrar el código QR de su hijo, o solo verlo? | La pantalla Código QR del padre | Uso por confirmar | Pantallas por rol |
| ¿El alumno y el padre ven en la app si el alumno está inactivo? | Las pantallas de alumno y padre | Por definir | Pantallas por rol |

---

## 4. Decisiones del usuario: lo que la app hace distinto al documento de roles

La app ya hace estas cosas de otra forma, o de más. Hay que decidir si se quedan así o se
cambian.

| Tema | Documento de roles | App hoy |
|---|---|---|
| Profesor: tipos de estatus | Falta, reporte o llamada de atención | También deja poner **castigo**, porque las reglas lo permiten. El documento dice "por definir (profesor o Control Escolar)" |
| Profesor: notificaciones | Pregunta abierta | Tiene la campana de Notificaciones en su Inicio |
| Profesor: horario del grupo | No está | Agregado (se abre desde el grupo) |
| Prefecto: Inicio | Acceso al escáner y últimos escaneos | Accesos (escáner y captura ya funcionan) y los grupos que salieron hoy, en tiempo real |
| Prefecto: calendario y horarios | Pregunta abierta / propuesta | Los tiene, como pestañas |
| Prefecto y profesor: Perfil | No está | Agregado, para cambiar la contraseña y cerrar sesión |
| Vencimiento de la contraseña | ¿Solo prefecto o todo el personal? | 90 días para cualquier rol (ver sección 3) |
| Textos del padre | Deben hablar del hijo | Siguen como en Figma: "Estudiante", "Mi credencial digital", "Mis entradas y salidas", "tu código", "tus horarios"; Notificaciones dice "Registraste tu entrada". Hay que definir los textos finales y cambiarlos también en Figma |

Origen: pantallas por rol (comparación con la app y ajustes pendientes en Figma), manual §11.

---

## 5. App Android: lo que falta hacer

| Pendiente | Detalle | Origen |
|---|---|---|
| **Publicar las reglas nuevas y pasar los datos de prueba a 14 dígitos** | Sin esto el alumno de prueba no entra: las reglas viejas buscan `matricula` y su número tiene 8 dígitos. Pasos en BASE_DE_DATOS §10.1 (lo hace el usuario en la consola). Las reglas ya incluyen lo de ADR-010. **Hecho el 20 oct:** las reglas publicadas y los perfiles de `roles` (pasos 1 y 2 de §10.1). Falta el alumno de prueba | ADR-009 (acciones 4 a 6), ADR-010 (acciones 3 y 4) |
| **Recuperar una contraseña olvidada** | "¿Olvidaste tu contraseña?" no abre nada. Los correos internos (`@cetis61.mx`) no reciben mensajes, así que hay que decidir otro camino. Mientras, Control Escolar la cambia en la consola (base de datos, receta §9.8) | ADR-003 (acción 5), base de datos §11, manual §11 |
| **Padre con varios hijos** | Hoy se muestra el primero de su lista `hijos`. Falta una pantalla para elegir hijo | ADR-004 (acción 4), base de datos §11, manual §11 |
| Textos del padre | Ver sección 4 | Manual §11 |
| Colores de advertencia | El Figma de prefecto y profesor usa `#B7740C` y su fondo `#FCF0D9` (estatus amarillos, aviso de contraseña vencida). Todavía no están en `colors_cetis.xml` | Figma |

*Resueltos el 19 de octubre con el menú ☰ (ADR-007):* el ☰ que no abría nada, el alto de la
barra de gestos en las vistas del alumno y pasar al alumno al esquema común (ya usa
`PantallaXml`).

---

## 6. Ajustar prefecto y profesor a su Figma

Las vistas de prefecto y profesor se hicieron el 19 de octubre **antes de encontrar sus
páginas en Figma**, con el estilo del alumno. El usuario pidió por ahora **solo guardar esta
información**; los cambios se hacen cuando lo pida. Los enlaces de cada página están en el
documento de Figma (`FIGMA.docx`).

**Prefecto:**

| Tema | Figma | App hoy |
|---|---|---|
| Pestañas | Inicio · Escanear · Horarios · Perfil | Inicio · Horarios · Calendario · Perfil |
| Inicio | Saludo, botón guinda "Escanear código QR", Horarios y Calendario, "Últimos escaneos" | Tarjeta de perfil, 4 accesos, "Salidas de hoy" |
| Salidas de grupo | En Notificaciones ("Grupo 102 ya salió de clases") | En el Inicio, en tiempo real |
| Horarios | Una pantalla: chips de grupo + chips de día + clases | Lista de grupos por turno → horario del grupo |
| Calendario | Se abre desde Inicio | Pestaña |
| Perfil | Agrega "Área" y dice "Tus datos los administra Control Escolar." | Usuario y Correo |
| Contraseña obligatoria | Aviso amarillo con candado "Tu contraseña venció"; campos "Nueva contraseña" y "Confirmar nueva contraseña"; botón "Guardar" | Aviso rojo de texto; "Contraseña nueva", "Repite la contraseña nueva"; "Guardar contraseña" |

**Profesor:**

| Tema | Figma | App hoy |
|---|---|---|
| Pestañas | Inicio · Grupos · Boletín · Perfil | Inicio · Boletín · Perfil |
| Inicio | Saludo, botón guinda "Dar salida al grupo" (grupo de la última clase de hoy), "Mis grupos" con el nombre del grupo en el círculo, tarjeta Boletín | Tarjeta de perfil y lista de grupos |
| Grupo | Buscador y lista de alumnos | Dar salida, Horario del grupo y lista de alumnos (sin buscador) |
| Estatus en la lista | Último estatus del alumno ("Sin incidencias", "Castigo", "Falta notificada", "Reporte"), en amarillo las faltas y los reportes | Estado de pago o salud ("Activo", "Inactivo por pago"…) |
| Perfil del alumno | 3 botones de acción e "Historial" con círculos amarillos | Datos, un botón "Cambiar estatus" y "Estatus registrados" |
| Cambiar estatus | **3 tipos: Falta, Reporte, Llamada de atención** (sin castigo); campo "Motivo" | 4 tipos (también castigo); campo "Descripción" |
| Dar salida | Tarjeta del grupo con la última clase; botón "Dar salida a todo el grupo"; solo en la última clase | Tarjeta de datos; botón "Dar salida"; a cualquier hora |
| Perfil | Agrega "Área" ("Docente") | Usuario y Correo |

**Ojo:** el Figma de prefecto y profesor también tiene barra inferior (pestañas). En la app ya
no hay barra: esas pestañas ahora son opciones del menú ☰ (ADR-007).

**En el propio Figma:** dibujar el menú ☰ y quitar la barra inferior del diseño, para que
Figma y la app coincidan. (Ya no hace falta corregir la capa `BottomNav`: la barra se quitó.)

Origen: Figma (diferencias), ADR-006 (acción 6), ADR-007 (acción 9), traspaso §8, bitácora
19-28.

---

## 7. Servidor y Firebase

| Pendiente | Detalle | Origen |
|---|---|---|
| Plan de Firebase | Ver sección 1 | ADR-002, ADR-005 |
| **Validar el escaneo en el servidor** | Una función `validarEscaneo` en Cloud Functions que revise estatus, guarde el registro y avise. Necesita el plan Blaze; mientras, las reglas limitan quién escribe | ADR-002 (acción 4), base de datos §11 |
| **Avisos push al padre** | Con Cloud Messaging. Ya todo llega al buzón `usuarios/{uid}/notificaciones` (ADR-010): basta una Cloud Function que escuche ese buzón y mande el push a `tokensFcm`. Depende de Cloud Functions | ADR-002 (acción 4), ADR-010 (acción 8) |
| **Borrar los avisos vencidos** | Cada aviso trae `venceEn` (60 días). Activar una política TTL sobre ese campo si la consola lo permite; si no, con una Cloud Function o desde la web | ADR-010 (acción 6) |
| Borrar las colecciones viejas | Ver sección 1 | Base de datos §11 |

---

## 8. Web de Control Escolar y Enfermería

**Primera versión hecha el 20 de octubre** (ADR-011, carpeta `web/`, cómo usarla en
[web/README.md](../web/README.md)): acceso, alumnos (búsqueda y por grupo), perfil, pase de
salud, activar/desactivar por salud y por pago, nuevo estatus, datos de salud, entradas y
salidas del día, avisos al padre y notificaciones. Falta **publicarla** (§1, punto 5).

Lo que falta en la web:

| Rol | Pendiente | Detalle |
|---|---|---|
| Control Escolar | **Alta de alumnos, padres y personal** | La web no puede crear cuentas de Authentication sin un servidor: hace falta el script de administración con el Admin SDK (ARQUITECTURA.md E1), que también sirve para la importación masiva (§9) |
| Control Escolar | Editar datos del alumno | Hoy solo se ven; al editar el nombre hay que cambiar las dos copias (`alumnos` y `usuarios`) |
| Control Escolar | Publicar anuncios y eventos | Hoy desde la consola (BASE_DE_DATOS §9.6) |
| Ambos | App Check | ARQUITECTURA.md B3 |

Estas pantallas siguen pendientes y, además, **ni siquiera tienen wireframe**:

| Rol | Pantalla | Para qué sirve |
|---|---|---|
| Control Escolar | Estatus del alumno | Consultar y modificar cualquier estatus (**consultar y agregar ya está** en el perfil; falta modificar y borrar) |
| Control Escolar | Altas para prácticas | Registrar al alumno que sale a prácticas |
| Control Escolar | Cambio de grupo o turno | Mover al alumno de grupo o turno |
| Control Escolar | Horarios | Consultar y administrar los horarios de los grupos |
| Control Escolar | ~~Entradas y salidas~~ | Hecho (ADR-011) |
| Enfermería | ~~Perfil del alumno~~ | Hecho (ADR-011) |

Origen: pantallas por rol, base de datos §11, traspaso §8.

---

## 9. Datos y trabajo escolar

| Pendiente | Detalle | Origen |
|---|---|---|
| **Importación masiva de alumnos** (CSV o Excel) | Para dar de alta cientos de alumnos por semestre; a mano no se puede con 2,500 | ADR-005 (acción 7), base de datos §11 |
| **Cambio de semestre** | Receta o herramienta para crear el periodo nuevo y sus grupos, reinscribir a los alumnos y cambiar `periodoActual` | ADR-005 (acción 9), base de datos §11 |
| No. de control y especialidades | Cargarlas cuando el plantel las dé (sección 3) | ADR-005 (acción 10) |

---

## 10. Pruebas

| Qué probar | Cómo | Origen |
|---|---|---|
| Padre | Entrar con `padre.prueba` y revisar todas sus pantallas con datos reales | ADR-006 (acción 5), traspaso §8 |
| Profesor | Entrar con `profesor.prueba`, guardar un estatus y dar salida al grupo | ADR-006 (acción 5), traspaso §8 |
| Prefecto | Entrar con `prefecto.prueba`: la salida que dio el profesor debe aparecer en su Inicio, en tiempo real | ADR-006 (acción 5), traspaso §8 |
| Escaneo | Con `prefecto.prueba`: escanear el QR del alumno de prueba (en otro teléfono), probar Entrada y Salida, teclear el No. de control y provocar cada alerta cambiando campos en la consola. Revisar que el resultado diga el motivo y que la alerta salga en "Alertas de hoy" | ADR-009 (acción 7), ADR-010 |
| Avisos | Con `padre.prueba` abierto: el prefecto registra una entrada y el profesor pone una falta; el aviso debe aparecer solo, la campana encenderse y el menú ☰ decir "2". "Marcar todas como leídas" apaga la campana | ADR-010 (acción 7) |
| Menú ☰ de padre, prefecto y profesor | Abrirlo en cada pantalla principal; elegir una opción; "Notificaciones"; cerrarlo tocando fuera y con "atrás"; "Cerrar sesión" pregunta y sale. **Con el alumno ya se probó y funciona** (bitácora 19-33 y 19-34) | ADR-007 (acción 8) |
| Otros tamaños | Nunca se ha probado en tabletas, en horizontal ni con la letra del sistema agrandada | Manual §11 |

Las contraseñas de las cuentas de prueba las tiene el usuario. Después de probar, revisar
juntos lo que salga.

---

## 11. Limitaciones conocidas (no son errores)

- El menú ☰ se abre con el botón; **no** se abre deslizando el dedo desde la orilla izquierda
  (ADR-007).
- En la vista previa de Android Studio no se ve el menú: lo pone el código al tocar el ☰.

---

## 12. Estructura del código (propuesta, ADR-008)

> **20 de octubre:** la revisión de **todo el sistema** está en [ARQUITECTURA.md](ARQUITECTURA.md),
> con más puntos (escaneo sin señal, hora del servidor, proyecto de pruebas, App Check,
> custom claims, respaldos, script de administración, GitHub Actions). Su §4 trae el orden
> que se propone. Esta tabla sigue siendo la del código de la app.

Mejoras que **el usuario decide** si se hacen. En orden: primero lo barato y seguro.

| # | Mejora | Esfuerzo | Cuándo |
|---|---|---|---|
| 1 | Borrar el código muerto: inicio de sesión viejo en `Main.kt`, `alumno_iniciar_sesion.xml` y los recursos sin uso que marca lint | Una tarde | Cuando sea |
| 2 | Ordenar carpetas: `ui/acceso/` para Bienvenida e Iniciar sesión; `AlumnoVistas.kt` y `AlumnoVariables.kt` a `ui/alumno/` | Una tarde | Cuando sea |
| 3 | Una sola navegación (`Navegacion<T>`) para los 4 roles, en lugar de 4 copias | Medio día | Cuando sea |
| 4 | Pruebas de `Formatos`, de No. de control → correo y de las reglas de contraseña | Un día | Cuando sea |
| 5 | Un `ViewModel` por rol (empezar por el profesor) | 2 a 3 días | **Antes del escaneo** |
| 6 | Paquete propio (por ejemplo `mx.cetis61.cetisasiste`) y registrar la app otra vez en Firebase | Medio día | **Antes de publicar** |
| 7 | Actualizar librerías | Un rato por semestre | Cada semestre |
| 8 | Decidir `minSdk` (las esquinas redondeadas solo se ven desde Android 12) | Decisión del plantel | Cuando se sepa qué teléfonos usan |
