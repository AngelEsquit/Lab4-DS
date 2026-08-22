# Laboratorio 4: Datos Geoespaciales

**CC3084 Data Science | Universidad del Valle de Guatemala | Semestre II 2026**

Detección de floraciones de cianobacterias en los lagos Atitlán y Amatitlán usando imágenes Sentinel-2 (Copernicus L2A), sobre 11 fechas por lago entre enero de 2025 y julio de 2026.

**Integrantes:** Javier España (23361), Ángel Esquit (23221), Roberto Barreda (23354)

## Contenido

- [`notebooks/Lab4_Datos_Geoespaciales.ipynb`](notebooks/Lab4_Datos_Geoespaciales.ipynb). **Parte I:** Descarga de las bandas desde Copernicus vía openEO, máscara de agua, cálculo de NDVI, NDWI, NDCI, FAI y clorofila-$a$ con el script [`cyanobacteria_chla_ndci_l1c`](https://github.com/sentinel-hub/custom-scripts/tree/master/sentinel-2/cyanobacteria_chla_ndci_l1c) de Sentinel Hub, y análisis temporal, espacial y comparativo entre lagos.
- [`notebooks/Lab4_Parte2_ML.ipynb`](notebooks/Lab4_Parte2_ML.ipynb). **Parte II:** Dataset a nivel de píxel (3.3 millones de observaciones), variable respuesta binaria con umbral de 30 mg/m³ de Chl-$a$, y modelos de Regresión Logística, Random Forest y XGBoost con validación aleatoria y validación espacial por bloques de 1 km.

El informe compilado está en [`docs/informe_lab4.pdf`](docs/informe_lab4.pdf).

## Cómo ejecutar

Se necesita Python 64-bit (3.10 a 3.12) y una cuenta gratuita en [Copernicus Data Space](https://dataspace.copernicus.eu).

```bash
python -m pip install -r requirements.txt
```
Las imágenes GeoTIFF se descargan una sola vez a `data/` y los resultados se escriben en `outputs/`.
