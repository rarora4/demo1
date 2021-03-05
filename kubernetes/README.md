# Kubernetes - Getting Started

### Kubernetes Commands
The first kubernetes command we will run return kubernetes server and client version and verifies it is working.
```sh
$ kubectl version
```

Next we see what see all commands we can run in docker
```
$ kubectl
```
It is a long list of commands, bus as part of our demo we will be using only a few of them.

Let us see how many nodes do we have in out cluster, are they ready and what is their role, IP address? 
```
$ kubectl get nodes
# or kubectl get nodes -o wide
```

Now, lets run a pod. For this step lets first define a YAML definition of the pod specification. Most of the Kubernetes object like Pod, Replicaset, Deployment etc. will have these fields

1. apiVersion
2. kind
3. metadata
4. spec

So definition of our pod would be, save it in a file name pod1.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
spec:
  containers:
  - name: my-first-container
    imagePullPolicy: IfNotPresent
    image: alpine:latest
    command: ["sleep"]
    args: ["99999"]
```

To run this we will use the command
```
$ kubectl create -f pod1.yaml
```
To check the status of the container run
```
$ kubectl get pod
```
To see the node on which the pod is running
```
$ kubectl get pod -o wide
```
You can verify that internally docker pulled the `alpine` image.
```
$ docker image ls | grep alpine
```
You can use two very useful commands to get more information on the pod using
```
$ kubectl get pod my-first-pod -o wide
# and
$ kubectl describe pod my-first-pod
```

You can also go run commands from within the pod using `kubectl exec` command
```
$ kubectl exec -it my-first-pod sh
```

Now lets try and create and object of different kind called `deployment`. Save the below YAML in a file called deployment1.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-first-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: alpine-app
  template:
    metadata:
      labels:
        app: alpine-app
    spec:
      containers:
      - image: alpine:latest
        command:
          - /bin/sh
          - "-c"
          - "sleep 60m"
        imagePullPolicy: IfNotPresent
        name: alpine
```

Then, run the same command we ran to create a pod but file name that of the deployment1.yaml
```
$ kubectl create -f deployment1.yaml
```
Now if you run the `kubectl get pod` command again, there will be 5 pods with the name `my-first-deployment-*`. Why is that?

If you look at the above deployment definition, the number of `replicas` has been set to 5 so the deployment creates 5 pods, but the deployment itself runs as a single instance.
```
$ kubectl get deployment
```
Now lets scale the number of replicas to 20, for this you will need to run:
```
$ kubectl scale --replicas=20 deployment/my-first-deployment
```
You will see 15 new alpine pods come up with the name `my-first-deployment-*`. Alternatively to scale-up or scale-down you can use `kubectl edit deployment my-first-deployment` command and change number of replicas.

Now, to remove these pods and deployment you can run the command
```
$ kubectl delete pod my-first-pod
$ kubectl delete deployment my-first-deployment
```


