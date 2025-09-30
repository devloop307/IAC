# 🧠 Detección de Tumores Cerebrales por MRI – Aplicación Streamlit

Clasificador **binario (sí/no)** para detección de tumores cerebrales en imágenes **MRI** con explicación visual usando **Grad-CAM**.  
Aplicación web construida con **Streamlit** y modelo **PyTorch (ResNet18)** preentrenado.

---

## 🚀 Tecnologías Utilizadas

![Python](https://img.shields.io/badge/Python-3.10-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![Render](https://img.shields.io/badge/Render-46E3B7?style=for-the-badge&logo=render&logoColor=white)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/15giXohyUo7ck9FjbVnmwcqPvXOPSuK2l?usp=sharing)

---

## 📦 Estructura del Repositorio

```
.
├── .streamlit/           # Carpeta de configuración de Streamlit
│   └── config.toml      # Configuración de tema y servidor
├── assets/              # Recursos multimedia
│   ├── capturas/        # Screenshots de la aplicación
│   ├── graficos/        # Gráficos de entrenamiento (loss, ROC, PR)
│   └── ejemplos/        # Imágenes de prueba (positivas/negativas)
├── LICENSE              # Licencia Apache 2.0
├── Procfile            # Definición de proceso para despliegue (Render)
├── README.md           # Este archivo
├── best_threshold.json # Umbrales sugeridos (youden_J, max_F1, recall_priority)
├── requirements.txt    # Dependencias para despliegue/entorno
├── resnet18_best.pt    # Checkpoint del modelo entrenado
├── runtime.txt         # Versión de Python para despliegue (3.10 recomendado)
└── streamlit_app.py    # Aplicación principal Streamlit (subir imagen → predicción → Grad-CAM)
```

---

## 🧠 ¿Qué hace?

1. **Subes** una imagen (PNG/JPG/TIFF) de una resonancia magnética.
2. La aplicación la **preprocesa** (escala de grises→3 canales, redimensión 224, normalización ImageNet).
3. El modelo **infiere** `prob_yes` (probabilidad de presencia de tumor).
4. Se compara contra un **umbral** (por defecto: *recall_priority* de `best_threshold.json`).
5. Si la predicción es **SÍ**, genera una visualización **Grad-CAM** + **caja delimitadora** para resaltar la región de mayor atención.

---

## 📸 Ejemplos Visuales

### Interfaz de la Aplicación
![Captura de predicción positiva](assets/capturas/prediccion_si.png)
*Ejemplo de detección positiva con visualización Grad-CAM*

![Captura de predicción negativa](assets/capturas/prediccion_no.png)
*Ejemplo de predicción negativa (sin tumor detectado)*

### Métricas de Entrenamiento

<p align="center">
  <img src="assets/graficos/loss_curve.png" width="400" alt="Curva de pérdida">
  <img src="assets/graficos/roc_curve.png" width="400" alt="Curva ROC">
</p>

<p align="center">
  <img src="assets/graficos/precision_recall.png" width="400" alt="Curva Precisión-Recall">
</p>

### Ejemplos de Predicción

| Tumor Detectado | Sin Tumor |
|----------------|-----------|
| ![Ejemplo tumor](assets/ejemplos/tumor_ejemplo1.jpg) | ![Ejemplo normal](assets/ejemplos/no_tumor_ejemplo.jpg) |

---

## 📓 Notebook de Entrenamiento en Google Colab

Explora el proceso completo de entrenamiento del modelo, desde el preprocesamiento de datos hasta la evaluación de métricas:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/15giXohyUo7ck9FjbVnmwcqPvXOPSuK2l?usp=sharing)

**Contenido del notebook:**
- 📊 Análisis exploratorio del dataset
- 🔧 Preprocesamiento y augmentación de imágenes
- 🏗️ Arquitectura del modelo ResNet18
- 📈 Entrenamiento con métricas (ROC-AUC, F1-Score, Recall)
- 🎯 Calibración de umbrales de decisión
- 🔥 Generación de mapas Grad-CAM para explicabilidad

