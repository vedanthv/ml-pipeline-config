# ML Pipeline Demo Configuration with MLFlow, PyCaret and Ploomber

This is a demo of an ML Pipeline with MLFlow, PyCaret and Ploomber libraries.

### Overview of the Tools

1. Ploomber
Ploomber is an open source framework used for building modularized data pipelines using a collection of python scripts, functions or Jupyter Notebooks. Imagine we have multiple Jupyter Notebooks, each for a different purpose, e.g. data cleaning, feature engineering, model training and model evaluation. Ploomber helps to concatenate all these notebook into sequence of steps based on a user defined pipeline.yaml file.

2. MLFlow

MLFlow is an open source platform to manage the ML lifecycle, including experimentation, reproducibility, deployment, and a central model registry. MLFlow offers 4 different components:

* MLFlow Tracking: Record and query experiments: code, data, config, and results
* MLFlow Projects: Package data science code in a format to reproduce runs on any platform
* MLFlow Models: Deploy machine learning models in diverse serving environments
* Model Registry: Store, annotate, discover, and manage models in a central repository.

3. PyCaret

PyCaret is an open-source, low-code automated machine learning (AutoML) library in python. PyCaret helps to simplify the model training process by automating steps such as data pre-processing, hyperparameter optimization, stacking, blending and model evaluation. PyCaret is integrated with MLFlow and automatically log the run’s parameters, metrics and artifact onto MLFlow server.

### TLDR of Objective

Now that we understand what each of the tool does, if you haven’t already guessed it, we will be cleaning the data with Pandas, training the models with PyCaret, logging our experiments with MLFlow and the entire pipeline will be orchestrated by Ploomber.

### Project Description

Let’s dive into an example. We will be using the Pima Indian Diabetes Dataset from the National Institute of Diabetes and Digestive and Kidney Diseases. The objective of the dataset is to diagnostically predict whether or not a patient has diabetes, based on certain diagnostic measurements included in the dataset.

### Complete Pipeline Config Steps and Flow

##### Project Directory Structure

Ploomber offers command line which creates a barebone project directory structure.

```ploomber scaffold <folder_name> --empty
```