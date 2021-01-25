## MPI job

### MPI job = `horovodrun`

The Horovod framework provides a convenient, Open MPI-based wrapper called `horovodrun` for users to launch their distributed Horovod jobs. Using `horovodrun` is simpler to configure, and we can recommend this path to users who do not need more fine-grained MPI control.

```yaml
command: >-
  horovodrun -np 16 --hostfile $AZUREML_MPI_HOSTFILE python train.py
code:
  path: ./src
environment: 
  docker:
    image: docker.io/pytorch
distributed:
  mpi:
    worker:
      count: 4
compute:
  infrastructure: managed
  instance_type: Standard_NC24
```



### MPI job
For fine-grained control, the user provides the MPI launch command (e.g. `mpirun` or `mpiexec`).

For MPI jobs, Azure ML will set an environment variable `AZUREML_MPI_HOSTFILE` that has the path to the MPI hostfile created by Azure ML.

```yml
code:
  path: ./src
environment: 
  docker:
    image: docker.io/pytorch
command: >-
  mpirun -np 16 --hostfile $AZUREML_MPI_HOSTFILE
  -bind-to none -map-by slot -x NCCL_DEBUG=INFO -x LD_LIBRARY_PATH -x PATH
  -mca pml ob1 -mca btl ^openib
  python train.py
distributed:
  mpi:
    worker:
      count: 4
      slots_per: 4 # optional, workers have defaults
compute:
  infrastructure: managed
  instance_type: Standard_NC24
```

### MPI job - launcher = worker env
```yml
command: >-
  mpirun -np 16 --hostfile $AZUREML_MPI_HOSTFILE
  -bind-to none -map-by slot -x NCCL_DEBUG=INFO -x LD_LIBRARY_PATH -x PATH
  -mca pml ob1 -mca btl ^openib
  python train.py
code:
  path: ./src
environment: 
  docker:
    image: docker.io/pytorch
compute:
  infrastructure: managed
  instance_type: Standard_NC24
distributed:
  mpi:
    worker:
      count: 4
```


### MPI job - K8s
For jobs running on k8s, users can specify additional configurations that are relevant to k8s training, such as specifying resource limits.

```yaml
environment: 
  docker:
    image: docker.io/pytorch
code:
  path: ./src
command: >-
    mpirun -np 16 --allow-run-as-root -bind-to none -map-by slot
    -x LD_LIBRARY_PATH -x PATH -mca pml ob1 -mca btl ^openib
    python train.py
distributed:
  mpi:
    launcher:
      resources:
        limits:
          cpu: 1
          memory: 2Gi
    worker:
      count: 4
      resources:
        limits:
          gpu: 4
compute:
  infrastructure: k8s
  target: my-aks-cluster
```

TO DO:
- Need to know the # of processes per node that user wants in order to create the hostfile -- processCount? slotsPerWorker? ProcessesPerNode?

## PyTorch jobs

### PyTorch job on AmlCompute

PyTorch provides a distributed launch utility `torch.distributed.launch`, which gets run on each node. The utility will launch the given number of processes per node.

For PyTorch jobs, Azure ML will set the `MASTER_ADDR`, `MASTER_PORT`, and `NODE_RANK` environment variables, which the user can reference in their launch command. The launch utility will then set the `WORLD_SIZE`, and `RANK` and `LOCAL_RANK` environment variables for each subprocess it starts (these environment variables are needed for PyTorch).

```yaml
command: >-
  python -m torch.distributed.launch --nproc_per_node 4 --nnodes 4
  --node_rank $NODE_RANK --master_addr $MASTER_ADDR --master_port $MASTER_PORT --use_env
  python train.py
code:
  path: ./src
environment: 
  docker:
    image: docker.io/pytorch
distributed:
  pytorch:
    worker:
      count: 4
compute:
  infrastructure: managed
  instance_type: Standard_NC24
```

### PyTorch job on k8s

```yml
command: python train.py --language $LANGUAGE
code:
  path: ./src
environment: 
  docker:
    image: docker.io/pytorch
compute:
  type: k8s
  target: my-aks-cluster
environment_variables:
  LANGUAGE: en-us
distributed:
  pytorch:
    worker:
      count: 4
    resource_limits:
      gpu: 4
```

## TensorFlow jobs

If training with TensorFlow 2 MultiWorkerMirroredStrategy, the user will provide the worker spec. If using the ParameterServerStrategy, the user will provide the parameter server and worker specs.

For TensorFlow jobs, Azure ML will set the `TF_CONFIG` environment variable which is required for distributed TF training.

### TensorFlow job - multiworker mirrored

**MultiWorkerMirroredStrategy:**

```yml
command: python train.py
code:
  path: ./src
environment: 
  docker:
    image: docker.io/tensorflow
compute:
  type: managed
  target: gpu-cluster
  instance_count: 4
distributed:
  tensorflow:
   worker: 
    count: 4
```

### Tensorflow job - parameter server + workers

```yml
command: python train.py --magic $MAGIC
code:
  path:
    ./src
environment: 
  docker:
    image: docker.io/tensorflow
environment_variables:
  MAGIC: number

distributed:
  tensorflow:
    worker: 
      count: 2
      resource_limits:
        cpu: 4
    parameter_server:
      count: 4
      environment:
        docker:
          image: docker.io/python
      resource_limits:
        gpu: 4
        memory: 2Gi
      environment_variables:
        MAGIC: newnumber
compute:
  infrastructure: managed
  target: gpu-cluster
```

