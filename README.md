# Escribe con nosotros · Perpetuo

Formulario para que autores manden propuestas a Perpetuo. Reemplaza al Google Form "Perpetuo - Escribe con nosotros".

## Archivos
- `index.html`: todo el formulario (HTML, CSS y JS en un solo archivo, sin paso de compilación).
- `assets/`: logo y los cuatro signos de puntuación del brandbook (copiados de `quiniela-nobel`).

## Flujo
1. **Tú**: nombre, correo, país, suscripción (ya suscrito / suscríbanme), biografía breve opcional y un enlace opcional a su trabajo.
2. **Sección**: El estelar, Los anteojos, El creativo, La sobremesa, Foto ensayo, Vainas varias, Diseño y arte.
3. **Tu propuesta**: los campos cambian según la sección (son los mismos saltos que tenía el Google Form).
   - Estelar / Anteojos / Creativo: idea o texto listo, título, propuesta o archivo (Creativo pide además el género).
   - Sobremesa: un párrafo sobre la persona, WhatsApp y enlaces a trabajo previo (opcionales) y la pregunta «¿Tienes una propuesta o un borrador?»: con propuesta escribe la idea del texto; con borrador adjunta el documento. Si n8n falla, el formulario acomoda esos datos en título, biografía y propuesta para que el respaldo de Apps Script no pierda nada.
   - Vainas varias: descripción breve (se sugieren 50 palabras, sin bloquear) y archivo opcional.
   - Foto ensayo: descripción breve (se sugieren 50 palabras, sin bloquear), más un .zip o un enlace a las fotos.

Ningún campo exige un mínimo ni un máximo de caracteres o palabras: los obligatorios solo tienen que no estar vacíos.
   - Diseño y arte: qué quiere hacer y su portafolio.
4. **Envío**: repaso de las respuestas, luego el envío y una pantalla con el folio.

El borrador se guarda en `localStorage` (sin el archivo) y se borra cuando el envío sale bien.

## Configuración
```js
const CONFIG = { WEBHOOK_URL: "https://esperpetuo.app.n8n.cloud/webhook/escribe-con-nosotros", MAX_MB: 15, CLAVE_LOCAL: "perpetuo-escribe-borrador" };
```
Si se deja `WEBHOOK_URL` vacío, el formulario funciona en **modo demostración**: no envía nada y muestra en la consola lo que habría enviado.

### Lo que se envía al webhook
`POST` en `multipart/form-data` con estos campos: `nombre, correo, pais, suscripcion, bio, enlace, seccion, seccionNombre, modo, titulo, genero, propuesta, descripcion, enlaceFotos, arte, portafolio, sobreTi, whatsapp, enlacesPrevios, folio, fecha`. Si hay archivo, va en el campo binario `archivo`.
Cualquier respuesta 2xx cuenta como recibido; un 413 muestra el aviso de archivo demasiado pesado. El webhook debe permitir CORS desde el dominio donde se publique el formulario.

## Backend (n8n)
Workflow **"Escribe con nosotros · Propuestas (webhook → Drive + Textos)"** en esperpetuo.app.n8n.cloud:
1. Valida los campos (nombre, correo y sección) y responde 400 si falta alguno.
2. Si trae archivo, lo sube a la carpeta de Drive **"Escribe con nosotros · Archivos"**.
3. Crea una ficha en **📖 Textos** (Notion) con Nombre = título, Autor, Correo, País, Biografía, Propuesta (lo que escribió: la propuesta o, en Vainas varias, Foto ensayo y Diseño y arte, la descripción) y Enlace del texto (el archivo en Drive, o el enlace a las fotos o al portafolio). La propuesta completa, el folio y los demás datos van dentro de la página.
4. Responde 200 `{ok, folio}`. Si la sección es Sobremesa, además avisa en **#sobremesa** (Slack) con la ficha, el borrador y el WhatsApp.
   Si Notion falla, responde 500 y el formulario le pide a la persona intentar de nuevo.

| Sección del formulario | Sección | Categoría | Estado inicial |
|---|---|---|---|
| El estelar | Estelares | — | Estatus: Inbound |
| Los anteojos | Anteojos | Anteojo | Estatus: Inbound |
| El creativo | Creativo | Poema / Ensayo / Cuento / Otro | Etapa: Sin lectura |
| La sobremesa | Sobremesa (opción nueva) | — | Etapa: Sin lectura |
| Foto ensayo | — | Foto ensayo | Etapa: Sin lectura |
| Vainas varias | Creativo | VV | Etapa: Sin lectura |
| Diseño y arte | — | Otro | Estatus: Inbound |

CORS permitido desde: `https://escribe.perpetuo.global`, `https://perpetuo.global`, `https://www.perpetuo.global` y `http://localhost:8765`. Si el formulario se publica en otro dominio, hay que agregarlo en el nodo "Recibir propuesta".
