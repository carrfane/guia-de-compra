# Productos similares

El catálogo no trae categorías: solo código y descripción. Por eso la similitud se calcula **a partir del texto de la descripción**, en el navegador, cuando se toca *Ver productos similares* por primera vez. Indexar unos 2.400 productos tarda alrededor de 100 ms.

El código está en `index.html`, en la sección `// ---- Productos similares ----`: `simStem`, `simTokens`, `buildSimIndex`, `findSimilar`, `showSimilar` y `renderSimilar`.

## Qué hace que dos productos sean "similares"

Revisando el catálogo se ve que la **primera palabra de la descripción suele ser el tipo de producto**: LLAVE, CLAVO, ENCHUFE, PLUMILLAS, DISCO, LUZ, COPLA, EXTRACTOR… Las palabras siguientes son variantes: material, medida, color o marca. Además, muchas familias comparten la raíz del código (`9066-16`, `9066-18`, `9066-20`; `HL-L-009-RO`, `HL-L-009-BL`).

## Algoritmo

1. **Normalizar:** minúsculas, sin tildes, separar por todo lo que no sea letra o número.
2. **Descartar palabras vacías** (`SIM_STOP`): de, del, la, para, con, sin, x, pcs, piezas, set, kit, juego, jgo, blister…
3. **Raíz simple** (`simStem`) para unir plurales y género:
   - `luces → luz` (terminación `-ces` → `z`)
   - `interruptores → interruptor` (`-es` tras consonante)
   - `clavos → clavo` (`-s`)
   - `roja / rojo → roj` (quita la `a`/`o` final si la palabra tiene más de 4 letras)
4. **Pesos (TF-IDF binario):** cada palabra pesa `log(1 + N / df)`, donde `df` es en cuántos productos aparece. Las palabras raras pesan más que las comunes. Encima de eso:
   - **Tipo de producto** (primera palabra significativa): peso × 2.
   - **Números y medidas** (`12v`, `1/2`, `4.2x38mm`): peso × 0.5, para que salgan los otros tamaños.
   - **Colores** (`SIM_COLORS`): peso × 0.4, para que salgan los otros colores.
5. **Puntaje** = similitud coseno entre los dos vectores de pesos, más dos bonos:
   - **+0.15** si comparten el tipo de producto.
   - **+0.10** si comparten la familia de código: el código sin su último segmento tras un separador (`9066-20` → `9066`), con al menos 3 caracteres.
6. Se muestran los **15 mejores con puntaje ≥ 0.2**. Si hay empate, se ordenan por código.

Los códigos que no están en el catálogo no tienen descripción, así que con ellos el botón aparece desactivado.

## Ejemplos (catálogo actual)

| Producto | Primeros similares |
|---|---|
| `Y00803` LLAVE STILSON 12YTS | Y00801/02/04 (Stilson 8, 10 y 14), luego TP-10M LLAVE 10… |
| `9066-20` PLUMILLAS IRM GOMA 20 | 9066-16/18/22, luego 9077-xx PLUMILLAS IRM METAL |
| `30523` CLAVO ROSCADO 4.8X45MM | Otros CLAVO ROSCADO de distintas medidas |
| `HL-L-009-RO` LUZ LED CON GOMA ROJA | Versiones blanca y amarilla, luego otras LUZ LED ROJA |
| `CP50` COPLA 50MM PVC SANITARIA | CP40, luego otras COPLA PVC |
| `TN0300` DISCO LIJA TRASLAPADO METAL P40 | Otros discos de lija (metal y madera), luego discos de corte |

**Caso débil conocido:** `LT-8302` AUTO OIL FILTER ROJO. El tipo detectado es "auto", así que después de su par `LT-8303` salen ampolletas "AUTO LED".

## Cómo ajustarlo

- **Sale ruido de otro tipo de producto:** subir el bono por tipo (`+0.15`) o el umbral (`0.2`).
- **Faltan variantes del mismo producto:** bajar el peso de números o colores, o subir el bono de familia de código.
- **Palabras genéricas que se toman como tipo** (p. ej. "auto", "moto", "kit"): agregarlas a `SIM_STOP` para que el tipo sea la siguiente palabra.
- **Abreviaturas del catálogo** (`JGO`, `P/`, `SPTO`…): agregarlas a `SIM_STOP` o normalizarlas en `simTokens`.

Para probar cambios rápido, se puede extraer `CATALOG` a un JSON y correr la misma lógica en Node para varios códigos, comparando los resultados antes y después.
