# Diseño inverso de un filtro de guía de onda en banda W mediante redes neuronales

Este repositorio contiene un cuaderno de Jupyter para explorar el diseño inverso de un filtro electromagnético basado en una guía de onda rectangular con cinco irises simétricos. El flujo combina simulaciones electromagnéticas en **MEEP**, un modelo sustituto (*surrogate model*) basado en una red neuronal y un algoritmo evolutivo para buscar geometrías que aproximen una respuesta espectral objetivo.

El ejemplo del cuaderno busca una respuesta de filtrado centrada en **90 GHz**, con un ancho de banda objetivo de **10 GHz**, y valida el diseño encontrado mediante una nueva simulación electromagnética.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `physics-informed-nn.ipynb` | Cuaderno principal: configuración de MEEP, carga de datos, entrenamiento de la red neuronal, optimización inversa y visualización de campos. |
| `filter_dataset.npz` | Conjunto de datos precalculado utilizado para entrenar y evaluar el modelo. Debe estar disponible en el directorio de trabajo. |

El archivo de datos contiene, entre otros elementos, los parámetros geométricos, las curvas espectrales de \(S_{11}\), las frecuencias de simulación y banderas que identifican las muestras válidas.

## Metodología

El cuaderno está organizado en las siguientes etapas:

1. **Carga del conjunto de datos.** Se descarga y carga el archivo `filter_dataset.npz`, que contiene resultados de simulaciones previas. Esto evita tener que generar desde cero todas las muestras electromagnéticas.
2. **Configuración de las simulaciones.** Se definen la banda de frecuencias, la resolución numérica, los límites geométricos y las funciones auxiliares para simular y visualizar la guía de onda mediante MEEP.
3. **Inspección de los datos.** Se separan los parámetros geométricos de las respuestas \(S_{11}\) y se visualizan ejemplos de geometría y respuesta espectral.
4. **Preparación de datos y modelo sustituto.** Se filtran las muestras válidas, se estandarizan las siete variables geométricas y se dividen los datos en entrenamiento (80 %), validación (10 %) y prueba (10 %).
5. **Entrenamiento y evaluación.** Se entrena una red neuronal `BottleneckSurrogate` para predecir la curva de \(S_{11}\) a partir de la geometría. El entrenamiento utiliza PyTorch, el optimizador AdamW, una función de pérdida MSE, reducción adaptativa de la tasa de aprendizaje y parada temprana.
6. **Diseño inverso.** Se emplea `differential_evolution` de SciPy para buscar los parámetros geométricos cuya respuesta predicha se aproxime a un espectro objetivo.
7. **Validación electromagnética.** La geometría optimizada se simula nuevamente con MEEP y se compara la respuesta simulada con la predicción del modelo sustituto y con el espectro objetivo.
8. **Visualización de campos.** Se ejecuta una simulación de onda continua a 90 GHz y se genera una animación de la componente \(E_z\) del campo electromagnético en la guía de onda.

## Parámetros geométricos

La geometría se describe mediante siete parámetros:

| Parámetro | Descripción |
|---|---|
| `a` | Altura de la guía de onda. |
| `notch_width` | Espesor longitudinal de los irises. |
| `h1`, `h2`, `h3` | Alturas de los tres irises independientes. Los cinco irises se distribuyen simétricamente como `[h1, h2, h3, h2, h1]`. |
| `s1`, `s2` | Separaciones entre irises. |

Los límites de búsqueda se encuentran definidos en el diccionario `PRIOR_BOUNDS` del cuaderno.

## Requisitos

El cuaderno utiliza principalmente:

- Python
- [MEEP](https://meep.readthedocs.io/) para las simulaciones electromagnéticas mediante FDTD.
- [NumPy](https://numpy.org/) para operaciones numéricas y manejo del conjunto de datos.
- [Matplotlib](https://matplotlib.org/) para gráficas y animaciones.
- [PyTorch](https://pytorch.org/) para definir y entrenar la red neuronal.
- [scikit-learn](https://scikit-learn.org/) para la partición de datos y la estandarización.
- [SciPy](https://scipy.org/) para la optimización evolutiva.
- [joblib](https://joblib.readthedocs.io/) para ejecución paralela.

El cuaderno comienza configurando un entorno de **Google Colab** mediante `condacolab` e instalando MEEP con Conda. Se recomienda ejecutarlo en Colab o en un entorno Linux donde MEEP esté correctamente instalado. La disponibilidad de GPU puede acelerar el entrenamiento de PyTorch, pero MEEP también requiere recursos de CPU para las simulaciones electromagnéticas.

## Ejecución

1. Clona el repositorio:

   ```bash
   git clone https://github.com/saint-germain/physics-informed-nn.git
   cd physics-informed-nn
   ```

2. Abre `physics-informed-nn.ipynb` en Google Colab o en Jupyter.

3. Ejecuta las celdas en orden. El cuaderno instala las dependencias de MEEP al comienzo y descarga el conjunto de datos.

4. Comprueba que el archivo `.npz` descargado se encuentre en el directorio de trabajo y que el nombre utilizado en las celdas de carga coincida con el nombre real del archivo.

5. Ejecuta el entrenamiento, la optimización inversa y las simulaciones de validación. Al finalizar, el cuaderno guarda el modelo entrenado como `best_surrogate_model.pth` y la animación como `waveguide_90GHz_resonance.mp4`.

> **Nota sobre los nombres de archivo:** en la versión actual del cuaderno, algunas celdas hacen referencia a `filename` o `filename2`, y la función de preprocesamiento tiene como nombre predeterminado `w_band_filter_dataset_test.npz`, mientras que la celda de descarga obtiene `filter_dataset.npz`. Antes de ejecutar el flujo completo, unifica estas referencias. Por ejemplo, puedes definir `filename2 = "filter_dataset.npz"` después de descargar el conjunto de datos y utilizar ese mismo nombre en todas las celdas de carga.

## Alcance del enfoque

La red neuronal implementada es un **modelo sustituto entrenado con resultados de simulaciones electromagnéticas de MEEP**. El cuaderno utiliza información física en la generación y validación de los datos, así como restricciones en la optimización geométrica. Sin embargo, en la implementación actual la función de pérdida de la red neuronal es el error cuadrático medio entre las curvas predichas y las curvas simuladas; no se observa la incorporación explícita de ecuaciones de Maxwell como términos de penalización en la función de pérdida. Por tanto, el modelo no implementa, en sentido estricto, una *Physics-Informed Neural Network* (PINN) basada en residuos de las ecuaciones físicas.

## Resultados generados

Durante la ejecución se producen:

- Gráficas de geometrías y curvas \(S_{11}\).
- Historial de pérdidas de entrenamiento y validación.
- Comparaciones entre las curvas de MEEP y las predicciones de la red neuronal.
- Parámetros geométricos óptimos encontrados por el algoritmo evolutivo.
- Comparación entre el espectro objetivo, la predicción del modelo sustituto y la simulación electromagnética de validación.
- Archivo `best_surrogate_model.pth` con los pesos del mejor modelo.
- Archivo `waveguide_90GHz_resonance.mp4` con la animación del campo electromagnético.

## Licencia y reproducibilidad

No se especifica una licencia de distribución en el cuaderno. Antes de reutilizar o redistribuir el código y los datos, añade al repositorio la licencia correspondiente y documenta las versiones de las dependencias, las características del equipo utilizado y las semillas aleatorias. Para reproducir los resultados, conserva también el conjunto de datos original y los parámetros de configuración de cada ejecución.
