**Measurements of 10 Applications with OZM v2 in two days**

This is the last repository of measurements using the new version of oVm v2 (three-phase version of oZM), whose tests began on 2023-03-07 and ended on 2023-06-09 at 13:36:19+02:00 in the Electrical Engineering Laboratory of the School of Industrial Engineering of the University of Almeria. In this repository, measurements with 3 three-phase ozm are analysed, making up a total of 10 applications plus the aggregate, all of these during 6 hours spread over two different days.


The analysis of the measurements of 10 applications including transients up to order 150 of voltage, current and power is presented in the notebooks attached to this repository. The measurements are made with 3 three-phase OpenZmeters (each with 4 measurement channels), making a total of 11 measurement channels distributed among the 10 applications, plus the aggregate.


The measurements correspond to W, VAR, VA, f, VLN, PF and A, plus harmonics up to order 50 of W, V and A, all with a 13-digit UNIX Epox timestamp.
Therefore, this new repository analyses the impact of taking extra time to train the model with the harmonics up to order 50 of the voltage, current and power provided by OZM v2. It should be noted that the attached Jupyter Notebooks contain not only the Python code but also the results of running the dataset. Also, to run this code we need to have the NILMTK toolkit installed (also available on Github), as well as the new dataset with the additional data.

**\*\*DUE TO ITS EVEN LARGER SIZE, THE CSV FORMAT DATA FILES ARE NOT AVAILABLE IN THIS PARTICULAR REPOSITORY, NOR IS THE DATASET, BUT THE COMPLETE DATASET WITH ALL HARMONICS WITH THE USUAL SAMPLING TIME IS AVAILABLE IN THE DSUALM10H REPOSITORY.. \*\***

Our goal is to provide NILM researchers with new data repositories to expand the existing range. As these new datasets can contain more than 150 electrical variables recorded at high frequency in different everyday applications, by offering this wide range of data, we hope to boost and improve NILM research.


In this repository we use 3 OZM v2 units that allow us to record real time electrical measurements from 10 devices plus the aggregate, and also in this case we will take an extra time of measurements (from 9:34 to 13:36).


This was the division of the dataset:

-   train\*\*.**set_window(start**=\*\*"2023-06-09 09:34:00", end**=**"2023-06-09 10:54:00")
-   test\*\*.**set_window(start**=\*\*"2023-06-09 12:55:00", end**=**"2023-06-09 13:36:00")
-   valid\*\*.**set_window(start**=\*\*"2023-06-09 10:55:00", end**=**"2023-06-09 13:36:00")

These data were trained with both the combinatorial algorithm (CO) and the Hidden Markov algorithm (FHMM), but the algorithm that returned the best results was CO, given that with FHMM it was impossible to run it with sampling times of less than 90 seconds due to a lack of physical memory (it even returned errors using machines with more than 64GB of RAM).


The generation of the Dataset was done with the new converter/converter designed for this occasion, starting from 11 csv files of measurements that resulted from joining each csv file of each application of each day with the csv of that same application and that same day.

**We recommend the reader to go to step 8 of Metrics where you can study the results of these metrics, which in principle seemed promising but have been shown not to improve the results compared to the metrics obtained against the dataset formed by the data of only one day*.*.

The main conclusions of taking only the odd harmonics, disregarding all harmonics, taking all harmonics (odd and even), or extending the sample time are presented in a generic way below.

**ARCHITECTURE**

For the unbundling process with OZM V2 smart meters (three-phase version of OZM) we will use the NILMTK Toolkit, the flow of which can be seen in the illustration below.

![Diagrama Descripción generada automáticamente](media/ae003a4fce3933cef4ec0c557d516ceb.png)

Figure 2-NILMTK flow diagram

1. **Generation of the new DSs**.


The models presented in this study employ the multiple operating hour records of various devices using the OZM API, for which we use three three-phase OZM devices to obtain 12 measurement channels, reserving one of these for aggregation.

