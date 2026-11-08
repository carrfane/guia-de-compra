# Contexto del proyecto (para próximas sesiones)

Proyecto personal: una página estática con una **guía de compras** para armar listas de productos a partir de un catálogo de ferretería y repuestos. La usa el dueño (y quien compra) desde el celular. La interfaz y la comunicación con el usuario son **en español**.

- Sitio: https://carrfane.github.io/guia-de-compra/ (GitHub Pages, rama `main`)
- Funcionalidades para el usuario: ver [README.md](README.md)
- Algoritmo de similares: ver [SIMILARES.md](SIMILARES.md)

## Forma de trabajo acordada

- **Hacer push directo a `main`**: es un proyecto personal, el usuario lo pidió explícitamente. No hace falta abrir PRs. Pages publica en uno o dos minutos.
- Antes de subir cambios, probar en un navegador headless (Playwright con Chromium). Sirve el archivo con un servidor HTTP local, emula un iPhone y revisa que no haya `pageerror`. También conviene probar la recarga (que la lista persista) y el modo impresión (`page.emulateMedia({ media: 'print' })`).
- Mantener todo en **un solo archivo** (`index.html`): sin build, sin dependencias y sin librerías externas, salvo Google Fonts (Barlow / Barlow Condensed).
- `index.html` pesa unos 130 KB, sobre todo por la línea de `CATALOG`. Conviene editarlo con reemplazos puntuales y no leerlo entero.

## Estructura de `index.html`

En orden:

1. **CSS** con tokens en `:root`: modo oscuro con `prefers-color-scheme` y `[data-theme]`, diseño móvil (en ≤560px la tabla pasa a grid) y `@media print`. El modo impresión oculta la entrada, las herramientas, los paneles y los botones, y muestra la fecha y las cantidades como texto.
2. **HTML**:
   - Sección `.entry`: buscador `#q` (combobox con `#sug`), `#qty`, `#add`, `#simBtn` y el panel `#simPanel`.
   - Herramientas: `#copy`, `#print`, `#sortColor`, `#clear`.
   - Paneles: `<details id="providers">`, `#saved`, el aviso post-impresión `#after`, `#legend`, la tabla `#rows`, el popover `#picker` y el toast `#toast`.
3. **JS**:
   - `const CATALOG = [["CÓDIGO","DESCRIPCIÓN"], …]` en una sola línea larga.
   - `strip()` (minúsculas y sin tildes) y `esc()` (escapa HTML; siempre se usa al insertar texto con `innerHTML`). `byCode` es un Map de código normalizado → item, e `index` es el arreglo de búsqueda.
   - `// ---- Guardado en el navegador ----`: IndexedDB y paleta `COLORS`.
   - Búsqueda (`search`, `updateEntry`, `pick`, `addCurrent`), `render()` y eventos.
   - `// ---- Productos similares ----`
   - `// ---- Colores / proveedores ----`: picker, leyenda, panel de proveedores y ordenar.
   - Al final: `render()` inicial y luego `load().then(render…)`.

## Datos y persistencia

- **IndexedDB** con base `guia-compras` y object store `lista`:
  - Registro `actual`: `{ items: [{code, qty, color?}], savedAt }`. El orden del arreglo es el orden de la lista; lo nuevo va arriba.
  - Registro `proveedores`: `{ colorId: "Nombre" }`. **No** se borra al vaciar la lista.
- Respaldo en `localStorage` si IndexedDB falla: `guia-compras-v1` para los items y `guia-compras-proveedores` para los proveedores. La primera versión del sitio guardaba en `guia-compras-v1`, y `load()` lo migra a IndexedDB y borra la clave antigua.
- Las escrituras se encadenan en una promesa (`writeChain`) para que no se pisen. `save()` no hace nada hasta que `load()` termina (`ready`).
- Se llama `navigator.storage.persist()` para pedir que el navegador no borre los datos.
- `cleanItems()` valida lo cargado: elimina duplicados por código normalizado, limita la cantidad a 1–99999 y acepta códigos fuera del catálogo y colores válidos.

## Decisiones tomadas

- **Borrado de la lista:** solo con *Vaciar lista* (pide un segundo toque) o después de *Imprimir / Guardar PDF*. El navegador no informa si el PDF se guardó o se canceló, así que se muestra el aviso `#after` con *Sí, vaciar lista* / *No, seguir editando*, en vez de borrar automáticamente.
- **Códigos fuera del catálogo:** se permiten. Se normalizan con `normCode` (mayúsculas, espacios colapsados, máximo 40 caracteres) y se muestran "Sin descripción" con la etiqueta *Código nuevo* (oculta al imprimir). Los duplicados se comparan con `strip()`, así que `zz-1` y `ZZ-1` son el mismo código.
- **Colores = proveedores:** paleta fija de 8 colores claros (amarillo, naranja, rojo, rosa, morado, azul, verde y gris), con texto oscuro para que se lea bien en ambos temas. Se asignan tocando el código en la lista (botón `.chip`). Al imprimir se usa `print-color-adjust: exact`. *Ordenar por color* reordena la lista de verdad (orden estable, sin color al final).
- **Similares:** calculados en el cliente con TF-IDF sobre las descripciones, con prioridad al tipo de producto (primera palabra) y a la familia de código. Detalles en SIMILARES.md. *+ 1* agrega una unidad directo; tocar el resultado lo pasa al buscador.

## Historial resumido

1. Página de guía de compras con catálogo, búsqueda y lista.
2. Imprimir / Guardar PDF.
3. Persistencia en IndexedDB y códigos fuera del catálogo.
4. Colores por proveedor: picker, nombres, leyenda, ordenar y columna en copiar.
5. Búsqueda de productos similares.
6. Documentación: README.md, CLAUDE.md y SIMILARES.md.

## Ideas pendientes / posibles mejoras

- Botón de "similares" también desde cada fila de la lista.
- Ajustar los pesos de similares si el usuario reporta casos malos (ver "Cómo ajustarlo" en SIMILARES.md).
- Sincronizar la lista entre dispositivos (hoy vive solo en el navegador de cada dispositivo).
- Actualizar el catálogo: reemplazar el arreglo `CATALOG` manteniendo el formato `[código, descripción]`.
