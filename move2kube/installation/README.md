# Move2Kube Installation

## Installing Move2Kube CLI

You can directly download the Move2Kube binary for your local machine matching your OS/arch from [here](https://github.com/konveyor/move2kube/releases/tag/v0.3.5).

To install the latest stable version of Move2Kube run the below command.

```console
$ MOVE2KUBE_TAG='v0.3.5' bash <(curl https://raw.githubusercontent.com/konveyor/move2kube/main/scripts/install.sh)
```

## Running Move2Kube UI

```console
$ docker run --rm -it -p 8080:8080 quay.io/konveyor/move2kube-ui:v0.3.5
```

After executing the above command, the Move2Kube web interface can be accessed at `http://localhost:8080/`.

For more details please refer- https://move2kube.konveyor.io/installation.
