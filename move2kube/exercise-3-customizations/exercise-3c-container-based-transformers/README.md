# Generating custom Helm charts to illustrate adding custom files and directories in custom locations using the Container-based Transformer

Move2Kube allows defining containerized transformers where-in the `directory_detect` and `transform` operations are scripts residing within the container that are invoked by Move2Kube. To illustrate this mode of defining containerized transformer, we take the example of altering the output structure of Move2Kube to generate one helm-chart per service, within each service directory. 

## Steps for Move2Kube CLI/UI

1. Run Move2Kube transform on the `enterprise-app` without any customizations and observe the output:

    ```console
    $ cd exercise-3-customizations/exercise-3c-container-based-transformers
    ```

    `--qa-skip` option allows us to run Move2Kube transform non-interactively to skip the questions and use the default answers.

    ```console
    $ move2kube transform -s ../../source/enterprise-app/ --qa-skip --overwrite
    ```
    The output of Move2Kube is generated within `myproject` folder such that one consolidated helm-chart for all services is generated within the `myproject/deploy/yamls-parameterized`.

1. We will use the containerized custom transformer in `customizations` folder. We specify the customization using the `-c` flag, and the configuration using the `-f` flag.
    ```console
    $ move2kube transform -s src/ -c customizations/ -f m2kconfig-containerized-transformer.yaml --overwrite
    ``` 

Once the output is generated, we can observe one helm-chart is generated for each service and placed within the service directory. Also, note that every helm-chart project is named after the service it is meant for. The contents are shown below for reference:
```
    myproject
    ├── config-utils
    │   ├── config-utils-helmchart
    │   ├── pom.xml
    │   └── src
    ├── customers
    │   ├── Makefile
    │   ├── README.md
    │   ├── customers-helmchart
    │   ├── mvnw
    │   ├── mvnw.cmd
    │   ├── pom.xml
    │   └── src
    ├── gateway
    │   ├── gateway-helmchart
    │   ├── manifest.yml
    │   ├── mvnw
    │   ├── mvnw.cmd
    │   ├── pom.xml
    │   └── src
    └── orders
        ├── README.md
        ├── manifest.yml
        ├── mvnw
        ├── mvnw.cmd
        ├── orders-helmchart
        ├── pom.xml
        └── src
```

## Anatomy of containerized transformer
The containerized transformer differs from other external transformers in the following ways:
1. A Dockerfile (see `customizations/Dockerfile`) needs to be specified to define the execution context of the containerized transformer.

1. The `transformer.yaml` (see `customizations/transformer.yaml`) will state that the class of the transformer is `Executable` and there will be an config section (as shown below) which defines the detect and transformer commands, working directory within the container, build parameters like container image name, and Dockerfile:

```
apiVersion: move2kube.konveyor.io/v1alpha1
kind: Transformer
metadata:
  name: ContainerizedTransformer
  labels: 
    move2kube.konveyor.io/inbuilt: false
spec:
  mode: "Container"
  class: "Executable"
  isolated: true
  override:
    matchLabels: 
      move2kube.konveyor.io/built-in: "true"
  directoryDetect:
    levels: 1
  consumes:
    Service:
      merge: false
  config:
    platforms: 
      - "darwin"
    directoryDetectCMD: ["python", "./detect.py", "input=$(M2K_DETECT_INPUT_PATH)", "output=$M2K_DETECT_OUTPUT_PATH"]
    transformCMD: ["python", "./transform.py"]
    container: 
      image: containerizedtransformer:latest
      workingDir: "/m2k/"
      keepAliveCommand: ["tail", "-f", "/dev/null"]
      build:
        dockerfile: "Dockerfile"
        context: "."
```
The above steps can be replicated in the UI, by uploading the zip of the custom transformer as an input of type `customization`.
For convenience we have provided the `customizations.zip` file in this exercise directory.