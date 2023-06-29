# Pods

Pods are the fundamental compute unit in Kubernetes. A Pod is analogous to a container but with some key differences. Pods can contain multiple containers, each of which share a context. The entire Pod will always be scheduled onto the same node. The containers within a Pod are tightly coupled so you should create a new Pod for each distinct part of your application, such as its API and database.

In simple situations, Pods will usually map one-to-one with the containers your application runs. In more advanced cases, Pods can be enhanced with init containers and ephemeral containers to customize startup behavior and provide detailed debugging.

## How to create a pod

You can create a pod using a command like this command:

```
kubectl run nginx --image nginx:latest
```

You can also create it by using a manifest by a yaml file, looks like config below

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

And execute it wit command

```
kubectl apply -f file-demo/simple-pod.yaml
```

## To list the pod

To get the list of the pod you can easily run this command

```
kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          36s

```

## Remove pod

To remove the pod you can execute this command

```
kubectl delete pods nginx
```

or via the yaml file

```
kubectl delete -f simple-pod.yaml
```

if you run `kubectl get pods` you will get the pod removed

```
kubectl get pod
No resources found in default namespace.
```