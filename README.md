#
# Measurements of 10 Applications with OZM v2 in two days**
#

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

As each csv file is obtained in the previous phase from the OZM files, it is necessary to number them, with number 1 corresponding to the main meter. To do this, the new function accesses all the aforementioned measurement data files located in the input folder "/electricity/", using the labels.csv file, a process that is shown in Illustration 5.

![](media/6ae09e667f0eab524fc770ce3f816fba.png)

Figure 5-Data file structure

Once the data files are located, the first step is to invoke the converter by calling the new function **convert_ualm**, passing it the metadata path and the new DS name. Basically the new converter performs the following steps for each measurement file:


- Reading of the numbered file.
- Conversion to date format of the timestamp field.
- Load the rest of the columns.
- Sort-index.
- Resample.
- Re-indexing of the file.

Once all the measurement files have been processed, we proceed to merge them in yaml format, to later add the metadata and finally generate a new DS in H5 format.


Once the new DS is finally created, we can perform a pre-analysis of the data, being especially interesting to represent the fraction of energy consumption of each device.

![Gráfico, Gráfico circular Descripción generada automáticamente](media/c55b5209b66b10dcb86ea5bcde9745ba.png)

Figure 6- Representation of measurements

It is also interesting to look at the voltage, power and current graphs for all the different applications as well as the aggregate. However, a very informative graph is to plot the submetered data together with the data of the main meter.

![Gráfico, Histograma Descripción generada automáticamente](media/d23ff6bb98853429d031c72d602effa9.png)

Table 2-Data of all meters

Finally, in this study it is necessary to study the correlation of the data, as well as its relation to possible changes in sampling.

![Gráfico, Histograma Descripción generada automáticamente](media/f3a4da44a09f9489b6988dd032b63af2.png)

Figure 7-Graph ot autocorrelation

**Análisis, Preprocesamiento, Entrenamiento, Validación, Desagregación y Métricas**

Once we have generated the new DS, we can use the implementations available in NILMTK to perform a quick diagnosis of the DS. It is particularly interesting to obtain the power profile in an area graph of the applications (Illustration 7).

![Gráfico, Gráfico de barras Descripción generada automáticamente](media/f80b6e170f22320613d24d9fdc1bdf5b.png)

Figure 8- Voltage profile

At this stage, the voltage profile is also obtained, and possible missing sections are calculated or samples with very low values are discarded by applying filters. It is also interesting to obtain the activity log shown below.

![Texto Descripción generada automáticamente](media/387c5ae802a667bc9d8f4e9f93075602.png)

Tabla 3-Registry of activity

Finally, after analysing the data, we will divide the DS into **training set, validation set and test set** To train the model, we use two of the disaggregation models available in NILMTK, such as the supervised algorithms CO and FHMM, using the active power signal of the devices. To do this, in addition to loading the necessary libraries, we will first define the DS, associate the labels associated with the appliances and finally define the training subset.


Once the training model is defined, thanks to the fact that NILMTK implements the two desegregation algorithms, we will run the two algorithms CO and FHMM at different sampling time intervals using three different filling methods (First, Mean and Median), saving the generated models in H5 format. Once all the options have been computed, we select the best evaluated model in the validation stage and we will be able to disaggregate the model.

![Gráfico, Gráfico de barras, Histograma Descripción generada automáticamente](media/5c356e3a7720f282ce9e44a7ec826bad.png)

Figure 9- Disaggregate chart

Finally, once the different applications have been disaggregated, it is time to obtain metrics to check how the best model has performed and, above all, to decide whether it is necessary, based on the results obtained, to change it for a more efficient one. To facilitate this task, in this last stage NILMTK implements numerous graphical and textual tools to help us in the selection of the model that offers the best possible metrics.

An example of a graph to help us select the infill method is the following which shows the behaviour of the F1, EAE, MNEAP and RMSE metrics for running the combinatorial algorithm with different infill methods.

![Una captura de pantalla de un celular Descripción generada automáticamente con confianza media](media/22f409336d75b20d796d9dc9b22aebb5.png)

Table 4- Final results of CO

**Final definitive measurements with OZM taken on 6 June 2023**

They started on 2023-06-09 at 09:34:38+02:00 and ended on 2023-06-09 at 13:36:19+02:00 in the Electrical Engineering Laboratory of the School of Industrial Engineering of the University of ALmeria. The measurements are also considered unitary measurements of day 6 of 6. The measurements are carried out with 3 three-phase OpenZMeters (each one with 4 measurement channels) making a total of 11 measurement channels that are distributed in the 10 applications, plus the aggregate.


