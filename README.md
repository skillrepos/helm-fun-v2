# Helm Fundamentals - lab setup

These instructions will guide you through configuring a GitHub Codespaces environment that you can use to run the course labs.

These steps **must** be completed prior to starting the actual labs.

## Step 1. To create your working environment for the labs, create a codespace by clicking on the button below:

Click on this button ⬇️
<br/><br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/skillrepos/helm-fun-v2?quickstart=1)

<br/><br/>
Then click on the option to create a new codespace.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Creating new codespace from button](./images/helmfun20.png?raw=true "Creating new codespace from button")

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**This will run for several minutes while it gets everything ready.**
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- After the codespace has initialized there will be a terminal present.

<br/><br/>

## Step 2. Start the Kubernetes cluster and complete setup

2. Setup an alias and run the setup command in the codespace's terminal (**This will take several minutes to run...**):

      ```
      alias k=kubectl
      minikube start
      ```

    - The output should look similar to the following.

```console
😄  minikube v1.31.2 on Ubuntu 20.04 (docker/amd64)
✨  Automatically selected the docker driver. Other choices: none, ssh
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.27.4 preload ...
    > preloaded-images-k8s-v18-v1...:  393.21 MiB / 393.21 MiB  100.00% 206.91 
    > gcr.io/k8s-minikube/kicbase...:  447.62 MiB / 447.62 MiB  100.00% 56.57 M
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```


## Step 3. Open Labs

3. After the codespace has started, open the labs document by going to the file tree on the left, find the file named **codespace-labs.md**, right-click on it, and open it with the **Preview** option.)

![Labs doc preview in codespace](./images/helmfun4.png?raw=true "Labs doc preview in codespace")

This will open it up in a tab above your terminal. Then you can follow along with the steps in the labs. 
Any command in the gray boxes is either code intended to be run in the console or code to be updated in a file.
