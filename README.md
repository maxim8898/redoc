# Forecasting API Product

# Markdown test (left column)

This part is displayed in the middle column with a youtube video
<a href="https://www.youtube.com/watch?v=OcTE_U35m_8" target="_blank"><img src="https://img.youtube.com/vi/OcTE_U35m_8/0.jpg" 
alt="IMAGE ALT TEXT HERE" width="240" height="180" border="10" /></a>


<PullRight>
This part will appear in the right pane with formatted code
```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```
</PullRight>



---

# Overview
## API overview and usage
This document describes the communication with the `forecasting` API. It makes future prediction based on past and present trends. It uses supervised machine learning techniques to learn the relationship between variables (input) at hand and the variable (the target) we want to forecast. 

For instance, this could be the forecast of a production line or the prediction of a daily building energy consumption. This API should be used when the target variable is supposed to have an underlying pattern and when historical measurements are available.

The API includes 4 services:

- **createModel:** Creates a forecasting model based on historical data
- **applyModel:** Applies a created model to a test data set
- **updateModel**: Updates an existing model that was build with the *createModel* service
- **getModelInformation**: Returns information of an existing forecasting model

This document provides a general tutorial for users who want to consume the forecasting API. For more detailed information about parameters, inputs, and outputs of the web services the user should refer to the API Reference provided as Swagger.

## Benefits and features
Use Cases: Provide comprehensive use cases that your audience can identify with. Complement them with sample code that uses your SDKs. It's much easier to understand how your API works by reading a story than by going through the full reference.

##	Customers (optional)
##	Pricing (optional)

---

# Developer Guide

