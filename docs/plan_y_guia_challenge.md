# PLAN DE PROYECTO + GUÍA PASO A PASO (PSEUDOCÓDIGO)
## Challenge 9 - Análisis de Datos de Producto

**Enfoque:** Simple, directo, sin complicaciones innecesarias.  
**Herramientas:** Python, Pandas, Matplotlib/Seaborn, Jupyter Notebook.

---

## FASE 0 - CONFIGURACIÓN INICIAL
**Objetivo:** Preparar el entorno y cargar el dataset.  
**Tiempo estimado:** 5 minutos.

### PSEUDOCÓDIGO:
```python
# Importar librerías necesarias
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Cargar el dataset
df = pd.read_csv("product_activity.csv")
```

**MÉTODOS A APRENDER:**
* `pd.read_csv()` → Carga un archivo CSV en un DataFrame.

---

## FASE 1 - EXPLORACIÓN INICIAL (Medir antes de limpiar)
**Objetivo:** Conocer el dataset tal cual está, sin modificar nada.  
**Criterio del challenge:** Regla 1.  
**Tiempo estimado:** 15 minutos.

### PASO 1.1 - Vista general del dataset
```python
# Ver las primeras 5 filas
df.head()

# Ver estructura: columnas, tipos de datos, nulos
df.info()

# Ver estadísticas numéricas
df.describe()
```

**MÉTODOS A APRENDER:**
* `df.head()` → Muestra las primeras N filas (default 5).
* `df.info()` → Muestra tipos de datos y conteo de no-nulos.
* `df.describe()` → Calcula estadísticas: media, min, max, etc.

### PASO 1.2 - Conteo de nulos y duplicados
```python
# Contar nulos por columna
df.isnull().sum()

# Contar filas duplicadas exactas
df.duplicated().sum()
```

**MÉTODOS A APRENDER:**
* `df.isnull()` → Devuelve True/False si el valor es nulo.
* `.sum()` → Suma los True (cuenta los nulos).
* `df.duplicated()` → Marca filas que son duplicados exactos.

### PASO 1.3 - Valores únicos y frecuencias
```python
# Para cada columna categórica, ver qué valores existen
df["plan_type"].value_counts()
df["post_category"].value_counts()
df["device_type"].value_counts()
```

**MÉTODOS A APRENDER:**
* `.value_counts()` → Cuenta la frecuencia de cada valor único.
* `.nunique()` → Cuenta cuántos valores únicos hay.

### PASO 1.4 - Chequeos lógicos
```python
# Convertir fechas temporalmente para el chequeo
signup = pd.to_datetime(df["created_at"], errors="coerce")
post   = pd.to_datetime(df["post_created_at"], errors="coerce")

# ¿Cuántos posts ocurren ANTES del signup?
posts_antes_signup = (post < signup).sum()
print(f"Posts antes del signup: {posts_antes_signup}")

# ¿Days_since_signup es consistente?
dias_calculados = (post - signup).dt.days
dias_originales = df["days_since_signup"]
inconsistentes  = (dias_calculados != dias_originales).sum()
print(f"Days_since_signup inconsistentes: {inconsistentes}")
```

**MÉTODOS A APRENDER:**
* `pd.to_datetime()` → Convierte texto a formato fecha.
* `errors="coerce"` → Los que no se pueden convertir quedan NaT.
* `.dt.days` → Extrae los días de una diferencia de fechas.

---

## FASE 2 - LIMPIEZA BÁSICA CON CRITERIO
**Objetivo:** Normalizar valores sucios y separar datos con errores.  
**Criterio del challenge:** Regla 2.  
**Tiempo estimado:** 25 minutos.

### PASO 2.1 - Eliminar duplicados exactos
```python
duplicados_antes = df.duplicated().sum()
df = df.drop_duplicates()
duplicados_removidos = duplicados_antes
```

**MÉTODOS A APRENDER:**
* `df.drop_duplicates()` → Elimina filas duplicadas, conserva la primera.