![Diagrama Descripción generada automáticamente](media/64ff0f26702e07a364d4358738a2d4d0.png)

Figure 3 - Schematic diagram of OZM's connections with the applications.

The applications used in the experiment are the following:


1 -Main


2 - Electric Furnace


3- Microwave (Micro Wave)


4 - Television


5 - Bulb


6 - Vacuum Cleaner (Vacuum cleaner)


7 - Electric Space Heater (Oil Radiator)


8 - Electric Shower Heater (Water Heater)


9 - Fan


10 - Fridge


11 - Freezer

The data collected by the OZMs are stored in files with 160 data fields. However, not all of these fields are relevant in all phases of this study, therefore, it is necessary to adapt them for use in NILMTK [1].


As a first step, a preliminary analysis of the data files or pre-processing of the data will be performed, where all the measurement files will be decompressed from the *parquet* format to the csv format, adding a header with the names of the measurements and substituting the angular values of the harmonics by the module. Also, a date and time analysis is performed, since there may be mismatches in the three date fields returned by ozm(*FistTimestamp, OriginaTimestamp* and *Time*). Finally, for each csv file, the headers are reorganised and unwanted characters are removed.

![Interfaz de usuario gráfica, Texto, Aplicación Descripción generada automáticamente con confianza media](media/b6d729bf82d005c27e0f5052f1fd0315.png)

Table 1- Different dates generated by oZM

The next task of loading and analysing the data consists mainly of converting the different measurement files preprocessed in the previous phase together with the metadata into a single file in HDF5 format, which will be stored in the run folder [2].


Normally, NILMTK uses standardised DS formats, but due to the exclusivity of the data provided by OZM, a new converter for the data is needed. For this, a new converter has been created, as well as a new function *convert_ualm* to load the measurement files in csv format from the OZM to the new DS in H5 format. For this purpose, in the NILMTK directory of the converters, not only the new Python code of the converter (based on the IAWE converter) will be included, but also a subdirectory will be created in "/metadata/" containing the metadata files in YAML format. Figure 4 shows the configuration of all the files needed for the new converter, as well as the required directory structure.

![](media/f0fc612321c8838d7279158e7cde4234.png)

Figure 4-Metadata file structure

Como cada fichero csv es obtenido en la fase de anterior a partir de los ficheros de los OZM, es necesario numerarlos, siendo el nº 1 el correspondiente al medidor principal. Para ello, la nueva función accede a todos los citados ficheros de datos de medidas localizados en la carpeta de entrada “/electricity/”, usando para ello el fichero de etiquetas labels.csv, proceso que representamos en la Ilustración 5.

![](media/6ae09e667f0eab524fc770ce3f816fba.png)

Figure 5-Data file structure

Ubicados los ficheros de datos, lo primero es invocar al conversor llamando a la nueva función **convert_ualm**, pasándole la ruta de los metadatos y el nuevo nombre del DS. Básicamente el nuevo convertidor realiza los siguientes pasos para cada fichero de medidas:

-   Lectura del fichero numerado.
-   Conversión a formato fecha del campo timestamp.
-   Carga del resto de columnas.
-   Sort-index.
-   Resample.
-   Re-indexación del fichero.

Una vez se han procesado todos los ficheros de medidas, procedemos a unir éstos en formato yaml, para posteriormente añadir los metadatos y generar finalmente un nuevo DS en formato H5.

Creado finalmente el nuevo DS, podemos realizar un preanálisis de los datos, siendo especialmente interesante representar la fracción del consumo de energía de cada aparato.

![Gráfico, Gráfico circular Descripción generada automáticamente](media/c55b5209b66b10dcb86ea5bcde9745ba.png)

Figure 6- Representation of measurements

Es asimismo interesante observar los gráficos de tensión, potencias y corriente para todos los diferentes aplicativos, así como del agregado. No obstante, un gráfico muy aclaratorio, consiste en trazar los datos submedidos junto los datos del medidor principal.

