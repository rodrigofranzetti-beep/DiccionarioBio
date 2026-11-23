# Diccionario Bio-Emocional 2016

Reconstrucción del JSON a partir del PDF entregado por Rodrigo. Fecha: 23/11/2026.

## Archivos

- `diccionario-bioemocional-2016.json`: archivo principal, array de 2.096 fichas únicas.
- `referencias-bibliograficas-2016.json`: las 2.065 notas numeradas del apéndice, con letra, número, texto y páginas.
- `control-calidad.json`: recuentos, procedencia, deduplicación, remisiones sin destino, autorreferencias y correcciones de títulos documentadas.

## Alcance y controles

El cuerpo del diccionario está entre las páginas 15 y 667 del PDF de 721 páginas. Se identificaron 2.097 encabezados tipográficos. PARESTESIA aparece dos veces, con la misma remisión a ADORMECIMIENTO, en las páginas 510 y 511. Se conserva una única ficha con ambas páginas de origen.

Resultado: 1.216 fichas con contenido propio y 880 remisiones. Se preservan también las entradas anatómicas, conceptos y otros encabezados del libro; no se reclasifican como diagnósticos.

La doble comprobación fue técnica:

1. Extracción por tipografía y posición para separar títulos, secciones, referencias y números de página. Se asignaron las 18.546 líneas del cuerpo a sus fichas, sin líneas sin asignar ni líneas duplicadas.
2. Comprobación de los 2.097 encabezados con una segunda extracción de texto independiente. Ningún encabezado ausente. Comprobación de claves, IDs, destinos, secciones y grafo de remisiones.

Se verificaron visualmente páginas representativas del cuerpo, la repetición de PARESTESIA y las referencias. Esto no equivale a una revisión visual manual de las 721 páginas ni a una validación científica del contenido.

Se conservan los saltos de línea de la extracción del PDF. Se quitaron la cabecera del sitio, los números de página y los separadores alfabéticos. Las secciones no se mezclan. Los marcadores de cita en superíndice se representan como `[A:1]`, por ejemplo, y se relacionan con `citations`.

Los recuentos de etiquetas coinciden: 1.216 Definición, 1.212 Técnico y 1.210 Sentido biológico. Hay 1.209 etiquetas Conflicto en 1.208 fichas porque una ficha contiene dos etiquetas de esa sección; se conserva el texto de ambas.

## Referencias: límites del propio PDF

Hay 3.273 remisiones registradas:

- 3.199 tienen un destino resuelto dentro del diccionario.
- 66 no tienen un destino inequívoco en el PDF y llevan `target_id: null`.
- 8 son autorreferencias del libro y se conservan sin enlace navegable.

No se fabricaron fichas para cubrir remisiones ausentes del original. Las erratas claras y variantes de títulos se resolvieron hacia títulos existentes; el texto original queda en `label` y `v_source`, y el criterio figura en `resolution`. Las remisiones a subapartados explícitos se identifican como tales. Todas las decisiones están listadas en `control-calidad.json`.

Ejemplos de destinos ausentes o ambiguos del original: MIOBLASTOMA, BARTONELOSIS, DISTROFIA MACULAR VITELIFORME y algunas referencias generales como NERVIOS o INTESTINO. No se reemplazaron por un diagnóstico parecido.

Se vincularon 2.530 citas bibliográficas por ficha. Tres números citados no aparecen con esa letra y número en el apéndice: D:241 en DOLOR, G:127 en GLÁNDULAS SALIVALES y L:427 en LORDOSIS. Se preservan y se marcan como ausentes, sin adivinar una corrección. Los textos bibliográficos que dicen Ibíd. se mantienen así, tal como figuran en el PDF.

## Estructura del JSON principal

Se mantiene un array y los nombres compactos que ya consume la aplicación:

