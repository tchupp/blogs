# Deleting Kubernetes Deployments Without Downtime

## Background

If you've ever tried to change the labels on your Kubernetes Deployment, you may have encountered this error:
```
The Deployment "my-app" is invalid: spec.selector:
Invalid value: 
  v1.LabelSelector{
    MatchLabels: map[string]string{"app":"my-app", "chart":"my-chart", "type":"app"}, 
    MatchExpressions: []v1.LabelSelectorRequirement(nil)
  }
  : field is immutable
```

You can find this field in the Deployment's yaml here:
```yaml
$ kubectl get deploy my-deployment

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
 ...
spec:
  selector:
    matchLabels:
      app: my-app
      chart: my-chart
      type: app
  ...
```

Since Kubernetes Deployment's label selectors are immutable, the only way to change this is to delete and re-create the Deployment. Anyone who has deleted a Deployment may know that, by default, the Pods created by the Deployment are also deleted. This isn't great news as it can cause downtime, and makes this opperation seem very difficult to pull off.

Fortunately, there is a way to do this without causing downtime!

## Explanation

While there are multiple ways to create Pods in Kubernetes, let's specifically focus on Pods related to Deployments.

You might already be familiar with Kubernetes Deployments and Pods. When you create a Deployment, you end up with similarly named Pods:
```bash
$ kubectl get deploy -l app=my-app
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
my-app   2/2     2            2           2h

$ kubectl get pods -l app=my-app
NAME                      READY   STATUS    RESTARTS   AGE
my-app-776b864c75-vk87p   1/1     Running   0          5h3m
my-app-776b864c75-whbqk   1/1     Running   0          5h2m
```

If you were to scale the Deployment up to 3, you would see an additional Pod start up:
```bash
$ kubectl scale deploy/my-app --replicas=3
deployment.extensions/my-app scaled

$ kubectl get pods -l app=my-app
NAME                      READY   STATUS    RESTARTS   AGE
my-app-776b864c75-vk87p   1/1     Running   0          5h4m
my-app-776b864c75-whbqk   1/1     Running   0          5h3m
my-app-776b864c75-hw932   1/1     Running   0          34s
```

Let's take a closer look at the name of the Pod `my-app-776b864c75-vk87p`. The template for a Pod name is actually `<deployment name>-<replica set id>-<random pod id>`.  
Under the hood, the Deployment creates a ReplicaSet based on it's Pod spec template. The ReplicaSet is **actually** responsible for managing the Pods!  
You can see these by listing the ReplicaSets:
```bash
$ kubectl get replicasets -lapp=my-app
NAME                DESIRED   CURRENT   READY   AGE
my-app-57549f8bdb   0         0         0       6h26m
my-app-798cbb959d   0         0         0       5h29m
my-app-776b864c75   3         3         3       5h18m
```

The `<replica set id>` section of the Pod name is consistent for all replicas managed by a single ReplicaSet. If you were to update the Deployment, you would expect a new ReplicaSet to get created and the new `<replica set id>` to appear on the new Pods.

Just to re-cap, the Deployment manages the ReplicaSets, which manage the Pods.

We can take advantage of this structure to be able to change the labels without downtime!

## Steps

1. List the current ReplicaSets for the Deployment
    ```bash
    $ kubectl get replicasets -lapp=my-app
    NAME                DESIRED   CURRENT   READY   AGE
    my-app-57549f8bdb   0         0         0       6h26m
    my-app-798cbb959d   0         0         0       5h29m
    my-app-776b864c75   3         3         3       5h18m
    ```

1. List the current Pods for the Deployment
    ```bash
    $ kubectl get pods -l app=my-app
    NAME                      READY   STATUS    RESTARTS   AGE
    my-app-776b864c75-vk87p   1/1     Running   0          5h14m
    my-app-776b864c75-whbqk   1/1     Running   0          5h13m
    my-app-776b864c75-hw932   1/1     Running   0          10m
    ```

1. (Optional, but recommended) Delete the old ReplicaSets, the ones with `0` Desired replicas
    ```bash
    $ kubectl delete replicasets my-app-57549f8bdb my-app-798cbb959d
    replicaset.extensions "my-app-57549f8bdb" deleted
    replicaset.extensions "my-app-798cbb959d" deleted
    ```

1. Delete your Deployment with the `--cascade=false` flag
    ```bash
    $ kubectl delete deploy my-app --cascade=false
    deployment.extensions "my-app" deleted
    ```

1. Verify that the pods still exist
    ```bash
    $ kubectl get pods -l app=my-app
    NAME                      READY   STATUS    RESTARTS   AGE
    my-app-776b864c75-vk87p   1/1     Running   0          5h14m
    my-app-776b864c75-whbqk   1/1     Running   0          5h13m
    my-app-776b864c75-hw932   1/1     Running   0          10m
    ```

1. Apply the new labels, typically by re-deploying with the process that initially failed

1. List the new ReplicaSets for the new Deployment, wait for the new Pods to be "Ready"
    ```bash
    $ kubectl get replicasets -lapp=my-app
    NAME                DESIRED   CURRENT   READY   AGE
    my-app-776b864c75   3         3         3       5h28m
    my-app-10p8a02k18   2         2         2       1m
    ```

1. Verify that your new Pods are healthy. This will be specific to your application.  
   If you used a Service to route traffic to your Pods, you can check that the new pods are selected by the Service by running the following
   ```bash
   $ kubectl get endpoints -lapp=my-app
   NAME        ENDPOINTS                                                                  AGE
   my-app   100.65.104.180:8080,100.65.119.120:8080,100.65.121.106:8080 + 12 more...     116d
   ```

1. **Once you have confirmed that your new ReplicaSet is healthy and receiving traffic as expected**, delete the old ReplicaSet
    ```bash
    $ kubectl delete replicasets my-app-776b864c75
    replicaset.extensions "my-app-776b864c75" deleted
    ```

You should now have your new labels on your Deployment, and have all the old Pods cleaned up!
