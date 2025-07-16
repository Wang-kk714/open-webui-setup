## Nvidia container toolkit

[Docker desktop didn't support this toolkit](https://forums.docker.com/t/cant-start-containers-with-gpu-access-on-linux-mint/144606/3)

[Install Nvidia container toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)


- Validate

```

docker run --rm --gpus all nvidia/cuda:12.9.1-cudnn-devel-ubi8 nvidia-smi

```


## Minikube

### Minikube Invidia addon

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

### Minikube Ingress addon
```
minikube addons enable ingress
```

## Helm

### openwebui deploy -> Helm chart

[openwebui docker start](https://docs.openwebui.com/#quick-start-with-docker-)

```
helm create open-webui

helm install open-webui ./open-webui --debug --dry-run

helm install open-webui ./open-webui --debug --namespace open-webui --create-namespace
```

### NOTES

Currently ingress-nginx exposed with Nodeport, the ingress host url should add port.
```
{{- $svc := (lookup "v1" "Service" "ingress-nginx" "ingress-nginx-controller") }}
      {{- if eq $svc.spec.type "NodePort" }}
        http://{{ $host.host }}:{{ index $svc.spec.ports 0 "nodePort" }}{{ .path }}
      {{- else }}
```

## LoadBalancer Problem

### original ingress-controller + minikube tunnel

- [ingress-controller](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)

Minikube addon will create Nodeport type ingress-controller service while installing it with helm, the service type will be LoadBalancer.

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

As minikube running docker(default setting), munikube tunnel should be setup to enbale LoadBalancer service creating. 

[Accessing Apps](https://minikube.sigs.k8s.io/docs/handbook/accessing/), Minikube didn't support tunnel in background as so far [issue](https://github.com/kubernetes/minikube/issues/3647)

```
minikube tunnel

Status:
        machine: minikube
        pid: 1952963
        route: 10.96.0.0/12 -> 192.168.49.2
        minikube: Running
        services: [ingress-nginx-controller]
    errors: 
                minikube: no errors
                router: no errors
                loadbalancer emulator: no errors
```
 
 ```
 kubectl get svc -n ingress-nginx
 NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.107.255.200   10.107.255.200   80:31624/TCP,443:32298/TCP   25m
 ```

 External ip not changing to the `minikube ip`

### TODO: Try https://metallb.io/
