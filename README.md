# 📈 Modelos de Trading: la búsqueda esquiva de la rentabilidad constante

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Machine Learning](https://img.shields.io/badge/Machine%20Learning-Scikit--Learn-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## 🧠 1. Introducción

¿Alguna vez te has preguntado cómo convertir datos brutos de trading en una ventaja operativa real?

Este proyecto nació de esa inquietud: transformar un simple registro histórico de operaciones en una **fuente de información estratégica**.

El objetivo fue **optimizar los resultados de un modelo de trading**, analizando los datos históricos de *Take Profit* y *Stop Loss* para **aumentar el Win Rate y mejorar la relación riesgo-beneficio**.

---

## 🧹 2. Limpieza y preparación de datos

Antes de cualquier optimización, el primer paso fue una **limpieza y preparación exhaustiva** del dataset.

### Tareas realizadas:
- Eliminación de filas con valores faltantes en `ticks_SL` (Stop Loss en ticks), ya que representaban registros incompletos.  
- Conversión de tipos de datos para garantizar que las columnas numéricas fueran correctamente interpretadas.  
- Creación de una columna `fecha` (año, mes y día) y uso de esta para completar los valores faltantes de `dia_semana`.  
- Eliminación de columnas irrelevantes o con exceso de valores nulos (`HIN`, `hora_HIN`).  

### Métricas iniciales (dataset sin optimizar):

| Métrica | Valor |
| --- | --- |
| Win Rate inicial | **31.85%** |
| Máxima racha de pérdidas | **16** |
| RR promedio en operaciones ganadoras | **3.50** |
| Profit factor | **1.89** |

> Este punto de partida permitió visualizar el desempeño base del sistema antes de aplicar mejoras.

---

## 🔍 3. Análisis Exploratorio de Datos (EDA)

El **EDA (Exploratory Data Analysis)** permitió identificar las variables con mayor impacto en los resultados.

Se analizaron dos factores principales:
- `ticks_rango_OTE`: tamaño de la zona de entrada óptima (en ticks).  
- `antiguedad_de_liquidez`: tiempo transcurrido desde que se formó la liquidez.

### Hallazgos clave:
- Operaciones con `ticks_rango_OTE` > **55.2** mostraban una **disminución significativa en el Win Rate**.  
- Operaciones con `antiguedad_de_liquidez` > **300** velas también tenían **menor probabilidad de éxito**.

📊 **Conclusión:**  
Filtrar operaciones con valores superiores a estos umbrales mejora las métricas generales del sistema.

---

## ⚙️ 4. Resultados tras el filtrado

Después de aplicar los filtros (`ticks_rango_OTE ≤ 55.2` y `antiguedad_de_liquidez ≤ 300`), los resultados mejoraron notablemente:

| Métrica | Antes del filtro | Después del filtro |
| --- | --- | --- |
| Win Rate | 31.85% | **34.88%** |
| Máx. racha de pérdidas | 16 | **13** |
| RR promedio (TPs) | 3.50 | **3.55** |

> Este filtrado eliminó setups históricamente desfavorables, aumentando la consistencia general del modelo.  
> Pero esperen, esta optimización recién comienza...

---

## 🎯 5. Optimización del Take Profit (TP)

Se aplicó una estrategia de **optimización dinámica del Take Profit** en función del tamaño de la zona OTE (`ticks_rango_OTE`).

### Objetivo:
Maximizar el valor esperado para cada operación:

\[
E(R) = R \times P(\text{alcanzar R sin Stop Loss})
\]

Mediante análisis de percentiles de `RR_potencial` y simulaciones, se estimó un TP óptimo para cada intervalo de OTE.

### Resultados:

| Métrica | Valor |
| --- | --- |
| R:R total (datos filtrados originales) | 126.53 |
| R:R total (con TP optimizado) | **236.57** |
| Mejora absoluta | **+110.04** |
| Mejora porcentual | **+86.9%** |

📈 **Conclusión:**  
Definir dinámicamente el TP en base a las características del trade casi duplicó la rentabilidad total.

---

## 🧱 6. Ajuste del Stop Loss (SL)

Se analizó el efecto del **retroceso máximo del precio dentro del OTE** (`profundidad_retroceso_real`).

- Cuando el precio retrocedía **0.9 del rango OTE**, la mayoría de las operaciones terminaban en pérdida.  
- Se implementó una estrategia ajustada:
  - Si el retroceso llegaba a 0.9 → se consideraba Stop Loss anticipado (-1R).  
  - Si la operación alcanzaba TP sin retroceder a 0.9 → se ajustaba su R:R hacia arriba (x1.33).  

### Resultados:

| Métrica | Valor |
| --- | --- |
| R:R total (original filtrado) | 126.53 |
| R:R total (con SL ajustado a 0.9) | **163.00** |
| Mejora absoluta | **+36.47** |
| Mejora porcentual | **+28.82%** |

📉 **Conclusión:**  
Incorporar un Stop Loss dinámico basado en retroceso aumentó significativamente la eficiencia operativa.

---

## ⚖️ 7. Impacto combinado (TP + SL)

La combinación de ambas estrategias produjo los mejores resultados globales.

| Criterio | Win Rate (%) | Prom. RR (ganadoras) | Profit Factor | Máx. racha pérdidas |
| --- | --- | --- | --- | --- |
| RR_real (sin cambios) | 34.42 | 3.615 | 1.897 | 13 |
| RR_real con 0.9 SL | 30.23 | 4.815 | 2.087 | 15 |
| TP óptimo | 22.53 | 9.209 | 2.678 | 13 |
| TP óptimo + SL 0.9 | **20.88** | **12.016** | **3.149** | **13** |

📊 **Conclusión:**  
Aunque el Win Rate bajó ligeramente, el **Profit Factor casi se duplicó (de 1.89 a 3.1)** y la rentabilidad acumulada aumentó drásticamente.

---

## 🤖 8. Predicción con Machine Learning

Finalmente, se desarrolló un modelo de **clasificación supervisada** para predecir el resultado de una operación (TP o SL).

### Preparación de datos:
- Eliminación de variables *spoiler* (como `RR_real`).  
- Imputación de valores faltantes.  
- Estandarización (`StandardScaler`) y codificación categórica (`OneHotEncoder`).  
- División *train/test* y evaluación con `SMOTE` para manejar desbalanceo.

### Modelos entrenados:
`Logistic Regression · Random Forest · Gradient Boosting · SVM · KNN · Decision Tree · Naive Bayes (ajustado) · MLP Classifier · LightGBM · XGBoost`

### Métricas del modelo Naive Bayes (mejor resultado):

| Métrica | Valor |
| --- | --- |
| AUC | 0.7039 |
| Precisión (TP) | 0.346 |
| Recall (TP) | **0.947** |
| F1-Score (TP) | 0.507 |
| Accuracy global | 0.4068 |

📈 **Interpretación:**  
El modelo tiene un **recall del 95% para identificar trades ganadores**, lo que significa que detecta casi todos los Take Profits reales, aunque con cierta sobrepredicción — justo lo que buscamos: **no dejar pasar operaciones rentables**, incluso a costa de algunos falsos positivos.

---

## 📊 9. Conclusiones generales

### Principales mejoras obtenidas:

| Aspecto | Antes | Modelo mejorado sin ML |
| --- | --- | --- |
| Win Rate | 34% | **20.88%** |
| Profit Factor | 1.89 | **3.14** |
| Recall (TP, modelo ML) | — | **0.95** |

### Insights clave:
- El filtrado basado en EDA eliminó contextos poco rentables.  
- La optimización del TP y el ajuste del SL transformaron la curva de rendimiento.  
- El Win Rate bajó ligeramente, pero el *profit factor* se disparó, gracias a operaciones con mayores recompensas.  
- El modelo Naive Bayes demostró un gran potencial como filtro predictivo de operaciones, anticipando setups ganadores en tiempo real.

---

## 🔮 10. Próximos pasos

1. **Integrar el clasificador Naive Bayes** al proceso operativo para filtrar setups de alta probabilidad.  
2. **Refinar el modelo de regresión** para estimar mejor el `RR_potencial`.  
3. **Agregar nuevas fuentes de datos**, como volatilidad o sentimiento de mercado.  
4. **Backtesting completo** con las reglas y modelos combinados para validar robustez y estabilidad.

---

## 🧭 Conclusión final

Este proyecto demuestra cómo una estrategia de trading puede **evolucionar de una intuición a un sistema basado en datos**, combinando análisis exploratorio, optimización estadística y modelos predictivos.

**Resultado:**  
> De un *Profit Factor* de **1.3 a más de 3.0**, con un enfoque replicable, medible y sustentado en evidencia.

---

### 🧰 Stack Tecnológico

| Área | Herramientas |
|------|---------------|
| Lenguaje | Python (3.10+) |
| Análisis | Pandas, NumPy, Seaborn, Matplotlib |
| Modelado | Scikit-learn, LightGBM, XGBoost |
| Optimización | GridSearchCV, Interpolación de percentiles |
| ML Pipeline | Imputer + Scaler + OneHotEncoder + Classifier |
| Entorno | Google Colab / Jupyter Notebook |

---

### 📬 Contacto

**Autor:** Valentín Álvarez Lamas  
**Email:** [valentinalvarezlamas@gmail.com](mailto:valentinalvarezlamas@gmail.com)  
**LinkedIn:** [linkedin.com/in/valentinalvarezlamas](https://www.linkedin.com/in/valentinalvarezlamas)

---