![Gráfico, Histograma Descripción generada automáticamente](media/d23ff6bb98853429d031c72d602effa9.png)

Table 2-Data of all meters

Finalmente, en este estudio se hace necesario estudiar la correlación de los datos, así como su relación con posibles cambios en el muestreo.

![Gráfico, Histograma Descripción generada automáticamente](media/f3a4da44a09f9489b6988dd032b63af2.png)

Figure 7-Graph ot autocorrelation

**Análisis, Preprocesamiento, Entrenamiento, Validación, Desagregación y Métricas**

Una vez hemos generado el nuevo DS, podemos usar las implementaciones que dispone NILMTK para realizar un diagnóstico rápido del DS. Es especialmente interesante obtener el perfil de la potencia en un gráfico de área de los aplicativos (Ilustración 7).

![Gráfico, Gráfico de barras Descripción generada automáticamente](media/f80b6e170f22320613d24d9fdc1bdf5b.png)

Figure 8- Voltage profile

En esta etapa también se obtiene el perfil del voltaje, se hace el cálculo posibles secciones faltantes o se descartan aquellas muestras con valores muy bajos aplicando filtros. Asimismo, es interesante obtener el registro de actividad que mostramos a continuación.

![Texto Descripción generada automáticamente](media/387c5ae802a667bc9d8f4e9f93075602.png)

Tabla 3-Registry of activity

Finalmente analizados los datos, dividiremos el DS en **set de entrenamiento, set de validación y set de pruebas.** Para entrenar el modelo, usamos dos de los modelos de desagregación que existen disponibles en NILMTK, como son los algoritmos supervisados CO y FHMM, usando la señal de potencia activa de los dispositivos. Para ello, previamente además de cargar las librerías necesarias, definiremos el DS, y asociaremos las etiquetas asociadas a los electrodomésticos y finalmente definiremos el subconjunto de entrenamiento.

Definido el modelo de entrenamiento, gracias a que NILMTK implementa los dos algoritmos de desegregación, ejecutaremos los dos algoritmos CO y FHMM en diferentes intervalos de tiempo de muestreo usando tres métodos diferentes de relleno (First, Mean y Median), salvando los modelos generados en formato H5. Una vez computadas todas las opciones, seleccionamos el mejor modelo evaluado en la etapa de validación y ya podremos desagregar el modelo.

![Gráfico, Gráfico de barras, Histograma Descripción generada automáticamente](media/5c356e3a7720f282ce9e44a7ec826bad.png)

Figure 9- Disaggregate chart

Finalmente, una vez desagregado las diferentes aplicativos, toca la fase de obtención de métricas para comprobar cómo ha respondido el mejor modelo, y sobre todo para decidir, si es necesario con los resultados obtenidos cambiar éste por otro más eficiente. Para facilitar esta labor, en esta última etapa precisamente NILMTK implementa numerosas herramientas gráficas y textuales para ayudarnos en la selección del modelo que nos ofrezca las mejores métricas posibles.

Un ejemplo de gráfico para ayudarnos a seleccionar el método de relleno es el siguiente que no muestra el comportamiento de las métricas F1, EAE, MNEAP y RMSE para la ejecución del algoritmo combinatorio con diferentes métodos de relleno.

![Una captura de pantalla de un celular Descripción generada automáticamente con confianza media](media/22f409336d75b20d796d9dc9b22aebb5.png)

Table 4- Final results of CO

**Medidas definitivas finales con OZM tomadas el 6 de junio de 2023**

Comenzaron el día 2023-06-09 a las 09:34:38+02:00 y terminaron el día 2023-06-09 a las 13:36:19+02:00 en el Laboratorio de Electrotecnia de la Escuela de Ingeniería Industrial de la Universidad de ALmeria. También se consideran medidas unitarias del día 6 del 6. Las Medidas se realizan con 3 OpenZMeter Trifásicos (cada uno con 4 canales de medida) conformando así en total 11 canales de medida que se distribuyen en los 10 aplicativos, más el agregado.

