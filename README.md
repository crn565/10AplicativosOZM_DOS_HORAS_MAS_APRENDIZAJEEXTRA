# Medidas de 10 Aplicativos con  OZM  en dos dias 

10 Aplicativos OZM_dos dias medidas( 3-7-2023  y 6-9-2023)



Comenzaron el dia 2023-03-07 y terminaron el dia 2023-06-09 a las 13:36:19+02:00 en el Laboratorio de Electrotecnia de la Escuela de Ingenieria Industrial de la Universidad de Almeria.

En este respositorio se analizaran medidas con 3 ozm trifasicos conformando en total 10 aplicativos mas el agregado, todo estos durante 6 horas repartidas en dos dias diferentes.

Se presenta en los cuadernos adjuntos a este repositorio el analisis de las medidas de 10 aplicativos incluyendo transitorios hasta el orden 150 de tension, corriente y potencia. Las Medidas se realizan con 3 OpenZMeter Trifásicos (cada uno con 4 canales de medida) conformando asi en total 11 canales de medida que se distribuyen en los 10 aplicativos, mas el agregado.

Las medidas corresponden a W, VAR, VA,f, VLN,PF y A, mas los transititoros hasta el orden 50 de W, V y A, todas con un marca de tiempo (Timestamp) de 13 dígitos tipo UNIX Epox. 


Para el entrenamiento se han definido tres periodos:

- TRAIN(start="2023-03-07 09:34:00", end="2023-06-09 12:54:00")

- VAL:(start="2023-06-09 12:55:00", end="2023-06-09 13:36:00")

- TEST: (start="2023-06-06 11:19:19", end="2023-06-06 11:40:28")

Estos datos se entrenaron, tanto con el algoritmo CO, como el algoritmo FHMM, pero el algoritmo que mejores resultados devolvio es CO, dado que con FHMM ha sido imposible ejecutarlo con tiempos de sampling inferiores a 90segundos por falta de memoria fisica ( incluso devolvió errores usando maquinas con mas de 64GB de RAM).


Los aplicativos usados en el experimento son los siguientes:

- 1 -Main

- 2 - Electric Furnace ( Horno)

- 3- Microwave (Microonda)

- 4 - Television

- 5 - Bulb ( bombilla)

- 6 - Vacuum Cleaner ( Aspiradora)

- 7- Electric Space Heater ( Radiador de aceite)

- 8 - Electric Shower Heater (Calentador de agua)

- 9 - Fan ( Ventilador)

- 10 - Fridge ( refrigerador)

- 11 - Freezer (congelador)


La generacion del Dataset se hizo con el nuevo convertidor/caonversor  diseñado para esta ocacion, partiendo de 11 ficheros csv de medidas que resultaron de unir cada fichero csv de cada aplicativo de cada dia  con el csv de ese mismo aplicativo y ese mimso dia. 


RESULTADOS FINALES 

La rápida expansión de NILM y el desarrollo de diferentes algoritmos, han hecho que sea esencial proporcionar una evaluación de rendimiento mediante el uso de métricas de desempeño. Las métricas de evaluación, comparan los resultados de la desagregaciónn (predicciones) de los modelos entrenados con los datos del set de validación (mediciones reales de cada proceso). NILMTK cuenta con el cálculo de métricas de evaluación mediante el uso del MeterGroup para la validación de los resultados mediante el set de validación entes y lo mas variadas posibles .
Recomendamos al lector se vaya al paso 8 de Metricas donde se pueden estudiar los resultados de esas metricas, que en principio parecian proometedoras pero se ha desmostrado no mejoran  los resultados frente a las metricas obtendias contra el  dataset formado por los datos de solo un dia.
