## MPI job

For an MPI job, the user will need to provide the launcher spec and the worker spec.


### MPI job on AmlCompute
For fine-grained control, the user provides the MPI launch command (e.g. `mpirun` or `mpiexec`).

For MPI jobs, Azure ML will set an environment variable `AZUREML_MPI_HOSTFILE` that has the path to the MPI hostfile created by Azure ML.

```yaml
name: sample-mpi-job
type: MpiJob
experiment_name: pytorch-mnist-horovod
launcher:
  spec:
    command: >-
      mpirun -np 16 --hostfile AZUREML_MPI_HOSTFILE
      -bind-to none -map-by slot -x NCCL_DEBUG=INFO -x LD_LIBRARY_PATH -x PATH
      -mca pml ob1 -mca btl ^openib
      python train.py
    code:
      path: ./src
    environment: azureml:pytorch-1.7:1
worker:
  spec:
    environment: azureml:pytorch-1.7:1
compute:
  type: AmlCompute
  target: nc24-cluster
  instance_count: 4
```

### MPI job on AmlCompute using `horovodrun`
The Horovod framework provides a convenient, Open MPI-based wrapper called `horovodrun` for users to launch their distributed Horovod jobs. Using `horovodrun` is simpler to configure, and we can recommend this path to users who do not need more fine-grained MPI control.

```yaml
name: sample-mpi-job
type: MpiJob
experiment_name: pytorch-mnist-horovod
launcher:
  spec:
    command: >-
      horovodrun -np 16 --hostfile AZUREML_MPI_HOSTFILE python train.py
    code:
      path: ./src
    environment: azureml:pytorch-1.7:1
worker:
  spec:
    environment: azureml:pytorch-1.7:1
compute:
  type: AmlCompute
  target: gpu-cluster
  instance_count: 4
```

### MPI job on Amlk8s
For jobs running on Amlk8s, users can specify additional configurations that are relevant to k8s training, such as specifying resource limits.

```yaml
name: sample-mpi-job-k8s
type: MpiJob
experiment_name: pytorch-mnist-horovod
launcher:
  spec:
    command: >-
      mpirun -np 16 --allow-run-as-root -bind-to none -map-by slot
      -x LD_LIBRARY_PATH -x PATH -mca pml ob1 -mca btl ^openib
      python train.py
    code:
      path: ./src
    environment: azureml:pytorch-1.7:1
    resource_limits:
      cpu: 1
      memory: 2Gi
worker:
  spec:
    environment: azureml:pytorch-1.7:1
    resource_limits:
      nvidia.com/gpu: 4
compute:
  type: AksCompute
  target: my-aks-cluster
```

TO DO:
- figure out where to get ProcessCount from the user (needed for Azure ML to create the hostfile), versus num replicas ? in k8s

## PyTorch job

## TensorFlow job
