# Writing the Transformation Logic in a Python-like Language using the Starlark Transformer

A lot of the common transformations are already covered by the transformers that are built into Move2Kube.
However, sometimes we need to perform very custom transformations on the input (like adding labels or annotations to the K8s yamls, conditionally changing the replica count for certain deployments based on fields in the yaml, changing the storage classes of certain PVCs after asking the user, etc.).

Move2Kube provides us a way to write the logic for these transformations in a Python-like language called [Starlark](https://docs.bazel.build/versions/2.1.0/skylark/language.html) . Same as the previous tutorials we provide this custom transformer to Move2Kube using the `--customizations` flag.


A `Starlark` class based transformer allows for writing a full fledged transformer in Starlark simply by implementing the `directory_detect` and `transform` functions.


Examples use-cases:
- [add-custom-files-directories-in-custom-locations](https://github.com/konveyor/move2kube-transformers/tree/main/add-custom-files-directories-in-custom-locations)
- [add-custom-kubernetes-annotation](https://github.com/konveyor/move2kube-transformers/tree/main/add-custom-kubernetes-annotation)
- [cloud-foundry-to-ce-iks-roks](https://github.com/konveyor/move2kube-transformers/tree/main/cloud-foundry-to-ce-iks-roks)
- [custom-default-transformer](https://github.com/konveyor/move2kube-transformers/tree/main/custom-default-transformer)
- [custom-m2kcollect-yaml-file-to-csv-file](https://github.com/konveyor/move2kube-transformers/tree/main/custom-m2kcollect-yaml-file-to-csv-file)

## Functions to Implement

* `directory_detect()` - It should be implemented to check if there are relevant files in the directory to invoke the transformer. See example [here](https://github.com/konveyor/move2kube-transformers/blob/0d6e555c85221493928cd1e899ffcb29ab69108a/add-custom-files-directories-in-custom-locations/customhelmchartgen.star#L17).
* `transform()` - It should be implemented to make changes on the input artifacts and output relevant artifacts for the rest of the transformation journey. See example [here](https://github.com/konveyor/move2kube-transformers/blob/0d6e555c85221493928cd1e899ffcb29ab69108a/add-custom-kubernetes-annotation/ingress-annotator.star#L18).

More details can be found [here](https://move2kube.konveyor.io/concepts/transformer#methods). 

## Built-in Functions

Move2Kube also makes available many functions, which can be used by transformer developers to interact with files, directories, parse json/yaml, etc.
We have listed a few relevant functions below, for more info see: [Built-in functions](https://move2kube.konveyor.io/transformers/external/starlark#in-built-functions)


| Function name in starlark | Purpose | Arguments | Returns |
| ----------- | ----------- | ----------- | ----------- | 
| `query` | Used to prompt Users for input. The function takes two arguments. | a dict of options and a string containing a validator function name | varies depending on the type of question |
| `exists`| Check if the provided file exists or not | filepath as string| boolean |
| `read`| Reads the given file | filepath as string | string |
| `read_dir`| Obtain list of files names in a directory | directory path as string | []string | 
| `is_dir`| Check if the given path is a directory or not | path as string | boolean | 
| `get_yamls_with_type_meta`| Return files by yaml kind | path to directory containing yamls as string <br/> kind of yamls required as string | []string| 
| `find_xml_path`| Return XML value based on the [XPath expression](https://www.w3schools.com/xml/xml_xpath.asp) in the given XML file | path to XML file as string <br/> XPath expression | []interface{}| 
| `get_files_with_pattern` | Get files in the given directory with a particular extension | path to directory as string <br/> list of extensions as []string | []string |
| `path_join` | Join two paths | first as path as string <br/> second path as string| string |
| `write` | Write data to a file and returns number of bytes written | path to a file as string <br/> data to be written as a string <br/> file permissions (optional) as an integer | int
| `path_base` | Return last element of the given path | path as string | string
| `path_rel` | Return relative path | base path as string <br/> target path as string | string

Example :

```python
isExists = fs.exists("path/to/the/file")
fileContent = fs.read("path/to/the/file")
ListOfFiles = fs.read_dir("path/to/the/directory")
isDirectory = fs.is_dir("path/to/the/directory")
yamlFiles = fs.get_yamls_with_type_meta("path/to/the/directory", "kind of resource")
XMLValue = fs.find_xml_path("path/to/XML/file", "XPath/expression")
FilesWithExt = fs.get_files_with_pattern("path/to/the/directory", ["txt", "yaml"])
path = fs.path_join("base/path", "/relative/path")
numberOfBytesWritten = fs.write("path/to/the/file", "some data", 777)
basePath = fs.path_base("some/path")
relativePath = fs.path_rel("base/path", "target/path")
```

## Global Variables
Move2Kube provides variables which expose values required by the transformer.

* `source_dir` - Exposes the source path in the environment.
* `context_dir` - Exposes the context path in the environment.
* `temp_dir` - Exposes the environment temporary path.
* `templates_reldir` - Exposes the relative path to the templates directory.
* `config` - Exposes the transformer meta data.
* `resources_dir` - Exposes the path to the resources directory.
* `output_dir` - Exposes the path to the output directory where the transformer output artifacts are produced.

## Steps

### Adding custom annotations to Kubernetes YAMLs

In this example, we illustrate how we could add an annotation specifying the ingress class to an Ingress yaml.


1. Let's start by going into the exercise directory. We will assume all commands are executed within this directory.
    ```console
    $ cd exercise-3-customizations/exercise-3b-starlark-based-transformers/
    ```

1. Lets use the [enterprise-app](https://github.com/konveyor/move2kube-demos/tree/main/samples/enterprise-app) as input.
    ```console
    $ ls ../../source/enterprise-app/
    README.md		config-utils		customers	docs			frontend		gateway			orders
    ```

1. Let's first run Move2Kube **without** any customizations. The output ingress does not have any annotations. We delete the output directory (`myproject/`) and files (`m2k-graph.json`, `m2kconfig.yaml`, `m2kqacache.yaml`) before moving on to the next step.
    ```console
    $ move2kube transform -s ../../source/enterprise-app/ --qa-skip && cat myproject/deploy/yamls/myproject-ingress.yaml && rm -rf myproject/ m2k*
    ```
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      creationTimestamp: null
      labels:
        move2kube.konveyor.io/service: myproject
      name: myproject
    ```


1. We will use the starlark based custom transformer given [here](https://github.com/konveyor/move2kube-transformers/tree/main/add-custom-kubernetes-annotation). We already have it in the `customizations` directory in this exercise.
    ```console
    $ ls customizations/add-custom-kubernetes-annotation/
    ```

1. Now lets transform with this customization. Specify the customization using the `--customizations` flag (short-hand `-c`). 
    ```console
    $ move2kube transform -s ../../source/enterprise-app/ -c customizations/ --qa-skip
    ```

Once the output is generated, we can observe from the below snippet of the ingress file (`myproject/deploy/yamls/myproject-ingress.yaml`) that there is an annotation for the ingress class added (`kubernetes.io/ingress.class: haproxy`):
```console
$ cat myproject/deploy/yamls/myproject-ingress.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: haproxy
  creationTimestamp: null
  labels:
    move2kube.konveyor.io/service: myproject
  name: myproject
```

## Anatomy of ingress annotator transformer

This custom transformer uses a configuration yaml (`ingress-annotator.yaml`) and a starlark script (`ingress-annotator.star`) which adds the annotation to the ingress yaml. The contents of custom transformer are as shown below:
  ```console
  $ ls customizations/add-custom-kubernetes-annotation/
  ingress-annotator.star  ingress-annotator.yaml
  ```
The configuration yaml specifies that the custom transformer consumes and produces kubernetes yaml artifact type as shown in the `consumes` and `produces` section.
  ```console
  $ cat customizations/add-custom-kubernetes-annotation/ingress-annotator.yaml
  ```
  ```yaml
  apiVersion: move2kube.konveyor.io/v1alpha1
  kind: Transformer
  metadata:
    name: IngressAnnotator
    labels: 
      move2kube.konveyor.io/built-in: false
  spec:
    class: "Starlark"
    consumes:
      KubernetesYamls: 
        merge: false
        # Ensures an artifact of this type gets immediately intercepted by this transformer
        mode: "MandatoryPassThrough" 
    produces:
      KubernetesYamls:
        disabled: false
    config:
      starFile: "ingress-annotator.star"
  ```

The code of the starlark script is shown below. At a high-level, the code requires only the `transform()` function as it acts upon any kubernetes yaml generated within Move2Kube. The `transform()` function loops through every yaml generated for every detected service, checks whether it is an ingress yaml, and if so, adds the annotation. The path mappings are meant to persist these changes.
```console
$ cat customizations/add-custom-kubernetes-annotation/ingress-annotator.star
```
```python
def transform(new_artifacts, old_artifacts):
    pathMappings = []
    artifacts = []

    for a in new_artifacts:
        yamlsPath = a["paths"]["KubernetesYamls"][0]
        serviceName = a["name"]
        artifacts.append(a)

        fileList = fs.read_dir(yamlsPath)
        yamlsBasePath = yamlsPath.split("/")[-1]
        # Create a custom path template for the service, whose values gets filled and can be used in other pathmappings
        pathTemplateName = serviceName + yamlsBasePath
        pathTemplateName = pathTemplateName.replace("-", "")
        tplPathData = {'PathTemplateName': pathTemplateName}
        pathMappings.append({'type': 'PathTemplate', \
                            'sourcePath': "{{ OutputRel \"" + yamlsPath + "\" }}", \
                            'templateConfig': tplPathData})
        for f in fileList:
            filePath = fs.path_join(yamlsPath, f)
            s = fs.read(filePath)
            yamlData = yaml.loads(s)
            if yamlData['kind'] != 'Ingress':
                continue
            if 'annotations' not in yamlData['metadata']:
                yamlData['metadata']['annotations'] = {'kubernetes.io/ingress.class': 'haproxy'}
            else:
                yamlData['metadata']['annotations']['kubernetes.io/ingress.class'] = 'haproxy'
            s = yaml.dumps(yamlData)
            fs.write(filePath, s)
            pathMappings.append({'type': 'Default', \
                    'sourcePath': yamlsPath, \
                    'destinationPath': "{{ ." + pathTemplateName + " }}"})

    return {'pathMappings': pathMappings, 'artifacts': artifacts}
```

The above steps can be replicated in the UI, by uploading the zip of the custom transformer as an input of type `customization`.
For convenience we have provided the `customizations.zip` file in this exercise directory.

For more detailed info see the full tutorial here - https://move2kube.konveyor.io/tutorials/customizing-the-output/custom-annotations
