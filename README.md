© 2025 Julián René Muñoz Burbano

This repository includes code and derived metadata from the ShapeNetCore v2 dataset.
Original 3D models are copyrighted by Princeton University and Stanford University
and provided for research and educational use only.

The contents of this repository (code, scripts, derived metadata, and documentation)
are released under the Creative Commons Attribution–NonCommercial 4.0 International License (CC BY-NC 4.0).

You may:
 - Share and adapt the materials for non-commercial research or educational purposes
 - Attribute the author (Julián René Muñoz Burbano)
You may not:
 - Use the materials for commercial purposes
 - Redistribute original ShapeNet files or any derivative dataset containing them

Uso de datos ShapeNet

Este proyecto utiliza datos derivados del dataset ShapeNetCore v2
(© Princeton University & Stanford University, 2015).
Los modelos 3D originales no se incluyen en este repositorio.
Los metadatos y análisis generados se comparten bajo licencia CC BY-NC 4.0,
únicamente para fines académicos y de investigación.
- https://modelnet.cs.princeton.edu/#
- https://3dshapenets.cs.princeton.edu/

# PRISMA-ETL ShapeNetCore v2 — Ejecución LOCAL (Anaconda)

Este repositorio/notebook implementa un **pipeline ETL + EDA** reproducible para **ShapeNetCore v2** en entorno **local** con **Anaconda/Jupyter**. Incluye:
- **Extracción** de metadatos desde la estructura `clase/id/model.binvox`
- **Transformación y Validación (PRISMA-ETL)** con checks SI/NO, detección de duplicados por `sha256` y KPIs de calidad
- **Carga** a **SQLite** y **Excel** con hojas `Metadatos` y `Resumen`
- **EDA** con visualizaciones en **matplotlib** (una gráfica por celda)
<img width="967" height="486" alt="image" src="https://github.com/user-attachments/assets/e2dbe57e-072a-42c0-b432-2e6d4b8c44c3" />
https://3dshapenets.cs.princeton.edu/poster.pdf

> Si buscas la versión para Google Colab, usa el cuaderno `PRISMA_ShapeNet_Colab.ipynb`. Este README está orientado a **Windows/Linux** con **Anaconda**.

---

## 1) Requisitos

- **Anaconda** (Python 3.10+ recomendado)
- **Dataset local**: estructura tipo `ShapeNetVox32` (subcarpetas por *clase* → *id* → `model.binvox`)
- **Espacio en disco** acorde al subset que vayas a indexar

## Anaconda
<img width="1202" height="794" alt="image" src="https://github.com/user-attachments/assets/8ccb6801-5f29-4210-a261-6926c4bc88eb" />


### Paquetes
Puedes instalar con **conda** o **pip**:

```bash
# Opción conda (recomendada)
conda create -n ETL python=3.10 -y
conda activate ETL
pip install pandas numpy matplotlib tqdm psutil openpyxl sqlalchemy python-binvox
```

---

## 2) Rutas locales (ajusta a tu PC)

En el cuaderno **local** (ej. `PRISMA_ShapeNet_Local.ipynb` o `Propuesta_maestria_EDA_actualizado-(Anaconda).ipynb`), configura:

```python
from pathlib import Path

# Carpeta con clases/id/model.binvox
BASE_DIR = Path(r"C:\Datasets\ShapeNetVox32")

# Carpeta de salida
OUT_DIR = Path(r"C:\Users\TU_USUARIO\Documents\ShapeNet_PRISMA")
OUT_DIR.mkdir(parents=True, exist_ok=True)
```

##Estructura de carpetas
<img width="600" height="800" alt="Arquitectura_pipeline_ver1png" src="https://github.com/user-attachments/assets/343cc334-2095-45d4-bc41-ffd8f28193a6" />


> En Linux, usa rutas como `/home/usuario/Datasets/ShapeNetVox32`.

---

## 3) Flujo de trabajo

1. **Extracción**  
   Recorre `BASE_DIR` y genera una tabla con: `file_id, class, file_name, type, bytes, path, source, acquired_at`.

