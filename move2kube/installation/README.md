# Move2Kube Installation

## Installing Move2Kube CLI

```console
$ MOVE2KUBE_TAG='v0.3.4' bash <(curl https://raw.githubusercontent.com/konveyor/move2kube/main/scripts/install.sh)
```

## Running Move2Kube UI

```console
$ docker run --rm -it -p 8080:8080 quay.io/konveyor/move2kube-ui:v0.3.4
```

After executing the above command, the Move2Kube web interface can be accessed at `http://localhost:8080/`.

For more details please refer- https://move2kube.konveyor.io/installation.
