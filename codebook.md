# **Codebook**

Diccionario de datos de los conjuntos derivados que produce el pipeline de notebooks, a partir de
`data/raw/youtube_videos.csv` y `data/raw/youtube_comments.csv`.

Los cinco archivos viven en `data/processed/`, no se versionan y se regeneran ejecutando los
notebooks en el orden que indica el README.

| Archivo | Filas | Lo genera |
|---|---|---|
| `comentarios_limpio.csv` | 406 | Notebook 02 |
| `videos_limpio.csv` | 293 | Notebook 02 |
| `red_nodos.csv` | 351 | Notebook 04 |
| `red_aristas.csv` | 343 | Notebook 04 |
| `comentarios_sentimiento.csv` | 406 | Notebook 07 |

---

## **comentarios_limpio.csv**

Cada fila es un comentario publicado en un video. Llave primaria: `comment_id`.

| Variable | Tipo | Descripción | Valores válidos y notas |
|---|---|---|---|
| `comment_id` | str | Identificador único del comentario | 26 caracteres en los principales; 49 con un punto en las respuestas |
| `video_id` | str | Video donde se publicó | Llave foránea hacia `videos_limpio.video_id`, 19 valores distintos |
| `author_channel_id` | str | Cuenta que escribió el comentario | 24 caracteres, 332 valores distintos. Es el identificador de nodo de autor |
| `channel_id` | str | Canal dueño del video comentado | No es el autor del comentario. Universo disjunto de `author_channel_id` |
| `author_name` | str | Nombre visible del autor | En realidad es un handle: empieza con `@` en el 100 % de los registros |
| `handle_autor` | str | Handle normalizado | **Derivada.** Sin la barra inicial y con la codificación porcentual decodificada |
| `es_respuesta` | bool | Si el registro es una respuesta a otro comentario | **Derivada** del formato de `comment_id`. True en 36 de 406 |
| `comment_id_padre` | str | Comentario al que responde | **Derivada.** Vacía en los 370 comentarios de primer nivel |
| `me_gusta` | int | "Me gusta" recibidos | **Derivada** de `like_count_text`. Rango 0 a 405 |
| `respuestas` | int | Respuestas declaradas por el comentario | **Derivada** de `reply_count`. Rango 0 a 7. No identifica a los autores |
| `published_text` | str | Antigüedad del comentario | Tiempo relativo del tipo "hace 6 meses". **No convertible a fecha** |
| `texto_original` | str | Contenido del comentario sin modificar | Idéntico a la columna `text` del crudo. Se conserva para auditoría y sentimiento |
| `texto_limpio` | str | Contenido lematizado | **Derivada.** Ver reglas de limpieza abajo. Vacío en 3 registros |
| `urls` | list | URLs extraídas del texto | **Derivada.** 1 aparición en todo el conjunto |
| `hashtags` | list | Hashtags extraídos del texto | **Derivada.** 1 aparición en todo el conjunto |
| `menciones` | list | Menciones extraídas del texto | **Derivada.** 5 apariciones |
| `emojis` | list | Emojis presentes en el comentario | **Derivada.** 199 apariciones en 61 comentarios |

## **videos_limpio.csv**

Cada fila es un video del catálogo. Llave primaria: `video_id`.

| Variable | Tipo | Descripción | Valores válidos y notas |
|---|---|---|---|
| `video_id` | str | Identificador único del video | 11 caracteres, 293 valores |
| `channel_id` | str | Canal que publicó el video | 97 valores distintos |
| `channel_name` | str | Nombre visible del canal | Coincide uno a uno con `channel_id` |
| `handle_canal` | str | Handle del canal normalizado | **Derivada.** Sin la barra inicial |
| `title` | str | Título del video | 19 títulos se repiten dentro del mismo canal, pero son videos distintos |
| `category` | str | Categoría asignada por YouTube | 11 categorías; News & Politics domina con 138 videos |
| `source_query` | str | Consulta con la que se encontró el video | 21 valores. **Describe el muestreo, no el tema del video** |
| `source_group` | str | Estrategia de búsqueda | `topic` (177), `official_gov` (105) o `channel` (11) |
| `query_hits` | list | Todas las consultas que recuperaron el video | Solo 5 videos fueron recuperados por más de una |
| `keywords` | list | Etiquetas del video | Vacía en 162 de los 293 registros |
| `description` | str | Descripción completa | Vacía en 26 registros |
| `vistas` | int | Visualizaciones | **Derivada** de `view_count`. Rango 2 a 8,190,449 |
| `vistas_texto` | float | Visualizaciones según el texto mostrado por YouTube | **Derivada** de `view_count_text`. NaN en 13 registros. Difiere de `vistas` en 53 de 280 casos comparables |
| `publish_date` | str | Fecha y hora de publicación | ISO 8601 con zona horaria. Única variable temporal absoluta |
| `published_time` | str | Antigüedad del video | Tiempo relativo. **No convertible a fecha.** Vacía en 13 registros |

## **red_nodos.csv**

Cada fila es un nodo de la red bipartita observada.

| Variable | Tipo | Descripción |
|---|---|---|
| `id` | str | `author_channel_id` si es autor, `video_id` si es video |
| `tipo` | str | `autor` (332 nodos) o `video` (19 nodos) |
| `etiqueta` | str | Handle del autor o título del video, para visualización |
| `grado` | int | Vecinos distintos. En autores, videos comentados; en videos, autores distintos |
| `grado_ponderado` | int | Suma de los pesos de sus aristas, es decir, comentarios |
| `canal` | str | Canal dueño del video. Vacío en los nodos de autor |
| `categoria` | str | Categoría del video. Vacío en los nodos de autor |
| `vistas` | float | Visualizaciones del video. Vacío en los nodos de autor |
| `me_gusta_recibidos` | float | "Me gusta" que acumuló el autor. Vacío en los nodos de video |