### PASO 2.2 - Normalización canónica (mapear variantes)
```python
# Paso previo: poner todo en minúsculas y quitar espacios
df["plan_type"]     = df["plan_type"].str.strip().str.lower()
df["device_type"]   = df["device_type"].str.strip().str.lower()
df["post_category"] = df["post_category"].str.strip().str.lower()

# Definir diccionarios de mapeo para variantes conocidas
# (Los valores exactos dependerán de lo que encuentres en 1.3)

# Diccionarios fijos según el challenge:
# plan_type:     {free, pro, enterprise}
# device_type:   {web, mobile, desktop}
# post_category: {tech, life, sports, science, finance, gaming, music, health, education, travel}

mapeo_plan = {
    "variante_encontrada_1": "free",
    "variante_encontrada_2": "pro",
    # ... completar según lo que encuentres
}

mapeo_device = {
    # ... completar según lo que encuentres
}

mapeo_category = {
    # ... completar según lo que encuentres
}

# Aplicar los mapeos
df["plan_type"]     = df["plan_type"].replace(mapeo_plan)
df["device_type"]   = df["device_type"].replace(mapeo_device)
df["post_category"] = df["post_category"].replace(mapeo_category)
```

**MÉTODOS A APRENDER:**
* `.str.strip()` → Elimina espacios en blanco al inicio y final.
* `.str.lower()` → Convierte todo el texto a minúsculas.
* `.replace()` → Reemplaza valores según un diccionario de mapeo.

### PASO 2.3 - Convertir fechas a datetime
```python
df["created_at"]      = pd.to_datetime(df["created_at"], errors="coerce")
df["post_created_at"] = pd.to_datetime(df["post_created_at"], errors="coerce")

# Reportar fechas no parseables
fechas_nulas_signup = df["created_at"].isna().sum()
fechas_nulas_post   = df["post_created_at"].isna().sum()
print(f"Fechas no parseables - signup: {fechas_nulas_signup}")
print(f"Fechas no parseables - post: {fechas_nulas_post}")
```

**MÉTODOS A APRENDER:**
* `pd.to_datetime()` → Convierte a datetime.
* `.isna()` → Detecta valores NaT (fecha nula).

### PASO 2.4 - Recálculo obligatorio de days_since_signup
```python
# Crear la columna recalculada
df["days_since_signup_calc"] = (df["post_created_at"] - df["created_at"]).dt.days

# Comparar con el original
mismatches = (df["days_since_signup"] != df["days_since_signup_calc"]).sum()
print(f"Mismatches en days_since_signup: {mismatches}")

# A partir de aquí, USAR days_since_signup_calc para todo el análisis
```

**MÉTODOS A APRENDER:**
* Resta de fechas → Produce un Timedelta.
* `.dt.days` → Extrae la cantidad de días del Timedelta.

### PASO 2.5 - Quarantine (Cuarentena)
```python
# Identificar filas con errores duros y asignar motivo

# Condición 1: post_created_at < created_at (post antes del signup)
cond_fecha = df["post_created_at"] < df["created_at"]

# Condición 2: fechas no parseables (NaT)
cond_nat = df["created_at"].isna() | df["post_created_at"].isna()

# Condición 3: valores que no están en el diccionario fijo
planes_validos     = ["free", "pro", "enterprise"]
devices_validos    = ["web", "mobile", "desktop"]
categorias_validas = ["tech", "life", "sports", "science", "finance",
                      "gaming", "music", "health", "education", "travel"]

cond_plan   = ~df["plan_type"].isin(planes_validos)
cond_device = ~df["device_type"].isin(devices_validos)
cond_cat    = ~df["post_category"].isin(categorias_validas)

# Crear columna reason_code
df["reason_code"] = ""
df.loc[cond_fecha,  "reason_code"] += "post_antes_signup;"
df.loc[cond_nat,    "reason_code"] += "fecha_no_parseable;"
df.loc[cond_plan,   "reason_code"] += "plan_invalido;"
df.loc[cond_device, "reason_code"] += "device_invalido;"
df.loc[cond_cat,    "reason_code"] += "categoria_invalida;"

# Separar: cuarentena vs core
mascara_quarantine = df["reason_code"] != ""
df_quarantine = df[mascara_quarantine].copy()
df_core       = df[~mascara_quarantine].copy()

# Limpiar columna temporal
df_core = df_core.drop(columns=["reason_code"])
```

