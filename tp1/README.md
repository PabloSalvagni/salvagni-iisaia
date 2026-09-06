# Prompts — TP 1

El registro del proceso, en orden. Tres prompts en una sola conversación de Gemini Canvas. El artefacto quedó terminado en el tercero.

---

## 1 — Prompt inicial

```
Construir un formulario de contacto que tenga un boton de ENVIAR dificil de encontrar para hacerle click.

Estructura:
- <header> con el titulo "Formulario de contacto ludico" 
- <main> que tenga el formulario, con 4 campos <input≥: y labels nombre y apellido, celular, email y asunto, un <textarea> para mensaje. 
- <footer> con un <button> centrado que diga "Comenzar envio de memsaje" y debajo un <small> con el texto "tienes 10 segundos para enviar el mensaje, o este se autodestruira" 

Estilo: 
- Estetica del formulario es MS DOS, con fondos negro y textos en verde con imputs en blanco y sin dedondear
- utilizar tipografias windows o alguna monospace 


Comportamiento:
- Estado inicial es de un formulario con 5 campos, 4 inputs y 1 text area.
- Una vez completo los 5 campos del formulario se habilita el boton que inicialmente esta disabled = true
- Al completarse los campos, se habilita el boton para se cliickeado.
- Al hacer click comienza un contador debajo del boton que va de 10 a 0 segundos.
- El boton cambia el texto a "Enviar" y tiene la "habilidad" de moverse cuando el usuario se acerca hasta 50px
- Si se acerca el mouse en posision x el boton de mueve en ese sentido, tanto de forma positiva como de negativa viendo de donde viene el mouse. Si se acerca en posicion y, se mueve en esas coordenadas, hacia abajo o hacia arriba.
- Cuando el boton se mueve y toca los bordes, recien ahi se queda quieto para poder hacer hover y 
- Si el contador llega a 0, se borran los textos cargados y se cambia el boton por el boton iniical.
- Si el usuario logra clickear el boton, aparece un mensaje sobre el formulario con el texto "Muy bien hecho, el mensaje fue enviado"

Constraints
- Un solo archivo HTML, con el CSS en un <style> y el JS en un <script>
- Vanilla JS, sin frameworks ni dependencias externas.
- El boton, textos y elementos del DOM son re posicionados con
  CSS. No usar <canvas>: quiero poder ver el estado reflejado en el DOM.
```

**Que buscaba lograr:** generarle al usuario estar atento y despierto al completar un formulario y enviarlo, que en general suele ser una tarea mas automatica de completar y hacer click de forma automatica.

**Que devolvio:** la estetica solicitada y las relgas de comportamiento muy exacto a lo pedido en el prompt. 

---



## 2 — Iterar sobre lo hecho para que sea reutilizable sin refrescar el navegador.

```
Agregar luego de que el mensaje se envia un timeout de 5 segundos, que al momento de llegar a 0 vuelva al estado inicial con campos vacios y boton con texto inicial.
```

**Que se intento lograr:** darle mejor usabilidad, y que no dependa de hacer F5 o recargar la pagina. Ya tiene la poca usabilidad de enviar el mensaje.

**Que devolvio:** el timeout y funcionalidd correcta. Luego de unos segundos se reseteaba el formulario.

---



## 3 - Agregado de otra dificultad para hacerlo mas ludico y poco funcional al Formulario

```
Agregar un captcha dentro de un <footer> arrriba del boton "y debajo un <small> con el texto "tienes 10 segundos para enviar el mensaje, o este se autodestrui" basado en el juego/puzzle llamado "Juego del 15" o "Taken" pero que en lugar de ser de 15 piezas, sea de 9, con 9 lugares y 8 piezas. 
Que tenga un <small> debajo indicando la tearea a realizar: "debe ordenar los numeros de 1 a 8, de izquierda a derecha, de arriba hacia abajo"
 
Estilo: 
- mantenga la estetica del formulario y tenga un tamaño de 180x180px

Comportamiento:
- que cada ficha tenga un numero
- que los numeros aparezcan al azar, 
- que no tenga timeout, el tiempo es ilimitado.
- cambiar la habilitacion del boton Enviar y que se hablite el timeout una vez que se ordena correctamente
```

**Que se intento lograr:** dificultar el envio y agregar un Captcha, componente que evita el uso de robots.

**Que devolvio:** Devolvio un juego simple que logro agregarle dificultad, pero termino siendo demasiado compleja la tarea e iba a llevar mucho tiempo, por lo que se decide bajar esta dificultad a 9 lugares. 

## 4 - Bajando de dificiltad del Captcha

```
Ajustar la dificultad del puzzle, reduciendolo a 6 lugares y 5 piezsa.
Tambien reducir tamaño a 120x180px con 2 filas y 3 columnas
```

**Que se intento lograr:** simplificar el juego y no demorar tanto la tarea final que es enviar el formulario.

**Que devlovio:** un captcha mas ludico y divertido para validar que el usuario no es un robot. 