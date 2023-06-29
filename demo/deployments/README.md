# Deployment

Deployments wrap ReplicaSets with support for declarative updates and rollbacks. They’re a higher level of abstraction that’s easier to control.

A Deployment object lets you specify the desired state of a set of Pods. This includes the number of replicas to run. Modifying the Deployment will automatically detect the required changes and scale the ReplicaSet as required. You can pause the rollout or revert to an earlier revision, features that aren’t available with plain ReplicaSets.

# Create Deployment

You can create it by only one command looks like this

```
kubectl create deployment nginx --image nginx:latest --replicas 3
```

Just like pod you can also create the manifest, so you can share it and manage it as a code.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

And run this command

```
kubectl apply -f file-demo/nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
# Get the deployment

Now run this command to se the list of the deployment
```
kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           67s
```

and if you check the pods you will get 3 pods

```
kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6b7f675859-2kjgz   1/1     Running   0          2m
nginx-deployment-6b7f675859-brp2n   1/1     Running   0          2m
nginx-deployment-6b7f675859-c8gv9   1/1     Running   0          2m
```

# Updating deployment

Let's update the nginx Pods to use the nginx:1.16.1 image instead of the nginx:1.14.2 image.

```
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
```

or 

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

Run `kubectl get rs` to see that the Deployment updated the Pods by creating a new ReplicaSet and scaling it up to 3 replicas, as well as scaling down the old ReplicaSet to 0 replicas.

```
kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-56f4d48746   0         0         0       6m43s
nginx-deployment-6b7f675859   0         0         0       11m
nginx-deployment-7d97946fb9   3         3         3       100s
```

To get details of your deployment

```
kubectl describe deployments

❯ kubectl describe deployments 
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 29 Jun 2023 23:29:51 +0700
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.16.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-7d97946fb9 (3/3 replicas created)
Events:
  Type    Reason             Age                    From                   Message
  ----    ------             ----                   ----                   -------
  Normal  ScalingReplicaSet  13m                    deployment-controller  Scaled up replica set nginx-deployment-6b7f675859 to 3
  Normal  ScalingReplicaSet  8m38s                  deployment-controller  Scaled up replica set nginx-deployment-56f4d48746 to 1
  Normal  ScalingReplicaSet  8m33s                  deployment-controller  Scaled down replica set nginx-deployment-6b7f675859 to 2 from 3
  Normal  ScalingReplicaSet  8m33s                  deployment-controller  Scaled up replica set nginx-deployment-56f4d48746 to 2 from 1
  Normal  ScalingReplicaSet  8m29s                  deployment-controller  Scaled down replica set nginx-deployment-6b7f675859 to 1 from 2
  Normal  ScalingReplicaSet  8m29s                  deployment-controller  Scaled up replica set nginx-deployment-56f4d48746 to 3 from 2
  Normal  ScalingReplicaSet  8m25s                  deployment-controller  Scaled down replica set nginx-deployment-6b7f675859 to 0 from 1
  Normal  ScalingReplicaSet  3m35s                  deployment-controller  Scaled up replica set nginx-deployment-7d97946fb9 to 1
  Normal  ScalingReplicaSet  3m18s                  deployment-controller  Scaled down replica set nginx-deployment-56f4d48746 to 2 from 3
  Normal  ScalingReplicaSet  3m10s (x4 over 3m18s)  deployment-controller  (combined from similar events): Scaled down replica set nginx-deployment-56f4d48746 to 0 from 1
```

## Rollover (aka multiple updates in-flight)

Each time a new Deployment is observed by the Deployment controller, a ReplicaSet is created to bring up the desired Pods. If the Deployment is updated, the existing ReplicaSet that controls Pods whose labels match .spec.selector but whose template does not match .spec.template are scaled down. Eventually, the new ReplicaSet is scaled to .spec.replicas and all old ReplicaSets is scaled to 0.

If you update a Deployment while an existing rollout is in progress, the Deployment creates a new ReplicaSet as per the update and start scaling that up, and rolls over the ReplicaSet that it was scaling up previously -- it will add it to its list of old ReplicaSets and start scaling it down.

For example, suppose you create a Deployment to create 5 replicas of nginx:1.14.2, but then update the Deployment to create 5 replicas of nginx:1.16.1, when only 3 replicas of nginx:1.14.2 had been created. In that case, the Deployment immediately starts killing the 3 nginx:1.14.2 Pods that it had created, and starts creating nginx:1.16.1 Pods. It does not wait for the 5 replicas of nginx:1.14.2 to be created before changing course.

## Rolling back a deployment

Suppose that you made a typo while updating the Deployment, by putting the image name as nginx:1.161 instead of nginx:1.16.1:

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.161
```

The rollout gets stuck. You can verify it by checking the rollout status:

```
kubectl rollout status deployment/nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
```

check the replicas will be only 1

```
kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-56f4d48746   0         0         0       14m
nginx-deployment-6b7f675859   0         0         0       19m
nginx-deployment-7d97946fb9   3         3         3       9m21s
nginx-deployment-7f6bb9949f   1         1         0       2m1s
```

Looking at the Pods created, you see that 1 Pod created by new ReplicaSet is stuck in an image pull loop

```
kubectl get pod
NAME                                READY   STATUS             RESTARTS   AGE
nginx-deployment-7d97946fb9-7s6nx   1/1     Running            0          10m
nginx-deployment-7d97946fb9-bk8n6   1/1     Running            0          10m
nginx-deployment-7d97946fb9-s6zb8   1/1     Running            0          10m
nginx-deployment-7f6bb9949f-52bfk   0/1     ImagePullBackOff   0          3m28s
```

to rolling back to previous revision you can do

```
kubectl rollout undo deployment/nginx-deployment
deployment.apps/nginx-deployment rolled back
```

if you check the deployment it will be back to up and 3 and will looks like this

```
kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           24m
```

## Remove

You can run this command

```
kubectl delete deployment nginx-deployment
```

or 

```
kubectl delete -f file-demo/nginx-deployment.yaml
```

**self notes:**
Edit the manifest if want to do changes on resources, it will be more easy to manage than do changes via command line instead