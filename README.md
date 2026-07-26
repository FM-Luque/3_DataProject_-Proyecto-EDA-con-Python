# Análisis Exploratorio de Datos — Campañas de Marketing Bancario

Proyecto de Análisis Exploratorio de Datos (EDA) sobre campañas de marketing directo (llamadas telefónicas) de una institución bancaria portuguesa, cuyo objetivo era conseguir la suscripción de un depósito a plazo por parte de los clientes.

## 1. Objetivo del proyecto

Realizar un análisis exploratorio completo —transformación y limpieza de datos, análisis descriptivo, visualización e informe de conclusiones— para entender qué factores se asocian a que un cliente suscriba el depósito a plazo bancario.

## 2. Estructura del repositorio

```
├── data/
│   ├── raw/                            # Datos originales, sin modificar
│   │   ├── bank-additional.csv
│   │   └── customer-details.xlsx
│   └── processed/                      # Datos generados en cada fase
│       ├── 01_datos_integrados.csv     # Salida de la Fase 1 (carga + integración)
│       └── 02_datos_limpios.csv        # Salida de la Fase 2 (limpieza + transformación)
│
├── notebooks/
│   ├── 01_carga_integracion.ipynb      # Fase 1: Carga e integración de datos
│   ├── 02_limpieza_transformacion.ipynb# Fase 2: Limpieza y transformación
│   └── 03_eda_visualizacion.ipynb      # Fase 3: EDA descriptivo y visualización
│
├── informe/
│   └── INFORME_ANALISIS.md             # Fase 4: Informe explicativo y conclusiones
│
├── requirements.txt
└── README.md
```

## 3. Fuentes de datos

Los datos están relacionados con campañas de *marketing* directo de una institución bancaria portuguesa, basadas en llamadas telefónicas. A menudo era necesario más de un contacto con el mismo cliente para determinar si suscribiría o no el producto (depósito a plazo).

- **`bank-additional.csv`** — información de la campaña: datos del cliente (edad, profesión, estado civil, educación, historial de impagos/préstamos), datos del contacto (canal, duración, número de contactos, resultado de campaña anterior), indicadores macroeconómicos del momento del contacto, y la variable objetivo `y` (¿suscribió o no?).
- **`customer-details.xlsx`** — información demográfica y de comportamiento de los clientes (ingreso anual, hijos/adolescentes en el hogar, fecha de alta como cliente, visitas mensuales a la web), repartida en 3 hojas según el año de alta del cliente (2012, 2013, 2014).

Ambos datasets se unen a través de un identificador único de cliente.

## 4. Herramientas utilizadas

- Python 3
- Pandas / NumPy
- Matplotlib / Seaborn
- Jupyter Notebook (Visual Studio Code)

## 5. Fases del Proyecto

El proyecto se ha estructurado de manera que se divide en Fases, esta estructura se ejecuta para facilitar su comprensión e interpretacion:

#### Fase 1. Carge e interpretación de los datos.

    La Carga de datos implica la Importación los datos desde una o varias fuentes y preparar un único dataset sobre el que trabajar.

    La Interpretación de los datos es crucial antes de limipiar y analizar los datos, porque debemos entender qué continte el DataFrame.

#### Fase 2. Limpieza y transformación de los datos.

En esta Fase se realiza la correccion de errores, tratamiento de datos incompletos y dejaremos el DataFrame preparado para el análisis en la Fase 3.

###### Flujo de la limpieza y transformación de datos

0. Carga de datos integrados y Exploracion inicial
1. Modificar datos para mejor comprensión si es necesario (Nombre columna)
2. Corregir tipos de datos
3. Tratar valores nulos
4. Eliminar duplicados
5. Limpiar datos inconsistentes
6. Eliminar variables irrelevantes
7. Detectar y tratar outliers
8. Validación final del resultado
9. Guardar el dataset limpio

#### Fase 3. Análisis Exploratorio de Datos (EDA): análisis descriptivo y visualizacion

El análisis exploratorio de datos (EDA) permite entender, comprender y detectar el comportamiento de los datos anteriormente limpiados y transformados mediante el uso de estaditacas y su visualización usando gráficos básicos.

###### Flujo del Análisis Exploratorio de Datos

0. Cargar el dataset limpio y comprobar la calidad de dato.
1. Realizar un Recordatorio Pre-EDA. (Revisar la estructura del dataset ya limpio).
2. Identificar variables categóricas y numéricas.
3. Realizar un Análisis descriptivo general.
4. Analizar variables univariantes categóricas.
5. Analizar variables univariantes numericas.
6. Análisis bivariante y su relación con la variable clave ( y )
7. Análisis de la relación entre variables numéricas
8. Análisis de la evolución temporal
9. Preguntas de negocio adicionales
10. Realización de un insights destacado para realizar el informe en Fase 4.

#### Fase 4. Realización del informe y conclusiones

En esta fase realizamos un informe esplicativo con las conclusiones y recomendaciones de negocio

## Resumen de las fases del proyecto

|  Fase  | Contenido                                                                                                                                       | Notebook / archivo                   |
| :----: | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| Fase 1 | Carga de`bank-additional.csv` y `customer-details.xlsx`, unión de las 3 hojas del Excel y `merge` de ambos datasets por el ID de cliente | `01_carga_integracion.ipynb`       |
| Fase 2 | Corrección de tipos de datos, tratamiento de nulos, revisión de duplicados e inconsistencias de texto, y análisis de outliers (IQR)          | `02_limpieza_transformacion.ipynb` |
| Fase 3 | Análisis descriptivo y visualización (univariante, bivariante frente a la variable objetivo, correlaciones y evolución temporal)             | `03_eda_visualizacion.ipynb`       |
| Fase 4 | Informe explicativo con las conclusiones del análisis y recomendaciones de negocio                                                             | `informe/INFORME_ANALISIS.md`      |

## Principales hallazgos

- Solo el **11,25 %** de los clientes contactados suscribió el depósito: el dataset está claramente desbalanceado.
- La **duración de la llamada** es el predictor individual más fuerte de conversión (correlación +0,41).
- El **historial de contacto previo** es la señal más potente: los clientes ya contactados antes (o con una campaña anterior exitosa) convierten a tasas de 60-65 %, muy por encima del ~9-11 % del resto.
- **`student`** y **`retired`** son las profesiones con mayor tasa de conversión (25-31 %).
- El canal **`cellular`** convierte más del doble que **`telephone`**.
- No se observa una tendencia temporal clara en los 5 años del dataset.

El análisis completo, con la justificación de cada transformación y todas las visualizaciones, está disponible en el INFORME_ANALISIS.md .
