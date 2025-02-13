---
layout: post
title: "Deploying LLMs in Clusters #1: running “vLLM production-stack” on a cloud VM"
thumbnail-img: /assets/img/stack-thumbnail.png
share-img: /assets/img/stack-thumbnail.png
author: LMCache Team
image: /assets/img/stack-thumbnail.png
---
<br>


## TL;DR
- vLLM boasts the largest open-source community in LLM serving, and **“vLLM production-stack”** offers a vLLM-based full inference stack with **10x** better performance and Easy cluster **management**.
- Today, we will give a step-by-step demonstration on how to deploy a proof-of-concept **“vLLM production-stack”** in a cloud VM.
- This is the beginning of our **Deploying LLMs in Clusters** series. We will be rolling out more blogs about serving LLMs with your own infrastructure during the next few weeks. Let us know which topic we should do next! [[poll]](https://forms.gle/cSeqDz1Nm2iwv7eH8)
[[Github Link]](https://github.com/vllm-project/production-stack) | [[Tutorial Video]](https://www.youtube.com/watch?v=EsTJbQtzj0g&ab_channel=JunchenJiang) | [[More Tutorials]](https://github.com/vllm-project/production-stack/tree/main/tutorials) | [[Interest Form]](https://forms.gle/Jaq2UUFjgvuedRPV8)

<iframe width="500" height="315" src="(https://www.youtube.com/watch?v=EsTJbQtzj0g&ab_channel=JunchenJiang" frameborder="0" allowfullscreen></iframe>


# The Context
<!-- Over the past year, LLM inference has raced to the forefront, powering everything from chatbots to code assistants and beyond. It’s quickly becoming critical infrastructure, much like the cloud was to big data, cellular was to mobile apps, and CDNs were (and still are!) to the broader Internet. -->


**vLLM** has taken the open-source community by storm, with unparalleled hardware and model support plus an active ecosystem of top-notch contributors. But until now, vLLM has mostly focused on **single-node** deployments.

**vLLM Production-stack** is an open-source **reference implementation** of an **inference stack** built on top of vLLM, designed to run seamlessly on a cluster of GPU nodes. It adds four critical functionalities that complement vLLM’s native strengths.

Below is a quick snapshot comparing vLLM production-stack with its closest counterparts:
<div align="center">
<img src="/assets/img/stack-table.png" alt="Icon" style="width: 90%; vertical-align:middle;">
</div>

\
vLLM production-stack offers superior performance than other LLM serving solutions by achieving higher throughput through smart routing and KV cache sharing\:
<div align="center">
<img src="/assets/img/stack-ttft.png" alt="Icon" style="width: 60%; vertical-align:middle;">
</div>



# Deploying a "production-stack" demo in the cloud
In this section, we will go through the general steps to set up the vLLM production-stack service in the cloud. If you prefer watching videos, please follow this [link](https://www.youtube.com/watch?v=EsTJbQtzj0g&ab_channel=JunchenJiang).
## Step 1: Prepare the machine
### Install Kubernetes
1. Clone the repository and navigate to the `utils/` folder:

   ```bash
   git clone https://github.com/vllm-project/production-stack.git
   cd production-stack/utils
   ```

2. Execute the script `install-kubectl.sh`:

   ```bash
   bash install-kubectl.sh
   ```

   This script downloads the latest version of `kubectl`, the Kubernetes command-line tool, and places it in your PATH for easy execution.

4. Expected Output:
   - Verification message using:

     ```bash
     kubectl version --client
     ```

   Example output:

   ```plaintext
   Client Version: v1.32.1
   ```
### Install Helm 
1. Execute the script `install-helm.sh`:

   ```bash
   bash install-helm.sh
   ```
This script downloads and installs Helm and places the Helm binary in your PATH. Helm is a package manager for Kubernetes and simplifies the deployment process

3. Expected Output:
   - Verification message using:

     ```bash
     helm version
     ```

   Example output:

   ```plaintext
   version.BuildInfo{Version:"v3.17.0", GitCommit:"301108edc7ac2a8ba79e4ebf5701b0b6ce6a31e4", GitTreeState:"clean", GoVersion:"go1.23.4"}
   ```

### Step 3: Install Minikube

1. Execute the script `install-minikube-cluster.sh`:

   ```bash
   bash install-minikube-cluster.sh
   ```
This script installs Minikube. Minikube configures the system to support GPU workloads by enabling the NVIDIA Container Toolkit and starting Minikube with GPU support. 

2. Expected Output:

   ```plaintext
   😄  minikube v1.35.0 on Ubuntu 22.04 (kvm/amd64)
   ❗  minikube skips various validations when --force is supplied; this may lead to unexpected behavior
   ✨  Using the docker driver based on user configuration
   ......
   ......
   🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
   "nvidia" has been added to your repositories
   Hang tight while we grab the latest from your chart repositories...
   ......
   ......
   NAME: gpu-operator-1737507918
   LAST DEPLOYED: Wed Jan 22 01:05:21 2025
   NAMESPACE: gpu-operator
   STATUS: deployed
   REVISION: 1
   TEST SUITE: None
   ```


## Step 2: Deploy the production-stack
Now you have everything needed, it is time to deploy production-stack! 

First create a yaml file as shown below. Be sure to include your model name, model url, replica-count, and vLLM configurations in the file. Note that the pvcStorage needs be be bigger than the model size.

### example.yaml
```yaml
servingEngineSpec:
  runtimeClassName: ""
  modelSpec:
  - name: "opt125m"
    repository: "vllm/vllm-openai"
    tag: "latest"
    modelURL: "facebook/opt-125m"

    replicaCount: 2

    requestCPU: 6
    requestMemory: "16Gi"
    requestGPU: 0.5

    pvcStorage: "10Gi"
    pvcAccessMode:
      - ReadWriteMany

    vllmConfig:
      maxModelLen: 1024
      extraArgs: ["--disable-log-requests", "--gpu-memory-utilization", "0.4"]
```

And deploy this configuration using Helm:

```bash
helm repo add vllm https://vllm-project.github.io/production-stack
helm install vllm vllm/vllm-stack -f example.yaml
```
Monitor the deployment status using:

```bash
sudo kubectl get pods
```

Expected output:

- Pods for the `vllm` deployment should transition to `Ready` and the `Running` state.

```plaintext
NAME                                               READY   STATUS    RESTARTS   AGE
vllm-deployment-router-859d8fb668-2x2b7        1/1     Running   0          2m38s
vllm-opt125m-deployment-vllm-84dfc9bd7-vb9bs   1/1     Running   0          2m38s
```

_Note_: It may take some time for the vLLM instance to become ready!

## Step 3: Send requests and test!  
#### 3.1: Forward the Service Port

Expose the `vllm-router-service` port to the host machine:

```bash
sudo kubectl port-forward svc/vllm-router-service 30080:80
```

#### 3.2: Query the OpenAI-Compatible API to list the available models

Test the stack's OpenAI-compatible API by querying the available models:

```bash
curl -o- http://localhost:30080/models
```

Expected output:

```json
{
  "object": "list",
  "data": [
    {
      "id": "facebook/opt-125m",
      "object": "model",
      "created": 1737428424,
      "owned_by": "vllm",
      "root": null
    }
  ]
}
```

#### 3.3: Query the OpenAI Completion Endpoint

Send a query to the OpenAI `/completion` endpoint to generate a completion for a prompt:

```bash
curl -X POST http://localhost:30080/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "facebook/opt-125m",
    "prompt": "Once upon a time,",
    "max_tokens": 10
  }'
```

Expected output:

```json
{
  "id": "completion-id",
  "object": "text_completion",
  "created": 1737428424,
  "model": "facebook/opt-125m",
  "choices": [
    {
      "text": " there was a brave knight who...",
      "index": 0,
      "finish_reason": "length"
    }
  ]
}
```

### Step 4: Uninstall

To remove the deployment, run:

```bash
sudo helm uninstall vllm
```

## Conclusion
We have demonstrated how to set up a **vLLM Production Stack** with a GPU VM. 

This is the first episode of our **Deploy LLMs in Clusters** Series. Stay tuned for multi-node deployment on Amazon EKS, serving multiple models with one cluster, LLM router deep dive, and many more! Fill this one question [poll](https://forms.gle/cSeqDz1Nm2iwv7eH8) to let us know which one we should do next!


Join us to build a future where every application can harness the power of LLM inference—reliably, at scale, and without breaking a sweat.
*Happy deploying!*

Contacts:
- **Github: [https://github.com/vllm-project/production-stack](https://github.com/vllm-project/production-stack)**
- **Chat with the Developers** **[Interest Form](https://forms.gle/mQfQDUXbKfp2St1z7)**
- **vLLM [slack](https://slack.vllm.ai/)**
- **LMCache [slack](https://join.slack.com/t/lmcacheworkspace/shared_invite/zt-2viziwhue-5Amprc9k5hcIdXT7XevTaQ)**
