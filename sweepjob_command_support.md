In v1 there are two modes for jobs: 
1. script + arguments 
1. command 

V2 will only support command mode.  

Sample v2 SweepJob yaml config:

```yaml
experiment_name: "sweep-trial-v3"
algorithm: random
job_type: Sweep
name: test_v3111
search_space:
  learning_rate:
    spec: uniform
    min_value: 0.001
    max_value: 0.1
  subsample:
    spec: uniform
    min_value: 0.1
    max_value: 1.0    
objective:
  primary_metric: accuracy
  goal: maximize
trial:
  command: >-
    python train.py --data {inputs.training_data} --lr {search_space.learning_rate} --subsample {search_space.subsample}
  environment: azureml:xgboost-env:1
  compute:
    target: azureml:goazurego
  code: 
    directory: train
  inputs:
    training_data:
      data: azureml:irisdata:1
      mode: Mount
limits:
  max_total_runs: 10
  max_concurrent_runs: 10
  max_duration_minutes: 20
```

HyperDrive will need to support the command mode, which means 
1. not restricting jobs to only Python runtime
2. in v2, cannot modify the arguments to append the command line argument with parameter value for each trial, e.g. `arguments=['--lr', 0.001]` (as script + arguments mode won't be an option)
3. Instead, in the CLI/SDK the user will use the `{search_space.*}` notation to denote which part(s) of the command will be injected with the hyperparameter value(s) for each trial. To do so, client side will resolve this notation to the appropriate environment variable, which HyperDrive will then use to specify the corresponding value for each trial.

    E.g. `{search_space.learning_rate}` gets resolved to `AZUREML_SWEEP_learning_rate`; `{search_space.subsample}` gets resolved to `AZUREML_SWEEP_subsample`