**MÉTODOS A APRENDER:**
* `.isin()` → Verifica si cada valor está en una lista dada.
* `~` (tilde) → Operador NOT, invierte True/False.
* `.loc[]` → Accede a filas por condición booleana.
* `.copy()` → Crea una copia independiente del DataFrame.
* `.drop(columns=)` → Elimina columnas especificadas.

---

## FASE 3 - DATA QUALITY REPORT
**Objetivo:** Presentar un resumen del estado de los datos.  
**Criterio del challenge:** Regla 3.  
**Tiempo estimado:** 10 minutos.

### PSEUDOCÓDIGO:
```python
filas_raw          = len(df)           # Total original
filas_core         = len(df_core)      # Total limpio
filas_quarantine   = len(df_quarantine)
pct_quarantine     = (filas_quarantine / filas_raw) * 100
pct_mismatches     = (mismatches / filas_raw) * 100

# Presentar tabla resumen (ejemplo manual o con DF)
```

| Métrica | Valor |
| :--- | :--- |
| Filas RAW (originales) | `filas_raw` |
| Filas CORE (limpias) | `filas_core` |
| Filas Quarantine | `filas_quarantine` |
| % Quarantine | `pct_quarantine` % |
| Duplicados removidos | `duplicados_removidos` |
| % Mismatches en fechas | `pct_mismatches` % |

**MÉTODOS A APRENDER:**
* `len()` → Cuenta el número de filas.
* `pd.DataFrame()` → Crea un DataFrame a partir de un diccionario.

---

## FASE 4 - MÉTRICAS Y ANÁLISIS
**Objetivo:** Calcular métricas descriptivas y segmentaciones.  
**Criterio del challenge:** Regla 4.  
**Tiempo estimado:** 30 minutos.  
*IMPORTANTE:* Desde aquí, trabajar SOLO con `df_core`.

### PASO 4.1 - Distribuciones (Volumen)
```python
# Usuarios únicos por plan
df_core.groupby("plan_type")["user_id"].nunique()

# Actividad (#posts) por país, categoría y dispositivo
df_core["country"].value_counts()
df_core["post_category"].value_counts()
df_core["device_type"].value_counts()

# Visualizar con gráficos de barras
df_core["plan_type"].value_counts().plot(kind="bar", title="Posts por Plan")
plt.show()
```

**MÉTODOS A APRENDER:**
* `.groupby()` → Agrupa datos por una columna.
* `.nunique()` → Cuenta valores únicos dentro del grupo.
* `.plot(kind="bar")` → Genera un gráfico de barras.
* `plt.show()` → Muestra el gráfico en pantalla.

### PASO 4.2 - Engagement (Votos)
```python
# Votos por plan: media y mediana
df_core.groupby("plan_type")["votes_received"].agg(["mean", "median"])

# Votos por país, categoría y dispositivo
df_core.groupby("country")["votes_received"].agg(["mean", "median"])
df_core.groupby("post_category")["votes_received"].agg(["mean", "median"])
df_core.groupby("device_type")["votes_received"].agg(["mean", "median"])
```

**MÉTODOS A APRENDER:**
* `.agg()` → Aplica múltiples funciones de agregación a la vez.
* `"mean"` → Calcula el promedio.
* `"median"` → Calcula la mediana.

### PASO 4.3 - Promedios e Interpretación
```python
# Promedio de votos por plan
votos_por_plan = df_core.groupby("plan_type")["votes_received"].mean()

# Posts promedio por usuario
posts_por_usuario = df_core.groupby("user_id")["post_id"].count().mean()
```

**OBLIGATORIO:** Escribir en Markdown una celda explicando:
* "La unidad de análisis es el EVENTO (cada fila = un post)."
* "Un usuario con muchos posts pesa más en el promedio general."
* "Esto puede generar sesgo si pocos usuarios dominan la actividad."
* "Los outliers (usuarios con votos extremos) distorsionan la media."

