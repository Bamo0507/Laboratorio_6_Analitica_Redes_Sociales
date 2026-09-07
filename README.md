# **Laboratorio 6. Análisis de redes sociales en YouTube**

## **Estructura del repositorio**

```
Laboratorio_6_Analitica_Redes_Sociales/
├── data/
│   ├── raw/          datos crudos, fuente de verdad, nunca se modifican
│   └── processed/    datos derivados, se regeneran con el notebook 02, no se versionan
├── notebooks/        un notebook por ejercicio del enunciado
├── requirements.txt  dependencias
├── codebook.md       diccionario de datos de los conjuntos derivados
└── README.md
```

---

## **Cómo ejecutar el análisis**

### **1. Crear el entorno virtual e instalar dependencias**

```bash
python3 -m venv .venv
source .venv/bin/activate          # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m spacy download es_core_news_sm
```

El modelo `es_core_news_sm` pesa unos 13 MB y es necesario para la lematización del ejercicio 2.
La primera ejecución del ejercicio 7 descarga además el modelo de sentimiento
`pysentimiento/robertuito-sentiment-analysis`, de aproximadamente 500 MB.

### **2. Abrir Jupyter**

```bash
jupyter notebook notebooks/
```

### **3. Ejecutar los notebooks en orden**

**El orden importa.** `data/processed/` no se versiona, así que al clonar el repositorio esa carpeta
está vacía y los notebooks del 03 al 10 fallarían. El notebook 02 es el que genera los datos
derivados y debe correrse primero.

| Orden | Notebook | Qué hace | Produce |
|---|---|---|---|
| 1 | `01_carga_integracion_datos.ipynb` | Carga, llaves e integración por `video_id` | nada |
| 2 | `02_calidad_limpieza_preprocesamiento.ipynb` | Diagnóstico, normalización, tipado y limpieza de texto | `comentarios_limpio.csv`, `videos_limpio.csv` |
| 3 | `03_analisis_exploratorio.ipynb` | Descriptivos, concentración y visualizaciones | nada |
| 4 | `04_red_bipartita_autor_video.ipynb` | Construcción de la red bipartita | `red_nodos.csv`, `red_aristas.csv` |
| 5 | `05_proyecciones_de_la_red.ipynb` | Proyecciones autor-autor y video-video | nada |
| 6 | `06_topologia_y_fragmentacion.ipynb` | Densidad, grados, componentes y cohesión | nada |
| 7 | `07_comunidades.ipynb` | Louvain y caracterización de comunidades | `comentarios_sentimiento.csv` |
| 8 | `08_nodos_centrales.ipynb` | Centralidad, puentes y articuladores | nada |
| 9 | `09_sentimiento.ipynb` | Sentimiento por video, canal y categoría | nada |
| 10 | `10_interpretacion_conclusiones.ipynb` | Interpretación, limitaciones y conclusiones | nada |

Los notebooks 08, 09 y 10 dependen únicamente de lo que producen el 02, el 04 y el 07, por lo que
basta con haber corrido esos tres antes.

---

## **Dependencias**

| Paquete | Para qué se usa |
|---|---|
| `pandas` | Manipulación de datos en todos los notebooks |
| `numpy` | Operaciones numéricas, viene como dependencia de pandas |
| `scipy` | Correlación de Spearman |
| `matplotlib` | Todas las visualizaciones |
| `networkx` | Construcción de redes, métricas topológicas y Louvain |
| `spacy` | Tokenización, stopwords y lematización en español |
| `emoji` | Detección y segmentación de emojis en el texto |
| `wordcloud` | Nube de palabras del ejercicio 3 |
| `pysentimiento` | Análisis de sentimiento en español, arrastra `transformers` y `torch` |
| `jupyter` | Ejecución de los notebooks |

---

## **Decisiones metodológicas que conviene conocer**

- **La semilla aleatoria es 123** en todo el proyecto: Louvain, los layouts de los grafos y la nube
  de palabras.
- **No se eliminó ningún registro** durante la limpieza. Los 406 comentarios llegan completos hasta
  el final, acorde a la política del curso de no descartar filas por valores faltantes.
- **Se conservan dos versiones del texto.** `texto_original` queda intacto para auditoría y para el
  análisis de sentimiento, donde mayúsculas, puntuación y emojis son señal; `texto_limpio` está
  lematizado y sin stopwords para el análisis de frecuencias.
- **Los emojis se conservan en `texto_limpio`.** El enunciado pide tratamiento de emojis, no
  eliminación, y quitarlos dejaba vacíos varios comentarios que sí comunican algo.
- **No se despojan tildes.** En español la tilde cambia el significado, así que en lugar de
  normalizar se corrigen las palabras mal escritas con un mapa curado de 43 entradas, documentado en
  el notebook 02.
- **36 de los 406 comentarios son respuestas, no comentarios de primer nivel.** Su `comment_id`
  tiene la forma `id_del_padre.id_de_la_respuesta`. El enunciado describe el archivo como si todos
  fueran principales, y no es así.

---

## **Limitaciones de los datos**

Solo 19 de los 293 videos tienen comentarios recolectados, un 6.5 % del catálogo, y varios de los
videos sin comentarios superan el millón de visualizaciones. Ese grado cero es un vacío del muestreo
y no una propiedad del contenido. Adicional, la muestra se armó con 21 consultas de búsqueda
dirigidas, las fechas de los comentarios son relativas y no absolutas, y un solo video concentra el
39.7 % de toda la participación observada. Las conclusiones aplican a esta muestra y no a YouTube
Guatemala en general.
