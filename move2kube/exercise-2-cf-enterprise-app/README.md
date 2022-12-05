# Migrate the Cloud Foundry enterprise-app sample to Minikube/Kubernetes using Move2Kube

## Steps for Move2Kube CLI/UI

1. Run the transform phase directly and answer the questions that Move2Kube asks.

    ```console
    cd exercise-2--cf-enterprise-app
    ```

    ```console
    $ move2kube transform -s ../source/enterprise-app -f config.yaml
    ```

    If you are using Minikube cluster, enter `localhost` for ingress host domain.

    If you want to quickly try out the deployment step, then you can use the images that we have already built and pushed to our registry. For this you will need to enter `quay.io` as the target image registry and `move2kube` as the namespace during the transform phase. Then you can directly move to the step 5.

    You can also run the transform phase without using the [config.yaml](./config.yaml) file if you want to go through all the questions that Move2Kube asks and use the [qacache.yaml](./qacache.yaml) file as reference to answer some of the questions.

    ```console
    $ move2kube transform -s ../source/enterprise-app
    ```

     If you are using the Move2Kube UI, create a new workspace and create a new project inside the workspace. Now, keep the the `Project input type` as `Source folder` and upload the [enterprise-app.zip](../source/enterprise-app.zip) file. Then, change the `Project input type` to `Config file` and upload the [config.yaml](./config.yaml) file (optional). Click on the `Start Planning button`. After the planning completes, click on the `Start Transformation` button.

2. Build the images using the `buildimages.sh` script inside `myproject/scripts` folder.

    ```console
    $ cd myproject/scripts
    ```

    ```console
    $ ./buildimages.sh
    ```

3. Push the images to your image registry using the `pushimages.sh` script.

    ```console
    $ ./pushimages.sh
    ```

4. Deploy the applications to your Minikube/Kubernetes cluster using the yamls inside `myproject/deploy/yamls` folder.

    ```console
    $ cd ../deploy/yamls
    ```

    ```console
    $ kubectl apply -f .
    ```

For more detailed instructions, you can check the full tutorial here - https://move2kube.konveyor.io/tutorials/migration-workflow.
