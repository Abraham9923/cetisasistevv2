# ADR-009: No. de control de 14 dígitos, QR con el No. de control y escaneo del prefecto

**Estado:** Aceptada y aplicada en la app (19 de octubre de 2026). Reglas publicadas y perfiles
actualizados en la consola el 20 de octubre. **Falta** volver a crear al alumno de prueba
(ver "Acciones").
**Fecha:** 19 de octubre de 2026
**Quién decide:** el equipo de CetisAsiste (pedido del usuario)
**Reemplaza a:** DECISIONES J6 (QR con texto al azar) y DECISIONES K6 (escaneo "Próximamente")

> Lo que se hizo, paso por paso: [BITACORA.md](../BITACORA.md) (entrada 19-35). Base de datos:
> [BASE_DE_DATOS.md](../BASE_DE_DATOS.md). Pantallas: [VISTAS_POR_ROL.md](../VISTAS_POR_ROL.md).

## Contexto

- La app hablaba de **"matrícula"** (8 dígitos de prueba, desde 25000000). El plantel usa el
  **No. de control**, impreso en la credencial: **14 dígitos**. El usuario mostró una credencial
  de ejemplo (sus datos no se guardaron en el proyecto; solo se tomó el formato).
- El QR del alumno llevaba un **texto secreto al azar** (`codigoQr`) que alguien tenía que
  generar y guardar para cada alumno. Para 2,500 alumnos no había forma práctica de hacerlo.
- El escaneo del prefecto (escáner QR, captura, resultado y alertas) estaba pendiente: sus
  tarjetas decían "Próximamente". La base, las reglas y el diseño en Figma ya estaban listos.

Respuestas del usuario (19 de octubre):

| Pregunta | Respuesta |
|---|---|
| ¿Hasta dónde se cambia "matrícula" por "No. de control"? | **Todo**: textos, documentos, código y base de datos |
| ¿Qué lleva el QR? | **El No. de control** (generado por la app) |
| ¿Se exigen 14 dígitos desde ya? | **Sí**; el usuario vuelve a crear al alumno de prueba con 14 dígitos |
| ¿Se empieza el escaneo? | Sí, con lo propuesto: zxing-android-embedded, guardar siempre el registro y no cambiar la ubicación si hay alerta |

## Decisión

1. **"No. de control" en todos lados.** En Firestore el campo `matricula` pasa a `noControl`
   (en `alumnos` y en `usuarios`); el valor `medio` de los registros pasa de `matricula` a
   `no_control`; el perfil `roles/alumno` dice `inicioDeSesion = no_control`. El id de
   `alumnos/{noControl}` sigue siendo el número. El correo interno es `noControl@cetis61.mx`.
2. **14 dígitos exactos** (`NoControl.esValido`). Si al iniciar sesión se escriben solo números
   y no son 14, la app avisa "El No. de control tiene 14 dígitos." sin llamar a Firebase. La
   captura del prefecto también exige 14.
3. **El QR lleva `CETIS61:` + el No. de control** (`NoControl.contenidoQr`). La app lo dibuja
   sola; ya no hay que capturar nada para cada alumno y el campo `codigoQr` deja de usarse
   (si un documento viejo lo trae, se ignora). El prefijo sirve para reconocer los QR de la app.
4. **Escaneo del prefecto** (Figma 77:500, 89:859, 77:501, 77:502):
   - **Escáner** (nueva opción "Escanear" del menú ☰ y tarjeta del Inicio): selector Entrada /
     Salida, cámara con la librería **zxing-android-embedded** (primera librería nueva desde
     DECISIONES H3) y botón "Escribir No. de control". Pide el permiso de cámara al abrirse.
   - **Captura por No. de control**: teclado propio de 14 dígitos (para cuando el QR no se lee).
   - **Resultado**: verde ("Entrada registrada") o rojo con el título de la alerta.
5. **Las alertas las decide `Escaneo.alerta`** (datos/Escaneo.kt), sin Firebase, con pruebas
   automáticas (`EscaneoTest`, 20 pruebas). Si aplican varias, gana la primera:
   - Entrada: no dado de alta → inactivo (pago o salud) → ya ingresó → castigo.
   - Salida: no dado de alta → no ha ingresado → castigo → salida fuera de horario.
   - Un alumno dado de baja o egresado cuenta como **no dado de alta**.
   - **Salida autorizada** = su profesor ya dio salida al grupo hoy, tiene pase de salud de hoy,
     está en prácticas o ya tenía `ubicacion = salida_autorizada`.
   - **Castigo vigente** = su `vigenteHasta` no ha pasado; si no tiene, **solo el día en que se
     puso** (el plantel todavía no dice cuánto dura un castigo).
6. **Se guarda todo de una vez** (`CetisRepositorio.registrarEscaneo`, un lote de Firestore):
   - el **registro siempre**, aunque haya alerta (es una bitácora);
   - **si no hubo alerta**, la `ubicacion` del alumno (dentro o fuera) y su `ultimoRegistro`;
     **con alerta la ubicación no cambia**;
   - **si no hubo alerta**, una notificación **dentro de la app** para cada padre o tutor
     ("Juan entró al plantel a las 7:05 am."). El aviso **push** (con la app cerrada) sigue
     pendiente: necesita Cloud Functions y el plan Blaze (ADR-002).
7. **El mismo QR no se registra dos veces seguidas**: la cámara lo sigue viendo, así que el
   mismo No. de control se ignora durante 20 segundos después de registrarlo.

## Opciones consideradas

### Para el contenido del QR

| Opción | A favor | En contra |
|---|---|---|
| **A. No. de control con prefijo (la elegida)** | Se genera solo; se puede imprimir en la credencial; nada que capturar | Cualquiera puede hacer un QR con un número ajeno. Pero el prefecto **ya puede teclear** cualquier número, y al escanear ve nombre e iniciales para comparar con la persona |
| B. Texto secreto al azar (`codigoQr`, como estaba) | No se puede adivinar; se cambia si se pierde el teléfono | Hay que generarlo y guardarlo para 2,500 alumnos; no se puede imprimir en una credencial fija |
| C. No. de control firmado (con una clave) | No se puede falsificar sin la clave | La clave tendría que ir dentro de la app, de donde se puede sacar; sin servidor no es seguro |

### Para leer el QR con la cámara

| Opción | A favor | En contra |
|---|---|---|
| **A. zxing-android-embedded (la elegida)** | La cámara es una vista XML (`BarcodeView`), igual que el resto de la app; usa ZXing, que ya estaba | Usa la API vieja de la cámara y casi no recibe cambios |
| B. CameraX + ML Kit | Lo más moderno; lee más rápido con poca luz | Dos librerías más y bastante más código |

## Consecuencias

- **Más fácil:** dar de alta un alumno (ya no hay `codigoQr`); escanear y capturar con el
  mismo número que trae la credencial; probar las alertas sin teléfono.
- **Más difícil:** cada cambio en la base (`matricula` → `noControl`) obliga a **publicar las
  reglas nuevas** y a corregir los documentos que ya existían (solo hay datos de prueba).
- **Volumen:** cada escaneo lee unos 4 documentos y escribe 2 o 3. Con 2,500 alumnos (entrada
  y salida) se acerca al límite gratis del plan Spark (ADR-005): hay que decidir el plan antes
  de usarlo con todo el plantel.
- **Seguridad:** la validación es en el teléfono del prefecto (ADR-002). Una app modificada
  podría registrar escaneos falsos; se corrige con Cloud Functions cuando haya plan Blaze.
- **Hay que revisar después:** qué hace el prefecto ante cada alerta, cuánto dura un castigo y
  qué efecto tienen falta, reporte y prácticas (preguntas del plantel en PENDIENTES §3).

## Acciones

1. [x] Cambiar "matrícula" por "No. de control" en textos, documentos, código, reglas
   (`firebase/firestore.rules`) y perfiles (`firebase/roles.json`).
2. [x] 14 dígitos al iniciar sesión y en la captura; QR con `CETIS61:` + No. de control.
3. [x] Escáner, captura y resultado; lógica de alertas con 20 pruebas; guardar en lote.
4. [x] (20 oct, la publicó el usuario) **Publicar `firebase/firestore.rules` en la consola** (sin esto, el alumno nuevo no
   puede leer sus datos: las reglas viejas buscan `matricula`).
5. [x] (20 oct, bitácora 20-04) **Actualizar `roles/alumno` en la consola** con lo de `firebase/roles.json`
   (`inicioDeSesion = no_control`, `camposDelUsuario = [noControl]`) y `roles/prefecto`.
6. [ ] **Volver a crear al alumno de prueba** con un No. de control inventado de 14 dígitos
   (pasos en BASE_DE_DATOS §10.1) y cambiar el `hijos` del padre de prueba.
7. [ ] Probar el escaneo en el teléfono del prefecto con el QR del alumno de prueba.
