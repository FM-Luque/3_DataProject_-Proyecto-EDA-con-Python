# Informe y Conclusiones del Análisis — Campañas de Marketing Bancario

**Proyecto:** Análisis Exploratorio de Datos (EDA) sobre campañas de marketing directo de una institución bancaria portuguesa.

**Fuentes de datos:** `bank-additional.csv` (datos de la campaña) y `customer-details.xlsx` (datos demográficos de los clientes, en 3 hojas: 2012, 2013, 2014).

**Notebooks del proyecto:** `01_carga_integracion.ipynb` → `02_limpieza_transformacion.ipynb` → `03_eda_visualizacion.ipynb`

---

## 1. ¿Cómo era el dataset original?

El proyecto partía de **dos fuentes independientes**:

- **`bank-additional.csv`**: 43.000 registros y 24 columnas con información de la campaña (edad, profesión, estado civil, educación, historial de impagos/préstamos, canal y duración del contacto, número de contactos, resultado de campañas anteriores, indicadores macroeconómicos y variable objetivo `y`).
- **`customer-details.xlsx`**: repartido en 3 hojas (2012, 2013, 2014) según el año de alta del cliente, con 5 variables demográficas (`Income`, `Kidhome`, `Teenhome`, `Dt_Customer`, `NumWebVisitsMonth`) y un identificador único (`ID`).

Ambas fuentes se integraron mediante un `merge` por el identificador de cliente (`id_` / `ID`), obteniendo un único dataset de **43.000 filas y 29 columnas**. El 100 % de los registros de la campaña encontraron información demográfica correspondiente, por lo que no se perdió ningún registro en la integración.

Desde el primer vistazo (`head()`, `info()`, `describe()`) se detectó que el dataset **no estaba listo para el análisis**: columnas numéricas cargadas como texto, fechas en formato no estándar, categorías con formato inconsistente y un volumen relevante de valores nulos.

## 2. ¿Qué problemas se detectaron durante la limpieza?

- **Tipos de datos incorrectos**: `cons.price.idx`, `cons.conf.idx`, `euribor3m` y `nr.employed` estaban almacenadas como texto porque usaban la **coma como separador decimal** (ej. `"93,994"`). La columna `date` venía en formato texto `día-mes_en_español-año` (ej. `2-agosto-2019`), y `Dt_Customer` también como texto.
- **Inconsistencias de formato**: `marital` y `poutcome` mezclaban mayúsculas (`MARRIED`, `NONEXISTENT`) con el resto de categóricas en minúsculas.
- **Variables binarias poco legibles**: `default`, `housing` y `loan` estaban codificadas como 0.0/1.0 en lugar de un texto interpretable.
- **Valores nulos** en 9 columnas, con distinta magnitud: desde un 0,2 % (`marital`) hasta un **21,5 %** (`euribor3m`) y un **11,9 %** (`age`).
- **Columna basura**: `Unnamed: 0`, un índice residual de la exportación original sin ningún valor analítico.
- **Código especial no documentado**: la columna `pdays` usa el valor `999` para representar *"cliente nunca contactado antes"*, algo que no es un error pero que distorsiona cualquier estadística descriptiva o de outliers si no se trata de forma diferenciada.

No se encontraron registros duplicados, ni columnas completamente constantes.

## 3. ¿Qué transformaciones se realizaron?

| Ámbito            | Transformación aplicada                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Tipos              | `date` y `Dt_Customer` → `datetime`; indicadores económicos → `float` (coma → punto); `default`/`housing`/`loan` → texto (`"si"`/`"no"`)                                                                                                                                                                                                                                                                                                 |
| Formato            | `marital`, `poutcome` → minúsculas; verificación de espacios en blanco en todas las categóricas                                                                                                                                                                                                                                                                                                                                                         |
| Nulos numéricos   | `age`, `cons.price.idx`, `euribor3m` → imputados con la **mediana** (robusta frente a outliers)                                                                                                                                                                                                                                                                                                                                                    |
| Nulos categóricos | `job`, `marital`, `education`, `default`, `housing`, `loan` → categoría **`"desconocido"`** (más honesto que asumir la moda)                                                                                                                                                                                                                                                                                                             |
| Nulos en`date`   | 248 filas (0,6 %) eliminadas, al ser una variable clave para el análisis temporal                                                                                                                                                                                                                                                                                                                                                                              |
| Columnas           | Eliminada`Unnamed: 0` en ambos datasets; `hoja_origen` renombrada a `anio_alta_cliente`                                                                                                                                                                                                                                                                                                                                                                   |
| Variable derivada  | Creada`contactado_previamente` (`"si"`/`"no"`) a partir del código especial `pdays = 999`, para poder analizarlo sin distorsionar las estadísticas de `pdays`                                                                                                                                                                                                                                                                                       |
| Outliers           | Analizados con el método IQR sobre`age`, `duration`, `campaign`, `previous`, `Income` y `NumWebVisitsMonth`. **Se decidió no eliminar ni recortar ningún valor**: los "outliers" detectados corresponden a observaciones reales de negocio (llamadas largas, clientes muy insistidos, clientes jubilados), y en el caso de `previous` el método IQR resulta directamente poco fiable por la alta concentración de ceros en la variable |

El dataset final tras la limpieza (`02_datos_limpios.csv`) contiene **42.752 filas y 30 columnas**, sin nulos ni duplicados.

