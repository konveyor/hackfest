# Migrate the apps inside language-platforms sample to Minikube/Kubernetes

## Steps for Move2Kube CLI/UI

1. Run the plan phase.

     ```console
    $ cd exercise-1-language-platforms
    ```

    ```console
    $ move2kube plan -s ../source/language-platforms
    ```

     If you are using the Move2Kube UI, create a new workspace and create a new project inside the workspace. Now, keep the the `Project input type` as `Source folder` and upload the [language-platforms.zip](../source/language-platforms.zip) file. Then, change the `Project input type` to `Config file` and upload the [config.yaml](./config.yaml) file (optional). Click on the `Start Planning button`.


2. Run the transform phase and answer the questions that Move2Kube asks.

    ```console
    $ move2kube transform -f config.yaml
    ```
    If you are using Minikube cluster, enter `localhost` for ingress host domain.

    For Move2Kube UI, click on the `Start Transformation` button.

    You can also run the transform phase without using the [config.yaml](./config.yaml) file if you want to go through all the questions that Move2Kube asks and use the [qacache.yaml](./qacache.yaml) file as reference to answer some of the questions.

    ```console
    $ move2kube transform
    ```

    If you want to quickly try out the deployment step, then you can use the images that we have already built and pushed to our registry. For this you will need to enter `quay.io` as the target image registry and `move2kube` as the namespace during the transform phase. Then you can directly move to the step 5.

3. Build the images using the `buildimages.sh` script inside `myproject/scripts` folder.

    ```console
    $ cd myproject/scripts
    ```

    ```console
    $ ./buildimages.sh
    ```

4. Push the images to your image registry using the `pushimages.sh` script.

    ```console
    $ ./pushimages.sh
    ```

5. Deploy the applications to your Minikube/Kubernetes cluster using the yamls inside `myproject/deploy/yamls` folder.

    ```console
    $ cd ../deploy/yamls
    ```

    ```console
    $ kubectl apply -f .
    ```

For more detailed instructions, you can check the full tutorials here - https://move2kube.konveyor.io/tutorials/cli and https://move2kube.konveyor.io/tutorials/ui.
