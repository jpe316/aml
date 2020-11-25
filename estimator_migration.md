# Estimator to ScriptRunConfig migration guide

Up until now, there have been multiple methods for configuring a training job in Azure Machine Learning via the SDK, including Estimators, ScriptRunConfig, and the lower-level RunConfiguration, which have generated a lot of ambiguity and inconsistency for users. To address this, we are simplifying the job configuration process in Azure ML to converge on using ScriptRunConfig as the recommended option for specifying training jobs. Estimators are deprecated with the 1.19 release of the Python SDK. You should also generally avoid explicitly instantiating a RunConfiguration object yourself, and instead configure your job using the ScriptRunConfig class.

**To migrate to ScriptRunConfig from Estimators, please make sure you are using >= v1.15 of the Python SDK.**

## Documentation & samples
For information on using ScriptRunConfig, you can refer to the following documentation:
* [Configure and submit training runs](https://docs.microsoft.com/azure/machine-learning/how-to-set-up-training-targets)
* [Configuring PyTorch training runs](https://docs.microsoft.com/azure/machine-learning/how-to-train-pytorch)
* [Configuring TensorFlow training runs](https://docs.microsoft.com/azure/machine-learning/how-to-train-tensorflow)
* [Configuring scikit-learn training runs](https://docs.microsoft.com/azure/machine-learning/how-to-train-scikit-learn)

In addition, you can refer to the following samples & tutorials:
* [Azure/MachineLearningNotebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks)
* [Azure/azureml-examples](https://github.com/Azure/azureml-examples)

## Migrating from estimators
Here we will cover some specific considerations when migrating from estimators.

### Defining the training environment
While the various framework estimators had preconfigured environments that were backed by Docker images, the Dockerfiles for these images were private and hence the user did not have a lot of transparency into what these environments contained. In addition, the estimators also took in environment-related configurations as individual parameters on their respective constructors.

When using ScriptRunConfig, all environment-related configurations are encapsulated in the Environment object that gets passed into the `environment` parameter of the ScriptRunConfig constructor. To configure a training job, you will need to provide an environment that has all the dependencies required for your training script. There are a couple of ways to provide an environment:

1) Use a curated environment - curated environments are predefined environments available in your workspace by default. There is a corresponding curated environment for each of the preconfigured framework/version Docker images that backed each framework estimator.
2) Define your own custom environment

Here is an example of using the curated PyTorch 1.6 environment for training:

```python
from azureml.core import Workspace, ScriptRunConfig, Environment

curated_env_name = 'AzureML-PyTorch-1.6-GPU'
pytorch_env = Environment.get(workspace=ws, name=curated_env_name)

compute_target = ws.compute_targets['my-cluster']
src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      compute_target=compute_target,
                      environment=pytorch_env)
```

If you want to specify **environment variables** that will get set on the process where the training script is executed, you should also do that using the Environment object:
```
myenv.environment_variables = {"MESSAGE":"Hello from Azure Machine Learning"}
```

For information on configuring and managing Azure ML environments, see:
* [How to use environments](https://docs.microsoft.com/azure/machine-learning/how-to-use-environments)
* [Curated environments](https://docs.microsoft.com/azure/machine-learning/resource-curated-environments)
* [Train with a custom Docker image](https://docs.microsoft.com/azure/machine-learning/how-to-train-with-custom-image)


### Using data for training
#### Datasets
If you are using an Azure ML dataset for training, pass the dataset as an argument to your script using the `arguments` parameter. By doing so, you will get the data path (mounting point or download path) in your training script via arguments.

The following example configures a training job where the FileDataset, `mnist_ds`, will get mounted on the remote compute.
```python
src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      arguments=['--data-folder', mnist_ds.as_mount()], # or mnist_ds.as_download() to download
                      compute_target=compute_target,
                      environment=pytorch_env)
```

#### DataReference (old)
While we recommend using Azure ML Datasets over the old DataReference way, if you are still using DataReferences for whatever reason, you must configure your job as follows:
```python
# if you want to pass a DataReference object, such as the below:
datastore = ws.get_default_datastore()
data_ref = datastore.path('./foo').as_mount()

src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      arguments=['--data-folder', str(data_ref)], # cast the DataReference object to str
                      compute_target=compute_target,
                      environment=pytorch_env)
src.run_config.data_references = {data_ref.data_reference_name: data_ref.to_config()} # set a dict of the DataReference(s) you want to the `data_references` attribute of the ScriptRunConfig's underlying RunConfiguration object.
```

For more information on using data for training, see:
* [Train with datasets in Azure ML](https://docs.microsoft.com/azure/machine-learning/how-to-train-with-datasets)

### Distributed training
If you need to configure a distributed job for training, you can do that by specifying the `distributed_job_config` parameter in the ScriptRunConfig constructor. You can pass in an `MpiConfiguration`, `PyTorchConfiguration`, or `TensorFlowConfiguration` for distributed jobs of the respective types.

The following example configures a PyTorch training job to use distributed training with MPI/Horovod:
```python
from azureml.core.runconfig import MpiConfiguration

src = ScriptRunConfig(source_directory='.',
                      script='train.py',
                      compute_target=compute_target,
                      environment=pytorch_env,
                      distributed_job_config=MpiConfiguration(node_count=2, process_count_per_node=2))
```

For more information, see:
* [Distributed training with PyTorch](https://docs.microsoft.com/azure/machine-learning/how-to-train-pytorch#distributed-training)
* [Distributed training with TensorFlow](https://docs.microsoft.com/azure/machine-learning/how-to-train-tensorflow)

### Miscellaneous
If you do want to access the underlying RunConfiguration object for a ScriptRunConfig for any reason, you can do so as follows:
```
src.run_config
```
