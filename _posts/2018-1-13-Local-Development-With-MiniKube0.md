---
layout: post
title: Local Kubernetes Development with MiniKube - Part 2
subtitle: Learning Kubernetes with Helm!
gh-badge: [star, fork, follow]
---
# Continuing Forward with MiniKube using Helm
This is part 2 of a series that illustrates to get started with local Kubernetes development using the MiniKube project, directly on your laptop. If you have not read part one of this series, I would recommend you read it [here](https://aric49.github.io/2018-01-13-Local-Development-With-MiniKube/).  Before continuing, it is important to make sure that you have MiniKube running on your laptop with the `kubectl` commandline tool configured to use the Kubernetes API that comes as apart of your MiniKube deployment.   To validate your Minikube installation is up and running, execute the `kubectl cluster-info` command. It should return details about your Minikube cluster as seen below:

```
$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:8443
```

Before moving forward, we can clean up the cluster by deleting the NGINX `test` deployment we created previously:  

```
$ kubectl delete deployment test
deployment "test" deleted
```

In the last tutorial, we learned how to launch Minikube as well as create and expose a simple deployment using the NGINX web server. In this lesson, we will learn about Helm, an opensource tool for installing Kubernetes *packages* and managing deployments and other Kubernetes abstractions.   Using Helm, we can manage Kubernetes resources as code to make installing applications and dependencies quickly and easily.

## What is Helm?
[Helm](https://helm.sh/) is a package manager for Kubernetes that provides a layer of abstraction on top of the standard Kubernetes API. Using Helm, Kubernetes resources can be expressed as Helm Charts which describe the desired state of the application running in the Kubernetes cluster.  Abstractions such as image version, tag, annotations, ingress resources, and secrets can be automated and packaged for consumption in any Kubernetes cluster. Helm Charts are essentially templatized Kubernetes manifests that are rendered with user defined variables when applied to a Kubernetes cluster. These make up the source code side of Helm and provide a consistent base for deploying applications into Kubernetes. Charts are useful for automating Kubernetes builds by giving developers a simple interface from which deployments can be updated or modified along with the project source code. In this tutorial we will look at installing Helm in the Minikube cluster and deploying a simple application.

## Initializing Helm
Helm is composed of two primary parts:  The Helm CLI tool that is installed on your laptop, and `Tiller` the server-side component that interfaces with Kubernetes and instructs the K8s API server to bring cluster resources into the desired state according to the Helm charts. Before we can start deploying charts into Kubernetes, we need to first install the Helm CLI tool onto our workstation.  We can do this by downloading the Helm binaries and extracting them to a `$PATH` directory similar to how we installed KubeCTL. In a similar way, Helm can be downloaded for Windows and MacOS. Be sure to check the [Helm releases](https://github.com/kubernetes/helm/releases) page for more details.

Let's first download and extract the Helm binaries:
```
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.8.0-linux-amd64.tar.gz

$ tar -xvf helm-v2.8.0-linux-amd64.tar.gz
linux-amd64/
linux-amd64/README.md
linux-amd64/helm
linux-amd64/LICENSE
```

From here, we can copy the primary Helm executable to a  $PATH location, and set the permissions to executable:

```
$ sudo cp linux-amd64/helm /usr/local/bin
$ sudo chmod +x /usr/local/bin/helm
```

Now that Helm has been installed locally, we can use the Helm CLI tool to bootstrap Tiller into our Minikube environment. To do this, we will use the `helm init` command to initialize Helm locally as well as in our cluster. It is important to note, that Helm will use your Kubernetes configuration file (kubeconfig) to authenticate to the cluster and install Tiller.

```
$ helm init                                                    
$HELM_HOME has been configured at /home/aric/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.                                                                                                                    
Happy Helming!                                          
```

The `Happy Helming!` response will indicate that Helm has been initialized locally as well as in the cluster.  From here, we can start deploying services using Helm.

## Installing a Test Application
To get a feel for how Helm works, we can install a chart into our Kubernetes cluster to test the functionality. By default, Helm can install any chart located in the official stable repositories located on [GitHub](https://github.com/kubernetes/charts) using the `helm install` command. For this example we are going to install the `stable/mysql` chart and validate that it is working as expected.  To do this, we are going to use the `helm install` command, providing a unique release name so we can easily identify our application. We can also override any variables the author of the Chart exposed in the `variables.yml` file, by using the `--set` flag.  In this example, are going to expose the MySQL port as a NodePort (high numbered port) that will be directly accessible from our workstation. You can view the `values.yaml` file to see what values the author exposed, [here](https://github.com/kubernetes/charts/blob/master/stable/mysql/values.yaml).  

```
$ helm install --name test-helm stable/mysql --set service.type=NodePort                                                                                                            
NAME:   test-helm                                                                                                                                                                                           
LAST DEPLOYED: Fri Mar  9 15:11:34 2018                                                                                                                                                                     
NAMESPACE: default                                 
STATUS: DEPLOYED                                   

RESOURCES:                                         
==> v1/PersistentVolumeClaim                       
NAME             STATUS  VOLUME                                    CAPACITY  ACCESS MODES  STORAGECLASS  AGE                                                                                                
test-helm-mysql  Bound   pvc-107f568c-23d6-11e8-88af-0800270bdc16  8Gi       RWO           standard      0s                                                                                                 

==> v1/Service                                     
NAME             TYPE      CLUSTER-IP   EXTERNAL-IP  PORT(S)         AGE                              
test-helm-mysql  NodePort  10.96.7.180  <none>       3306:32189/TCP  0s                               

==> v1beta1/Deployment                             
NAME             DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE                                         
test-helm-mysql  1        1        1           0          0s                                          

==> v1/Pod(related)                                
NAME                              READY  STATUS    RESTARTS  AGE                                      
test-helm-mysql-7f8d4fb59d-khdnc  0/1    Init:0/1  0         0s                                       

==> v1/Secret                                      
NAME             TYPE    DATA  AGE                 
test-helm-mysql  Opaque  2     0s                  
```

As you can see, installing the Helm chart installed a large number of Kubernetes resources into the cluster such as a Service, Deployment, Pod and a secret.  The status of these can be seen on the Helm status screen that displays upon a successful deployment. Also on this screen, you can see which high numbered node port has been assigned to the deployment. In my case, it is: `32189`.   We should be able to use curl and see if we can access the MySQL instance running on this port:

```
$ curl -v --output - 192.168.99.100:32189
* Rebuilt URL to: 192.168.99.100:32189/
*   Trying 192.168.99.100...
* TCP_NODELAY set
* Connected to 192.168.99.100 (192.168.99.100) port 32189 (#0)
> GET / HTTP/1.1
> Host: 192.168.99.100:32189
> User-Agent: curl/7.55.1
> Accept: */*
>
* Connection #0 to host 192.168.99.100 left intact
5.7.14AjE`R\rH#5I1L`mysql_native_passwordot packets out of order
```

As you can see, we received a response indicating that MySQL is running on this port. We can also query the Kubernetes API server to see details about this deployment:

```
$ kubectl get deployment      
NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE     
test-helm-mysql   1         1         1            1           31m
```

Similarly, Helm can give us details about the deployment using the `helm ls` command:

```
$ helm ls
NAME            REVISION        UPDATED                         STATUS          CHART           NAMESPACE
test-helm       1               Fri Mar  9 15:11:34 2018        DEPLOYED        mysql-0.3.4     default  
```


Finally, we can delete the deployment from our cluster using the `helm delete` command. By default, Helm will cache charts and deployments in the cluster historically, so it is important that we use the `--purge` flag to delete the cache as well:

```
$ helm delete --purge test-helm
release "test-helm" deleted
```

Using KubeCTL, we can validate that this command has succeeded and the chart has been purged from Tiller:

```
$ kubectl get deployments
No resources found.

$ kubectl get pods
No resources found.
```

This concludes this entry on the basics of Helm and Kubernetes. Stay tuned for the next installment in which we will cover more advanced Helm concepts including writing custom charts!