**MÉTODOS A APRENDER:**
* `.mean()` → Calcula el promedio de una serie.
* `.count()` → Cuenta el número de elementos.

### PASO 4.4 - Evento vs Usuario
```python
# Promedio de votos por FILA (nivel evento)
promedio_por_evento = df_core["votes_received"].mean()

# Promedio de votos agrupado por USUARIO
promedio_por_usuario = df_core.groupby("user_id")["votes_received"].mean().mean()

print(f"Promedio por evento: {promedio_por_evento}")
print(f"Promedio por usuario: {promedio_por_usuario}")
```

**OBLIGATORIO:** Escribir en Markdown una celda explicando:
* "Difieren porque en el nivel evento, los usuarios con más posts tienen más peso en el promedio (cada post es una fila)."
* "En el nivel usuario, primero se calcula el promedio de cada usuario y luego se promedian todos, dando peso igual a cada uno."

**MÉTODOS A APRENDER:**
* `.mean().mean()` → Doble mean: primero por grupo, luego general.

---

## FASE 5 - CONCENTRACIÓN Y TEMPORALIDAD
**Objetivo:** Medir si la actividad está concentrada en pocos usuarios.  
**Criterio del challenge:** Regla 5.  
**Tiempo estimado:** 20 minutos.

### PASO 5.1 - Concentración (Top 1%)
```python
# Posts por usuario
posts_usuario = df_core.groupby("user_id")["post_id"].count()

# Votos por usuario
votos_usuario = df_core.groupby("user_id")["votes_received"].sum()

# Calcular el umbral del top 1%
umbral_posts = posts_usuario.quantile(0.99)
umbral_votos = votos_usuario.quantile(0.99)

# Filtrar top 1%
top_posts = posts_usuario[posts_usuario >= umbral_posts]
top_votos = votos_usuario[votos_usuario >= umbral_votos]

# Calcular porcentaje del total
pct_posts_top1 = top_posts.sum() / posts_usuario.sum() * 100
pct_votos_top1 = top_votos.sum() / votos_usuario.sum() * 100

print(f"El top 1% genera {pct_posts_top1:.2f}% de los posts y {pct_votos_top1:.2f}% de los votos")
```

**MÉTODOS A APRENDER:**
* `.quantile(0.99)` → Calcula el percentil 99 (umbral del top 1%).
* `.sum()` → Suma total de los valores.

### PASO 5.2 - Tendencia temporal
```python
# Crear columna de semana y mes
df_core["semana"] = df_core["post_created_at"].dt.isocalendar().week
df_core["mes"]    = df_core["post_created_at"].dt.to_period("M")

# Actividad por mes
actividad_mes = df_core.groupby("mes")["post_id"].count()
actividad_mes.plot(kind="line", title="Actividad por Mes")
plt.show()

# Engagement por mes
engagement_mes = df_core.groupby("mes")["votes_received"].mean()
engagement_mes.plot(kind="line", title="Engagement por Mes")
plt.show()
```

**MÉTODOS A APRENDER:**
* `.dt.to_period("M")` → Convierte fecha a período mensual.
* `.dt.isocalendar().week` → Extrae el número de semana ISO.
* `.plot(kind="line")` → Genera un gráfico de línea.

---

## FASE 6 - PRODUCT DECISIONS
**Objetivo:** Responder preguntas de negocio con evidencia.  
**Criterio del challenge:** Regla 6.  
**Tiempo estimado:** 15 minutos.

### PSEUDOCÓDIGO:
*No hay código nuevo aquí. Es redacción en celdas Markdown.*
*Usar los datos ya calculados para responder:*

* **PREGUNTA 1:** ¿Qué segmento priorizarías y por qué?
* **PREGUNTA 2:** ¿Qué parte del tablero "mentía" antes de limpiar?
* **PREGUNTA 3:** ¿Qué nuevo dato agregarías al tracking?

**ACCIONES (2 acciones concretas):**
* Proponer acciones basadas en hallazgos con respaldo visual (tabla/gráfico).
* Mencionar las limitaciones del dataset.

---

