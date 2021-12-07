# How to deploy locally a cluster Kubernetes on macOS
---
We will deploy a simple Kubernetes cluster with the Kubernetes dashboard. 
You need to have Docker Desktop and `kubectl` installed.

Make sure you have enabled the Kubernetes cluster option in Docker Desktop. You can check if you have pods running with `kubectl get po -A`.
You should get something like:
```
kube-system            coredns-78fcd69978-8ztf6                    1/1     Running   0             119m
kube-system            coredns-78fcd69978-ct6c2                    1/1     Running   0             119m
kube-system            etcd-docker-desktop                         1/1     Running   0             119m
kube-system            kube-apiserver-docker-desktop               1/1     Running   0             119m
kube-system            kube-controller-manager-docker-desktop      1/1     Running   0             119m
kube-system            kube-proxy-mqhdg                            1/1     Running   0             119m
kube-system            kube-scheduler-docker-desktop               1/1     Running   0             119m
kube-system            storage-provisioner                         1/1     Running   0             119m
kube-system            vpnkit-controller                           1/1     Running   7 (23m ago)   119m
```


## Hello World deployment
We will first deploy a simple webserver:

```
kubectl expose deployment hello --type=NodePort --port=8080
```

```
kubectl create deployment hello --image=k8s.gcr.io/echoserver:1.4
```

## Dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```
In order to skip the login (for educational purposes) we have to patch our deployement:
```
kubectl patch deployment kubernetes-dashboard -n kubernetes-dashboard --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-skip-login"}]'
```

## Dashbaord metrics

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.2/components.yaml
```
We also have to patch this one so we can fetch metrics from Docker Desktop
```
kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```

### Cluster Role

Finally, we patch the roles to give the correct accesses to the dashboard (note you can find the file `cluster-roles.yaml` in this repo):

```
kubectl apply -f cluster-roles.yaml
```

You can finally run `kubectl proxy` and go to [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)