En este repositorio se analizarán todas las medidas eléctricas por medio de 3 OpenPowerMeter (OzM) con soporte de 4 canales medidor (en total 12 canales de medidas eléctricas), conformando así en total 11 canales útiles de los cuales 10 son para aplicativos, uno al agregado, y una última vacante realizándose todas las medidas estas durante 4 horas a la máxima frecuencias de muestreo.

Todo el proceso se realiza en los cuadernos adjuntos a este repositorio fundamentalmente versando en el análisis de las medidas de 10 aplicativos incluyendo transitorios hasta el orden 150 de tensión, corriente y potencia.

Las medidas corresponden a W, VAR, VA, f, VLN, PF y A, más los ARMONICOS hasta el orden 150 de W, V y A, todas con una marca de tiempo (Timestamp) de 13 dígitos tipo UNIX Epox.

Para el entrenamiento se han definido tres periodos:

-   TRAIN(start="2023-06-09 09:34:00", end="2023-06-09 12:54:00")
-   VAL(start="2023-06-09 12:55:00", end="2023-06-09 13:36:00")
-   TEST (start="2023-06-06 11:19:19", end="2023-06-06 11:40:28")

Estos datos se entrenaron, tanto con el algoritmo CO, como el algoritmo FHMM.

C. Mejoras necesarias asociadas al aumento del número de aplicativos

Los resultados obtenidos para 5 aplicativos en experimentos previos [20] nos han corroborado que NILMTK con los nuevos datos de OZM responde muy bien para 5 aplicativos con tres horas de toma de muestras, obteniendo excelentes métricas (por cierto, similares tanto con CO como con FHMM) usando tiempos de muestreo óptimos entre 1 seg y 30seg y sin necesidades de cómputo específicas para procesar el Dataset (especialmente con FHMM).

![Tabla Descripción generada automáticamente](media/297c59ec44c1b7470918f21972f879d2.png)

Tabla 5-Results for 5 apliances

Desgraciadamente a medida que se aumenta el número de aplicativos, aumenta también la complejidad de procesar esta gran cantidad de datos con resultados aceptables. Prueba de ello, han sido las numerosas tentativas que se han realizado experimentalmente hasta llegar a la publicación del presente conjunto de datos.

Dado que la métrica F1-score mide la precisión global de la clasificación de los dispositivos eléctricos, si la comparamos para diversos aplicativos y en diferentes experimentos estos nos van a dar una idea de la precisión en la clasificación de los dispositivos en general, lo cual podemos apreciar en la gráfica siguiente.

![Gráfico, Gráfico de barras Descripción generada automáticamente](media/a01eede30d7e3c556a2bcdaca8ff4990.png)

Tabla 6- Results of F1 in the experiments

Por otro lado, RMSE (Root Mean Squared Error) es una métrica utilizada habitualmente para evaluar la precisión de un modelo a la hora de predecir el consumo eléctrico de aparatos individuales o el consumo energético agregado de un hogar. Para ello el RMSE mide la diferencia cuadrática media entre los valores de consumo de energía predichos y los reales, y se expresa en las mismas unidades que los datos de consumo de energía. Un valor alto de RMSE en NILMTK indica que el modelo no predice con exactitud el consumo energético de los electrodomésticos o del hogar, lo cual significa que los valores predichos están muy alejados de los valores reales.

![Gráfico, Gráfico de barras Descripción generada automáticamente](media/a525306d9ba678c40261f40d2690bff8.png)

Tabla 7- Results of RMSE in the experiments

En general, un valor de RMSE bajo indica que el modelo predice mejor el consumo de energía, mientras que un valor de RMSE alto indica que el modelo tiene margen de mejora, lo cual podemos ver reflejado en el siguiente gráfico.

