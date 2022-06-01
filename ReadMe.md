ref: refs/heads/master
# Databricks and MLFlow in R

## Overview

This document will provide information related to how to train models using Databricks and MLFlow in R. When training a model the user often will need to use a Graphics Processing Unit (GPU), please make sure that the Databricks cluster is set up appropriately and is using an ML version of the runtime. Please note that a new runtime is released about every 6 months, and support for each runtime usually lasts 6 months unless the user selects a Long-Term Support runtime which is supported for 2 years.

## Install and Load Libraries

Prior to beginning an MLFlow run, please import mlflow and set the experiment context. This will allow the user to track each run within the experiment with ease.

## Create Experiment

An experiment directory needs to be initially created prior to setting and running experiments.

Two options for creating an experiment directory:

In a cmd cell:

Enter the following code:

mlflow_create_experiment(name,
  artifact_location = NULL,
  client = NULL,
  tags = NULL
)


In the tab bar:



## Set Experiment

## 