In this repository all the electrical measurements will be analysed by means of 3 OpenPowerMeter (OzM) with support of 4 measuring channels (a total of 12 channels of electrical measurements), thus forming a total of 11 useful channels of which 10 are for applications, one for the aggregate, and a last vacancy, all the measurements being carried out during 4 hours at maximum sampling frequencies.


The whole process is carried out in the notebooks attached to this repository, mainly dealing with the analysis of the measurements of 10 applications, including transients up to order 150 of voltage, current and power.

The measurements correspond to W, VAR, VA, f, VLN, PF and A, plus the ARMONICS up to order 150 of W, V and A, all with a 13-digit UNIX Epox timestamp.


Three periods have been defined for the training:

-   TRAIN(start="2023-06-09 09:34:00", end="2023-06-09 12:54:00")
-   VAL(start="2023-06-09 12:55:00", end="2023-06-09 13:36:00")
-   TEST (start="2023-06-06 11:19:19", end="2023-06-06 11:40:28")

These data were trained with both the CO algorithm and the FHMM algorithm.


C. Necessary improvements associated with increasing the number of applications


The results obtained for 5 applications in previous experiments [20] have corroborated that NILMTK with the new OZM data responds very well for 5 applications with three hours of sampling, obtaining excellent metrics (incidentally, similar with both CO and FHMM) using optimal sampling times between 1sec and 30sec and without specific computational needs to process the Dataset (especially with FHMM)..

![Tabla Descripción generada automáticamente](media/297c59ec44c1b7470918f21972f879d2.png)

Tabla 5-Results for 5 apliances

Unfortunately, as the number of applications increases, so does the complexity of processing this large amount of data with acceptable results. Proof of this has been the numerous attempts that have been made experimentally up to the publication of the present dataset.


Given that the F1-score metric measures the overall accuracy of the classification of electrical devices, if we compare it for different applications and in different experiments, it will give us an idea of the accuracy in the classification of devices in general, which can be seen in the following graph.

![Gráfico, Gráfico de barras Descripción generada automáticamente](media/a01eede30d7e3c556a2bcdaca8ff4990.png)

Tabla 6- Results of F1 in the experiments

On the other hand, RMSE (Root Mean Squared Error) is a metric commonly used to assess the accuracy of a model in predicting the electricity consumption of individual appliances or the aggregate energy consumption of a household. For this purpose, RMSE measures the root mean square difference between predicted and actual energy consumption values, and is expressed in the same units as the energy consumption data. A high RMSE value in NILMTK indicates that the model does not accurately predict the energy consumption of the appliances or the household, which means that the predicted values are far away from the actual values.

![Gráfico, Gráfico de barras Descripción generada automáticamente](media/a525306d9ba678c40261f40d2690bff8.png)

Tabla 7- Results of RMSE in the experiments

In general, a low RMSE value indicates that the model predicts the power consumption better, while a high RMSE value indicates that the model has room for improvement, which can be seen in the graph below.


As an example of the fluctuations that can occur due to noise, misalignment of the current sensor, or current sensor failure, we reproduce here the power measurements of 25/01/2023 of the hoover where 7056 measurements with clearly erroneous negative power (marked in yellow) were observed.

![Gráfico Descripción generada automáticamente](media/a580ca601dc1b012a274e27953f9dca2.png)

Figure 10-Evolution of power for vacuum cleaner

This same result, thanks to the graphical tools provided by NILMTK, can be seen differentiated for voltage, power (active or reactive) and current:

![Gráfico Descripción generada automáticamente](media/9dd979458ced15e0f5778ee5a09d0d33.png)

Figure 11-Power, current and voltage for vac

If we finally compare the main metrics with the successive improvements made in successive experiments, such as sample time extension, electrical noise reduction, sensor switching and improved alignment of active phases in split-core sensors, we see significant signs of improvement.

![Gráfico, Gráfico de líneas Descripción generada automáticamente](media/93e8658d5396e28e3b58b6c5bdb385d5.png)

Figure 12- Improvement of metrics in the different experiments

As the number of applicatives in the training is increased using the combinatorial algorithm there are no problems even with very small sampling times, but with FHMM there can be significant difficulties in running the algorithm with the new dense data set, which requires either reducing the training time or reducing the sampling time (or even reducing the number of applicatives), as illustrated in the table below.

![](media/6302c2338200f183cc06be7770ecd77a.emf)

Table 8- Training possibilities with FHMM

Table 8 shows that in order to run FHMM with large numbers of high-frequency samples, it is necessary to either reduce the training period or increase the sampling to produce a satisfactory model.

**4. RESULTS**

NILMTK has the calculation of evaluation metrics through the use of the MeterGroup for the validation of the results by means of the validation set. It is necessary to run different metrics such as FEAC, F1, EAE, MNEAP and RMSE on the models obtained, which for the best model (CO with 1sec sampling) gives us an output similar to Table 9.