Como ejemplo de las fluctuaciones que pueden llegar a producirse por ruido, mal alineamiento del sensor de corriente, o fallo de este, reproducimos aquí las medidas de potencia del 25/01/2023 de la aspiradora donde se observaron 7056 medidas con potencia negativa claramente erróneas (marcadas en amarillo).

![Gráfico Descripción generada automáticamente](media/a580ca601dc1b012a274e27953f9dca2.png)

Figure 10-Evolution of power for vacuum cleaner

Esto mismo resultado, gracias a las herramientas graficas que cuenta NILMTK lo podemos apreciar diferenciado para la tensión, potencias (activa o reactiva) y la corriente:

![Gráfico Descripción generada automáticamente](media/9dd979458ced15e0f5778ee5a09d0d33.png)

Figure 11-Power, current and voltage for vac

Si finalmente comparamos las principales métricas con las mejoras sucesivas realizadas en sucesivos experimentos, como son la ampliación del tiempo de las muestras, la reducción del ruido eléctrico, cambio de sensores y la mejora en el alineamiento de las fases activas en los sensores de núcleo dividido, vemos como hay señales de mejora significativas.

![Gráfico, Gráfico de líneas Descripción generada automáticamente](media/93e8658d5396e28e3b58b6c5bdb385d5.png)

Figure 12- Improvement of metrics in the different experiments

A medida que se aumenta el número de aplicativos en el entrenamiento usando el algoritmo combinatorio no hay problemas incluso con tiempos de sampling muy pequeños, pero con FHMM puede haber dificultades importantes a la hora de ejecutar el algoritmo con el nuevo denso conjunto de datos, para lo cual se hace necesario reducir el tiempo de entrenamiento o bien reducir el tiempo de sampling (o incluso reducir el número de aplicativos), tal y como se ilustra en la siguiente tabla.

![](media/6302c2338200f183cc06be7770ecd77a.emf)

Table 8- Training possibilities with FHMM

En la tabla 8 vemos como se hace patente que para poder ejecutar FHMM con gran cantidad de muestras de alta frecuencia, se hace necesario o bien reducir el periodo de entrenamiento o bien aumentar el sampling para producir un modelo satisfactorio.

**4. RESULTADOS**

NILMTK cuenta con el cálculo de métricas de evaluación por medio del uso del MeterGroup para la validación de los resultados mediante el set de validación. Es preciso ejecutar para ello, sobre los modelos obtenidos, diferentes métricas como son FEAC, F1, EAE, MNEAP y RMSE, que para el mejor modelo (CO con sampling de 1seg) nos da una salida algo similar a la Tabla 9.

![Tabla Descripción generada automáticamente](media/2163dc9c3c3c2e4bc4f7349a0a5fa103.png)

Table 9-Metrics of CO for 10 apliances

Es decir, obtenemos una puntuación para F1-Score de Media de 45.41, un valor excelente de EAE (0 de media), un valor muy bajo perfecto de MNEAP (media 1.9809) y un valor aceptable de RMSE (media 372.8472).

**Resumen de resultados sin armónicos**

En general la eliminación de armónicos para el mismo periodo entrenamiento, mismo algoritmo (CO) y un periodo de sampling de 1seg, empeora levemente casi todas las métricas para casi todos los aplicativos. En la siguiente tabla podemos ver los resultados:

![Tabla Descripción generada automáticamente](media/0508e1d7129202cdea3b8468637f0dd4.png)

Table 13- Summary metrics without harmonics

Es decir, obtenemos una puntuación para F1-Score de Media 47,29 (algo mejor que incorporando los armónicos), un valor bastante peor de EAE (0,472 de media), un valor bajo de MNEAP (media 6,0115) pero peor que incluyendo armónicos cuya media era de 1.9809) y un valor algo peor de RMSE (media 382,2369 frente a 372.8472).

**Resumen de resultados sin armónicos impares**

En general la incorporación de solo los armónicos pares despreciando los armónicos impares para el mismo periodo de entrenamiento, mismo algoritmo (CO) y un periodo de sampling de 1seg, NO mejora las métricas para casi todos los aplicativos. En la siguiente tabla podemos ver los resultados:

