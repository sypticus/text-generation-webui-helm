# Text Generation Webui Helm

This is a helm chart for the excellent Oobagooba [Text generation web UI](https://github.com/oobabooga/text-generation-webui), which allows you to host your own LLM chatbot using any model, on your own hardward.
The goal of this Helm chart is to make it easy for anyone with a Kubernetes cluster to quickly and predictably deploy the entire application with a few config changes and commands.

https://github.com/sypticus/text-generation-webui-helm.git


### NOTE: This project is under construction and will be completed shortly.

## Build the Docker image.
You will need a prebuilt docker image for your specific platform before deploying. The easiest way is to use one of the prebuilt models here from [Atinoda](https://github.com/Atinoda/text-generation-webui-docker)
At the moment, this chart is designed to work with the default cpu chart from Atinoda.


## Installing the Chart.

```console
git clone https://github.com/sypticus/text-generation-webui-helm.git
helm install text-generation-ui ./text-generation-webui/ -n oobabooga --create-namespace
```

You should then be able to access the dashboard.

```console
POD=$(kubectl get pods -l app.kubernetes.io/instance=text-generation-ui   -o jsonpath="{.items[0].metadata.name}" -n oobabooga)
kubectl -n proxy port-forward $POD 7860

```
In a browser window, open http://localhost:7860

However without a model loaded, the chatbot will not work.

## Persistence
By default the config and model info folders are dropped at each deploy. Turn on `persistence.enabled` to allow storage.
You will need to define an PVC you would like to use and set it in `existingClaim`.


## Configuration

Text-Generation-Webui is configured in a few different ways. The first is the [command line flags](https://github.com/oobabooga/text-generation-webui?tab=readme-ov-file#updating-the-requirements)
These are set with the `additionalParams` field of the `values.yaml`. This will inject the params into the `CMD_FLAGS.txt` file which is passed into the application at startup

```yaml
additionalParams:
  - "--listen"
  - "--api"
```

There are also a number of Environment Variables which can be set. In the Docker version, these were set in the `.env` file. 
Now they are set in the `environment` field of `values.yaml`. 
This will include environment variables needed for the application, as well as CUDA config (more on that in a bit), or any Kubernetes envvars needed.

```yaml
environment:
  - name: CONTAINER_API_PORT
    value: "5000"
  - name: BUILD_EXTENSIONS
    value: ""
  - name: APP_RUNTIME_GID
    value: "6972"
```

## Loading a model
Models can be downloaded directly in the app in the `models` tab. You also can download them from [HuggingFace](https://huggingface.co/models). The downloaded models should be moved to the 
saved to the attached models PVC volume in the `/models/` directory.



## CUDA 
Cuda can be used to access Nvidia GPU resources from video cards on the host nodes. 

### NOTE: The NVIDIA and Cuda drivers need to be installed on the host, and the Kubernetes cluster needs to be configured to work with these.
### For K3S, see https://github.com/sypticus/nvidia-k3s-cuda for instructions, otherwise Nvidia has documentation on how to install.

Cuda can be enabled by setting `cuda.enabled` in the values.yaml. Here you can also set the other required params. 
This will set the needed ENV params and runtimeClassName for Cuda to work for at least k3s, but you may need to add other ENV params for your flavor of K8s. 
This can be done in the `extraEnvVars` field
You will also need to ensure that the pods are created on a node with available GPU resources. 
This can be done manually via `nodeSelector` etc, 

```yaml
nodeSelector:
  kubernetes.io/hostname: "my-llm-host"
```

Or if you are using the Nvidia `k8s-device-plugin`, which will detect and label GPU nodes automatically, you can set required resources. 

```yaml
resources:
    limits:
      nvidia.com/gpu: 1 # requesting 1 GPU
```


## Configuration

Documentation of configuration can be found on the Oobabooga [GitHub](https://github.com/oobabooga/text-generation-webui?tab=readme-ov-file#updating-the-requirements),
as well as from [Atinoda](https://github.com/Atinoda/text-generation-webui-docker?tab=readme-ov-file#configuration)

Further info can be found in the chart's included [values.yaml](https://github.com/sypticus/text-generation-webui-helm/blob/main/text-generation-webui/values.yaml)


The following are the most of the relevant config values which can be set in the helm chart.
Further environment params can be added with the `environmentParams` field, and command line params can be passed in with `additionalCmdLineParams`

| Parameter                   | Description                                                                                                               | Default                         |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------|---------------------------------|
| `image.repository`          | Image repository                                                                                                          | `atinoda/text-generation-webui` |
| `image.tag`                 | Image tag                                                                                                                 | `default-cpu`                   |
| `image.pullPolicy`          | Image pull policy                                                                                                         | `IfNotPresent`                  |
| `service.type`              | Kubernetes service type                                                                                                   | `ClusterIP`                     |
| `service.webUiPort`         | Port to be used for the web ui.                                                                                           | `7860`                          |
| `service.apiPort`           | Port to be used for the api calls.                                                                                        | `5000`                          |
| `service.loadBalancerIP`    | If type is LoadBalancer, set the IP                                                                                       |                                 |
| `persistence.enabled`       | Use a PVC to store data.  (models, characters, etc)                                                                       | `false`                         |
| `persistence.existingClaim` | An existing VPC to use for storage.                                                                                       |                                 |
| `extraEnvVars`              | Extra environment params to be passed into the container                                                                  |                                 |
| `additionalCmdLineParams`   | Additional CMD line params to be passed to the webui server at startup                                                    | ["--listen", "--api"]           |
| `cuda.enabled`              | Enable CUDA to access GPU resources on host                                                                               | `false`                         |
| `cuda.visibleDevices`       | Needed by K3s                                                                                                             | `all`                           |
| `cuda.capabilities`         | Needed by K3s                                                                                                             | `all`                           |
| `cuda.runtimeClassName`     | Needed for some K8s flavors. You would have created this when setting up your cluster for CUDA                            | `nvidia`                        |
| `cuda.torchCudaArchList`    | Compute capabilities of your [GPU](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#compute-capabilities) |                                 |
| `cuda.appRuntimeGID`        | The host group id to use                                                                                                  | `6972`                          |
