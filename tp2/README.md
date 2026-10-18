# TP 2 — API de socios y cuenta corriente de un club

Un `openapi.yaml` que describe la API del TP final, la Gestión de Clubes ([demo.clubdesocios.com.ar](https://demo.clubdesocios.com.ar)): cada club tiene sus socios, y cada socio su cuenta corriente de cargos y pagos. Seis endpoints en cuatro paths, sin nada implementado: el entregable es el contrato.

### Cómo se usa

Abrir [openapi.yaml](openapi.yaml) en [editor.swagger.io](https://editor.swagger.io/?url=https://raw.githubusercontent.com/PabloSalvagni/salvagni-iisaia/main/tp2/openapi.yaml), que lo carga directamente. Del lado derecho aparece la documentación navegable, con cada endpoint desplegable y su "Try it out".

### Qué me propuse construir

El dominio que mejor conozco, porque es el de mi TP final, y en el que la jerarquía se justifica sola: un socio no existe fuera de su club, y un cargo o un pago no existen fuera de su socio. Por eso los tres niveles van anidados en el path:

```
/clubs/{slug}/socios
/clubs/{slug}/socios/{socioId}
/clubs/{slug}/socios/{socioId}/cargos
/clubs/{slug}/socios/{socioId}/pagos
```

Me quedé con la parte del admin del club que toca la plata: alta, consulta y baja de socios, consulta de la cuenta corriente y registro de pagos. El contrato describe un diseño, no documenta el código; al final anoto en qué se diferencia de lo que el TP final hace hoy.

### Cómo lo armé

Trabajé con dos herramientas. Con **Claude Code** planifiqué: elegí el dominio y el recorte, discutimos las alternativas y quedaron escritas una spec y un plan. Claude me propuso los endpoints, los schemas y las decisiones de diseño, y yo elegí o aprobé cada una. Después redactó los prompts y armó un verificador automático del yaml contra la spec.

El contrato lo dicté en **una sola conversación con otra IA**, enviando esos prompts. Después de cada respuesta pasé el yaml por el verificador y por un linter OpenAPI con Claude Code, y con esos hallazgos decidí qué corregir.

### Decisiones tomadas por mí

**El club va en el path y nunca en el body.** Ningún schema tiene `clubId`. El `slug` del path identifica al club y es la única vía: es la regla multi-tenant del TP final llevada al contrato. Si el club viajara también en el body, el contrato tendría que decir cuál gana cuando no coinciden.

**Schemas de entrada y de salida separados.** `SocioInput` no tiene `id`, `nroSocio`, `activo` ni `bajaAt`, porque los maneja el servidor. `PagoInput` no tiene `monto`: lo calcula el servidor sumando los cargos que se pagan. Si el cliente mandara el monto, podría no coincidir con los cargos, y de nuevo habría que decidir cuál vale.

**No hay `POST` de cargos.** Los cargos los genera el sistema (la cuota de cada período, las clases reservadas, los eventos). La única forma de cambiar un cargo es pagarlo, con `POST .../pagos`, igual que en el TP final.

**`DELETE` del socio es una baja lógica.** Devuelve `204` sin cuerpo y el socio queda `activo=false` con `bajaAt`; no se borra porque tiene historial de cargos y pagos. Usé `DELETE` y no un `PATCH` de `activo` porque, para quien consume la API, el socio sale del padrón activo: que el servidor conserve el registro es un detalle suyo. Y no hay `DELETE` de cargos ni de pagos: la deuda y los cobros no se borran.

**`?activo` y `?estado` van como query, no en el path.** Filtran una colección; no identifican un recurso distinto.

**`409` y no `400` para el DNI repetido.** El body está bien formado; lo que choca es el estado del servidor, donde ya hay un socio con ese DNI en el club.

**`404` y no lista vacía.** Si se piden los cargos de un socio que no existe, devolver `[]` diría que existe y no debe nada. Un socio sin cargos devuelve `200` con lista vacía; uno que no existe, `404`.

**`401` y `403` separados, definidos una sola vez.** `401` es no tener sesión; `403` es tenerla pero no ser admin de ese club. Están en `components/responses` y cada endpoint los referencia con `$ref`.

## Qué salió mal y cómo lo corregí

En el prompt 2 dicté el medio de pago solo para la entrada: `medio: EFECTIVO|TRANSFERENCIA|OTRO` en `PagoInput`. El modelo hizo lo razonable con eso: lo extrajo como un único schema `MedioPago` y lo usó también en `Pago`, la salida.

En la entrada está bien, porque un admin no registra a mano un pago online. Pero en la salida es un error: en el TP final el socio puede pagar desde la app, y ese pago queda con medio `ONLINE`. Con el contrato como estaba, un pago real no se podía representar. Lo grave es que el yaml era válido y pasaba el linter: el error no era de sintaxis, era de dominio.

Lo detectó Claude Code al revisar el yaml de P2, comparando los enums contra el modelo de datos del TP final; lo confirmé y lo corregí con el prompt 4. El arreglo fue el prompt 4: el medio de `Pago` tiene su propio enum, que suma `ONLINE`.

Igual que en el error clásico de repetir en el body un dato que ya está en el path, el problema vino del prompt y no del modelo. Pensé el pago desde lo que carga el admin y no desde lo que existe en el sistema. Lo que me llevo es que entrada y salida no se separan solo en los campos: también en los valores que acepta cada campo.

Hubo además un problema que no fue del contrato: la respuesta del prompt 1 era larga y se cortó al copiarla, y tuve que volver a copiarla entera. Por eso desde el prompt 2 pedí siempre "el yaml completo en un solo bloque de código".

## Versiones

| Prompt | Qué cambió | Estado |
|---|---|---|
| 1 | Socios (`GET` lista, `POST`, `GET` uno) y cargos (`GET`), con entrada y salida separadas | Válido; el linter pide seguridad |
| 2 | `DELETE` del socio (baja lógica) y `POST` de pagos | Válido; entra el error de `MedioPago` |
| 3 | Seguridad por cookie de sesión y `401`/`403` reutilizables | Pasa el linter sin errores |
| 4 | Corrección: `Pago.medio` con su propio enum, que incluye `ONLINE` | Final |

Después de cada versión revisé el yaml entero, no solo lo nuevo, para comprobar que el modelo no hubiera cambiado lo anterior.

### Contrato vs. implementación

El contrato no es una copia del TP final, y hay una diferencia a propósito: para el DNI repetido el contrato dice `409`, mientras que el TP final hoy responde `400`. Ahí la base de datos rechaza el duplicado y el manejador de errores de la API convierte todo error de negocio en `400`. El contrato muestra cómo debería ser.

## Prompts

El registro completo está en [prompts.md](prompts.md). El que más pesó fue el primero, que fija la jerarquía y la separación entre entrada y salida; el cuarto es la corrección.