## 4. ¿Qué patrones se encontraron en el EDA?

- **El dataset está desbalanceado**: solo el **11,25 %** de los clientes contactados suscribió el depósito a plazo.
- **La duración de la llamada** es la variable con el patrón más claro: la mediana de duración en clientes que suscriben (≈449 s) es casi el triple que en los que no (≈163 s).
- **El historial de contacto previo es determinante**: los clientes que ya habían sido contactados antes convierten al **≈64 %**, frente al **≈9,2 %** de los clientes nuevos; si la campaña anterior tuvo éxito (`poutcome = success`), la conversión llega al **≈65 %**.
- **El perfil profesional influye claramente**: `student` (≈31 %) y `retired` (≈25 %) presentan tasas de conversión muy superiores a la media, mientras que `blue-collar` y `services` convierten peor (≈7-8 %).
- **El canal de contacto importa**: `cellular` convierte más del doble que `telephone` (14,7 % vs 5,2 %).
- **La edad**, el **número de contactos en la campaña actual** y las **variables demográficas del Excel** (`Income`, `NumWebVisitsMonth`) muestran poca o nula relación con la conversión.
- **No existe una tendencia temporal clara**: la tasa de suscripción mensual se mantiene estable (entre el 8,6 % y el 14,3 %) a lo largo de los 5 años del dataset, sin estacionalidad evidente.

## 5. ¿Qué variables parecen más relevantes?

Ordenadas por su relación (correlación / diferencia de tasas) con la variable objetivo `y`:

1. **`duration`** (correlación +0,41) — la más relevante de todo el dataset.
2. **`poutcome`** / **`contactado_previamente`** — el mayor salto de tasa de conversión observado en todo el análisis (de ~9 % a ~65 %).
3. **`previous`** (correlación +0,23) — coherente con el punto anterior.
4. **`job`** — diferencias de hasta 24 puntos porcentuales entre profesiones.
5. **`contact`** — el canal duplica la tasa de conversión.
6. **Indicadores macroeconómicos** (`nr.employed`, `pdays`, `emp.var.rate`, `euribor3m`) — correlaciones moderadas (entre -0,27 y -0,36), coherentes con el ciclo económico.

Por el contrario, **`Income`**, **`NumWebVisitsMonth`**, **`age`** y **`campaign`** no muestran una relación relevante con la conversión en este dataset.

## 6. ¿Existen relaciones entre variables?

- Existe una relación clara y consistente entre **historial de contacto previo** (`previous`, `poutcome`, `contactado_previamente`) y la probabilidad de conversión — probablemente la relación más fuerte de todo el análisis, tanto por tamaño de efecto como por consistencia entre las distintas variables que la reflejan.
- Los **indicadores macroeconómicos** (`emp.var.rate`, `euribor3m`, `nr.employed`) están fuertemente correlacionados entre sí, ya que reflejan el mismo ciclo económico visto desde distintos ángulos — es esperable y no indica un error en los datos.
- No se detecta relación relevante entre las variables demográficas (`Income`, `NumWebVisitsMonth`, `Kidhome`, `Teenhome`) y la variable objetivo, ni tampoco correlación lineal clara entre `age` y `duration` en el análisis bivariante (scatterplot).
- La duración de la llamada se relaciona con la conversión de forma consistente **en todas las profesiones** (no es un efecto explicado por un único segmento), lo que refuerza su fiabilidad como señal.

## 7. Conclusiones y recomendaciones

**Conclusión principal:** la probabilidad de que un cliente suscriba el depósito depende, sobre todo, de **cómo se desarrolla el propio contacto** (duración de la llamada, canal, y si el cliente ya tenía relación previa con campañas anteriores) — mucho más que de sus características demográficas o económicas fijas.

**Recomendaciones de negocio derivadas del análisis:**

1. **Priorizar clientes con contacto previo**, especialmente aquellos con una campaña anterior exitosa: son, por amplio margen, el segmento con mayor probabilidad de conversión.
2. **Favorecer el canal `cellular`** sobre `telephone` siempre que sea posible.
3. **Focalizar esfuerzos comerciales en estudiantes y jubilados**, segmentos con tasas de conversión muy superiores a la media, y revisar el guion/oferta para perfiles como `blue-collar` o `services`, donde la conversión es baja.
4. **No sobrecargar de contactos** a un mismo cliente dentro de la misma campaña: `campaign` no mejora la conversión y añade coste comercial.
5. Dado que la duración de la llamada correlaciona con la conversión, podría ser útil **formar a los agentes en mantener conversaciones más largas y de calidad** en lugar de llamadas breves y transaccionales — aunque conviene recordar que esta relación es una correlación observacional, no necesariamente causal (es posible que llamadas más largas ocurran *porque* el cliente ya está interesado, y no al revés).

**Limitaciones del análisis:**

- Las variables demográficas del Excel (`Income`, `Kidhome`, `Teenhome`, `NumWebVisitsMonth`) no mostraron relación con la conversión; no puede descartarse que estén generadas de forma independiente a la campaña real, por lo que sus conclusiones deben tomarse con cautela.
- Este es un análisis descriptivo (EDA), no un modelo predictivo: las relaciones encontradas son asociaciones, no relaciones causales confirmadas.
