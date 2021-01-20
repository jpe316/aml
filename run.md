## mlflow

### concepts
* **parameters**: Key-value input parameters of your choice. Both keys and values are strings.
* **metrics**: Key-value metrics, where the value is numeric. Each metric can be updated throughout the course of the run (for example, to track how your model’s loss function is converging), and MLflow records and lets you visualize the metric’s full history.
* **artifacts**: Output files in any format. For example, you can record images (for example, PNGs), models (for example, a pickled scikit-learn model), and data files (for example, a Parquet file) as artifacts.

**runs**
- can be logically grouped into "experiments"
- nested child runs possible:
  - parent run
    - child run 1
    - child run 2

### artifacts cli
`mlflow artifacts download --run-id <run_id>`

`mlflow artifacts list --run-id <run_id>`

`mlflow artifacts log-artifact/log-artifacts`

### runs cli
`mlflow runs delete --run-id <run_id>`

`mlflow runs describe --run-id <run_id>`

`mlflow runs list --experiment-id <experiment_id>`

`mlflow runs restore --run-id <run_id>`

## azureml job vs. run
**job**: resource in ARM
* a job can have 1 or more runs
* a job <==> root run, where rootRunId == job name
