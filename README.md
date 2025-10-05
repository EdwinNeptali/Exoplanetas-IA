# 🔭 Un mundo aparte: IA para buscar exoplanetas (MVP)

Este proyecto es un **MVP listo para correr** para el desafío *“Un mundo aparte: En busca de exoplanetas con IA”*.  
Toma datos abiertos de la NASA (KOI/TOI/TCE) y **curvas de luz Kepler/TESS** (MAST vía Lightkurve), ejecuta **BLS** para detectar tránsitos
y usa un **clasificador** (Random Forest) que prioriza candidatos a planeta. Incluye **app web (Streamlit)**.

> ⚠️ Requisitos de internet para descargar datos. Si corres esto sin conexión, usa el modo de **simulación** (ver más abajo).

## 🚀 Quickstart

```bash
# 1) (opcional) crear entorno conda
conda create -n exoai python=3.11 -y
conda activate exoai

# 2) instalar dependencias
pip install -r requirements.txt

# 3) (opcional) descargar etiquetas KOI/TOI y crear CSVs
python scripts/download_labels.py --out data

# 4) construir tabla de features BLS para algunos objetivos de demo
python scripts/build_feature_table.py --targets data/demo_targets.csv --out data/features_bls.csv --mission Kepler

# 5) entrenar modelo base y evaluarlo
python scripts/train.py --features data/features_bls.csv --labels data/labels_merged.csv --out model/rf_baseline.joblib

# 6) lanzar la app web
streamlit run app/streamlit_app.py
```

## 📂 Estructura

```
UnMundoAparte_ExoplanetasAI/
 ├─ app/streamlit_app.py           # demo web interactiva
 ├─ src/
 │   ├─ data/
 │   │   ├─ nasa_exoplanet.py      # descarga KOI/TOI/TCE desde API del Exoplanet Archive
 │   │   └─ lightcurves.py         # descarga/lectura de curvas Kepler/TESS (Lightkurve) o FITS
 │   ├─ features/bls_features.py   # cálculo de BLS + features (period, depth, SNR, odd/even, etc.)
 │   ├─ models/train_baseline.py   # entrenamiento y evaluación del modelo
 │   └─ utils/plotting.py          # utilidades de plotting
 ├─ scripts/
 │   ├─ download_labels.py         # crea data/labels_*.csv desde la API
 │   ├─ build_feature_table.py     # calcula features para una lista de objetivos
 │   └─ train.py                   # orquestador de entrenamiento (lee features+labels)
 ├─ data/
 │   └─ demo_targets.csv           # lista de IDs (KIC/TIC) de ejemplo
 ├─ model/                         
 ├─ requirements.txt
 └─ README.md
```

## 🧠 Datos y referencias (docs oficiales)

- **Exoplanet Archive API (KOI/TOI/TCE)**: ver *program_interfaces.html*, columnas de **KOI**, **TOI** y **TCE**.  
  - API base: `https://exoplanetarchive.ipac.caltech.edu/cgi-bin/nstedAPI/nph-nstedAPI`
  - Columnas KOI: `API_kepcandidate_columns.html`
  - Columnas TOI: `API_TOI_columns.html`
  - Columnas TCE: `API_tce_columns.html`
- **Curvas de luz Kepler/TESS** vía **Lightkurve** (MAST).  
- **BLS (Astropy)**: `astropy.timeseries.BoxLeastSquares` y guía del periodograma BLS.

> En el *pitch/README* cita las fuentes oficiales (NASA Exoplanet Archive, MAST/Lightkurve, Astropy).

## 🧪 Modo simulación (sin internet)
```bash
python scripts/simulate_and_run.py --out data/simulated_curve.csv
python scripts/build_feature_table.py --csv data/simulated_curve.csv --out data/features_bls_sim.csv --mode csv
python scripts/train.py --features data/features_bls_sim.csv --labels data/labels_sim.csv --out model/rf_baseline.joblib
```
Esto genera una curva sintética con un tránsito artificial para validar el flujo extremo a extremo.

## 📝 Notas
- **Split por estrella** cuando entrenes para evitar fuga de información.
- Métricas recomendadas: **PR-AUC** y *Recall@Top-K* (útil para priorizar vetting).
- Si no cargas `model/rf_baseline.joblib`, la app mostrará sólo resultados de BLS (útil para demo).


# 3b) (opcional recomendado) construir una lista de objetivos desde las etiquetas
python scripts/make_targets_from_labels.py --labels data/labels_merged.csv --out data/targets_from_labels.csv --n_pos 60 --n_neg 60 --mission Kepler

# 4b) construir features para esa lista
python scripts/build_feature_table.py --targets data/targets_from_labels.csv --out data/features_bls.csv


## 🔗 Ingesta desde tus propios enlaces
Si tienes un CSV/TSV con columnas `url,id,mission`, usa:
```bash
python scripts/ingest_from_links.py --links mis_enlaces.csv --outdir data/downloads --targets_out data/targets_from_links.csv
python scripts/build_feature_table.py --targets data/targets_from_links.csv --out data/features_bls.csv
```
Luego entrena con `scripts/train.py` como en el Quickstart.
