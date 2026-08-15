# Lab 4.

**CC3084 Data Science | Universidad del Valle de Guatemala | Semestre II 2026**

Detección de floraciones de cianobacterias en los lagos **Atitlán** y **Amatitlán** con imágenes
Sentinel-2, sobre las 11 fechas oficiales por lago. El índice de cianobacteria reproduce en Python
el script [`cyanobacteria_chla_ndci_l1c`](https://github.com/sentinel-hub/custom-scripts/tree/master/sentinel-2/cyanobacteria_chla_ndci_l1c)
de Sentinel Hub.

## Integrantes
- Javier España #23361
- Ángel Esquit #23221
- Roberto Barreda #23354

## Setup

```bash
python -m pip install -r requirements.txt
```

Se necesita una cuenta gratuita del [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu).
La celda de autenticación del notebook imprime un enlace con un código de un solo uso; al aprobarlo
en el navegador, `openEO` guarda un *refresh token* y las siguientes ejecuciones ya no lo piden.

## Cómo ejecutar

Abrir `Lab4_Datos_Geoespaciales.ipynb` y correrlo de arriba abajo. La primera ejecución descarga
22 imágenes (~1 min cada una) a `data/`; las siguientes las reutilizan desde disco.

## Estructura

| Ruta | Contenido |
|---|---|
| `Lab4_Datos_Geoespaciales.ipynb` | Notebook con todo el análisis |
| `data/` | GeoTIFF crudos de Copernicus — no se versionan, los genera el notebook |
| `outputs/` | Índices derivados (`.npz`) y tablas de resultados (`.csv`) |
| `docs/` | Informe entregable |
