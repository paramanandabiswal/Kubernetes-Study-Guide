Hello everyone one, and welcome back.

Ok, a little while ago, we saw how you can do pod-to-pod communication using IP addresses. We're going to do the same thing but this timee using DNS names. To do this we need to make use of Kubernete's own internal DNS service, which is called Kubernetes DNS. Kubernetes DNS is actually an addon that you can install into your cluster. However, it actually comes preinstalled by default in most installer options anyway, such as minikube and kubeadm. 

Ok before we show how kubernetes DNS works, let me do a quick recap on the ip address approach. 

Here I've opened up a bash terminal inside this video's topic folder. So let's create the same 2 pods that we used in our earlier demo. 

```
$ pwd
$ code configs/pod-centos.yml configs/pod-httpd.yml
$ kubectl apply -f configs/pod-httpd.yml -f configs/pod-centos.yml
$ kubectl get pods -o wide
$ kubectl exec pod-centos -- curl --silent http://172.17.0.9
```
==== keep using centos, since centos image now comes with dnf. 
==== introduce ubi as part of docker-registry video. 
By the way you may have noticed that in my earlier demos I used the centos Docker image to create my dummy test pod. However I've now switched to using the Universal Base Image, or UBI for short. The UBI image is an enterprise grade image developed by redhat. And the best part is that this image is available for free. UBI is not available on docker hub though, instead you need to download it from redhat's own registry. That's why I needed to specify the full image path. If you just specify the name, then kubernetes will default to prefixing the dockerhub's registry, to the image name behind the scenes. The docker hub's registry url is docker.io. For example if you specify busybox, then kubernetes will assume you meant docker.io/busybox. I'll cover more about using third party docker registries later in this course. 
====

So that's how far we got to last time. Now in this demo we still want to run a curl command, but this time using a DNS name rather than an IP addresse.


So to start using DNS, we first need to create a service object. So here's the service we're going to create:

```
code config/svc-nodeport-httpd.yaml
```

I'm going to close the centos tab to make room. 

I don't want to get sidetracked by going over this service file just yet. Instead I'll go through it in the next video. For now, the only thing I'll say is that this service uses label&selectors to associate itself with this pod, and this service will accept internal traffic from port xxxxx and forward it to port xxxx, which is the port our apache pod is listening on. 


So let's go ahead and create this service.

```
$ kubectl apply -f configs/svc-nodeport-httpd.yml
$ kubectl get services -o wide
```

During this service's creation process, kubernetes took this service's name, along with it's ip address, and used them to register a dns record in Kubernetes DNS. We can do an nslookup to confirm that's thee case.

```
$ kubectl exec -it pod-centos -- bash
$ nslookup svc-nodeport-httpd
```

Ok it looks like nslookup isn't installed, so let's install it.


```
dnf whatprovides */nslookup
```

Here we can see that nslookup comes as part of the bind-utils package.

```
dnf install -y bind-utils
```

Now let's try that again:

```
nslookup svc-nodeport-httpd
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   svc-nodeport-httpd.default.svc.cluster.local
Address: 10.105.186.109
```

As you can see, we now have a new dns record, which is the fqdn of our service along with it's ip address. So when we created this service, this dns entry was created as well under the covers. 

Should this ip address change in the future then kubernetes DNS will automatically update itself with the new ip address straight away. 



Now, finally,  let's try out our new dns record:


```
[root@pod-centos /]# curl http://svc-nodeport-httpd.default.svc.cluster.local:3050
<html><body><h1>It works!</h1></body></html>
```

Awesome that worked! That means we no stop using IP addresses. This service object now acts as our gateway to our pod.   


We had to use port 3050 here because in the service spec we said that this service can only accept internal traffic on this port.

The 'default' in the fqdn actually refers to the namespace the service live's in. 

Also Since our centOS pod lives in the same namespace as the service itself, it means we can get away with just curling the service's name:

```
$ curl http://svc-nodeport-httpd:3050
<html><body><h1>It works!</h1></body></html>
```

This worked because the resolv.conf has a default basename that's used as a fallback if you don't specify the fqdn. 

```
$ cat /etc/resolv.conf 
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

Now, there's one final thing I wanted to show you before I end this video, and that's to do with the nameserver (red box). This ip address is something that kubernetes has automatically put into the resolv.conf at the time of creating this pod. This is the Kubernetes DNS service's IP address. So let's see what that ip address points to:



```
$ kubectl get all -o wide --all-namespaces | grep 10.96.0.10
kube-system            service/kube-dns                    ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   4h34m   k8s-app=kube-dns
```

Here we find that it belongs to a service called kube-dns. This service lives in the kube-system namespace, and is configured to forward traffic to pods with the label key-pair value of k8s-app=kube-dns.

And if we look at what pods this service forwards traffic too, we find they are coredns pods. 


```
$ kubectl get pods --namespace=kube-system -l k8s-app=kube-dns 
NAME                       READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-5wm6q   1/1     Running   0          4h45m
coredns-5644d7b6d9-kzpjj   1/1     Running   0          4h45m
```

That's because, undr thee covers, Kubernetes DNS is actually built on top of another open source project called, CoreDNS. These coredns pods that are responsible for providing the DNS service to the cluster. So whenever we create a service object, a new dns record gets added to the coredns pod's internal dns database. 

Also if coredns receives a nslookup request for a dns record that it has no knowledge of, such as doing an nslookup for codingbee.net, then coredns will forward that request on to one of the internet's public dns servers, and then feedback the results back to the requester. 

```
$ kubectl exec xxxxxx -- nslookup codingbee.net
```

That's it for this video. I'll see you in the next one. 


