# Flask image, registry and minikube

## Instructions

### SETUP - Follow minikube installation instructions

* [Minikube get started](https://minikube.sigs.k8s.io/docs/start/)

In this assignment you will build and push a docker image with a Python flask server. You will then install minikube and deploy a pod in your own local kubernetes cluster.

### STEP 1 - Code your flask app

1. In your github account setup a devops_cicd project `https://github.com/your_username/devops_cicd`
2. Setup a `.gitignore` from a github official python template
3. Create a hello world python flask application (copying it from the previous lesson).
4. Create a file `requirements.txt` containing:

    ```text
    Flask==2.2.2
    ```

5. Test to start your from the root project folder:

    ```bash
    FLASK_APP=hello_flask flask run
    ```

6. Create a file `.dockerignore` containing:

    ```text
    .git
    .vscode
    .gitignore
    ```

7. Create a `Dockerfile`
    1. Use the image `python:3.11-slim`
    2. Set `WORKDIR` app
    3. `COPY` requirements.txt to your workdir
    4. `RUN` pip3 install -r requirements.txt
    5. `COPY` the rest of the files after pip install, this is a trick that will cache your build more often.
    6. `EXPOSE` 5000
    7. Set the `ENV` FLASK_APP hello_flask
    8. At last add a `CMD` [ "flask", "run", "--host=0.0.0.0"]

8. Build and test your docker image!

### STEP 2 - Push image to GitHub Container Registry

1. Build, tag for the image `my_flask`, remember to put the full url starting with `ghcr.io`. Use the format `ghcr.io/your_username/devops_cicd/my_flask`
2. Push the docker flask image to your GitHub Container Registry
3. Verify that the images is uploaded in your GitHub account
4. Run your image in Docker and test it in your browser

    ```bash
    docker run -P -d ghcr.io/your_username/devops_cicd/my_flask

    # Print the port of your newly created container
    docker port $(docker ps -ql)
    # Open the page in your browser 127.0.0.1:<port>

    # Make sure to remove the container after the test
    docker rm --force container_id
    ```

### STEP 3 - Configure GitHub secret in minikube

1. Setup a GitHub token for the registry, note it as minikube
2. The next step is to add a docker registry secret to your minikube kubernetes cluster, allowing it to pull images from your GitHub. Follow the guide [pull image from private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)

    ```bash

    # NOTE typing credentials in the terminal is not good, since the access token can be retrieved e.g from history, the guide has another method "Create a Secret based on existing Docker credentials" that could be used on some setups. You can also set a space before the command.

    kubectl create secret docker-registry regcred \
        --docker-server=ghcr.io \
        --docker-username=<your-name> \ # Enter username here
        --docker-password=<your-pword> \ # Enter access token here
        --docker-email=<your-email> # Enter email here


    # You can inspect the created secret with
    kubectl get secret regcred --output=yaml

    # You can also decode and view the secrets data with
    kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
    ```

3. Go to minikube dashboard in your browser and take a screenshot of the created secret.

### STEP 4 - Deploy a flask pod to your minikube

1. Modify the `create_pod.yaml` file image to use your `container registry`
2. Execute `kubectl apply -f` on the modified file
3. Make sure the pod is `running` with `kubectl get pods -l "app=my_flask"`
4. Create a service to expose the pod `kubectl expose pod flask-server-pod --selector "app=my_flask" --type=LoadBalancer --port=5000`
5. Make sure the service was created with `kubectl get services flask-server-pod`
6. Use the minikube command `minikube service flask-server-pod` to access the service in a browser. If this doesn't work you can use `kubectl port-forward service/flask-server-pod 32312:5000` and manually open your browser with url `http://127.0.0.1:32312`
7. Take a screenshot of the terminal output showing the tunnel and the browser successfully loading the page.

### Cleanup

If you want to delete the service and pod after you completed the hand in.

```bash
# To delete the pod
kubectl delete pod flask-server-pod

# To delete the service
kubectl delete service flask-server-pod
```

## Hand in instructions

* Push screenshots to this repo
