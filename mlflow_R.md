# Databricks and MLFlow in R

## Overview

This document will provide information related to how to train models using Databricks and MLFlow in R. When training a model the user often will need to use a Graphics Processing Unit (GPU), please make sure that the Databricks cluster is set up appropriately and is using an ML version of the runtime. Please note that a new runtime is released about every 6 months, and support for each runtime usually lasts 6 months unless the user selects a Long-Term Support runtime which is supported for 2 years.

## Install and Load Libraries

Prior to beginning an MLFlow run, please import mlflow and set the experiment context. This will allow the user to track each run within the experiment with ease.

## Create Experiment

An experiment directory needs to be initially created prior to setting and running experiments.

Two options for creating an experiment directory:

In a cmd cell:

![alt text](CreateExperiment.PNG)

Enter the following code:

        mlflow_create_experiment(name,
          artifact_location = NULL,
          client = NULL,
          tags = NULL
        )

Create an experiment only once.

To log runs the exeripment must be set each time.


## Set Experiment

Run this prior to starting an mlflow run.

![alt text](SetExperiment.PNG)


## Begin Training


After setting the experiment, you can begin training. Depending on the type of model you are developing the requirements and architecture for the model will be different. However, there are some key mlflow commands we recommend looking into prior to the beginning of the training.

with(mlflow_start_run() is a useful commands when it comes to launching runs and logging metrics. Please see the MLFlow documentation for details on specific tracking functions. https://www.mlflow.org/docs/latest/R-api.html

Use the template below for starting and tracking a model.

![alt text](start_mlflow.PNG)

        with(mlflow_start_run(), {
  
          model <- ModelType(y ~ x1 + x2, data = train_x)
          test$predictions <- predict(model, test)
  
          preds <- as.numeric(test$predictions)
          real <- as.numeric(test$y)

          mae <- mean(as.numeric(abs(test$predictions - test$y)), na.rm = TRUE)
          rmse <- sqrt(mean(as.numeric((test$predictions - test$y)), na.rm = TRUE))
          r2 <- cor(preds, real, use = "complete.obs") ^ 2

          message(" RMSE: ", rmse)
          message(" MAE: ", mae)
          message(" R2: ", r2)

          mlflow_log_metric("rmse", rmse)
          mlflow_log_metric("mae", mae)
          mlflow_log_metric("r2", r2)

          predictor <- carrier::crate(~ ModelLibrary::predict.ModelType(!!model, .x), !!model)
          mlflow_log_model(predictor, artifact_path="Model Title") 


        })
          }

For any questions about carrier::crate or uncommenting with "!!" please reference the following link:

**Crate::Carrier Documentation**
https://www.rdocumentation.org/packages/carrier/versions/0.1.0


Once the training begins, the user will be able to follow along with the model’s progress by clicking into the Experiment tab. This will take the user to another interface where you will be able to view various metrics and the model that gets saved from the run. Make sure the Machine Learning persona is selected. If the Experiments tab is not available it is likely because the persona is set to Data Science & Engineering or SQL.


![alt text](ExperimentTab.PNG)

![alt text](Experiments.PNG)

# Experiment Troubleshoot

If no metrics are appearing in your experiment results or if you get an error message like the one below while attempting log additional runs, your mlflow model experiment needs debugging. Before troubleshooting, it's important to run the "mflow_end_run" command to stop the mlflow process. 
        
 ![alt text](ErrorMessage.PNG)


        mlflow_end_run()


# Registering the Model

Once you have determined your "best" model run it is important to register that model. The advantage of registering a model is it creates transparency for determining models in staging and production. Additionally, through registering a model it allows for rapid deployments of newly refined models without changing the prediction code.

Start by creating a model landing zone by selecting the Models tab.

![alt text](CreateModel.PNG)

Select Create Model and give it a reference name. 

![alt text](NameModelRepo.PNG)

This is the repository the model will be registered in.

![alt text](ModelLanding.PNG)

Return to the Experiment tab and select the "best fit" model.

![alt text](SelectRun.PNG)

Register the model run.

![alt text](RegisterRun.PNG)

Select the model repository created earlier.

![alt text](RegisterModelRun.PNG)


When the model is registered the button will disapear and the link to the model in the repository will appear. Select this link.


![alt text](ModeRegistered.PNG)

If this is the first time saving a model to this repository it will be seen as Version 1. Select Version 1.
With the model ready for the Staging phase select the "Stage:" option and select Transition to Staging. 

![alt text](ModelStage.PNG)

Leaving a comment may be important for logging the model purpose or changes associated with the model version. It can also be used for notes to the approver as to why the model needs to be transitioned. 

![alt text](Transition.PNG)

## Model Prediction

With model fully registered it can be utilized for predictions.

Open a new notebook and load the same libraries and data as was done in the training notebook.

After that is done use the code below as an example of using the trained model to test on another sample of data.

        logged_model <- "models:/R_Tutorial/staging"


        best_model <- mlflow_load_model(model_uri = logged_model)


        ## Generate prediction on 5 rows of data 
        predictions <- data.frame(mlflow_predict(best_model, data = dataset[1:5, !(names(dataset) == "y")]))

![alt text](ModelPredict.PNG)

## Predict Troubleshooting

Some models, for instance basic stats::lm() linear models will often run an error with the mflow_predict command. 

In those situations try the mlflow_load_model like a function call.

For example if the crate trained model looks like this:

        prediction <- carrier::crate(~ stats::predict.lm(!!trained_model, .x), !!trained_model)
        mlflow_log_model(prediction, artifact_path="Linear Model")  

The prediction may need to be ran like this:

        best_model <- mlflow_load_model("models:/R_Tutorial/staging")
        output <- as.data.frame(best_model(.x = dataset))
        base::names(output)[1] <- "predictions"
