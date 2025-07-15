## Nvidia container toolkit

[Docker desktop didn't support this toolkit](https://forums.docker.com/t/cant-start-containers-with-gpu-access-on-linux-mint/144606/3)

[Install Nvidia container toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)


- Validate

```

docker run --rm --gpus all nvidia/cuda:12.9.1-cudnn-devel-ubi8 nvidia-smi

```


## Minikube

https://minikube.sigs.k8s.io/docs/tutorials/nvidia/


- validate
```
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: gpu-test
spec:
  restartPolicy: OnFailure
  containers:
  - name: ctr
    image: ubuntu:22.04
    command: ["nvidia-smi", "-L"]
    resources:
      limits:
        nvidia.com/gpu: 2
EOF
```

- [Accessing Apps](https://minikube.sigs.k8s.io/docs/handbook/accessing/)


## Helm

```
helm create open-webui

helm install open-webui ./open-webui --debug --dry-run

helm install open-webui ./open-webui --debug --namespace open-webui --create-namespace
```
