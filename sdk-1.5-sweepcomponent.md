# SDK 1.5 SweepComponent

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
  policy_type: truncation
  evaluation_interval: 100
  delay_evaluation: 200
limits:
  max_total_trials: 20
  max_concurrent_trials: 5
  timeout_minutes: 10000
  ```
