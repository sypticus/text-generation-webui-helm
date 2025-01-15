# Oobabooga Helm

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
helm install text-generation-ui ./text-generation-webui/ -n oobabooga --create-namespace`
```

You should then be able to access the dashboard.

```console
POD=$(kubectl get pods -l app.kubernetes.io/instance=text-generation-ui   -o jsonpath="{.items[0].metadata.name}" -n oobabooga)
kubectl -n proxy port-forward $POD 9117

```
In a browser window, open http://localhost:9117

However without a model loaded, the chatbot will not work.

## Persistence
By default the config and model info folders are dropped at each deploy. Turn on `persistence.enabled` to allow storage.
If you already have a PVC you would like to use, set it in values.yaml, otherwise a new PVC will be created for you and used.
Ensure that your default storage class is set up correctly so you don't loose data.


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
//TODO: It is possible to set this up using helm. I'm in the process of cleaning up and unifying my scripts and should have something posted soon 



## Configuration

Documentation of configuration can be found on the Oobabooga [GitHub](https://github.com/oobabooga/text-generation-webui?tab=readme-ov-file#updating-the-requirements),
as well as from [Atinoda](https://github.com/Atinoda/text-generation-webui-docker?tab=readme-ov-file#configuration)

Further info can be found in the chart's included [values.yaml](https://github.com/sypticus/text-generation-webui-helm/blob/main/text-generation-webui/values.yaml)


The following are the most of the relevant config values which can be set in the helm chart.
Further environment params can be added with the `environmentParams` field, and command line params can be passed in with `additionalCmdLineParams`

| Parameter                       | Description                                                              | Default                         |
|---------------------------------|--------------------------------------------------------------------------|---------------------------------|
| `image.repository`              | Image repository                                                         | `atinoda/text-generation-webui` |
| `image.tag`                     | Image tag                                                                | `default-cpu`                   |
| `image.pullPolicy`              | Image pull policy                                                        | `IfNotPresent`                  |
| `service.type`                  | Kubernetes service type                                                  | `ClusterIP`                     |
| `service.webUiPort`             | Port to be used for the web ui.                                          | `7860`                          |
| `service.apiPort`               | Port to be used for the api calls.                                       | `5000`                          |
| `service.loadBalancerIP`        | If type is LoadBalancer, set the IP                                      |                                 |
| `persistence.enabled`           | Use a PVC to store data.  (models, characters, etc)                      | `false`                         |
| `persistence.existingClaim`     | An existing VPC to use for storage. If empty, a new one will be created. |                                 |
| `persistence.storageClass`      | Storage class to use if creating a new PVC                               |                                 |
| `environmentParams`             | Extra environment params to be passed into the container                 |                                 |
| `additionalCmdLineParams`       | Additional CMD line params to be passed to the webui server at startup   | ["--listen", "--api"]           |
