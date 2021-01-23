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

**MPI job on AmlCompute using `horovodrun`**

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
      gpu: 4
compute:
  type: AksCompute
  target: my-aks-cluster
```

TO DO:
- Need to know the # of processes per node that user wants in order to create the hostfile -- processCount? slotsPerWorker? ProcessesPerNode?
## PyTorch job

### PyTorch job on AmlCompute

PyTorch provides a distributed launch utility `torch.distributed.launch`, which gets run on each node. The utility will launch the given number of processes per node.

For PyTorch jobs, Azure ML will set the `MASTER_ADDR`, `MASTER_PORT`, and `NODE_RANK` environment variables, which the user can reference in their launch command. The launch utility will then set the `WORLD_SIZE`, and `RANK` and `LOCAL_RANK` environment variables for each subprocess it starts (these environment variables are needed for PyTorch).

```yaml
name: sample-pytorch-job
type: PytorchJob
experiment_name: distr-pytorch-mnist
worker:
  spec:
    command: >-
      python -m torch.distributed.launch --nproc_per_node 4 --nnodes 4
      --node_rank NODE_RANK --master_addr MASTER_ADDR --master_port MASTER_PORT --use_env
      python train.py
    code:
      path: ./src
    environment: azureml:pytorch-1.7:1
compute:
  type: AmlCompute
  target: gpu-cluster
  instance_count: 4
```

### PyTorch job on Amlk8s

```yaml
name: sample-pytorch-job-k8s
type: PytorchJob
experiment_name: distr-pytorch-mnist
worker:
  spec:
    command: python train.py
    code:
      path: ./src
    environment: azureml:pytorch-1.7:1
    resource_limits:
      gpu: 4
compute:
  type: AksCluster
  target: my-aks-clutser
  instance_count: 4
```

## TensorFlow job

If training with TensorFlow 2 MultiWorkerMirroredStrategy, the user will provide the worker spec. If using the ParameterServerStrategy, the user will provide the parameter server and worker specs.

For TensorFlow jobs, Azure ML will set the `TF_CONFIG` environment variable which is required for distributed TF training.

### TensorFlow job on AmlCompute

**MultiWorkerMirroredStrategy:**

```yaml
name: sample-tf-job
type: TensorflowJob
experiment_name: distr-tf-mnist
worker:
  num_replicas: 4
  spec:
    command: python train.py
    code:
      path: ./src
    environment: azureml:tensorflow-2.3:1
compute:
  type: AmlCompute
  target: gpu-cluster
  instance_count: 4
```

**ParameterServerStrategy:**

```yaml
name: sample-tf-job
type: TensorflowJob
experiment_name: distr-tf-mnist
parameter_server:
  num_replicas: 2
  spec:
    command: python train.py
    code:
      path:
        ./src
    environment: azureml:tensorflow-2.3:1
worker:
  num_replicas: 4
  spec:
    command: python train.py
    code:
      path: ./src
    environment: azureml:tensorflow-2.3:1
compute:
  type: AmlCompute
  target: gpu-cluster
  instance_count: 4
```


### TensorFlow job on Amlk8s

```yaml
name: sample-tf-job-k8s
type: TensorflowJob
experiment_name: distr-tf-mnist
parameter_server:
  num_replicas: 2
  spec:
    command: python train.py
    code:
      path:
        ./src
    environment: azureml:tensorflow-2.3:1
  resource_limits:
    cpu: 1
worker:
  num_replicas: 4
  spec:
    command: python train.py
    code:
      path: ./src
    environment: azureml:tensorflow-2.3:1
  resource_limits:
    gpu: 4
compute:
  type: AmlCompute
  target: gpu-cluster
```