2. **Transformación y Validación (PRISMA-ETL)**  
   - `hash (sha256)` para detectar **duplicados** por **contenido**  
   - Checks SI/NO: `normalized, linked_ok, nulls_removed, format_valid, tipo_valido`  
   - KPIs:  
     - `completitud` = % de campos clave presentes  
     - `consistencia` = min(normalized, linked_ok, nulls_removed, format_valid)  
     - `duplicated` ∈ {0,1}  
     - `score_calidad` = 0.5·completitud + 0.25·consistencia + 0.15·(1-duplicated) + 0.10·tipo_valido

3. **Carga de resultados**  
   - **SQLite**: `OUT_DIR/shapenet_prisma_etl.db`, tabla `metadatos`  
   - **Excel (openpyxl)**: `OUT_DIR/EDA_ShapeNetCore_v2_PRISMA_ETL.xlsx`  
     - Hoja `Metadatos`: tabla completa
     - Hoja `Resumen`: KPIs globales

4. **EDA (gráficas)**  
   - Conteo por `type` y top 15 `class`  
   - Histograma `log10(bytes)`  
   - **Boxplot seguro** por `type` (requiere ≥2 registros por grupo; usa `tick_labels` en Matplotlib ≥3.9)  
   - Gráfico de pastel por `type`  
   - Serie temporal diaria usando `acquired_at`

---

## 4) Ejecución (paso a paso)

1. Abre **Anaconda Prompt** y activa el entorno:
   ```bash
   conda activate ETL
   ```

2. Abre **Jupyter Notebook** o **JupyterLab**:
   ```bash
   jupyter notebook
   # o
   jupyter lab
   ```

3. Abre el cuaderno **local** (por ejemplo `PRISMA_ShapeNet_Local.ipynb`).  
4. Edita las **rutas** en la celda de **configuración** y ejecuta **todas las celdas**.

---

## 5) Estructura de salidas

- **Base de datos**: `OUT_DIR/shapenet_prisma_etl.db`  
  - Tabla: `metadatos`

- **Excel**: `OUT_DIR/EDA_ShapeNetCore_v2_PRISMA_ETL.xlsx`  
  - Hojas: `Metadatos`, `Resumen`

- **Visualizaciones**: embebidas en el notebook (matplotlib)

---

## 6) Estructura sugerida del repositorio

```
.
├─ notebooks/
│  ├─ PRISMA_ShapeNet_Local.ipynb
│  └─ Propuesta_maestria_EDA_actualizado-(Anaconda).ipynb
├─ src/
│  └─ index_shapenet.py         # (opcional) versión script
├─ data/
│  └─ raw/ShapeNetVox32/        # datos de entrada (no versionar completos)
├─ db/
│  └─ shapenet_prisma_etl.db    # salida
├─ reports/
│  └─ EDA_ShapeNetCore_v2_PRISMA_ETL.xlsx
├─ environment.yml (opcional)
└─ README_LOCAL_Anaconda.md  ← (este archivo)
```

> Agrega un `.gitignore` para no subir datos pesados (`data/raw/*`, `db/*`, `reports/*`).

---

## 7) Troubleshooting

- **Notebook “vacío” (0 archivos):** revisa `BASE_DIR` (ruta correcta y permisos).  
- **Error de Matplotlib con `labels=` en boxplot:** usa `tick_labels` (Matplotlib ≥3.9) o emplea el bloque **boxplot seguro** del notebook.  
- **Excel sin fórmulas/hojas:** verifica que el proceso llegó a la sección **Carga**.  
- **Lento al calcular hash:** aumenta el tamaño de bloque del SHA o limita el subset (ej. por clases).

---

## 8) Licencia y uso de datos

- **ShapeNet** es un dataset de terceros (Princeton/Stanford). Respeta su licencia y términos de uso.  
- Este proyecto se entrega con fines académicos (**no comercial**). Recomendada: **CC BY-NC 4.0** para los metadatos y scripts.

---

## 9) Cita recomendada

> Muñoz Burbano, J. R. (2025). *PRISMA-ETL para ShapeNetCore v2 (ejecución local)*. Repositorio académico (Anaconda/Jupyter).