## FASE 7 - EXPORTACIÓN DE ARCHIVOS
**Objetivo:** Generar los 3 CSV obligatorios.  
**Criterio del challenge:** Entregables.  
**Tiempo estimado:** 5 minutos.

### PSEUDOCÓDIGO:
```python
# 1. Dataset limpio
df_core.to_csv("clean_product_activity.csv", index=False)

# 2. Dataset de cuarentena
df_quarantine.to_csv("quarantine_product_activity.csv", index=False)

# 3. Tabla resumen de métricas
metricas = pd.DataFrame({
    "metrica": ["usuarios_unicos", "total_posts_core", "promedio_votos_evento", "promedio_votos_usuario", "pct_posts_top1", "pct_votos_top1"],
    "valor":   [df_core["user_id"].nunique(), len(df_core), promedio_por_evento, promedio_por_usuario, pct_posts_top1, pct_votos_top1]
})
metricas.to_csv("metrics_summary.csv", index=False)
```

**MÉTODOS A APRENDER:**
* `.to_csv()` → Exporta el DataFrame a un archivo CSV.
* `index=False` → No incluye el índice como columna en el CSV.

---

## RESUMEN DE TODOS LOS MÉTODOS UTILIZADOS

| Categoría | Métodos |
| :--- | :--- |
| **Carga y Estructura** | `pd.read_csv()`, `df.head()`, `df.info()`, `df.describe()` |
| **Detección de Problemas** | `df.isnull().sum()`, `df.duplicated().sum()`, `.value_counts()`, `.nunique()` |
| **Limpieza** | `.str.strip()`, `.str.lower()`, `.replace()`, `pd.to_datetime()`, `df.drop_duplicates()`, `.isin()`, `.loc[]`, `.copy()`, `.drop(columns=)` |
| **Análisis** | `.groupby()`, `.agg()`, `.mean()`, `.median()`, `.sum()`, `.count()`, `.quantile()` |
| **Fechas** | `.dt.days`, `.dt.to_period("M")`, `.dt.isocalendar().week` |
| **Visualización** | `.plot(kind="bar")`, `.plot(kind="line")`, `plt.show()` |
| **Exportación** | `.to_csv()` |

---

## ESTRUCTURA FINAL DEL NOTEBOOK

1. **Celda 1:** [Código] Imports + Carga del CSV
2. **Celda 2:** [Código] `head()`, `info()`, `describe()`
3. **Celda 3:** [Código] Nulos y duplicados
4. **Celda 4:** [Código] `value_counts()` de categóricas
5. **Celda 5:** [Código] Chequeos lógicos (fechas, `days_since_signup`)
6. **Celda 6:** [Markdown] Resumen de hallazgos de exploración
7. **Celda 7:** [Código] Eliminar duplicados
8. **Celda 8:** [Código] Normalización canónica (mapeos)
9. **Celda 9:** [Código] Conversión de fechas
10. **Celda 10:** [Código] Recálculo de `days_since_signup_calc`
11. **Celda 11:** [Código] Quarantine (separar errores duros)
12. **Celda 12:** [Código] Data Quality Report (tabla resumen)
13. **Celda 13:** [Markdown] Interpretación del DQR
14. **Celda 14:** [Código] Distribuciones (volumen) + gráficos
15. **Celda 15:** [Código] Engagement (votos) por segmento
16. **Celda 16:** [Código] Promedios por plan y posts por usuario
17. **Celda 17:** [Markdown] Explicación unidad de análisis y sesgos
18. **Celda 18:** [Código] Evento vs Usuario
19. **Celda 19:** [Markdown] Explicación de por qué difieren
20. **Celda 20:** [Código] Concentración (top 1%)
21. **Celda 21:** [Código] Tendencia temporal + gráficos
22. **Celda 22:** [Markdown] Product Decisions (preguntas y acciones)
23. **Celda 23:** [Código] Exportación de los 3 CSV

---

## ARCHIVOS DE SALIDA ESPERADOS
1. `clean_product_activity.csv`
2. `quarantine_product_activity.csv`
3. `metrics_summary.csv`
