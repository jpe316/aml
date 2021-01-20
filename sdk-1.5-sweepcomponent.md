# SDK 1.5 SweepComponent

Example SweepComponent config:
```yaml
$schema: http://azureml/sdk-2-0/SweepComponent.json
name: microsoft.com.azureml.samples.tune
version: 0.0.4
display_name: Tune
type: SweepComponent
description: A dummy hyperparameter tuning module
tags: {category: Component Tutorial, contact: amldesigner@microsoft.com}
inputs:
  training_data:
    type: path
    description: Training data organized in the torchvision format/structure
    optional: false
  max_epochs:
    type: Integer
    description: Maximum number of epochs for the training
    optional: false
algorithm: random
search_space:
  learning_rate:
    type: uniform
    min_value: 0.001
    max_value: 0.1
  subsample:
    type: uniform
    min_value: 0.1
    max_value: 0.1
objective:
  primary_metric: accuracy
  goal: maximize
trial: # takes a CommandComponent
  command: >-
    python train.py --training_data {inputs.training_data} --max_epochs {inputs.max_epochs}
    --learning_rate {search_space.learning_rate} --subsample {search_space.subsample}
  code: ./src
  environment: <your environment here>
early_termination:
  policy_type: truncationSelection
  evaluation_interval: 100
  delay_evaluation: 200
limits:
  max_total_trials: 20
  max_concurrent_trials: 5
  timeout_minutes: 10000
  ```
  
  * Supported algorithm types: random, grid, bayesian
  * Supported early termination policies: bandit, medianStopping, truncationSelection (each policy has its own properties to be specified)
  * Supported hyperparameter expression types:
  ```yaml
  definitions:
  HyperparameterExpression:
    oneOf:
      - properties:
          spec:
            type: string
            enum: [choice]
          values: 
            type: array
      - properties:
          spec:
            type: string
            enum: [loguniform, uniform]
          min_value:
            type: number
          max_value:
            type: number
      - properties:
          spec:
            type: string
            enum: [qloguniform, quniform]
          min_value:
            type: number
          max_value:
            type: number
          q:
            type: number
      - properties:
          spec:
            type: string
            enum: [lognormal, normal]
          mu: 
            type: number
          sigma:
            type: number
      - properties:
          spec:
            type: string
            enum: [qlognormal, qnormal]
          mu: 
            type: number
          sigma:
            type: number
          q:
            type: number
      - properties:
          spec:
            type: string
            enum: [randint]
          upper:
            type: number
    required: [spec]
    type: object
  ```
  
  ## Notes
  * the search space can also be nested for conditional search space scenario, see this spec: https://github.com/Azure/azureml_run_specification/blob/master/specs/sweepjob.md#conditional-search-space-support
  * in the CLI/SDK the user will use the `{search_space.*}` notation to denote which part(s) of the command will be injected with the hyperparameter value(s) for each trial. To do so, client side will resolve this notation to the appropriate environment variable, which HyperDrive will then use to specify the corresponding value for each trial.

    E.g. `{search_space.learning_rate}` gets resolved to `AZUREML_SWEEP_learning_rate`; `{search_space.subsample}` gets resolved to `AZUREML_SWEEP_subsample`
