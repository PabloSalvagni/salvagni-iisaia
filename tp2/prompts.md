# Prompts — TP 2

El registro del proceso, en orden. Cuatro prompts en una sola conversación: tres para construir el contrato y uno para corregir un error que apareció al revisarlo.

Los prompts los redacté con asistencia de Claude Code a partir de la spec; los envié tal cual a la conversación del contrato.

---

## 1 — Prompt inicial: socios y cargos

```
Necesito un openapi.yaml (OpenAPI 3.1) para la API de un club que gestiona
sus socios y su cuenta corriente. Devolvémelo completo en un solo bloque de código.

recursos (entrada y salida son schemas distintos):
  SocioInput  { nombre, apellido, dni, email?, tipoSocioId }
  Socio       { id, nroSocio, nombre, apellido, dni, email?, tipoSocioId, activo, bajaAt? }
                id y nroSocio los genera el servidor
  Cargo       { id, concepto: CUOTA|CLASE|EVENTO, descripcion, periodo "YYYY-MM",
                monto entero en pesos, vencimiento fecha, estado: PENDIENTE|PAGADO|ANULADO }
                solo de salida: los cargos los genera el sistema
  Error       { error: string }

endpoints:
  GET  /clubs/{slug}/socios?activo=            → 200 lista / 404 si el club no existe
  POST /clubs/{slug}/socios                    → 201 / 400 si falta un campo / 404 / 409 si el DNI ya existe
  GET  /clubs/{slug}/socios/{socioId}          → 200 / 404
  GET  /clubs/{slug}/socios/{socioId}/cargos?estado=  → 200 lista / 404 si el socio no existe
```

**Qué buscaba:** fijar la jerarquía club → socio → cargos y separar los schemas de entrada y de salida desde el arranque. Si no lo pido, el modelo tiende a reusar `Socio` como entrada con el `id` opcional, y se pierde que `id` y `nroSocio` los genera el servidor.

Volvió el yaml con los cuatro endpoints en tres paths, `SocioInput` y `Socio` separados, `Cargo` solo de salida, `EstadoCargo` como schema reutilizado en el filtro `?estado=`, y respuestas `NotFound` y `BadRequest` reutilizables. Agregó por su cuenta restricciones que no le pedí: `pattern` para el DNI y el `slug`, `minLength` y `additionalProperties: false` en la entrada.

La respuesta era larga y se cortó al copiarla; tuve que volver a copiarla entera.

---

## 2 — Baja lógica y pagos

```
Agregar dos endpoints sin tocar lo que ya está:

  DELETE /clubs/{slug}/socios/{socioId}   → 204 sin cuerpo / 404
     es una baja lógica: el socio queda activo=false con bajaAt, no se borra
     porque tiene historial de cargos y pagos

  POST /clubs/{slug}/socios/{socioId}/pagos   → 201 / 400 / 404
     entrada PagoInput { cargoIds: lista de ids (mínimo 1), medio: EFECTIVO|TRANSFERENCIA|OTRO }
     salida  Pago      { id, monto, medio, fecha (fecha y hora), cargos: lista de Cargo }
     el monto lo calcula el servidor sumando los cargos; no viaja en la entrada
     400 si algún cargo no está PENDIENTE o no es de ese socio

Devolvé el yaml completo en un solo bloque de código.
```

**Qué buscaba:** completar las escrituras con su semántica de negocio. Pedí `204` explícito y aclaré que es una baja lógica, porque si no el modelo tiende a devolver `200` con el objeto. Y quería comprobar dos cosas: que no metiera `monto` en la entrada, y que no tocara nada de lo anterior.

Las dos se cumplieron (lo comparé contra la versión anterior). Pero acá entró el error que corrijo en el prompt 4: el modelo extrajo un único enum `MedioPago` y lo usó en la entrada y en la salida.

---

## 3 — Seguridad

```
Agregar seguridad sin tocar los endpoints:
  - un securityScheme de sesión por cookie, aplicado a toda la API
  - respuestas 401 (sin sesión) y 403 (no es admin de ese club), definidas
    una sola vez en components/responses y referenciadas con $ref en todos los endpoints

Devolvé el yaml completo en un solo bloque de código.
```

**Qué buscaba:** una iteración sobre algo transversal en vez de un endpoint más, y ver si el modelo reutiliza o repite. Lo pedí explícito con `$ref` porque si no suele copiar el mismo bloque de `401` en cada endpoint.

Volvió con `security` global, el scheme `sessionCookie` y `Unauthorized` / `Forbidden` definidos una vez y referenciados en los seis endpoints. Es el primer yaml que pasa el linter sin errores.

---

## 4 — Corrección: el medio de pago de la salida

```
MedioPago se reusa en PagoInput y en Pago, y eso está mal: en la entrada
solo van EFECTIVO|TRANSFERENCIA|OTRO (un pago ONLINE no lo carga un admin),
pero en la salida un Pago puede ser ONLINE si el socio pagó desde la app.
Separalo: el medio de Pago acepta EFECTIVO|TRANSFERENCIA|ONLINE|OTRO.
No toques nada más. Devolvé el yaml completo en un solo bloque de código.
```

**Qué buscaba:** corregir el error del prompt 2. Ver el detalle en el README.

Separó el enum de `Pago.medio` y le agregó una descripción que explica por qué difiere del de la entrada. No tocó nada más.

---

## Conversación completa

Una sola conversación, sin reiniciar el hilo. El yaml final tiene 6 endpoints repartidos en 4 paths.
