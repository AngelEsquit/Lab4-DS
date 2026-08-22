# Laboratorio 4 — Parte 2: Modelos con Datos Geoespaciales
## Documento Tecnico de Avance y Coordinacion de Equipo

**Proyecto:** Monitoreo y deteccion de floraciones de cianobacterias mediante Machine Learning y Sentinel-2
**Curso:** CC3084 Data Science — Universidad del Valle de Guatemala
**Autores:** Roberto Barreda (#23354), Javier Espana (#23361), Angel Esquit (#23221)
**Fecha:** Semestre II — 2026
**Estado:** Ejercicios 1 al 6 completados y validados

---

## 1. Resumen Ejecutivo

Se completo el desarrollo tecnico, experimental y analitico correspondiente a los **Ejercicios 1 al 6** de la Parte 2 del Laboratorio 4 en el notebook interactivo 
otebooks/Lab4_Parte2_ML.ipynb.

El flujo abarca:
1. Extraccion y depuracion de 3,321,327 observaciones satelitales a nivel de pixel (20 m) para los lagos Atitlan y Amatitlan en las 11 fechas oficiales por cuerpo de agua.
2. Definicion formal y justificada de la variable respuesta binaria basada en los estandares internacionales de la OMS.
3. Seleccion de 16 variables predictoras (espectrales, morfometricas, temporales y de ingenieria) con prevencion rigurosa de fuga de informacion (*data leakage*).
4. Entrenamiento y ajuste de hiperparametros de tres familias de modelos: Regresion Logistica, Random Forest y XGBoost.
5. Evaluacion multidimensional en conjunto de prueba fijo con analisis limnologico y sanitario del error.
6. Validacion espacial cruzada sobre una cuadricula regular metrica de 1 km x 1 km en coordenadas proyectadas UTM Zona 15N (EPSG:32615), comparando el desempeno frente a la particion aleatoria tradicional y analizando el efecto de la autocorrelacion espacial.

El dataset consolidado y listo para los ejercicios siguientes se encuentra almacenado en:
- outputs/dataset_ml_parte2.csv (3,321,327 filas x 24 columnas).

---

## 2. Detalle de los Ejercicios Completados

### Ejercicio 1: Preparacion de los Datos para Machine Learning
- **Estructura tabular:** Cada observacion corresponde a un pixel de 20 m x 20 m georreferenciado dentro de la huella morfologica estable del lago.
- **Variables extraidas por pixel:** Coordenadas (x_utm, y_utm, lon, lat), echa, lago, 9 bandas espectrales (B02, B03, B04, B05, B07, B08, B8A, B11, B12), indices (
dvi, 
dwi, 
dci, chla, ai), y la distancia euclidiana a la linea de costa (distancia_orilla_m).
- **Control de calidad y limpieza:**
  - Se restringio el analisis a la huella morfologica estable derivada de la Parte I (outputs/*_huella.npz).
  - Se eliminaron ceros radiometricos, valores NoData y sombras topograficas donde B04 <= 0 o B05 <= 0.
  - Se descartaron 70,697 observaciones correspondientes a fechas con cobertura valida inferior al 70% (Confiable == False en indices_<lago>.csv), manteniendo 3,321,327 pixeles validos (2,942,535 en Atitlan y 378,792 en Amatitlan) con 0% de valores faltantes.
- **Analisis Exploratorio (EDA):** Se incluyeron histogramas/KDE por lago, boxplots en escala logaritmica de Clorofila-a y matriz de correlacion de Pearson con mapa de calor.

### Ejercicio 2: Construccion de la Variable Respuesta
- **Definicion binaria:**
  alta_cianobacteria = 1 si Chl-a >= 30 mg/m3, 0 en caso contrario.
- **Fundamento cientifico del punto de corte (30 mg/m3 / 30 ug/L):**
  - Basado en las *Guidelines on Recreational Water Quality* de la Organizacion Mundial de la Salud (OMS / WHO, 2021) y el marco de Chorus & Welker (2021).
  - El umbral se situa en la transicion hacia el **Nivel de Alerta 2** (> 24 ug/L Chl-a), donde se forman natas densas superficiales y se presentan concentraciones peligrosas de microcistinas y otras cianotoxinas.
- **Desbalance de clases:**
  - Total global: 65,204 positivos (1.96%) y 3,256,123 negativos (98.04%). Ratio de desbalance aproximado de 1 : 50.
  - Heterogeneidad interlacustre: En Amatitlan (hipereutrofico), la clase 1 representa el **15.56%** (58,936 pixeles); en Atitlan (oligotrofico), la clase 1 representa apenas el **0.21%** (6,268 pixeles).
  - Implicacion: El Accuracy es una metrica enganosa (un clasificador nulo obtendria 98.04% de exactitud). Se requiere ponderacion de clases (class_weight, scale_pos_weight) y optimizacion de Recall, F1 y ROC-AUC.
- **Prevencion de fuga de informacion (*Data Leakage*):**
  - Se excluyeron categoricamente chla (variable base de la regla) y 
dci (transformacion matematica directa de la reflectancia en la que se basa la formula de Chl-a).

### Ejercicio 3: Seleccion y Construccion de Variables Predictoras
- **Conjunto de 16 predictores seleccionados:**
  1. Bandas espectrales (9): B02 (Azul), B03 (Verde), B04 (Rojo), B05 (Red Edge 1), B07 (Red Edge 3), B08 (NIR), B8A (NIR narrow), B11 (SWIR 1), B12 (SWIR 2).
  2. Indices espectrales (2): 
dvi (vegetacion superficial), 
dwi (contenido hidrico).
  3. Variables espaciales y morfometricas (3): distancia_orilla_m (cercania a afluentes y descargas costeras), x_utm, y_utm (coordenadas proyectadas en metros).
  4. Variables temporales e ingenieria (2): mes (1 a 12), estacion_cod (0: seca, 1: lluviosa), indice_turbidez_swir (B11 / B08, indicador de sedimentos y solidos en suspension).

### Ejercicio 4: Construccion de Modelos de Machine Learning
- **Division:** Particion convencional 70% entrenamiento / 30% prueba estratificada por clase con semilla fija (
andom_state=42).
- **Modelos implementados:**
  1. **Regresion Logistica:** Con pipeline de preprocesamiento (StandardScaler), penalizacion L2 (C=1.0) y class_weight='balanced'.
  2. **Random Forest:** RandomForestClassifier configurado con 
_estimators=80, max_depth=12, min_samples_leaf=5, class_weight='balanced' y max_samples=0.20.
  3. **XGBoost:** XGBClassifier con 	ree_method='hist', 
_estimators=100, max_depth=6, learning_rate=0.1, submuestreo del 80% y ponderacion scale_pos_weight = N_neg / N_pos.
- **Ajuste de hiperparametros:** Todos los modelos fueron calibrados para penalizar adecuadamente los errores en la clase minoritaria sin incurrir en sobreajuste.

### Ejercicio 5: Evaluacion de Modelos y Analisis Ambiental del Error
- **Resultados en conjunto de prueba convencional (30% test):**
  - **Regresion Logistica:** Accuracy: 0.9938 | Precision: 0.7610 | Recall: 0.9993 | F1: 0.8640 | ROC-AUC: 0.9999
  - **Random Forest:** Accuracy: 0.9946 | Precision: 0.8376 | Recall: 0.8967 | F1: 0.8661 | ROC-AUC: 0.9989
  - **XGBoost:** Accuracy: 0.9953 | Precision: 0.8124 | Recall: 0.9885 | F1: 0.8918 | ROC-AUC: 0.9997
- **Analisis de compensacion ambiental:**
  - *Falso Negativo (No detectar una floracion real):* Consecuencias criticas de salud publica (consumo inadvertido de agua con cianotoxinas, intoxicaciones agudas, mortandad de peces, retraso en contingencias sanitarias). Costo severo e irreversible.
  - *Falso Positivo (Emitir alerta donde no hay floracion):* Costo operativo menor (toma de muestras de campo de verificacion, restriccion preventiva temporal de bano).
  - *Conclusion:* Se debe priorizar el **Recall** como metrica primaria, seguido del **F1-Score** y **ROC-AUC**. XGBoost ofrece el mejor equilibrio entre alta sensibilidad (Recall = 98.85%) y precision aceptable (Precision = 81.24%).

### Ejercicio 6: Validacion Espacial
- **Estructuracion de la Cuadricula Regular:**
  - Reproyeccion y segmentacion en bloques de 1000 m x 1000 m (1 km x 1 km) en UTM 15N (EPSG:32615).
  - Se obtuvieron **198 bloques espaciales unicos** (165 en Atitlan y 33 en Amatitlan), con una mediana de ~17,000 pixeles por bloque, garantizando un soporte muestral robusto.
- **Cartografia de Bloques:** Se genero el mapa de asignacion de bloques con paleta categorica para ambos lagos.
- **Spatial Cross-Validation (GroupKFold 5 splits agrupados por loque_id):**
  - Todos los pixeles de un mismo bloque de 1 km permanecen juntos en entrenamiento o en validacion.
  - **Resultados Promedio Espaciales:**
    - Regresion Logistica: Accuracy: 0.9940 | Precision: 0.7647 | Recall: 0.9969 | F1: 0.8649 | ROC-AUC: 0.9999
    - Random Forest: Accuracy: 0.9934 | Precision: 0.8221 | Recall: 0.8564 | F1: 0.8371 | ROC-AUC: 0.9981
    - XGBoost: Accuracy: 0.9955 | Precision: 0.8179 | Recall: 0.9773 | F1: 0.8896 | ROC-AUC: 0.9995
- **Discusion Cientifica:**
  - Se evidencio la caida en metricas de prueba al pasar de la validacion aleatoria a la espacial (por ejemplo, en Random Forest el F1-Score disminuye de 0.8661 a 0.8371).
  - *Explicacion:* En la particion aleatoria convencional, pixeles contiguos (a 20 m de distancia) quedan repartidos entre train y test. Por la Primera Ley de Tobler (autocorrelacion espacial), el modelo memoriza el vecindario inmediato (*spatial leakage*). La validacion espacial elimina este artefacto y proporciona la **unica estimacion realista y honesta** de como funcionara el modelo al predecir sobre nuevas bahias o zonas no observadas.

---

## 3. Hoja de Ruta para los Ejercicios Restantes (7 al 10)

Para las siguientes secciones del informe y notebook:

### Ejercicio 7: Generalizacion entre Lagos
- **Experimento A:** Entrenar con el 100% de datos de Atitlan y evaluar sobre Amatitlan.
- **Experimento B:** Entrenar con el 100% de datos de Amatitlan y evaluar sobre Atitlan.
- **Pregunta clave a responder:** Puede un modelo transferirse entre lagos con regimenes troficos diametralmente opuestos (oligotrofico profundo vs hipereutrofico somero)? Discutir las diferencias espectrales de fondo y turbidez.

### Ejercicio 8: Interpretabilidad y Explicabilidad (SHAP)
- Extraer el ranking global de importancia de caracteristicas de XGBoost.
- Calcular los valores SHAP (shap.TreeExplainer) sobre una muestra balanceada y generar el SHAP Summary Plot (beeswarm).
- Interpretar fisicamente la direccion del impacto: como influyen valores altos de Red Edge (B05), NDVI y distancias cortas a la orilla en el aumento de la probabilidad de cianobacteria.

### Ejercicio 9: Generacion de Mapas Predictivos
- Con el mejor modelo (XGBoost), calcular las probabilidades continuas para cada pixel de las 22 imagenes.
- Reconstruir matricialmente las predicciones en formato raster 2D.
- Generar mapas coropleticos categorizados en 4 niveles de riesgo:
  - Muy Baja probabilidad (p < 0.25)
  - Baja probabilidad (0.25 <= p < 0.50)
  - Alta probabilidad (0.50 <= p < 0.75)
  - Muy Alta probabilidad (p >= 0.75)
- Comparar visual y cuantitativamente los mapas predichos con los mapas de Chl-a de la Parte I, identificando zonas de falsos positivos y falsos negativos en bahias cerradas.

### Ejercicio 10: Analisis y Conclusiones Finales
- Sintetizar la utilidad operativa del modelo satelital como sistema de alerta temprana.
- Discutir limitaciones inherentes: resolucion espacial (20 m), revisita temporal (5 dias), nubosidad en epoca lluviosa y saturacion espectral.
- Proponer fuentes de datos complementarias (temperatura superficial del agua, velocidad del viento, batimetria y estaciones telemetricas in situ).

---

## 4. Archivos Clave del Repositorio

- 
otebooks/Lab4_Parte2_ML.ipynb: Notebook principal con todo el codigo ejecutable y texto explicativo hasta el Paso 6.
- outputs/dataset_ml_parte2.csv: Dataset procesado completo con las 16 variables y la variable respuesta.
- outputs/amatitlan_huella.npz y outputs/atitlan_huella.npz: Huellas morfologicas de agua.
- outputs/indices_amatitlan.csv y outputs/indices_atitlan.csv: Metricas y control de calidad por fecha.
- docs/informe_lab4.tex: Plantilla del informe final para redactar los resultados consolidados.