## How it works
![Analytics forecasting API - How it works](https://github.com/sieiieowoer/jens/blob/master/forecasting.png?raw=true)

## 1. createModel Webservice

------

The `createModel` web service creates a forecasting model. The created forecasting model uses information available from driver measurements and process parameters to model the relationship between the drivers and the target we wish to forecast. 

The principle of this service is the following:

●	Historical training data depicting how a system works is available.

●	A target variable, i.e., data of special importance for which we want to build a forecasting model, is known.

●	Some pattern in the data is suspected, i.e. some relationship between some of the available data and the target can be observed.
●	This pattern can be used to create a forecasting model to generate predictions.



### URL:

The **end point Full URL**  is the following :

`<end_point_full_exchange_url> = <end_point_base_url>/3c1bc709b6474b9fbb2f3dc653c4302a`

### createModel - Input

As input, the `createModel` web service takes a single data-frame in JSON format. It must include a `datetime` column with the time-stamp of observations. Note that the time-stamps can be given in any standard  ISO 8601 formats, but any output will convert the given format into the following: *2018-03-22T10:39:19Z*. The `target` column includes the (numeric) observations of the variable for which a prediction model is created. Moreover, the data-frame has to include 15 additional columns `att01,att02,..,att15` with numeric observations of the driver variables (the final drivers used to build the model can be defined by the user through one of the parameters). Missing observations should be included as `NaN` values. The API enforces a strict schema, therefore all variables should be present with their name as shown in the Request-Response example and no modifications in the column names can be done. If you have less than 15 drivers, then you should include the additional drivers, fill them with `NaN` or random values and make sure to remove them from the `normalDriverName` parameter. As such, they will not be used by the API. 

An **example input data-frame** can be seen below:

| datetime                     | target | att01 | att02 | att03 | ...  | att15 |
| ---------------------------- | ------ | ----- | ----- | ----- | :--: | ----- |
| 2015-10-08T08:00:00.000+0200 | 47.0   | 50.40 | 25.93 | 25.93 | ...  | 50.40 |
| 2015-10-08T08:30:00.000+0200 | 46.0   | 40.96 | NaN   | 12.10 | ...  | 40.96 |
| 2015-10-08T09:00:00.000+0200 | 45.0   | 43.55 | 25.70 | 25.70 | ...  | 43.55 |
| 2015-10-08T09:30:00.000+0200 | 45.0   | 50.40 | NaN   | 11.10 | ...  | 50.40 |
| 2015-10-08T10:00:00.000+0200 | 45.0   | 53.29 | 25.61 | 25.61 | ...  | 53.29 |
| 2015-10-08T10:30:00.000+0200 | 46.0   | 67.23 | NaN   | 67.23 | ...  | 67.23 |
| 2015-10-08T11:00:00.000+0200 | 47.0   | 84.63 | 25.61 | 25.61 | ...  | 84.63 |
| 2015-10-08T11:30:00.000+0200 | 48.0   | 92.16 | NaN   | 15.2  | ...  | 92.16 |
| 2015-10-08T12:00:00.000+0200 | 48.0   | 67.23 | 25.80 | 25.80 | ...  | 67.23 |

### createModel - Parameters

The input is processed according to a set of user-defined parameters that describe the functioning of the service. The following parameters can be defined by the user:

#### Basic Parameters

- datetimeName (**str**)

  Mandatory name of the time variable in the training data-frame.

- targetName (**str**) 

  Mandatory name of the target variable to forecast.

- predictionTimeHorizonMs (**int**)

  Mandatory time delta in milliseconds between the prediction (t) and the prediction timestamp (t+delta). Must be a multiple of the sampling period.

- onIndicatorName (**str**, default = NaN)

  Name of the variable indicating if the process is working or not (set to 1 if the process is ON, or 0 if process if OFF). If an observation is defined OFF, it will not be considered in the learning process. Note that columns defined as process indicators cannot be used as drivers to build the forecasting model. 

#### Normal and Forecast Drivers

- normalDriverNames (**str**, default = NaN)

  List of driver names that are used to build the forecasting model (separated by a comma).

- normalDriversPastWindowLengthMs (**int**, default = 0)

  Sliding window (in milliseconds) to use for the normal drivers.

- normalDriversPastSamplingPeriodMs (**int**, default = 0) 

  Size of the sampling period in milliseconds to use inside the sliding window for normal drivers. This allows users to have large sliding windows (which capture higher system dynamics)m while using only a few points inside it (which leads to faster and more robust models).

- usePastTargetAsNormalDriver (**int**, default = False)

  If True the past values of the target until time t will be used as a normal driver to predict the future value. If False, they will not be used.

- forecastDriverNames (**str**, default = NaN)

  Comma separated list of column names that include the forecast drivers for the model. Values of these drivers are assumed to be known in advance (e.g. weather forecast).

- forecastDriversPastWindowLengthMs (**int**, default = 0)

  Size of sliding windows used for forecast drivers in milliseconds. If no value is provided, the same value as for the normal drivers is used.

- forecastDriversPastSamplingPeriodMs (**int**, default = 0) 

  Size of the sampling period in milliseconds to use inside the sliding window for forecastdrivers. This allows users to have large sliding windows (which capture higher system dynamics), while using only a few points inside it (which leads to faster and more robust models).

#### Time Related Drivers

- localTimeZone (**str**, default = Europe/Paris) 

  Time zone (in Olson format) that should be applied  to determine any time-related information. Can be given as string or taken from the python *pytz* package.

- usePublicHolidaysAsDriver (**boolean**, default = False) 

  Adds a new variable `TimeDescriptors_publicHoliday` to the dataset. The values equal 1 during the public holidays, or 0 otherwise. The national holidays of different countries and regions are defined by the `publicHolidayLocale` parameter.

- publicHolidayLocale (**str**, default = fra) 

  Defines the country and region from which the public holidays are considered in the model creation. Valid choices are country codes and regions that can be queried from this web page: http://holidays.kayaposoft.com/ .

- useWeekendsAsDriver (**boolean**, default = False) 

  Adds a new variable`TimeDescriptors_weekend` to the dataset. The values are equal to 1 during weekends, or 0 otherwise. Weekend days are defined by the `weekendDays` parameter. 

- weekendDays (**String**, default = Saturday, Sunday) 

  List of weekend days (separated by a comma).

- useTimeCyclesAsDrivers (**boolean**, default = False) 

  Add variables to take into account daily, weekly, monthly, and yearly cycles.

#### Advanced parameters

- throwErrorOnMissingValues (**boolean**, default = False)

  Boolean flag indicating if forecasting provider should throw error if training data includes missing data. By default this variable is set to true so that users become aware of the quality of their dataset.

- modelType (**enum : [linear_regression, random_forest]**, default = random_forest)

  Defines the type of model use. *linear_regression* will perform a linear regression while *random_forest* will perform a random forest regression.

- testDataRatio (**float**, default = 0.2)

  Data-ratio of the training set used for model validation. Once the performance is evaluated, the temporary model is thrown away and ALL data is used to create the model returned in the output.

- testDataSelectionStrategy (**enum : [last, random]**, default = last)

  Data to be used for model validation. Uses either the last `testDataRatio`% of the data, or chooses the data at random.

- speedStrategy (**enum [fast, medium, robust]**, default = robust)

  Parameter to increase the processing speed by reducing the number of learning observations. `fast` equals 1000 observations (i.e., sliding windows), `medium` refers to 10.000 observations (i.e., sliding windows), and a the complete training set is used to create the model if the parameters is set to `robust` .

- optimizeModelParameters (**boolean**, default = False)

  This parameter is only configured for *random forest* models. If this boolean flag is set to *true*, the hyper-parameters will be optimized before the final model is created.

### createModel - Output

The `createModel` service output consists of the following outputs in JSON format:

- **modelID** represents the identification of the created model, which is securely stored on the Schneider Electric platform. This ID can be used to retrieve the model created in the `createModel` service.

- **diagInfo** is a data-frame summary including relevant information of the created model. 

  An example table of the provided output can be seen below:

| phase_id                     | property                       | value                      |
| ---------------------------- | ------------------------------ | -------------------------- |
| data_validation              | start_time                     | 2018-05-02 09:56:05.936818 |
| data_validation              | nb_observations                | 31560                      |
| data_validation              | nb_columns                     | 10                         |
| data_validation              | nb_cells                       | 315600                     |
| data_validation              | nb_missing_values              | 31340                      |
| data_validation              | columns_with_missing_values    |                            |
| data_validation              | was_datetime_sorted            | True                       |
| data_validation              | sampling_period_ms             |                            |
| data_validation              | end_time                       | 2018-05-02 09:56:06.006818 |
| data_validation              | elapsed_seconds                | 0.07                       |
| preprocessing                | start_time                     | 2018-05-02 09:56:06.006818 |
| preprocessing                | non_selected_columns           | flag                       |
| preprocessing                | remaining_renamed_columns      |                            |
| preprocessing                | nb_rows_in_off_state_removed   | 15777                      |
| preprocessing                | end_time                       | 2018-05-02 09:56:06.086818 |
| preprocessing                | elapsed_seconds                | 0.08                       |
| preprocessing_model_specific | start_time                     | 2018-05-02 09:56:06.086818 |
| preprocessing_model_specific | nb_removed_constant_attributes | 0                          |
| preprocessing_model_specific | removed_constant_attributes    |                            |
| preprocessing_model_specific | nb_observations                | 261                        |
| preprocessing_model_specific | nb_input_variables_for_model   | 6                          |
| preprocessing_model_specific | rows_columns_ratio             | 43.5                       |
| preprocessing_model_specific | has_sufficient_nb_observations | True                       |
| preprocessing_model_specific | end_time                       | 2018-05-02 09:56:06.106818 |
| preprocessing_model_specific | elapsed_seconds                | 0.02                       |
| training                     | start_time                     | 2018-05-02 09:56:06.106818 |
| training                     | end_time                       | 2018-05-02 09:56:06.136818 |
| training                     | elapsed_seconds                | 0.03                       |
| performance_evaluation       | start_time                     | 2018-05-02 09:56:06.136818 |
| performance_evaluation       | temporary_train_set_size       | 209                        |
| performance_evaluation       | temporary_test_set_size        | 52                         |
| performance_evaluation       | split_method                   | split-last                 |
| performance_evaluation       | end_time                       | 2018-05-02 09:56:13.184830 |
| performance_evaluation       | elapsed_seconds                | 7.048012                   |



- **modelPerformance** includes different performance metrics of the test and training sets, such as the mean absolute error, the normalized absolute error, etc.

  An example table of the provided output can be seen below:

  | indicatorName                      | dataset  | value               |
  | ---------------------------------- | -------- | ------------------- |
  | mean_absolute_error                | training | 0.1874              |
  | root_mean_squared_error            | training | 0.2319917           |
  | normalized_mean_absolute_error     | training | 0.0253755           |
  | relative_error                     | training | 0.0254059916        |
  | relative_error_lenient             | training | 0.02452284036       |
  | relative_error_strict              | training | 0.0258087           |
  | normalized_root_mean_squared_error | training | 0.03204602498456187 |
  | relative_absolute_error            | training | 0.737781268         |
  | root_relative_squared_error        | training | 0.7373640686543463  |
  | r_squared                          | training | 0.4560837           |
  | mean_absolute_error                | test     | 0.161033            |
  | root_mean_squared_error            | test     | 0.20663             |
  | normalized_mean_absolute_error     | test     | 0.0225436734        |
  | relative_error                     | test     | 0.02277708872       |
  | relative_error_lenient             | test     | 0.022040115         |
  | relative_error_strict              | test     | 0.02284             |
  | normalized_root_mean_squared_error | test     | 0.02899             |
  | relative_absolute_error            | test     | 0.82132271          |
  | root_relative_squared_error        | test     | 0.85856834          |
  | r_squared                          | test     | 0.262862            |

  

- **driversUsed** and **driversUsageDetails** summarize information on the driver variables and their usage to create the model.

  Example tables of the provided outputs can be seen below:

| driver_name | driver_ type | max_ value | mean_  value | min_ value | reason | std_value | used_in_model |
| ----------- | ------------ | ---------- | ------------ | ---------- | ------ | --------- | ------------- |
| att01       | normal       | 47.55      | 46.46        | 43.20      |        | 1.58      | TRUE          |
| att02       | normal       | 15.33      | 14.79        | 13.33      |        | 0.35      | TRUE          |
| att03       | normal       | 53.85      | 50.43        | 47.20      |        | 1.33      | TRUE          |
| att04       | normal       | 42.50      | 39.18        | 35.70      |        | 1.38      | TRUE          |
| att05       | normal       | 32.40      | 30.85        | 29.20      |        | 0.65      | TRUE          |
| att06       | normal       | 82.70      | 58.35        | 49.30      |        | 3.30      | TRUE          |

| driver_name  | time_shift_used_for_prediction | driver_importance | driver_type |
| ------------ | ------------------------------ | ----------------- | ----------- |
| DRIVER_att01 | 0                              | 1.0               | normal      |
| DRIVER_att02 | 0                              | 1.0               | normal      |
| DRIVER_att03 | 0                              | 1.0               | normal      |
| DRIVER_att04 | 0                              | 1.0               | normal      |
| DRIVER_att05 | 0                              | 1.0               | normal      |
| DRIVER_att06 | 0                              | 1.0               | normal      |



- **predictionConfidenceIntervals** summarize the differences of real and predicted values under the consideration of different significance levels.

  An example table of the provided output can be seen below:

| scale    | tails    | dataset  | nb_folds | nb_bootstrap | significance_level | value | std   |
| -------- | -------- | -------- | -------- | ------------ | ------------------ | ----- | ----- |
| Relative | Positive | training | 1        | 100          | 0.05               | 0.072 | 0.006 |
| Relative | Negative | training | 1        | 100          | 0.05               | 0.051 | 0.005 |
| Relative | Both     | training | 1        | 100          | 0.05               | 0.065 | 0.005 |
| Absolute | Positive | training | 1        | 100          | 0.05               | 0.482 | 0.042 |
| Absolute | Negative | training | 1        | 100          | 0.05               | 0.399 | 0.036 |
| Absolute | Both     | training | 1        | 100          | 0.05               | 0.455 | 0.026 |
| Relative | Positive | training | 1        | 100          | 0.1                | 0.062 | 0.006 |
| Relative | Negative | training | 1        | 100          | 0.1                | 0.044 | 0.003 |
| Relative | Both     | training | 1        | 100          | 0.1                | 0.051 | 0.004 |
| Absolute | Positive | training | 1        | 100          | 0.1                | 0.427 | 0.044 |
| Absolute | Negative | training | 1        | 100          | 0.1                | 0.335 | 0.027 |
| Absolute | Both     | training | 1        | 100          | 0.1                | 0.373 | 0.034 |
| Relative | Positive | test     | 1        | 100          | 0.05               | 0.056 | 0.017 |
| Relative | Negative | test     | 1        | 100          | 0.05               | 0.032 | 0.008 |
| Relative | Both     | test     | 1        | 100          | 0.05               | 0.052 | 0.011 |
| Absolute | Positive | test     | 1        | 100          | 0.05               | 0.364 | 0.079 |
| Absolute | Negative | test     | 1        | 100          | 0.05               | 0.254 | 0.058 |
| Absolute | Both     | test     | 1        | 100          | 0.05               | 0.348 | 0.055 |
| Relative | Positive | test     | 1        | 100          | 0.1                | 0.046 | 0.009 |
| Relative | Negative | test     | 1        | 100          | 0.1                | 0.031 | 0.008 |
| Relative | Both     | test     | 1        | 100          | 0.1                | 0.043 | 0.005 |
| Absolute | Positive | test     | 1        | 100          | 0.1                | 0.315 | 0.054 |
| Absolute | Negative | test     | 1        | 100          | 0.1                | 0.222 | 0.059 |
| Absolute | Both     | test     | 1        | 100          | 0.1                | 0.306 | 0.056 |



- **modelErrorStatistics** define the distribution of the prediction errors. 

  An example table of the provided output can be seen below:

| variable | queue | dataset  | nb_folds | nb_boostrap | value   |
| -------- | ----- | -------- | -------- | ----------- | ------- |
| IQR      | No    | training | 1        | 100         | 0.2933  |
| IQR      | Yes   | training | 1        | 100         | 0.3220  |
| mean     | No    | training | 1        | 100         | -0.0013 |
| mean     | Yes   | training | 1        | 100         | 0.0010  |
| stdev    | No    | training | 1        | 100         | 0.2009  |
| stdev    | Yes   | training | 1        | 100         | 0.2316  |
| IQR      | No    | test     | 1        | 100         | 0.1917  |
| IQR      | Yes   | test     | 1        | 100         | 0.2007  |
| mean     | No    | test     | 1        | 100         | 0.1073  |
| mean     | Yes   | test     | 1        | 100         | 0.1098  |
| stdev    | No    | test     | 1        | 100         | 0.1320  |
| stdev    | Yes   | test     | 1        | 100         | 0.1750  |

- **modelInformation** summarizes relevant model properties.

  An example table of the provided output can be seen below:

| property                       | value                      |
| ------------------------------ | -------------------------- |
| creation_date                  | 2018-05-02 09:56:06.106818 |
| has_sufficient_nb_observations | True                       |
| initial_model_creation_date    | 2018-05-02 09:56:06.106818 |
| max_prediction                 | 0                          |
| min_prediction                 | 0                          |
| nb_drivers_used_in_model       | 0                          |
| nb_observations_testing        | 0                          |
| nb_observations_total          | 0                          |
| nb_observations_training       | 0                          |
| ts_uniform_sampling_period_ms  | 0                          |



## 2. applyModel Webservice

------

The `applyModel` service can be used to apply an existing forecasting model to new data. A request-response example to consume the `applyModel` web service (in Swagger and non-Swagger format) can be seen below. 

- URL:
  The **end point Full URL**  is the following :

  `<end_point_full_exchange_url> = <end_point_base_url>/b80d3357a77042c38ba53a2e57b116a0 `

  ### applyModel - Input

  The `apply` service needs two different inputs:

  - A test data set with the same variables and format as the training set described in the `createModel` service
  - The dataset should include a target column (even if you do not have the corresponding value). **Attention**: If you don't have the value of the target, you need to fill the column with a NaN (or any random number). 
  - The ID of the model created in the `createModel` service

  #### applyModel - Parameters

  The input is processed according to the following user-defined parameter:

  - joinPredictionToData (**boolean**, *default = False*)

    Defines whether or not the input data is included in the prediction output table.

  ### applyModel - Output

  The `applyModel` service output consists of a data-frame with a single data-frame as output. This table (**appliedModelOutput**) includes:

  - the actual target real - this can be used to compare the predictions and real value.
  - the predicted value
  - information whether or not the input is valid (input is considered as not valid if it deviates over 5% of the extreme points of the training data) or if it has a missing value (NaN)
  - 2- and 3-sigma confidence intervals.

  | datetime                  | TARGET_target_t+1 | nanInput | invalid Input | PREDICTED target_t+1 | invalid Prediction | min Confident 2Sigma | max Confident 2Sigma |
  | ------------------------- | ----------------- | -------- | ------------- | -------------------- | ------------------ | -------------------- | -------------------- |
  | 2015-10-08 06:00:00+00:00 | 46.5              | False    | False         | 47.0                 | FALSE              | 46.986               | 47.14                |
  | 2015-10-08 06:30:00+00:00 |                   |          |               |                      |                    |                      |                      |
  | 2015-10-08 07:00:00+00:00 | 44.7              | False    | False         | 45.0                 | FALSE              | 44.986               | 45.14                |
  | 2015-10-08 07:30:00+00:00 |                   |          |               |                      |                    |                      |                      |
  | 2015-10-08 08:00:00+00:00 | 46.8              | False    | False         | 45.0                 | FALSE              | 44.986               | 45.14                |
  | 2015-10-08 08:30:00+00:00 |                   |          |               |                      |                    |                      |                      |
  | 2015-10-08 09:00:00+00:00 | 47.1              | False    | False         | 47.0                 | FALSE              | 46.986               | 47.14                |
  | 2015-10-08 09:30:00+00:00 |                   |          |               |                      |                    |                      |                      |

  

## 3. updateModel Webservice

------

- This service updates an existing model created with the `createModel` web service. The input should include the same variables as the ones used to create the already available prediction model. Outputs of this service include the updated model ID (whereas the updated model is created with past and updated data), and the same output information as provided in the `createModel` service. 

- URL:
  The **end point Full URL**  is the following :

  `<end_point_full_exchange_url> = <end_point_base_url>/16c5c61045f547cbb6f3e33d86504b32 `

  ### updateModel - Input

  As input, this service needs the model ID (i.e., the secure location in the Schneider Electric platform) of the existing forecasting model for which information is supposed to be retrieved in form of a data-table in JSON format. 

  ### updateModel - Output

  The output of this service is the same as the one described for the `createModel` service. Note that a new **modelID** will be returned to represent the identification of the updated model, which is securely stored on the Schneider Electric platform. 

  

## 4. getModelInformation Webservice

------

- This service returns model information on an existing prediction model that is securely stored on the Schneider Electric platform.

- URL:
  The **end point Full URL**  is the following :

  `<end_point_full_exchange_url> = <end_point_base_url>/857cabb1b2c44579a04be29ff232a62 `


### getModelInformation- Input

As input, this service needs the model ID (i.e., the secure location in the Schneider Electric platform) of the existing forecasting model for which information is supposed to be retrieved in form of a data-table in JSON format. 

### getModelInformation - Output

The output of this service is the same as the one described for the `createModel` service. Note that no **modelID** information will be returned.

## Payload size limits
How large files do we accept

## End Point base URL's of the component:

The **end point base URL**  is the following :

`<end_point_base_url> = https://api.exchange.se.com/analytics-forecasting-api-prod` 
`<end_point_base_url> = https://api-sandbox.exchange.se.com/analytics-forecasting-api-prod` 


## Micro application
https://try-analytics-se.azurewebsites.net/forecasting

## Getting Started / Tutorial
The Getting Started guide provides a detailed account of how to quickly start working with your API. The emphasis in the guide should be on ensuring consumers reach success with your API as quickly as possible, hand holding them throughout this journey. 

### Run in Postman collection
First, you need to download the free [Postman application](https://www.getpostman.com/downloads/) for your operating system. Postman currently supports Mac, Windows, and Linux. Once you have Postman installed, you can import the Forecasting API Postman collection by clicking the orange “Run in Postman” button. At this point, you should see the Forecasting API folder in the left-hand column under the Collections tab.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://www.getpostman.com/run-collection/82e6e5ef-40c3-4784-bf0f-5d0573c3b8c4)

<a href="https://github.com/sieiieowoer/jens/blob/master/forecasting_nominalErrors.postman_collection.json" target="_blank"><img src="https://run.pstmn.io/button.svg" alt="Run forecasting in Postman"></a>



### Step 1: Setup environment
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur

### Step 2: Prepare Data
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur

### Step 3: Create a new forecasting model
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur

### Step 4: Train model 
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur

### Step 5: Apply an existing forecasting model to new data
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur

### Step 6: Get model information on an existing prediction model
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur

## Authentication guide
In order to be authorized to get access to the API please provide in the request header the following parameter:
> Authorization: <Subscription_Key>

For steps on how to generate a <Subscription_Key> please refer to the Features tab.

## Response Codes

We follow the error response format proposed in [RFC 7807](https://tools.ietf.org/html/rfc7807)also known as Problem Details for HTTP APIs.  As with our normal API responses, your client must be prepared to gracefully handle additional members of the response.

### Unauthorized
<RedocResponse pointer={"#/components/responses/Unauthorized"} />

### AccessForbidden
<RedocResponse pointer={"#/components/responses/AccessForbidden"} />


### 400
BadRequest
Your request could not be processed.
This is a generic error.
InvalidAction
The action requested was not valid for this resource.
This error is returned when you try to access an action that doesn't exist. For example, /campaigns/{id}/actions/send.
InvalidResource
The resource submitted could not be validated.
For field-specific details, see field_warnings or field_errors objects. This error means that the object submitted to a POST or PATCH request failed to validate against JSON schema, and could relate to campaign, interest group, merge field, or any other available object.
JSONParseError
We encountered an unspecified JSON parsing error.
This error means that your JSON was formatted incorrectly or was considered invalid or incomplete.
### 401
APIKeyMissing
Your request did not include an API key.
This error suggests that your API key was missing from your request, or that something was formatted or named improperly in your header. To learn more, check out Get Started with the Mailchimp API.
APIKeyInvalid
Your API key may be invalid, or you've attempted to access the wrong data center.
Check that your API key was input correctly, and verify which data center to access. To learn more, check out Get Started with the Mailchimp API.
### 403
Forbidden
You are not permitted to access this resource.
This is a generic error.
UserDisabled
This account has been disabled.
The Mailchimp account is deactivated. Contact our support team if you need more help.
WrongDatacenter
The API key provided is linked to a different data center.
This error suggests that you tried to contact the wrong data center. It's often associated with misconfigured libraries.
### 404
ResourceNotFound
The requested resource could not be found.
This error tells you a specific resource doesn't exist. It's possible that the resource has been moved or deleted, or that there's a typo in your request.
### 405
MethodNotAllowed
The requested method and resource are not compatible. See the Allow header for this resource's available methods.
This error means that the requested resource does not support the HTTP method you used. Find out which methods are allowed for each resource in the API Reference.
### 414
ResourceNestingTooDeep
The sub-resource requested is nested too deeply.
This uncommon error appears if you've tried to generate a URL with too many resources.
### 422
InvalidMethodOverride
You can only use the X-HTTP-Method-Override header with the POST method.
This error lets you know you've tried to override an incompatible method. The Mailchimp API only permits method override with POST.
RequestedFieldsInvalid
The fields requested from this resource are invalid.
This error suggests there is a typo in your field request or some other type of syntax error or problem that invalidates your request.
### 429
TooManyRequests
You have exceeded the limit of 10 simultaneous connections.
When you reach the connection limit, we'll throttle server response. If any of your requests time out after you've reached the limit, those requests could still be considered open and continue to slow your connection. Contact the Mailchimp API support team at apihelp@mailchimp.com if you think this is the case.
### 500
InternalServerError
An unexpected internal error has occurred. Please contact Support for more information.
This error lets you know our servers have experienced a problem. Although this is rare, please contact apihelp@mailchimp.com to let us know that you've encountered this error.
### 503
ComplianceRelated
This method has been disabled.

## Authentication

We love security!
<SecurityDefinitions />

## Micro application
https://try-analytics-se.azurewebsites.net/forecasting
# Support
# Blogs
---
# API Reference
Some description
