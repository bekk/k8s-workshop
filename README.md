# Kubernetes workshop

## Getting started

### Required tools

We'll use the following tools:

* `kubectl`
* `kubens` (from the `kubectx` package)
* `gcloud`

For macOS, you can use `brew install kubectl kubectx gcloud`.

After installing the tools, we'll need to install some additional plugins to `gcloud`: 

```sh
gcloud components install gke-gcloud-auth-plugin
```

### GUI/TUI:

Using a GUI or TUI is quicker than running commands.

* Monokle is a free GUI app that can display cluster resources and edit YAML manifests: https://monokle.io/download.
* k9s is a TUI for viewing and performing operations on cluster resources



### Autocompletion

Autocompletion simplifies writing the commands in this workshop.

The commands below show how to add it to the current shell, you will have to run it for every new shell opened. For a permanent setup, consult `kubectl completion -h`.

**Bash:**
```sh
source <(kubectl completion bash)
```

**ZSH:**
```sh
source <(kubectl completion zsh)
```

**Fish:**
```sh
kubectl completion fish | source
```

**Powershell:**
```sh
kubectl completion powershell | Out-String | Invoke-Expression
```

### Logging in

1. Go to [the Google Cloud Console](https://console.cloud.google.com/) and login with your account.

2. Run `gcloud auth login` and follow the steps in the browser to login the `gcloud` CLI.

3. Add the Google Kubernetes Engine (GKE) cluster context to the `kubectl` config:

    ```sh
    gcloud container clusters get-credentials cloudlabs-25 --region=europe-north1
    ```

4. Confirm successful `kubectl` setup by running `kubectl get namespaces`.

5. An online UI is available at [kube-dashboard.cloudlabs-gcp.no](https://kube-dashboard.cloudlabs-gcp.no). It requires an authentication token. Run `gcloud auth print-access-token` to generate a token, and copy it to online UI to log in.


## Your first pod - the imperative way

We'll create a namespace and then deploy a simple app and connect to it, all using `kubectl`.

1. Create a namespace: `kubectl create namespace <namespace-name>`.

    `<namespace-name>` should be lowercase with words separated by dashes, e.g., `my-name`.

    Run `kubectl get namespace` to verify that your namespace is in the list.


2. Let's start by creating a single pod that runs `podinfo`:
    
    `kubectl run --namespace <namespace-name> <pod-name> --image=stefanprodan/podinfo --port=80`
    
    You will get a warning about resources, ignore the warning for now.

> [!TIP]
> Use `kubectl <subcommand> --help` to get help on any `kubectl` subcommand. E.g., `kubectl run --help`.


3. Let's look at the app created. Run `kubectl get --namespace <namespace-name> pods`. You should get output looking like:

    ```text
    NAME                READY   UP-TO-DATE   AVAILABLE   AGE
    <deployment-name>   2/2     2            2           2m34s
    ```

    You can view the app information using `kubectl describe --namespace <namespace-name> pod <pod-name>`.

> [!TIP] 
> Most `kubectl` commands support abbreviations of the types. For example, `kubectl get pods` can be shortened to `kubectl get po`.
>
> Similarly, flags can be abbreviated. `--namespace` can be shortened to `-n`, and `--watch` can be shortened to `-w`. So, `kubectl get --namespace <namespace-name> pods` can be written as `kubectl get -n <namespace-name> po`.
>
> Feel free to use these abbreviations to save time.

4. Let's change the container image running in the pod to `nginx`.

    * Run `kubectl edit --namespace <namespace-name> pod <pod-name>`. This will open the pod manifest in your default text editor.
    * Find the line with `image: stefanprodan/podinfo` and change it to `image: nginx`. Save and close the editor.
    * Open a new shell, and run `kubectl get --namespace <namespace-name> pods --watch`. This will show all pods in the namespace, and the `--watch` flag also print new lines for every update.
    * Observe the output from the watch command in the second shell. You should see that the pod is terminated and a new pod is created with the updated image.

5. To access the pod and verify nginx is running, we'll do a port-forwarding from our local computer to the cluster. In your shell, run:

    ```sh
    kubectl --namespace <namespace-name> port-forward <pod-name> 8080:80
    ```

    This will create a tunnel from your local computer on port 8080, to the pod in the cluster on port `80`. Open [localhost:8080](http://localhost:8080) in your browser to confirm that the app is running smoothly.

    Use `CTRL-C` or equivalent to cancel the port-forwarding.


6. Continue watching the pod updates in the second shell, and run `kubectl delete --namespace <namespace-name> pod <pod-name>`.

7. Delete the namespace by running `kubectl delete ns <namespace-name>`.



## Your first pod - the declarative way

Running commands to spin up pods is not ideal. We want to use code to specify our resources.


1. Create a folder called, `podinfo`, and put the configuration you create in the following tasks in this folder.

    For most resource creation commands, you can add `--dry-run=client -o yaml` to generate the corresponding YAML configuration. The configuration can then be applied by running `kubectl apply`. E.g., to create a namespace like before, run `kubectl create namespace <namespace-name> --dry-run=client -o yaml`. Then copy the configuration into a code editor, into a file called `podinfo/namespace.yaml`. 

> [!TIP]
> If you're running commands in Bash/Zsh or a similarly compatible you can pipe the configuration directly into a file using `>`.
> E.g., `kubectl create namespace <namespace-name> --dry-run=client -o yaml > podinfo/namespace.yaml`.

2. Run `kubectl apply -R -f podinfo/` to apply all manifests in the `podinfo/` folder. Verify that the namespace is created, like before.


> [!TIP]
> We've specified the namespace flag for every command so far. We can simplify the commands by using `kubens`.
> 
> Run `kubens <namespace-name>`. From now on, you can write commands without `--namespace <namespace-name>`. Commands in the workshop will explicitly use the argument, even if it's not needed.

3. Similarly, create a manifest for a pod running `nginx` in `podinfo/pod.yaml`, using `kubectl run --namespace <namespace-name> <pod-name> --image=nginx --port=80 --dry-run=client -o yaml`. Apply the manifests using `kubectl apply -R -f podinfo/` after creating it.

4. Verify that the pod is created and in `Running` state in the correct namespace.

## Exploring TUIs/GUIs

The following two tasks explore the Monokle GUI and k9s TUI. You can choose to do both, or the one you prefer. Monokle includes more GUI-like features, like creating and editing manifests, while k9s is more focused on viewing and managing cluster resources.

We will edit a file inside the running container, and port-forward to view the changes.

### Monokle

1. Open Monokle, and add a new workspace. Select the `podinfo/` folder you created earlier as the workspace folder. Monokle will report some warnings about missing security settings, limits and more, ignore these.

2. Connect Monokle to your cluster by clicking the `Connect to Cluster` button in the top right. Select the correct context for your cluster. Select your namespace to view resources in your namespace.

3. Looking at the "Cluster Dashboard". The left side menu shows the workloads and other resources. Click on "Pods" to view the pod you created earlier. Click on the pod. You can see the pod status, logs, and other information in the side panel.

4. Enter the "Shell" tab in the side panel, and execute the command `echo "Hello Monokle world" > /usr/share/nginx/html/index.html` to change the default nginx page.

5. Monokle does not yet support port-forwarding, so run the command: `kubectl port-forward --namespace <namespace-name> <pod-name> 8080:80` in your terminal.

6. Verify that the text of the page is correct by running `curl localhost:8080` or navigating to `http://localhost:8080` in your browser.

7. Stop the port-forwarding by pressing `CTRL-C` in your terminal.

8. If you prefer, you can use this GUI to edit manifests from now on.

### k9s

1. Open k9s by running `k9s` in your terminal.

> [!TIP]
> Use the command `:help` or `?` in k9s to get help on keybindings.

2. Press `:` to start command mode, and type `ns` to list namespaces. Select your namespace by navigating to it and pressing `ENTER`. You can navigate using arrow keys or vim-style keybindings (`j` or `k`) You can also switch namespaces by pressing `0` (zero) and selecting the namespace, or by using the command (after `:`) `ns <namespace-name>`.

3. By default, k9s shows pods. You should see the pod you created earlier. Show the state of the pod by pressing `d` ("describe"). Go back by pressing `ESC`.

4. Press `s` to open a shell into the container. Run the command `echo "Hello k9s world" > /usr/share/nginx/html/index.html` to change the default nginx page. Exit the shell by typing `exit` and pressing `ENTER`.

5. With the pod still selected, press `Shift-f` to start port-forwarding. Use `80` as the container port, but change the local port to `8080`. Press `ENTER` to start port-forwarding.

6. Verify that the text of the page is correct by running `curl localhost:8080` or navigating to `http://localhost:8080` in your browser.

7. Stop the port-forwarding in k9s by pressing `f` to display port-forwards. Select the port-forward and delete it using `ctrl-d`.

## Deployments

Deployments are a higher-level abstraction that manages Pods. They ensure that a specified number of replicas are running and handle updates to the configuration in predictable ways.

1. Generate a deployment manifest:

    `kubectl create deployment --namespace <namespace-name> <deployment-name> --image=stefanprodan/podinfo --replicas=2 --port=80 --dry-run=client -o yaml > podinfo/deployment.yaml`

2. Apply the manifest like before: `kubectl apply -R -f podinfo/`

3. Verify using either the CLI, or Monokle/k9s. You should see 1 deployment and 2 pods running. The pods have names prefixed with the deployment name.
    * CLI: `kubectl get deployments --namespace <namespace-name>` and `kubectl get pods --namespace <namespace-name>`. 
    * k9s: Go to the namespace using `:ns`. View deployments using `:deployments` or `:deploy` to view deployments. View pods using `:pods` or `:po`.
    * Monokle: Select the namespace, and view workloads > deployments and workloads > pods

4. Describe the deployment using either CLI, or Monokle/k9s. Note the port configuration.

    * CLI: `kubectl describe deployment <deployment-name> --namespace <namespace-name>`
    * k9s: Select the deployment and press `d` to describe.
    * Monokle: Select the deployment to view its details in the side panel.

5. We used `--port=80` in the command to create the deployment, and in the previous step you should have confirmed that the `port` is configured to be `80`. However, this port is not exposed by the container. This is documented in the app documentation, but we have no way of knowing otherwise. The correct port is `9898`.

6. Let's edit the deployment.

    * CLI: Run `kubectl edit --namespace <namespace-name> deployment <deployment-name>`. Find the line with `containerPort: 80` and edit the port number to `9898`. Save and close, and observe the output from `kubectl get --namespace <namespace-name> pods --watch` (or in k9s/Monokle)
    * k9s: Select the deployment and press `e` to edit. Find the line with `containerPort: 80` and edit the port number to `9898`. Save and close. Navigate to the pods view to observe the pod updates.
    * Monokle: Select the deployment to view its details in the side panel. Click "Manifest" to view the manifest. Find the line with `containerPort: 80` and edit the port number to `9898`. Save the file by clicking "Update". Navigate to the pods view to observe the pod updates.

7. We could also have edited the manifest file `podinfo/deployment.yaml` directly, and reapplied it using `kubectl apply -R -f podinfo/`. This is the preferred way of managing resources, as it allows for version control and easier collaboration. Update the manifest to make sure it's not out of sync.


## Services

Pods are ephemeral; if they crash or are rescheduled, their IP addresses change. Services provide a stable IP and DNS name to access a set of Pods.

1. Generate a service manifest:

    `kubectl expose deployment --namespace <namespace-name> <deployment-name> --name=<service-name> --port=80 --target-port=9898 --type=ClusterIP --dry-run=client -o yaml > podinfo/service.yaml`

2. Apply it: `kubectl apply -f podinfo/service.yaml`

3. Verify.
    * CLI: `kubectl get services --namespace <namespace-name>`.
    * k9s: `:services` or `:svc`
    * Monokle: View services under the Network subheading.

3. Use port-forwarding to access the service. In k9s, find the service and use `shift-f` like before. the For the CLI, use 

    ```sh
    kubectl port-forward --namespace <namespace-name> service/<service-name> 9898:80
    ```

> [!NOTE]
> Take a look at how traffic is routed when refreshing in the browser. Which pod answers the request?
> 
> The service normally load balances requests across all pods matching its selector. However, port-forwarding towards a service will always send traffic to the first pod in the list managed by the service.

4. Cancel the port-forwarding before proceeding.


## Externally exposed services

In GKE, there are multiple ways to expose services externally: LoadBalancers, Ingresses and Gateway API. We will first try to use LoadBalancers.

1. Update the service to be of type LoadBalancer. You can do this by editing the manifest file `podinfo/service.yaml`, changing `type: ClusterIP` to `type: LoadBalancer`. Then reapply the manifest.

2. Look at the updates to the service, and fetch the new field "EXTERNAL-IP". Visit `http://<your-ip>/` and see your app in action. Refresh multiple times to see different pods responding.


## Configuration and Resources

The following tasks are more open-ended. You will need to look up the documentation to find out how to solve them.

### Resources

By default, containers have no resource constraints. It is good practice to specify how much CPU and memory (RAM) each container needs.

1. Modify your `podinfo/deployment.yaml` to add resource requests and limits for the container.
    *   Set memory request to `64Mi` and limit to `128Mi`.
    *   Set CPU request to `10m`.

> [!TIP]
> See [Managing Resources for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for syntax and examples.

2. Apply the changes and verify that the pods are recreated with the new configuration. You can check this with `kubectl describe pod <pod-name>`.

### Environment Variables

You can inject configuration into your application using environment variables.

1. Modify `podinfo/deployment.yaml` to add an environment variable named `PODINFO_UI_COLOR` with the value `#ffffff` (or any other hex color you like).

    > [!TIP]
    > See [Define Environment Variables for a Container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/).

2. Apply the changes.
3. Visit the external IP of your service again. The UI header color should have changed.

### ConfigMaps

Hardcoding environment variables in the deployment manifest is not always ideal. ConfigMaps allow you to decouple configuration artifacts from image content.

1. Create a new file `podinfo/configmap.yaml`.
2. Define a ConfigMap named `podinfo-config` containing the data `ui-color: "#34577c"`.
3. Modify `podinfo/deployment.yaml` to load the `PODINFO_UI_COLOR` environment variable from the ConfigMap instead of defining the value directly.

    > [!TIP]
    > See [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data).

4. Apply both files (`kubectl apply -R -f podinfo/`).
5. Verify that the application still works and uses the configured color.

### Secrets

Secrets are similar to ConfigMaps but are intended to hold sensitive information.

1. Create a new file `podinfo/secret.yaml`.
2. Define a Secret named `podinfo-secret` containing a key `api-key` with some dummy value.
3. Modify `podinfo/deployment.yaml` to inject this secret as an environment variable named `PODINFO_API_KEY`.

    > [!TIP]
    > See [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables).

4. Apply the changes.
5. Verify the secret is injected by running `kubectl exec -it <pod-name> -- env | grep PODINFO_API_KEY`.

## Debugging and Observability

When things go wrong, you need tools to investigate.

### Logs

Logs are the first place to look when an application is behaving unexpectedly.

1. View logs for a pod: `kubectl logs <pod-name>`.
2. Follow logs in real-time: `kubectl logs -f <pod-name>`.
3. If a pod has multiple containers, specify the container name: `kubectl logs <pod-name> -c <container-name>`.

> [!TIP]
> In k9s, you can press `l` to view logs for a selected pod.

### Events

Kubernetes emits events when resources change state or when errors occur (e.g., scheduling failures, image pull errors).

1. List events in the namespace: `kubectl get events`.
2. Describe a resource to see events related to it: `kubectl describe pod <pod-name>`.

### Executing commands

Sometimes you need to poke around inside a running container.

1. Start an interactive shell: `kubectl exec -it <pod-name> -- /bin/sh` (or `/bin/bash`).
2. Check file system, environment variables, network connectivity, etc.

> [!NOTE]
> Not all containers can run a shell. Some minimal containers may not have a shell installed.

## Security

Security is a critical aspect of Kubernetes. By default, pods are quite permissive. Let's lock them down.

### Security Context

You can configure security settings for a Pod or Container using the `securityContext` field.

1. Modify `podinfo/deployment.yaml` to add a `securityContext` to the container.
2. Configure the following settings:
    *   **Read-only filesystem**: `readOnlyRootFilesystem: true`. This prevents the application from writing to the root filesystem.
    *   **Disallow privilege escalation**: `allowPrivilegeEscalation: false`. This prevents the process from gaining more privileges than its parent process.
    *   **Run as non-root**: `runAsNonRoot: true` and `runAsUser: 1000` (or another non-root UID). This ensures the container runs as a regular user.
    *   **Drop capabilities**: Drop `ALL` capabilities. This removes all default Linux capabilities from the container.

> [!TIP]
> See [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

3. Apply the changes.
4. Verify the changes by inspecting the pod: `kubectl get pod <pod-name> -o yaml`.
5. Try to write to the filesystem inside the container: `kubectl exec -it <pod-name> -- touch /tmp/test`. This should fail if the filesystem is read-only (note: you might need to mount a volume to `/tmp` if the app needs to write there, but for this exercise, seeing it fail is the goal).

## Cleanup

To cleanup the resources created in this workshop, run:

```sh
kubectl delete namespace <namespace-name>
```
