# ML Pipeline Demo Configuration with MLFlow, PyCaret and Ploomber

This is a demo of an ML Pipeline with MLFlow, PyCaret and Ploomber libraries.

Here is the blog link I referred : [Piplines with Ploomber](https://towardsdatascience.com/machine-learning-pipeline-with-ploomber-pycaret-and-mlflow-db6e76ee8a10)
by [Edwin Tan](https://www.linkedin.com/in/edwintyh)

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

```
ploomber scaffold <folder_name> --empty

```

##### What are each of the directories in ```diabetes_ml```?

tasks : contains Jupyter Notebooks to be executed

products: stores executed Jupyter Notebooks, intermediate and final outputs

mlflow: stores parameters, results and artifacts that were logged using MLFlow

input_data: stores raw data

### MLFlow Server

Lets launch a MLFlow tracking server on localhost to log the experiments.

```cd sample-ml/mlflow
mlflow server --backend-store-uri sqlite:///expt.db --default-artifact-root "file:\\My Drive\\Data-Science\\Projects\\ploomber\\sample-ml\\mlflow\\ml-artifacts"
```

* backend-store-uri: URI to which to persist experiment and run data. We are using an SQLite database for this.

* default-artifact-root: Directory in which to store artifacts for any new experiments created

### Training Pipeline

**Defining and Configuring pipeline.yaml**

A Ploomber pipeline is a .yaml file that contains tasks and relationships between tasks. Lets define our training pipeline in the pipeline.yaml file.

We defined 3 tasks in the pipeline with the name clean, visualize and train. For each task we defined the following:

* source: path to Jupyter Notebooks (.ipynb) or python scrips (.py) that we intend to run
name: name of the task

* upstream: name of the upstream task. This tells us about the dependencies between notebooks. For example the visualize task has clean task as its upstream indicating that the visualize task requires the output from the clean task.

* product: are the output(s) (if any) that is generated by the task. There can be multiple products for each notebook. For example, in clean_data.ipynb we have two products: (1) nb: path to a copy of clean_data.ipynb that was executed. This notebook copy will contain the outputs of the Jupyter Notebook cells. (2) data: path to intermediate data output by clean_data.ipynb
params are parameter(s) which are passed to the Jupyter Notebook when it is executed

* params are parameter(s) which are passed to the Jupyter Notebook when it is executed

The {{root}} placeholder refers to the directory path which the pipeline.yaml file is in.

### Creating the Notebooks

Now that we have defined the pipeline, lets create the Jupyter Notebooks for the 3 tasks.

```
ploomber scaffold
```

We are using ploomber scaffold again, however instead of creating a project directory structure with an empty pipeline.yaml file, Ploomber reads the existing pipeline.yaml file and creates the missing Jupyter Notebooks. Now there are 3 new Jupyter Notebooks in the tasks folder.

### Injecting the Notebooks

We have previously defined the upstream and product information in pipeline.yaml. Lets transfer the information into the Jupyter Notebook using cell injection.

```
ploomber nb --inject
```

Ploomber helps to populate input (aka upstream) and output (aka product) path for each notebook based on the pipeline.yaml file with cell injection. A new “parameter” cell is injected into the notebook with the product and upstream information we defined earlier. The first 3 cells are not useful and we can delete them.

#### Creating the Notebooks

* In the clean_data.ipynb script, we have imputed the NaN values with median imputer
* In the visualize.iipynb script, we have used pandas_profiling to generate the plots

**Steps to Train Model**

* Import packages
* set the MLFlow tracking URI
* Read the cleaned data that was output by clean_data.ipynb
* Setup PyCaret for training
* Save PyCaret setup configurations
* Train, tune and finalize models

#### Parameters for the setup command of PyCaret

* data : the training DataFrame
* target: column name of the target variable
* train_size defined the ratio of the training DataFrame to be used for training. The remaining will be used as the holdout set.
* shuffle_fold: set as True to shuffle the data while doing cross validation
* html: Set as False to prevent runtime display of monitor
* silent: Set as True to skip the confirmation of input data types
* experiment_name: MLFlow experiment name
* log_experiment: Set as True to log metrics, parameters and artifacts to MLFlow server
* fix_imbalance: Set as True to balance the distribution of the target class. SMOTE is applied by default to create synthetic * datapoints for the minority class
* numeric_imputation: Imputation method for numerical values e.g. mean or median or zero
* session_id: Controls the randomness of an experiment. Use to generate reproducible experiments

PyCaret’s ```compare_models``` function trains and evaluates performance of all estimators available in the model library using cross validation and outputs a score grid with the average cross validated score. These estimators are trained on their default hyperparameters, there are no hyperparameter tuning at this stage.

##### Parameters:

* include_models: selected list of models to train and evaluate. Note that we have passed this parameter into the notebook from * pipeline.yaml using cell injection.
* sort: sort the output results based on selected metric
* n_select: select top n models to return
We have defined n_select as 1, hence only the best model based on AUC is selected.

#### Finalizing the model

Next we perform hyperparameter tuning on the Logistic Regression model using tune_model, select the best performing model based on AUC and finalize the model. Pycaret’s finalize_model function trains a given estimator on the entire dataset including the holdout set.

```
final_model = finalize_model(tune_model(best, choose_better = True, optimize = 'AUC'))
```
