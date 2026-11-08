# Guía de compras

Página web para armar listas de compra a partir de un catálogo de productos de ferretería y repuestos (≈2.400 códigos). Se usa desde el celular o el computador: buscas un producto, eliges la cantidad, lo agregas a la lista y al final la imprimes o la guardas en PDF.

**Sitio publicado:** https://carrfane.github.io/guia-de-compra/

## Qué se puede hacer

- **Buscar productos** por código (`LT-8302`) o por palabras de la descripción (`gancho`, `filtro rojo`). Las sugerencias se eligen con el dedo, el mouse o las flechas del teclado.
- **Agregar con cantidad.** Si el producto ya está en la lista, se suman las unidades.
- **Agregar códigos que no están en el catálogo.** Quedan "Sin descripción", con la etiqueta *Código nuevo*.
- **Ver productos similares.** Con un código del catálogo, el botón *Ver productos similares* muestra hasta 15 productos con descripción parecida. Cada uno se puede elegir o agregar directo con *+ 1*. Ver [SIMILARES.md](SIMILARES.md).
- **Colores por proveedor.** Al tocar el código de un producto en la lista se le asigna uno de 8 colores. En *Colores y proveedores* se puede poner el nombre del proveedor de cada color. Hay una leyenda con los colores usados y un botón *Ordenar por color* para agrupar la lista por proveedor.
- **Editar la lista:** cambiar cantidades, quitar productos (con *Deshacer*) o vaciarla (hay que tocar dos veces).
- **Copiar la lista** como texto con tabuladores, para pegar en Excel, WhatsApp o un correo. Incluye la columna *Proveedor* si hay colores.
- **Imprimir / Guardar PDF**, con un diseño limpio: fecha, colores y leyenda, sin botones. El archivo se nombra `Guia-de-compras-AAAA-MM-DD`.
- **La lista queda guardada en el dispositivo** (IndexedDB del navegador). Al volver a la página sigue ahí. Solo se borra con *Vaciar lista*, o después de imprimir, si confirmas *Sí, vaciar lista*.

## Cómo está hecho

- Todo vive en un solo archivo: [`index.html`](index.html), con HTML, CSS, JavaScript y el catálogo. No hay build, dependencias ni servidor.
- El catálogo está embebido en la constante `CATALOG` como un arreglo `[["CÓDIGO","DESCRIPCIÓN"], …]`.
- Se publica con GitHub Pages desde la rama `main` (`.nojekyll` desactiva Jekyll). Cada push a `main` se publica en uno o dos minutos.
- La interfaz está en español y funciona en modo claro y oscuro.

## Probar en local

Basta abrir `index.html` en el navegador. Si quieres servirlo:

```bash
python3 -m http.server 8000
```

y luego abrir http://localhost:8000.

## Más contexto

- [CLAUDE.md](CLAUDE.md): contexto técnico y decisiones, para quien siga desarrollando (personas o Claude Code).
- [SIMILARES.md](SIMILARES.md): cómo se calculan los productos similares y cómo ajustarlo.