![Tabla Descripción generada automáticamente](media/2163dc9c3c3c2e4bc4f7349a0a5fa103.png)

Table 9-Metrics of CO for 10 apliances

That is, we obtain a score for F1-Score of Mean of 45.41, an excellent value for EAE (mean 0), a very low perfect value for MNEAP (mean 1.9809) and an acceptable value for RMSE (mean 372.8472).


**Summary of results without harmonics**

In general, the elimination of harmonics for the same training period, same algorithm (CO) and a sampling period of 1sec, slightly worsens almost all metrics for almost all applications. In the following table we can see the results:


![Tabla Descripción generada automáticamente](media/0508e1d7129202cdea3b8468637f0dd4.png)

Table 13- Summary metrics without harmonics

That is, we obtain a score for F1-Score of Mean 47.29 (somewhat better than incorporating harmonics), a much worse value for EAE (mean 0.472), a low value for MNEAP (mean 6.0115) but worse than including harmonics (mean 1.9809) and a somewhat worse value for RMSE (mean 382.2369 vs. 372.8472).
**Resumen de resultados sin armónicos impares**

In general, the incorporation of only the even harmonics disregarding the odd harmonics for the same training period, same algorithm (CO) and a sampling period of 1sec, does NOT improve the metrics for almost all the applications. In the following table we can see the results:

![Interfaz de usuario gráfica, Aplicación Descripción generada automáticamente](media/f2965dc14f31dfad59bb853854b8504b.png)

Table 11- Metrics for even harmonics only

That is, we obtain a score for F1-Score of Mean 44.01 (worse than with or without harmonics), an excellent value for EAE (mean 0), a very low value for MNEAP (mean 2.2877) and an acceptable value for RMSE (mean 401.3312) but considerably worse than the value obtained with harmonics of 372.8472.

**Comparison with other DS**

For the IAWE DS, the results show that the most efficient algorithm for this DS is the combinatorial (CO) algorithm using the Mean method and **10-minute period, as opposed to only the 1 second needed with the OZM**data.

![Tabla Descripción generada automáticamente](media/8ebd0ec88d9f344dac9a694ecfd873f8.png)

Table 14- Results of IAWE

On the other hand, the results obtained for the DS of DEPS show a better performance for the CO algorithm, Mean method, but **at a sampling time of half an hour, compared to only 1 second for the OZM**data..

![Tabla Descripción generada automáticamente](media/8a92a55f7a375c6f472b1acb1a9a86a7.png)

Figure 15-Results of DEPS

Likewise, if we compare GT and Pred for the DS of DEPS, the divergences are very important, ranging between 1.4%, 4.6% and 4.9% compared to 0 and 1.6% for DSUAL.

![Gráfico, Gráfico circular Descripción generada automáticamente](media/2b62711a9cf58ecc96ca66410a1d8862.png)

Figure 16- Comparison GT with Pred for DEPS

**V. CONCLUSIONS**

In this work in the field of NILMTK, in addition to incorporating both the metrics and the tools available in the toolkit, a new 13-digit timestamp format has been incorporated as well as new converters and converters for the measurements obtained from OZM, thus eliminating the entry barrier for any researcher who has one or more OZMs and wishes to access NILM. Precisely by comparing the SD obtained (with or without transients), it has been shown that the use of harmonics can improve the result of the metrics, depending, of course, on the application to be considered.


On the other hand, if we compare the results of the metrics obtained on DSUALM or DSUALMT, compared to the DS of IAWE or DEPS, the results are much worse in the latter, especially in terms of the sampling period required. The minimum error in the disaggregation in DSUALM also stands out, as well as the best values obtained for the MNEAP and RMSE metrics.

**Publications**

There is an article by me about the NILM using single-phase OZM hardware instead of OZM v2:

-   C. Rodriguez-Navarro, A. Alcayde, V. Isanbaev, L. Castro-Santos, A. Filgueira-Vizoso, and F. G. Montoya, “DSUALMH- A new high-resolution dataset for NILM,” *Renewable Energy and Power Quality Journal*, vol. 21, no. 1, pp. 238–243, Jul. 2023, doi: 10.24084/repqj21.286.

Also, in order to make all this work replicable, a new open multi-counter called OMPM has been developed and is published in the scientific journal "Inventions".

-   C. Rodríguez-Navarro, F. Portillo, F. Martínez, F. Manzano-Agugliaro, and A. Alcayde, “Development and Application of an Open Power Meter Suitable for NILM,” *Inventions*, vol. 9, no. 1, p. 2, Dec. 2023, doi: 10.3390/inventions9010002.
