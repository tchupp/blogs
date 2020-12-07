# Kubernetes QoS classes

Every Pod in kubernetes will be assigned a QoS class:
- Best Effort
- Burstable
- Guaranteed

Kubernetes will assign the QoS class based on your resource specifications.

## Guaranteed

For a Pod to be given a QoS class of Guaranteed, every Container in the Pod must have a resource requests and limit; and they must be the same.

Below is an example of a pod that will receive a `Guaranteed` QoS class.  
```
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed-qos-demo
spec:
  containers:
  - name: guaranteed-qos-demo
    image: busybox
    resources:
      limits:
        memory: 250Mi
        cpu: 500m
#      requests:
#        memory: 250Mi
#        cpu: 500m
```
Technically, the `spec.containers[].resources.requests` definition is optional here. If you specify a limit but don't specify a request, kubernetes will automatically assign the requests to equal the limits.  
If we copy this YAML and run `pbpaste | kubectl apply -f -`, we can view the spec of the running Pod via `kubectl get pod guaranteed-qos-demo -o yaml`.  
Note the `status.qosClass`:
```
spec:
  containers:
    ...
    resources:
      limits:
        memory: 250Mi
        cpu: 500m
      requests:
        memory: 250Mi
        cpu: 500m
    ...
status:
  qosClass: Guaranteed
```

## Reference

https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/
https://medium.com/google-cloud/quality-of-service-class-qos-in-kubernetes-bb76a89eb2c6
