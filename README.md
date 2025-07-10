# Recomendación de películas

En esta era de la conectividad constante, las plataformas ofrecen una amplia gama de programas, películas y documentales. El precio accesible, el amplio catálogo de contenidos, la flexibilidad de horarios, la ausencia de parones publicitarios y la comodidad de estar en tu propia casa son algunas de las ventajas del streaming respecto a la televisión. En ocasiones, el espectador encuentra dentro de la vorágine del día un espacio de desconexión y lo destina a ver una película, pero seleccionar que mirar puede representar gran parte del tiempo disponible. Por tal motivo, sería ventajoso contar con un análisis que permita al espectador guiarse en su elección de forma rápida según sus preferencias. También puede ser utilizado por las empresas cinematográficas.

Por todo esto se planta como objetivo, a partir de un modelo de aprendizaje supervisado, predecir la calificación final de una película, tomando esta variable como indicador de aceptación por el público.

---

## Selección de Dataset

El Dataset seleccionado fue extraído de [https://www.kaggle.com/datasets/ky1338/10000-movies-letterboxd-data](https://www.kaggle.com/datasets/ky1338/10000-movies-letterboxd-data). Los datos se recopilaron en una lista de Letterboxd. Esta lista, diseñada como una ruleta de películas, cuenta con 10 002 entradas de diversos géneros y décadas. Los datos fueron tomados el 8/4/2025.

---

## Análisis exploratorio de datos

**Caracterización del DataFrame:** Se caracterizó el DataFrame según cantidad y tipo de variables.

**Análisis de variables categóricas:** en algunas columnas encontramos las variables categóricas como listas. Las separamos y conservamos el primer valor. Este procedimiento lo realizamos para las columnas Cast, Genres, Countries, Studios y Director. Cuando exploramos el resultado observamos valores repetidos, así que procedimos a unificarlos.

**Creación de una nueva variable:** creamos una nueva variable, Porcentaje_likes. Creemos que será de utilidad en el modelo predictivo.

**Eliminación y renombramiento de columnas:** eliminamos aquellas columnas que no serán de utilidad para la aplicación del modelo no supervisado y renombramos en castellano las que conservamos.

**Datos faltantes:** realizamos el análisis de los datos faltantes numéricos y luego categóricos. En el primer caso aplicamos imputación simple. Respecto a los valores categóricos eran solo 2 por lo tanto procedimos a la eliminación de las filas que los contenían.

**Outliers:** primero identificamos aquellas columnas que tenían valores numéricos para luego proceder al análisis de cada una. De cada una visualizamos su distribución para luego definir el manejo. En caso de distribución normal utilizamos zscore para detectar outliers y en caso de distribución sesgada utilizamos rango intercuartílico. Se justificó en cada caso la no eliminación de los outliers. En el caso de la duración se detectó que había series en la lista por lo que en ese caso se procedió a identificarlas y eliminarlas.

**Selección de columnas para aplicar modelo de aprendizaje no supervisado:** realizamos una última selección de las columnas que utilizaremos para el análisis y eliminamos el resto.

**Matriz de correlación (mapa de calor):** una vez seleccionadas las columnas con las que vamos a aplicar el modelo, realizamos el mapa de calor para asegurarnos que las variables no están altamente correlacionadas. Observamos que las variables Porcentaje_likes y Calificación_pormedio y Ratings y Vistas tienen alta correlación por lo tanto, eliminaremos Porcentaje_likes (conservaremos Calificación_promedio porque es la variable que seleccionamos predecir en los modelos supervisados) y Vistas. Evidentemente la nueva variable formulada, no fue de utilidad.

---

## Transformación de datos

**Transformación con One Hot Encoding:** Debido a que las variables a transformar son categóricas y no tienen un orden predeterminado utilizamos OneHotEncoding. Las variables categóricas transformadas son 'Director', 'Genero', 'Pais', 'Lenguaje', 'Productora' y 'Protagonista'. Por último, transformamos las variables booleanas a enteros.

---

## Guardamos el dataset

---

## Modelo de aprendizaje no supervisado

**Cargamos y visualizamos tipos de variables del Dataset**

---

### Clustering

#### Kmeans

##### Elbow

Aplicamos el método Elbow para encontrar el valor de k óptimo. A partir de dicho valor, aplicaremos kmeans.

Debido a que no se observó un codo claramente, incluso la función arroja error, y tampoco pudimos encontrarlo de forma analítica, aplicamos a continuación tres opciones:

1. Utilizamos Silhouette Score en vez de Elbow  
2. Realizamos PCA antes de aplicar Elbow  

---

### PCA

Elegimos automáticamente cuántos componentes PCA conservar para explicar al menos el 90% de la varianza total. Debido a que el resultado fue muy elevado utilizamos el método gráfico para ver cuánta varianza se explica con menos componentes. Nos permitió visualizar si necesitábamos realmente 5915 o si con menor cantidad de componentes explicábamos aproximadamente el 90%.

En el gráfico se observó claramente que necesitamos una gran cantidad de componentes para explicar el 90% de la varianza y que por la forma de la curva una disminución de los componentes implica una disminución importante en la varianza explicada. Esto se debe a que aplicamos One hot encoding y cada categoría nueva crea una columna. Numerosas columnas categóricas ocupadas con muchos 0 como valor originaron información redundante o casi nula.

Debido a esto realizamos la elección y la carga manual de los componentes (k=2), graficamos y aplicamos Elbow.

---

3. Ampliamos el rango en Elbow

El rango hace referencia al conjunto de valores de k (número de clusters) que evaluamos para encontrar el óptimo. Modificamos el límite 10 a 20.

Las primeras dos opciones nos arrojaron diferentes valores de k y la tercera ningún valor. Procedimos a realizar KMeans con los primeros dos resultados (k=2 y k=3)

---

### Aplicación de kmeans

En los dos casos se observó un importante desbalance en la cantidad de datos que integran los clusters. Debido a que el dataframe presentaba outliers, que fueron conservados por no pertenecer a errores de registro, kmeans es sensible a ellos y no sería el método más apto para la formación de clusters.

---

### DBSCAN

Debido a que con kmeans no pudimos establecer un resultado, aplicamos DBSCAN posteriormente a PCA. Se observó desbalanceo de los datos por clusters y no se observó una separación clara de clusters, por lo tanto, optimizamos los parámetros eps y min_samples.

**min_samples:** utilizamos 3 debido a que contamos con PCA reducido a 2 componentes  
**eps:** k-distancia y KneeLocator

Volvimos a aplicar DBSCAN con los parámetros optimizados, pero tampoco obtuvimos un resultado favorable.

---

### Gaussian Mixture Models (GMM)

- GMM no asume que los clusters son esféricos o equidistantes.
- Permite que los clusters tengan formas elípticas y diferentes tamaños.
- Es más flexible para datos con estructuras complejas.

Aplicamos este modelo luego de aplicar PCA y analizamos las métricas. Observamos que este modelo se ajustó mejor a nuestros datos y por lo tanto, procedimos a optimizar los parámetros.

**n_componentes:** para la reducción con PCA  
**n_componentes (GMM):** número de clusters en el modelo

Con los parámetros optimizados volvimos a aplicar GMM. Analizamos las métricas y el modelo presentó mejor ajuste que anteriormente. Realizamos el análisis de los clusters.

---

## Faltaría un análisis mejor de los clusters y las conclusiones