- `id`: entero consecutivo desde 0. Coincide con la posición del array.
- `t`: título de la ficha, tal como aparece en el PDF.
- `d`: definición, cuando existe.
- `tc`: texto técnico, cuando existe.
- `b`: sentido biológico, cuando existe.
- `c`: conflicto, cuando existe.
- `e`: etapa como string, solo cuando el texto técnico identifica una única etapa.
- `r`: booleano. true solo para una remisión sin contenido propio.
- `v`: texto de compatibilidad para mostrar las remisiones. NO usar para resolver enlaces dividiéndolo por comas.

Campos añadidos para evitar pérdida de información:

- `stages`: todas las etapas explícitamente indicadas junto a Etapa/Etapas en la sección técnica. La aplicación debe usar este array si quiere filtrar fichas de varias etapas.
- `v_source`: texto de las remisiones del PDF, sin sustituir sus títulos.
- `refs`: array de objetos con `label`, `target_id`, `target_title`, `status` y, cuando corresponde, `resolution`.
- `source`: documento, páginas y avisos de la ficha.
- `citations`: claves bibliográficas citadas, con texto y páginas del apéndice cuando existen. Las citas repetidas dentro de una ficha se listan una vez, pero sus marcadores se conservan en el texto.

Estados de `refs`:

- `resolved`: `target_id` apunta a una ficha existente.
- `unresolved_source_reference`: falta un destino inequívoco en el original.
- `self_reference_in_source`: la remisión vuelve a la misma ficha; se conserva el destino original en `source_target_id` y `source_target_title` para auditoría.

## Integración con la web

Se inspeccionó el repositorio en el commit `b0cd6a9777a66c8113b00de39f5c876dbe9cf664`:

https://github.com/rodrigofranzetti-beep/DiccionarioBio

https://diccionario-bio.vercel.app/

No se modificó GitHub ni el sitio.

El `index.html` actual contiene 2.552 objetos incrustados en `window.DATA`; no carga un archivo JSON externo. El número mostrado estático de entradas no es una medición fiable del array. También hay otro HTML del diccionario en el repositorio.

Para integrar el nuevo archivo, hace falta aprobar e implementar por separado:

1. Cargar el JSON antes de construir `searchIndex`, o sustituir el array incrustado. La web actual no descubrirá este archivo sola.
2. Crear los enlaces desde `refs` usando `target_id`. Desactivar los enlaces con destino nulo y mostrar una nota de remisión no resuelta. No dividir `v` por comas: hay títulos con comas y títulos con barras.
3. El código actual abre una entrada con `window.DATA[id]`. Este archivo hace coincidir ID y posición, empezando en 0. Los IDs antiguos empezaban en 1 y no deben mezclarse con estos.
4. Los favoritos y notas personales están guardados en el navegador por ID. Al cambiar el dataset, deben migrarse por título o guardarse con una nueva versión; no conservarlos asociados ciegamente a IDs antiguos.
5. Si se quieren filtrar todas las etapas de una ficha, usar `stages`. El filtro actual solo compara `e` contra una etapa.
6. Revisar la búsqueda de tildes: el código normaliza la consulta pero el índice no se normaliza del mismo modo.
7. El service worker usa caché offline primero y el nombre `bioemocional-v1`. Una publicación nueva debe revisar la actualización de la caché para no seguir sirviendo los datos viejos.
8. La pantalla modal de referencias no tiene una colección bibliográfica conectada en los datos actuales. `citations` y el archivo de notas permiten implementarla sin mezclarla con las remisiones entre fichas.

## Aviso sobre el contenido

Es una transcripción estructurada de una obra aportada por el usuario, no una fuente médica validada. Las afirmaciones del libro sobre causas emocionales de enfermedades no se presentan como hechos clínicos ni sustituyen diagnóstico o tratamiento profesional. No deben emplearse para suspender tratamientos o atribuir responsabilidad a una persona por su enfermedad. El archivo conserva contenido y terminología del original, incluso afirmaciones y clasificaciones que pueden resultar estigmatizantes.
