# ReplicaSet

A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

## How a ReplicaSet works 

A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod template.

A ReplicaSet is linked to its Pods via the Pods' metadata.ownerReferences field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their ownerReferences field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.

A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a Controller and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

## When to use a ReplicaSet 

A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.

This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.

## Example

first create a manifest `file-demo/frontend.yaml` looks like below

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

Now execute this command

```
kubectl apply -f file-demo/frontend.yaml
```

You can then get the current ReplicaSets deployed:

```
kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         0       34s
```

You can also check on the state of the ReplicaSet:

```
kubectl describe rs frontend
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  7m45s  replicaset-controller  Created pod: frontend-9t9q2
  Normal  SuccessfulCreate  7m45s  replicaset-controller  Created pod: frontend-vl79m
  Normal  SuccessfulCreate  7m45s  replicaset-controller  Created pod: frontend-mt6hp
```

And lastly you can check for the Pods brought up:

```
kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-9t9q2   1/1     Running   0          16m
frontend-mt6hp   1/1     Running   0          16m
frontend-vl79m   1/1     Running   0          16m

```

You can also verify that the owner reference of these pods is set to the frontend ReplicaSet. To do this, get the yaml of one of the Pods running

```
❯ kubectl get pods frontend-9t9q2 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: d266a2ab76a30a6e0048184d5244dab4bd96471c6810922f9f69b356672eed3c
    cni.projectcalico.org/podIP: 10.233.71.18/32
    cni.projectcalico.org/podIPs: 10.233.71.18/32
  creationTimestamp: "2023-06-29T17:50:06Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-9t9q2
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: 3f431257-9d5d-4d22-b2ca-27a02db17e8f
  resourceVersion: "29517"
  uid: a79ac83e-c07b-4bf2-9473-076f43afe33a
...

```

## Writing a ReplicaSet manifest 

As with all other Kubernetes API objects, a ReplicaSet needs the apiVersion, kind, and metadata fields. For ReplicaSets, the kind is always a ReplicaSet.

When the control plane creates new Pods for a ReplicaSet, the .metadata.name of the ReplicaSet is part of the basis for naming those Pods. The name of a ReplicaSet must be a valid DNS subdomain value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a DNS label.

A ReplicaSet also needs a .spec section.

Pod Template
The .spec.template is a pod template which is also required to have labels in place. In our frontend.yaml example we had one label: tier: frontend. Be careful not to overlap with the selectors of other controllers, lest they try to adopt this Pod.

For the template's restart policy field, .spec.template.spec.restartPolicy, the only allowed value is Always, which is the default.

Pod Selector
The .spec.selector field is a label selector. As discussed earlier these are the labels used to identify potential Pods to acquire. In our frontend.yaml example, the selector was:

```
matchLabels:
  tier: frontend
```
In the ReplicaSet, .spec.template.metadata.labels must match spec.selector, or it will be rejected by the API.

### ReplicaSet as a Horizontal Pod Autoscaler Target

A ReplicaSet can also be a target for Horizontal Pod Autoscalers (HPA). That is, a ReplicaSet can be auto-scaled by an HPA. Here is an example HPA targeting the ReplicaSet we created in the previous example.

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

run the manifest

```
kubectl apply -f file-demo/hpa-rs.yaml
```

Alternatively, you can use the kubectl autoscale command to accomplish the same

```
kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50
```