# Laboratorio 4 — Análisis de Datos Geoespaciales

**CC3084 Data Science | Universidad del Valle de Guatemala | Semestre II 2026**

Monitoreo y detección de floraciones de cianobacterias en los lagos **Atitlán** y **Amatitlán** a partir de imágenes satelitales multiespectrales **Sentinel-2** (Copernicus L2A), abarcando las 11 fechas oficiales por lago entre enero de 2025 y julio de 2026.

El flujo de trabajo reproduce en Python el script oficial [`cyanobacteria_chla_ndci_l1c`](https://github.com/sentinel-hub/custom-scripts/tree/master/sentinel-2/cyanobacteria_chla_ndci_l1c) (CyanoLakes / Sentinel Hub), aplicando filtrado multiespectral de agua (WBI), cálculo del índice NDCI (borde rojo), estimación de Clorofila-$a$ ($\text{mg/m}^3$), detección de natas flotantes (FAI) y cálculo de NDVI y NDWI.

---

## Integrantes
- Javier España #23361
- Ángel Esquit #23221
- Roberto Barreda #23354

---

## Estructura del Proyecto

```text
.
├── data/                                 # Imágenes GeoTIFF descargadas de Copernicus (no versionadas)
│   ├── amatitlan/                        # 11 escenas Sentinel-2 L2A (bandas B02-B12, 20 m)
│   └── atitlan/                          # 11 escenas Sentinel-2 L2A (bandas B02-B12, 20 m)
├── docs/                                 # Documentación e informe entregable
│   ├── img/                              # Figuras y gráficos exportados en alta resolución (PNG)
│   ├── informe_lab4.tex                  # Documento fuente en LaTeX (15 páginas)
│   ├── informe_lab4.pdf                  # Informe compilado final en PDF
│   └── Laboratorio 4. Datos Geoespaciales. 2026.pdf # Guía oficial del laboratorio
├── notebooks/
│   └── Lab4_Datos_Geoespaciales.ipynb    # Notebook interactivo con los Ejercicios 1 al 8
├── outputs/                              # Productos derivados y resultados del análisis
│   ├── amatitlan/                        # Arrays comprimidos (.npz) por fecha (Chl-a, NDCI, NDVI, NDWI, FAI)
│   ├── atitlan/                          # Arrays comprimidos (.npz) por fecha (Chl-a, NDCI, NDVI, NDWI, FAI)
│   ├── amatitlan_huella.npz              # Huella morfológica estable y mapa de frecuencia de agua
│   ├── atitlan_huella.npz                # Huella morfológica estable y mapa de frecuencia de agua
│   ├── amatitlan_meta.json               # Georreferencia y CRS (EPSG:32615 / UTM 15N)
│   ├── atitlan_meta.json                 # Georreferencia y CRS (EPSG:32615 / UTM 15N)
│   ├── indices_amatitlan.csv             # Tabla de métricas y estadísticas por fecha
│   └── indices_atitlan.csv               # Tabla de métricas y estadísticas por fecha
├── requirements.txt                      # Dependencias del entorno virtual
└── README.md
```

---

## Contenido de los Ejercicios

1. **Conexión API (openEO / Copernicus Data Space Ecosystem):** Autenticación vía OpenID Connect (OIDC) y acceso al catálogo `SENTINEL2_L2A`.
2. **Descarga Selectiva de Bandas:** Recorte espacial por *bounding box* y remuestreo a 20 m de las 9 bandas necesarias (`B02`, `B03`, `B04`, `B05`, `B07`, `B08`, `B8A`, `B11`, `B12`).
3. **Cálculo de Índices:** Reproducción de máscara de agua multiespectral (WBI), estimación de Clorofila-$a$ vía NDCI, detección de natas flotantes (FAI) y cálculo de NDVI/NDWI.
4. **Análisis Temporal:** Series de tiempo de Clorofila-$a$, identificación objetiva de picos ($\mu + 1\sigma$), evaluación de tipología de floración (uniforme vs parches) y control de calidad radiométrica.
5. **Análisis Espacial:** Mapa interactivo (`folium`) con capas informativas y cuadrículas comparativas fecha a fecha (`matplotlib`).
6. **Correlación de Índices:** Análisis de correlación de Pearson y regresión lineal entre $\text{Chl-}a$, NDVI y NDWI.
7. **Comparación Interlacustre:** Diagnóstico limnológico comparativo, balances morfométricos (profundidad y volumen) y presiones de cuenca antrópica.
8. **Análisis Exploratorio Adicional:** Extensión espacial de biomasa crítica ($\ge 30\text{ mg/m}^3$), mapas de persistencia espacial píxel a píxel, asimetría estadística en boxplots y evaluación del patrón estacional (lluvia vs estiaje).

---

## Requisitos y Configuración

### 1. Entorno de Python
Se requiere **Python 64-bit** (3.10 a 3.12). Instalar las dependencias con:

```bash
python -m pip install -r requirements.txt
```

### 2. Cuenta en Copernicus Data Space Ecosystem
Se requiere una cuenta gratuita en [Copernicus Data Space](https://dataspace.copernicus.eu). 

Al ejecutar la celda de conexión en el notebook:
1. Se imprimirá una URL de autenticación OIDC (`https://identity.dataspace.copernicus.eu/...`) con un código de dispositivo de un solo uso.
2. Inicia sesión en el navegador y autoriza el acceso.
3. `openEO` almacenará localmente el *refresh token* para las siguientes sesiones.

---

## Cómo Ejecutar

1. Abrir [`notebooks/Lab4_Datos_Geoespaciales.ipynb`](notebooks/Lab4_Datos_Geoespaciales.ipynb).
2. Seleccionar el kernel del entorno virtual (`.venv`).
3. Ejecutar las celdas de forma secuencial de arriba a abajo.
   - Las imágenes GeoTIFF se guardarán en `data/` y solo se descargan una vez.
   - Los índices calculados, tablas y huellas se exportarán automáticamente a `outputs/`.
