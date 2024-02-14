---
title: "Air Quality Monitoring using Electrochemical Low-Cost Sensor Calibrated by Lasso Regression"
date: 2024-02-14T08:30:00+09:00
category: Data-Analysis
tag: Projects
header:
  teaser: /assets/images/low-cost-sensor/teaser.jpg
---

## Analysis Goal

Urban air quality monitoring requires a dense monitoring network to effectively map out the air quality distribution across regions. However, due to the inherent high cost of certified analyzers, the density of monitoring stations has been low. Low-cost sensors have been proposed to be the solution to this challenge as they are cheap to deploy and they can be calibrated by a certified analyzer for a short time, and then operate in the field for a long time.

This dataset contains raw signals from two low-cost sensor nodes and ground truth data from reference stations. The sensor nodes contain NO, O3, NO2, temperature, and relative humidity sensors, as described in the [original paper](https://www.sciencedirect.com/science/article/pii/S2352340922007934#bib0011) publishing the dataset. The reference data contains hourly concentrations of CO, NO, NO2, NOx, O3, and SO2, downloaded from an [online database](https://mediambient.gencat.cat/es/05_ambits_dactuacio/atmosfera/qualitat_de_laire/vols-saber-que-respires/descarrega-de-dades/descarrega-dades-automatiques/). The goal is to develop a calibration/regression model for the low-cost sensors to accurately predict the target gas concentration for efficient air-quality monitoring.

This analysis will showcase one approach for sensor calibration using Lasso regression. This method differs from generic multiple linear regression as it introduces an additional regularization term to control the sparsity of fitting coefficients. By optimizing the regularization strength, we can obtain multiple linear regression models with sparse coefficients, leading to better interpretability. This makes Lasso regression naturally suitable for sensor calibration, where simplicity is desired. The performance of Lasso regression is compared with a series of baseline models from the Python package 'lazypredict' to confirm its good performance.

The calibration results indicate that the calibration for CO and SO2 is understandably poor because the sensor nodes only contain NO, O3, and NO2 sensors. However, with only two weeks of training data, the regression quality for four months of NO, NO2, NOx, and O3 data is very good, with R2 scores around and above 0.85. This validates the idea of using low-cost sensor nodes to increase monitoring network density for improved air quality monitoring performance.

The full analysis with code can be found in this [Kaggle notebook](https://www.kaggle.com/code/chaozhuang/low-cost-sensor-calibration-w-lasso-regression).

<figure style="width: 1000px" class="align-center">
  <img src="/assets/images/low-cost-sensor/setup.jpg" alt="A figure that illustrate the experiment setup with sensors collocated with an authorized reference air quality monitoring station, and the internal circuit of the sensors.">
  <figcaption>The experimental setup.</figcaption>
</figure>

## EDA

Let's take a first look at the data using a correlation matrix and scatter matrix for a sanity check, confirming that no weird correlations arise from errors during the data loading phase.

Several things can be observed from the correlation matrix:

- Same sensors from different nodes show good correlation, confirming the reproducibility among sensors from the same manufacturer.
- The target analyte and its corresponding specialized sensor (O3, NO, etc.) have a strong correlation, confirming the selectivity of these dedicated sensors.
- The NO and NOx levels are highly correlated because NOx contains NO. Therefore, the sensor nodes can also be calibrated to estimate the overall NOx level.

<figure style="width: 1000px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig1.png" alt="Correlation matrix showing sensor reproducibility, selectivity, and cross-sensitivity.">
  <figcaption>A correlation matrix of the combined dataframe.</figcaption>
</figure>

### Scatter Matrix

The scatter matrix below provides a more detailed representation of the correlation coefficients presented in the correlation matrix.

<figure style="width: 1000px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig2.png" alt="Scatter matrix visualizing correlations among sensors and target analytes.">
  <figcaption>A scatter matrix between all features.</figcaption>
</figure>

## Calibration Models

To calibrate the sensors for air quality monitoring, we will use Lasso regression to calculate the coefficients of all sensor features for all target analytes, starting with CO.

In the following, we have defined a helper function to visualize the optimization of the regularization term. When the regularization is weak, Lasso regression is reduced to generic multiple linear regression, where all sensor features are used in the regression. When the regularization strength is high, it promotes sparsity and penalizes the use of unimportant features, significantly reducing the number of features required during calibration. We select the best regularization strength that minimizes the root mean square error (RMSE).

```python
Best Alpha: 0.007663410868007463
Lowest RMSE: 0.06704093498084798
R-square (R2): 0.3081041404664757

Important features in ranked order:
NO-1-mean: 0.053
NO2-2-mean: 0.029
T-2-std: -0.018
T-1-mean: -0.011
T-2-mean: -0.008
RH-2-std: -0.008
```

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig3.png" alt="Optimizing alpha in Lasso regression to minimize RMSE. The graphs shows the RMSE and coefficient strength as a function of Alpha.">
  <figcaption>(Left) Model RMSE of CO calibration as a function of alpha. (Right) Coefficient strengths as a function of alpha.</figcaption>
</figure>

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig4.png" alt="The results of Lasso regression, indicating a decent fit despite of the absence of direct correlation.">
  <figcaption>(Left) True vs. calibrated CO readings. (Right) Model error as a function of time.</figcaption>
</figure>

#### Machine Learning Models

To obtain an overview of the performance of all other available machine learning models, we use a package called 'lazypredict' to generate baseline performance from a library of models. The list of models below shows that the optimized Lasso regression we use has a similar R2 score to the top-performing OrthogonalMatchingPursuitCV. Notice also that simple linear regression gives a negative R2 score, which means its performance is worse than using the average of all signals for calibration. This further validates the effectiveness of our calibration model.

```python
Adjusted R-Squared  R-Squared  RMSE  Time Taken
Model                                                                         
OrthogonalMatchingPursuitCV                  0.30       0.31  0.07        0.08
ExtraTreesRegressor                          0.29       0.29  0.07        0.27
LarsCV                                       0.28       0.29  0.07        0.14
LassoLarsCV                                  0.28       0.29  0.07        0.15
GammaRegressor                               0.28       0.29  0.07        0.04
LassoCV                                      0.28       0.29  0.07        0.22
ElasticNetCV                                 0.28       0.29  0.07        0.22
PoissonRegressor                             0.26       0.26  0.07        0.07
...
```

### NO

We have a NO sensor in the low-cost sensor node, so we expect much better calibration performance for this target. Using the same Lasso regressor, we obtain an R2 score of 0.89 with 14 non-zero coefficients out of 18 candidates. The most important feature is, unsurprisingly, the mean NO sensor signals. The optimized Lasso also outperforms the best model provided by lazypredict.

```python
Best Alpha: 0.10353218432956626
Lowest RMSE: 3.3233438447063244
R-square (R2): 0.8901232879755219

Important features in ranked order:
NO-1-mean: 17.050
T-1-mean: -2.978
NO2-2-mean: 2.865
NO-1-std: 2.623
NO2-2-std: -2.239
T-2-std: -1.330
O3-1-mean: 1.252
O3-2-std: 1.191
NO2-1-std: 0.838
RH-2-mean: 0.793
RH-1-std: -0.470
RH-1-mean: 0.360
T-1-std: 0.319
T-2-mean: -0.197
```

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig5.png" alt="Optimizing alpha in Lasso regression to minimize RMSE. The graphs shows the RMSE and coefficient strength as a function of Alpha.">
  <figcaption>(Left) Model RMSE of NO calibration as a function of alpha. (Right) Coefficient strengths as a function of alpha.</figcaption>
</figure>

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig6.png" alt="The results of Lasso regression, indicating a decent fit despite of the absence of direct correlation.">
  <figcaption>(Left) True vs. calibrated NO readings. (Right) Model error as a function of time.</figcaption>
</figure>

#### Machine Learning Models

```python
Adjusted R-Squared  R-Squared  RMSE  Time Taken
Model                                                                         
ExtraTreesRegressor                          0.88       0.88  3.42        0.31
ElasticNetCV                                 0.88       0.88  3.52        0.18
LinearSVR                                    0.87       0.87  3.64        0.06
SGDRegressor                                 0.86       0.86  3.78        0.06
HuberRegressor                               0.86       0.86  3.80        0.07
LassoCV                                      0.85       0.85  3.82        0.20
LassoLarsIC                                  0.85       0.85  3.90        0.07
LarsCV                                       0.85       0.85  3.93        0.13
RidgeCV                                      0.84       0.85  3.94        0.05
Ridge                                        0.84       0.85  3.94        0.04
```

### NO2

Running the calibration model on NO2 concentration gives an R2 score of 0.86 with 7 features, and the most important feature is NO2 sensor signals. Results from lazypredict indicate that further improvement can be obtained using OrthogonalMatchingPursuitCV.

```python
Best Alpha: 1.0473708979594507
Lowest RMSE: 5.541482212190994
R-square (R2): 0.8569124344981707

Important features in ranked order:
NO2-1-mean: 14.588
O3-1-mean: -2.432
NO-1-mean: 1.037
NO-1-std: 0.720
T-2-std: 0.679
NO2-2-mean: 0.064
O3-2-mean: -0.056
```

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig7.png" alt="Optimizing alpha in Lasso regression to minimize RMSE. The graphs shows the RMSE and coefficient strength as a function of Alpha.">
  <figcaption>(Left) Model RMSE of NO2 calibration as a function of alpha. (Right) Coefficient strengths as a function of alpha.</figcaption>
</figure>

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig8.png" alt="The results of Lasso regression, indicating a decent fit despite of the absence of direct correlation.">
  <figcaption>(Left) True vs. calibrated NO2 readings. (Right) Model error as a function of time.</figcaption>
</figure>

#### Machine Learning Models

```python
Adjusted R-Squared  R-Squared   RMSE   Time Taken
Model                                                                 
OrthogonalMatchingPursuitCV                  0.87       0.87   5.34   0.08
LinearSVR                                    0.86       0.86   5.46   0.06
RANSACRegressor                              0.86       0.86   5.51   0.19
Lasso                                        0.86       0.86   5.55   0.05
LassoLars                                    0.86       0.86   5.55   0.06
LassoCV                                      0.85       0.85   5.62   0.21
LassoLarsCV                                  0.85       0.85   5.62   0.16
HuberRegressor                               0.85       0.85   5.65   0.07
LassoLarsIC                                  0.85       0.85   5.68   0.08
ExtraTreesRegressor                          0.83       0.84   5.94   0.32
```

### NOx

The calibration for NOx is the best so far with an R2 score of 0.97, with strong contributions from both NO and NO2 sensors. This is understandable because NOx includes both NO and NO2.

```python
Best Alpha: 0.2196385372416547
Lowest RMSE: 4.638807395477192
R-square (R2): 0.970297075476178

Important features in ranked order:
NO-1-mean: 27.923
NO2-2-mean: 19.134
NO-1-std: 4.610
T-1-mean: -3.536
O3-2-std: 2.004
NO2-2-std: -1.461
RH-2-mean: 1.277
RH-1-std: -0.840
O3-2-mean: -0.762
T-2-std: -0.483
RH-1-mean: 0.444
NO2-1-mean: 0.421
T-1-std: 0.212
```

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig9.png" alt="Optimizing alpha in Lasso regression to minimize RMSE. The graphs shows the RMSE and coefficient strength as a function of Alpha.">
  <figcaption>(Left) Model RMSE of NOx calibration as a function of alpha. (Right) Coefficient strengths as a function of alpha.</figcaption>
</figure>

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig10.png" alt="The results of Lasso regression, indicating a decent fit despite of the absence of direct correlation.">
  <figcaption>(Left) True vs. calibrated NOx readings. (Right) Model error as a function of time.</figcaption>
</figure>

#### Machine Learning Models
```python
                               Adjusted R-Squared  R-Squared    RMSE   Time
Model                                                                  
PassiveAggressiveRegressor                   0.97       0.97    4.72   0.05
LinearSVR                                    0.97       0.97    4.73   0.06
HuberRegressor                               0.97       0.97    5.00   0.07
RANSACRegressor                              0.96       0.96    5.07   0.18
OrthogonalMatchingPursuitCV                  0.96       0.96    5.50   0.08
LassoCV                                      0.95       0.95    5.83   0.19
LassoLarsCV                                  0.95       0.95    5.84   0.14
LassoLars                                    0.95       0.95    6.21   0.06
Lasso                                        0.95       0.95    6.21   0.06
ElasticNetCV                                 0.94       0.94    6.53   0.18
LassoLarsIC                                  0.94       0.94    6.58   0.08
```

### O3

The calibration for O3 is also very good (R2 score: 0.94) with the main contributions from O3 sensors, and only five features are used in this calibration. Lazypredict shows that Lasso is the best model for this calibration.

```python
Best Alpha: 0.4937047852839004
Lowest RMSE: 5.585452993378217
R-square (R2): 0.9427919832964997

Important features in ranked order:
O3-2-mean: 9.233
NO2-1-mean: -7.326
O3-1-mean: 6.512
NO-1-mean: 1.504
RH-1-mean: -0.157
```

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig11.png" alt="Optimizing alpha in Lasso regression to minimize RMSE. The graphs shows the RMSE and coefficient strength as a function of Alpha.">
  <figcaption>(Left) Model RMSE of O3 calibration as a function of alpha. (Right) Coefficient strengths as a function of alpha.</figcaption>
</figure>

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig12.png" alt="The results of Lasso regression, indicating a decent fit despite of the absence of direct correlation.">
  <figcaption>(Left) True vs. calibrated O3 readings. (Right) Model error as a function of time.</figcaption>
</figure>

#### Machine Learning Models

```python
                               Adjusted R-Squared  R-Squared  RMSE  Time Taken
Model                                                                         
LassoCV                                      0.94       0.94  5.92        0.21
LassoLarsCV                                  0.93       0.94  5.95        0.15
LassoLars                                    0.93       0.93  5.96        0.05
Lasso                                        0.93       0.93  5.96        0.06
LarsCV                                       0.93       0.93  6.24        0.13
OrthogonalMatchingPursuitCV                  0.93       0.93  6.27        0.08
PassiveAggressiveRegressor                   0.91       0.91  6.97        0.06
LinearSVR                                    0.91       0.91  7.03        0.10
ElasticNetCV                                 0.90       0.90  7.51        0.18
HuberRegressor                               0.89       0.89  7.68        0.07
```

### SO2

Another sanity check, aside from CO, is to confirm that the good fit for NO, NO2, NOx, and O3 is a result of the dedicated sensors in the sensor nodes. Not all analytes can be calibrated. In the calibration for SO2, although the best model has only two features - the mean and standard deviation of the NO sensor - the R2 score is -0.52. This means that using the signal average can yield a better fit than the calibration model. Lazypredict gives the same result, so there is no combination of sensor signals that can be predictive of SO2.

```python
Best Alpha: 0.025826187606826773
Lowest RMSE: 0.6775644967412245
R-square (R2): -0.5156341060941354

Important features in ranked order:
NO-1-mean: 0.048
NO-1-std: 0.010
```

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig13.png" alt="Optimizing alpha in Lasso regression to minimize RMSE. The graphs shows the RMSE and coefficient strength as a function of Alpha.">
  <figcaption>(Left) Model RMSE of SO2 calibration as a function of alpha. (Right) Coefficient strengths as a function of alpha.</figcaption>
</figure>

<figure style="width: 750px" class="align-center">
  <img src="/assets/images/low-cost-sensor/fig14.png" alt="The results of Lasso regression, indicating a decent fit despite of the absence of direct correlation.">
  <figcaption>(Left) True vs. calibrated SO2 readings. (Right) Model error as a function of time.</figcaption>
</figure>

#### Machine Learning Models

```python
                               Adjusted R-Squared  R-Squared  RMSE  Time Taken
Model                                                                         
SVR                                         -0.28      -0.27  0.62        0.08
NuSVR                                       -0.47      -0.46  0.66        0.18
OrthogonalMatchingPursuit                   -0.52      -0.51  0.68        0.04
RandomForestRegressor                       -0.54      -0.53  0.68        0.40
ExtraTreeRegressor                          -0.54      -0.53  0.68        0.05
GradientBoostingRegressor                   -0.54      -0.53  0.68        0.32
XGBRegressor                                -0.54      -0.53  0.68        0.17
ExtraTreesRegressor                         -0.54      -0.53  0.68        0.20
LassoLars                                   -0.55      -0.54  0.68        0.05
Lasso                                       -0.55      -0.54  0.68        0.05
```

## Conclusions

This dataset consists of two low-cost sensor nodes and ground truth air quality data from an authorized reference station. By using the first two weeks of data for training and the following four months of data for validation, we show that Lasso regression can yield very good sensor calibration for several markers of air quality, including NO, NO2, NOx, and O3. The R2 scores for all these analytes are around and above 0.85, indicating good sensor stability over time. We also tested the calibration performance for non-target analytes CO and SO2, but the calibration accuracy is poor because the two sensor nodes do not contain any sensors that are sensitive to these gases.

In conclusion, Lasso regression provides a middle ground between overly simplistic multiple linear regression and uninterpretable machine learning models. Its ability to generate a sparse calibration relationship offers an efficient alternative to conventional calibration methods for low-cost air quality monitoring sensor arrays.