![Interfaz de usuario gráfica, Aplicación Descripción generada automáticamente](media/f2965dc14f31dfad59bb853854b8504b.png)

Table 11- Metrics for even harmonics only

Es decir, obtenemos una puntuación para F1-Score de Media 44,01 (peor que con armónicos o sin ellos), un valor excelente de EAE (0 de media), un valor muy bajo de MNEAP (media 2,2877) y un valor aceptable de RMSE (media 401,3312) pero bastante peor que el valor obtenido con armónicos de 372.8472.

**Comparación con otros DS**

Para el DS de IAWE los resultados nos evidencian que el algoritmo más eficiente para este DS es el combinatorio (CO) usando el método Mean y **periodo 10 minutos, frente sólo al 1 segundo necesarios con los datos del OZM**.

![Tabla Descripción generada automáticamente](media/8ebd0ec88d9f344dac9a694ecfd873f8.png)

Table 14- Results of IAWE

Por otro lado, los resultados obtenidos para el DS de DEPS nos evidencian un mejor rendimiento para el algoritmo CO, método Mean, pero **a un tiempo de muestreo de media hora, frente sólo al 1 segundo necesario con los datos del OZM**.

![Tabla Descripción generada automáticamente](media/8a92a55f7a375c6f472b1acb1a9a86a7.png)

Figure 15-Results of DEPS

Asimismo, si comparamos GT y Pred para el DS de DEPS, las divergencias son muy importantes, oscilando entre 1,4%, 4,6% y 4,9% frente al 0 y 1,6% que tenemos en DSUAL.

![Gráfico, Gráfico circular Descripción generada automáticamente](media/2b62711a9cf58ecc96ca66410a1d8862.png)

Figure 16- Comparison GT with Pred for DEPS

**V. CONCLUSIONES**

En este trabajo en el ámbito de NILMTK, además de incorporar tanto las métricas como las herramientas disponibles en el toolkit, se ha incorporado como novedad, el nuevo formato de timestamp de 13 dígitos además de nuevos conversores y convertidores para las medidas obtenidas de OZM, de modo que así se elimina la barrera de entrada a todo aquel investigador que cuente con uno o varios OZM y desee acceder al NILM. Precisamente comparando los DS obtenidos (con o sin transitorios), se ha demostrado que el uso de armónicos puede mejorar el resultado de las métricas, dependiendo, eso sí, del aplicativo a considerar.

Por otro lado, si comparamos los resultados de las métricas obtenidas sobre DSUALM o DSUALMT, frente a los DS de IAWE o DEPS los resultados son mucho peores en estos últimos, especialmente en cuanto al periodo de muestreo necesario. Destaca además el error mínimo en la desagregación en DSUALM, así como los mejores valores obtenidos para las métricas MNEAP y RMSE.

**Publicaciones**

Hay un artículo de mi autoría sobre el NILM que usa el hardware OZM monofásico en lugar del OZM v2:

-   C. Rodriguez-Navarro, A. Alcayde, V. Isanbaev, L. Castro-Santos, A. Filgueira-Vizoso, and F. G. Montoya, “DSUALMH- A new high-resolution dataset for NILM,” *Renewable Energy and Power Quality Journal*, vol. 21, no. 1, pp. 238–243, Jul. 2023, doi: 10.24084/repqj21.286.

Asimismo, con el fin de hacer replicable todo este trabajo se ha desarrollado un nuevo multi contador abierto llamado OMPM esta publicada en la revista científica “Inventions:”

-   C. Rodríguez-Navarro, F. Portillo, F. Martínez, F. Manzano-Agugliaro, and A. Alcayde, “Development and Application of an Open Power Meter Suitable for NILM,” *Inventions*, vol. 9, no. 1, p. 2, Dec. 2023, doi: 10.3390/inventions9010002.
