# Forecasting API

<img src=".\content.png" style="zoom:67%;" />

---

# Overview

## API overview and usage

This document explain the usage of the `Forecasting` API.

This API allows to make future prediction based on past and present trends. It uses supervised machine learning techniques to learn the relationship between variables (input) at hand and the variable (the target) we want to forecast. 

For instance, it can be used to prediction the production of a manufacturing line or the prediction of a building energy consumption. This API should be used when the target variable is supposed to have an underlying pattern and when historical measurements are available.

This document provides a general tutorial for users who want to consume the forecasting API.

## Features
The `Forecasting`  API includes 4 features:

Learn a model from past input and target data (createModel).

Apply the model on new data to forecast the target (applyModel).

Update an existing model with actual data (updateModel).

Get information on an existing model (getModelInformation).

## How it works
![](.\Forecasting_Principles.jpg)

## Hands-on application

Take the tour and experiment on your own data prior to any development through our hands-on application !

https://try-analytics-se.azurewebsites.net/forecasting

![](.\MicroApp.png)



# Developer Guide

## createModel endpoint

------

The `createModel` endpoint creates (train) a forecasting model. The created forecasting model uses information available from driver measurements and process parameters to model the relationship between the drivers and the target we wish to forecast. 

The `createModel` endpoint needs several inputs:

●	Historical training data depicting how a system works is available.

●	A target variable, i.e., data of special importance for which we want to build a forecasting model, is known.

●	Some pattern in the data is suspected, i.e. some relationship between some of the available data and the target can be observed.
●	This pattern can be used to create a forecasting model to generate predictions.

Then generated model, learned from the inputs, is then securely stored in Schneider-Electric platform.

The `createModel` endpoint return the ID of this newly created model for future use.

## applyModel endpoint

------

The `applyModel` endpoint can be used to apply (score) an existing forecasting model to new data. 

The `applyModel` endpoint needs two different inputs:

- A test data set with the same variables and format as the training set described in the `createModel` service
- The ID of the model created in the `createModel` service

The `applyModel` endpoint then generates predicted values based on the previously learned model and given the test data set.

## updateModel endpoint

------

This service updates an existing model created with the `createModel` web service.

The input should include the same variables as the ones used to create the already available prediction model.

Outputs of this service include the updated model ID (whereas the updated model is created with past and updated data), and the same output information as provided in the `createModel` service. 

## getModelInformation endpoint

------

This service returns information on an existing prediction model that is securely stored on the Schneider-Electric platform.



## Payload size limits
How large files do we accept



## Getting Started 
### Run in Postman collection
First, you need to download the free [Postman application](https://www.google.com) for your operating system. Postman currently supports Mac, Windows, and Linux. Once you have Postman installed, you can import the Forecasting API Postman collection by clicking the orange “Run in Postman” button. At this point, you should see the Forecasting API folder in the left-hand column under the Collections tab.

<a href="https://github.com/sieiieowoer/jens/blob/master/forecasting_nominalErrors.postman_collection.json" target="_blank"><img src="https://run.pstmn.io/button.svg" alt="Run forecasting in Postman"></a>


### Step 1: Setup environment
### Step 2: Prepare Data
### Step 3: Create a new forecasting model
### Step 4: Train model 
### Step 5: Apply an existing forecasting model to new data
### Step 6: Get model information on an existing prediction model

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
When you reach the connection limit, we'll throttle server response. If any of your requests time out after you've reached the limit, those requests could still be considered open and continue to slow your connection. Contact the Exchange support team at exchange.support@se.com if you think this is the case.

### 500
InternalServerError
An unexpected internal error has occurred. Please contact Support for more information.
This error lets you know our servers have experienced a problem. Although this is rare, please contact exchange.support@se.com to let us know that you've encountered this error.

### 503
ComplianceRelated
This method has been disabled.


# Support
# Blogs
---
# API Reference