## **red_aristas.csv**

Cada fila es una arista de la red bipartita. Una arista significa que ese autor comentó en ese video,
y nada más: no implica amistad, conversación ni acuerdo con el contenido.

| Variable | Tipo | Descripción |
|---|---|---|
| `origen_autor` | str | `author_channel_id` del autor |
| `destino_video` | str | `video_id` del video |
| `peso` | int | Comentarios que ese autor publicó en ese video. Rango 1 a 6; 40 aristas superan 1 |
| `autor` | str | Handle del autor, para lectura humana |
| `video` | str | Título del video, para lectura humana |
| `canal` | str | Canal dueño del video |

## **comentarios_sentimiento.csv**

Cada fila es un comentario con su clasificación de sentimiento.

| Variable | Tipo | Descripción |
|---|---|---|
| `comment_id` | str | Llave hacia `comentarios_limpio.csv` |
| `video_id` | str | Video donde se publicó |
| `sentimiento` | str | Clase predicha: `NEG` (249), `NEU` (79) o `POS` (78) |
| `prob_neg` | float | Probabilidad de la clase negativa, de 0 a 1 |
| `prob_neu` | float | Probabilidad de la clase neutra, de 0 a 1 |
| `prob_pos` | float | Probabilidad de la clase positiva, de 0 a 1 |
| `puntaje_sentimiento` | float | **Derivada.** `prob_pos` menos `prob_neg`, de -1 a 1. Media global -0.3787 |

El modelo es `pysentimiento/robertuito-sentiment-analysis`, entrenado en tuits en español, y se
aplica sobre `texto_original` porque mayúsculas, puntuación y emojis son señal para el clasificador.

---

## **Reglas de limpieza aplicadas**

1. **Marcadores de vacío unificados.** El crudo trae tres formas distintas de faltante: cadena
   vacía, espacio en blanco y lista vacía `[]`. Se identificaron y documentaron por columna.
2. **Identificadores intactos.** Ningún identificador se sustituyó por un nombre visible. Solo se
   recortó espacio sobrante, y no había ninguno.
3. **Handles normalizados.** Se quitó la barra inicial y se decodificó la codificación porcentual de
   14 handles de autor, por ejemplo `/@AlejandroP%C3%A9rez-b6r` pasó a `@AlejandroPérez-b6r`.
4. **Conteos convertidos a número.** Se eliminó la coma de miles y el texto acompañante. No se
   encontraron abreviaturas del tipo K o M.
5. **Texto limpio.** Se eliminaron caracteres invisibles y saltos de línea; se separaron URLs,
   hashtags y menciones a sus propias columnas; se lematizó con `es_core_news_sm`, descartando
   stopwords, puntuación, números y tokens de un carácter; los lemas se pasaron a minúscula después
   de lematizar, no antes.
6. **Emojis conservados en su posición.** Se segmenta cada comentario en tramos de texto y de emoji,
   se lematizan solo los tramos de texto y se reensambla en orden.
7. **Corrección ortográfica curada.** 43 entradas revisadas a mano que unifican palabras mal
   escritas, por ejemplo `pais` a `país` y `exelente` a `excelente`. **No se despojan tildes.** Tres
   pares se dejaron sin fusionar a propósito por ser palabras distintas: `ano` y `año`, `renuncie` y
   `renuncié`, `echo` y `hecho`.

---

## **Política de valores faltantes**

**No se imputó ningún valor y no se eliminó ninguna fila.** Los 406 comentarios y los 293 videos
llegan completos al dataset final. Rellenar es una decisión de análisis que corresponde al EDA, y
borrar filas por un faltante impondría a todos los análisis la restricción de uno solo.

Los faltantes del dataset final se clasifican así:

- **Ausentes en el origen.** `description` vacía en 26 videos, `keywords` vacía en 162,
  `published_time` y `vistas_texto` vacías en 13. Nunca se capturaron, y NaN es la representación
  honesta.
- **Vacíos por definición.** `comment_id_padre` está vacío en los 370 comentarios de primer nivel
  porque no tienen padre, y `canal`, `categoria`, `vistas` o `me_gusta_recibidos` están vacíos en
  `red_nodos.csv` según el tipo de nodo. No son faltantes sino ausencias esperadas.
- **Reparados, no inventados.** Los 189 `like_count_text` en blanco se convirtieron a 0, porque
  YouTube omite el número cuando un comentario no tiene "me gusta". Se verificó que esos blancos son
  el único origen de valores no convertibles en esa columna.
- **Descartados por inservibles.** `viewer_rating` estaba vacía en los 406 registros e `is_pinned`
  era constante en `False`, por lo que ninguna entra al dataset final.

---

## **Fuente de los datos**

Los archivos `youtube_videos.csv` y `youtube_comments.csv` fueron proporcionados por el curso CC3084
Data Science de la Universidad del Valle de Guatemala para el Laboratorio 6, semestre II 2026.
Provienen de una recolección dirigida sobre YouTube Guatemala mediante 21 consultas de búsqueda y
navegación de canales, ejecutada en un solo momento.
