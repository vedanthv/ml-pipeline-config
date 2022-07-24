# ML Pipeline Demo Configuration with MLFlow, PyCaret and Ploomber

This is a demo of an ML Pipeline with MLFlow, PyCaret and Ploomber libraries.

### Overview of the Tools

1. Ploomber
Ploomber is an open source framework used for building modularized data pipelines using a collection of python scripts, functions or Jupyter Notebooks. Imagine we have multiple Jupyter Notebooks, each for a different purpose, e.g. data cleaning, feature engineering, model training and model evaluation. Ploomber helps to concatenate all these notebook into sequence of steps based on a user defined pipeline.yaml file.

2. MLFlow

MLFlow is an open source platform to manage the ML lifecycle, including experimentation, reproducibility, deployment, and a central model registry. MLFlow offers 4 different components:

