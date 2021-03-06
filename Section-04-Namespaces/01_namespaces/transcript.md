# transcript

In Kubernetes


. You see, in Kubernetes we can group our objects into a construct known as **namespaces**. We can use the   get command to list out our namespace:

```bash
$ kubectl get namespaces
NAME          STATUS    AGE
default       Active    11h
kube-public   Active    11h
kube-system   Active    11h
```

Our minikube cluster comes included with these namespaces, however Kubectl can only interact with one namespace at a time. We can run the get-context config command to see which namespace our kubectl client is currently using:

```bash
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube
```

This command lists all the contexts defined in the kubectl config file, in my case I only have one context.

```popup animation
~/.kube/config
```

We'll cover more about this file later in the course.

If the current context's namespace shows as blank, like it is here, then it means that we're using the default namespace [red box]. So when we ran the get-pods command we only retrieved a list of pods that are in the default namespace.


If you want to interact with a different namespace, then one way to do that is by  specifying the namespace flag:


```bash
$ kubectl get pods xxxxx --namespace=kube-system
```

The kube-system namespace is used by Kubernetes to house objects that it itself needs for it's own internal working. Here we see that we have a lot of pods. This explains where all those containers came from earlier.

In Kubernetes, you can create your own custom namespaces and organise your objects into them. Here's a yaml file to create a namespace called ns-dev1:

```bash
$ tree configs/
$ code configs/namespace-dev1.yml
```

namespaces are quite a basic type of object, which is why the yaml definition is quite short compared to other object types. The name that you give to your namespaces should be something that makes sense in your workplace, such as naming them after    team names, project names, environment names, and etc.

Lets now create this namespace:

```bash
kubectl apply -f configs/namespace-dev1.yml
kubectl get namespaces
```

Next you'll want to add objects to your namespace. So let's try to create the following pod:

```bash
code configs/pod-httpd-any-namespace.yml
```

This is the same yaml definition that we used in our hello world demo earlier. Now to create this pod in our new namespace, we can use the namespace flag:

```bash
kubectl apply -f configs/pod-httpd.yml --namespace ns-dev1
kubectl get pods
kubectl get pods --namespace=ns-dev1
```

This is a great way to deploy the same pod across several namespaces. All you have to do is run the above command multiple times with a different namespace each time. Now if you want to work with a particular namespace for a prolonged period, then constantly writing out the namespace flag can get tedious. In that scenario, it's more convenient to persistently change the default namespace by using the set-context command:

```bash
kubectl config current-context
kubectl config set-context $(kubectl config current-context) --namespace=ns-dev1
kubectl config get-contexts
```

This effectily makes a one line change in the kubectl's config file. Now kubectl will use the ns-dev1 namespace as the new default, unless told otherwise.

```bash
$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          3m10s
$ kubectl get pods -o yaml --namespace=ns-dev1 | grep 'namespace:'
    namespace: ns-dev1
```

If you want to limit a pod, so that it can only be created inside a specific namespace, then you can hard code the namespace into the pod's   definition.

```code
code configs/pod-httpd-specfic-namespace.yml
```

Here we set the restriction by specifying the metadata.namespace setting. Let's try this out now:

```bash
kubectl apply -f configs/pod-httpd-specfic-namespace.yml
kubectl get pods --namespace=kube-public
```

ok, So far, We have only explored namespace using pod objects. But the same concepts apply to other object types as well such as services. However, some objects are cluster level objects and can't be namespaced. You can use the api-resources command to get a list of which objects can be namespaced and which cant:

```bash
# zoom out
$ kubectl api-resources -o wide
```


There's a lot of output here so I'll just scroll up to the beginning.

I'll zoom out a little bit so that each row fits on a single line.


Here we have the namespaced column. This tells us which of these objects can be put inside a namespace.


the namespace changing command is quite long. I've got it set as an alias so I'm able to bring it up faster. however there's another tool you can use that makes it easier.


With this you have the tab option to quickly pick your namespace.