---

## 🔧 Requisitos

- Python **3.10** (recomendado)
- Compatible con CPU (funciona sin GPU)
- Paquetes listados en `requirements.txt`

> **Nota**: El archivo del modelo preentrenado (`resnet18_best.pt`) debe agregarse al repositorio. Si el archivo es grande, considera usar **Git LFS**:
>
> ```bash
> git lfs install
> git lfs track "*.pt"
> git add .gitattributes
> git commit -m "chore(lfs): habilitar LFS para archivos .pt"
> ```

---

## ▶️ Ejecutar Localmente

```bash
# 1) Crear y activar entorno virtual
python -m venv .venv

# Windows:
.venv\Scripts\activate
# macOS/Linux:
# source .venv/bin/activate

# 2) Instalar dependencias
pip install --upgrade pip
pip install -r requirements.txt

# 3) Ejecutar la aplicación
streamlit run streamlit_app.py
# Se abre en: http://localhost:8501
```

---

## ⚙️ Umbrales de Decisión

`best_threshold.json` contiene tres opciones de umbral:

- **youden_J**: Balance entre especificidad/sensibilidad en curva ROC
- **max_F1**: Maximiza el puntaje F1 en curva PR  
- **recall_priority** (por defecto): Prioriza el recall para reducir falsos negativos

Puedes cambiar el umbral activo editando el archivo JSON o ajustando 1-2 líneas en `streamlit_app.py`.

---

## 🖼️ Uso

1. Sube una imagen de MRI
2. Haz clic en **"🔍 Predecir"**
3. Visualiza los resultados:
   - **Predicción**: SÍ (anomalía detectada) o NO
   - **prob_yes** (puntuación de probabilidad 0-1)
   - Si es SÍ: Visualización Grad-CAM con caja delimitadora estimada

---

## 🚢 Despliegue en Render (Tier Gratuito)

1. Sube el repositorio a GitHub
2. Crea un Web Service en Render → Python
3. Asegúrate de tener:
   - `requirements.txt`
   - `runtime.txt` con `3.10`
   - `Procfile` con:
     ```
     web: streamlit run streamlit_app.py --server.port=$PORT --server.address=0.0.0.0
     ```
4. Render instalará las dependencias e iniciará la aplicación
5. Accede en: `https://<tu-servicio>.onrender.com`

> **Nota del Tier Gratuito**: La aplicación entra en suspensión sin tráfico; la primera solicitud puede tomar varios segundos en despertar.

---

## ❓ Preguntas Frecuentes

**¿Puedo usar imágenes diferentes al conjunto de datos de entrenamiento?**  
Sí, siempre que sean imágenes MRI en escala de grises o RGB. La aplicación convierte a 3 canales y normaliza apropiadamente.

**¿Por qué mi prob_yes es exactamente 1.000 o 0.000?**  
El modelo podría estar muy confiado. Considera ajustar el umbral, calibrar probabilidades o revisar el conjunto de validación.

**¿La caja delimitadora del Grad-CAM es una localización clínica real?**  
Es explicativa, no detección clínica. Para localización robusta, considera entrenar un detector/segmentador con anotaciones (ej., YOLO/Faster-RCNN/U-Net).

---

## 🧪 Notas Técnicas

- **Arquitectura**: ResNet18 (capa final reentrenada para 2 clases)
- **Preprocesamiento**: Escala de grises→3 canales, Redimensión(224), Normalización(ImageNet)
- **Explicabilidad**: Grad-CAM en la última capa conv de layer4
- **Métricas/Umbrales**: Pre-generadas (ROC/PR) y guardadas en `best_threshold.json`

---
## 📄 Licencia
Este proyecto se distribuye bajo la licencia Apache-2.0. Consulta el archivo [LICENSE](./LICENSE) para más detalles.

![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)
