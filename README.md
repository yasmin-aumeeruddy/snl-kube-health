## Configuring a Readiness Probe for a Java Microservice

The online terminal is a pre-configured Linux environment that can be used as a regular console (you can type commands).For information regarding setting up a Kubernetes cluster please look at the following tutorial:
[Setting up a cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/) 

## Deploy Java microservices

Let's begin by ensuring your Kubernetes environment is set up by running by executing the following command in the terminal:

`kubectl version`

Clone the repository with the required configuration files by entering the following command in the terminal and then navigate to the correct directory:

`git clone https://github.com/yasmin-aumeeruddy/snl-kube-health.git`

`cd snl-kube-health/finish`

Build the applications by running the following command:

`mvn clean package`

Now that you have your Kubernetes cluster up and running, you can deploy your microservices using the following command:

`kubectl apply -f kubernetes.yaml`

While you are waiting for your services to start up, take a look at the provided kubernetes.yaml file that you have used to deploy your two Java microservices. 

> [File -> Open] snl-kube-health/kubernetes.yaml

Here you can see configuration for two deployments and two services. The first deployment **name-deployment** has two replicas which means it will have two pods running. We also specify the Docker image name and the container ports we would like to map outside the container, **9080**. This deployment contains a simple Java microservice that displays a brief greeting and the name of the container it runs in.

The second deployment **ping-deployment** does not specify any replicas as we only require one pod for this tutorial. This deployment pings the Kubernetes service that encapsulates the pod running the name microservice. This demonstrates how communication can be established between pods in your cluster.

For each deployment, you can find information relating to the readiness probe, provided by Kubernetes, underneath the **readinessProbe** attribute. We have specified a delay of 15 seconds that will give the deployment sufficient time to start up. The polling period is set to 5 seconds so it will check the pods health every 5 seconds and if it gets one bad request it will mark that pod as unhealthy.

The Kubernetes readiness probes in these services are implemented using MicroProfile health. The two Docker images that are being used for this tutorial have classes annotated with **@Health** that are integrated with CDI. Run the following command to have a look inside one of the classes used in this tutorial. This is just a simple class that contains a method **setUnhealthy()** that will make the service unhealthy for 60 seconds that allows the tutorial to demonstrate how useful this can be with Kubernetes. Once you have had a look at the code behind the service please move on to the next step.

> [File -> Open] snl-kube-health/finish/system/src/main/java/io/openliberty/guides/system/SystemReadinessCheck.java

The microservices are fully deployed and ready for requests when the **READY** column indicates 1/1 for each deployment. Repeat the previous command until all deployments are ready before continuing. Now that your microservices are deployed and running, you are ready to send some requests.

Firstly check the node ports by running the following command in the terminal:

`kubectl get services` 

There are two ports specified under the **Port(s)** collumn for each service and they are shown as **{target port}/{node port}/TCP**, for example, **9080:31006/TCP** where 9080 is the target port and 31006 is the node port. Take note of each node port shown from the command.

Set the **namePort** and **pingPort** variables to the correct node ports for each service:

`namePort={port}`
and 
`pingPort={port}`

Check that they have been set correctly: 

`echo $namePort && echo $pingPort`

You should see an output consisting of both node ports. 

To find the IP addresses required to access the services, use the following command:

`kubectl describe pods`

*Note: to make your terminal full screen, double click the terminal heading

This command shows the details for both pods. The IP addresses for the nodes that the pods are deployed on are listed in the output. Look for the IP address that is stated next to the label **Node:** for each pod. For example, in the following case, the IP address would be **10.114.85.172** for the name deployment and **10.114.85.161** for the ping deployment. 

```
Name:           name-deployment-855f7d4b98-77qgl
Namespace:      sn-labs-yasminaumeer
Priority:       0
Node:           10.114.85.172/10.114.85.172
Start Time:     Tue, 18 Feb 2020 14:15:42 +0000
...
Name:           ping-deployment-5bb7ff78cf-s88w4
Namespace:      sn-labs-yasminaumeer
Priority:       0
Node:           10.114.85.161/10.114.85.161
Start Time:     Tue, 18 Feb 2020 14:15:42 +0000
...
```

Like you did with the node ports, set the **nameIP** and **pingIP** variables to the right IP addresses for the services:

`nameIP={IP address}`
`pingIP={IP address}`

Check that they have been set correctly: 

`echo $nameIP && echo $pingIP`

You should see an output consisting of both IP addresses.

When you run the following command it will use the IP address of your cluster (This may take several minutes).

`curl http://$nameIP:$namePort/api/name`

You should see a response similar to the following:

`Hello! I'm container [container name]`

Similarly, navigate to `curl http://$pingIP:$pingPort/api/ping/name-service` and observe a response with the content pong.

## Turn one of your Microservices Unhealthy

Now that your microservices are up and running and you have made sure that your requests are working, we can monitor the microservices’ health. Let’s start by making one of our microservices unhealthy. To do this, you need to make a POST request to a specific URL endpoint provided by the MicroProfile specification, which allows you to make a service unhealthy:

`curl -X POST http://$nameIP:$namePort/api/name/unhealthy`

If you now check the health of your pods you should notice it is not ready as shown by **0/1**.

`kubectl get pods`

Now if you send a request to the endpoint again you will notice it will not fail as your other microservice will now handle the request:

`curl http://$nameIP:$namePort/api/name`

## Test out the readiness Probe

The unhealthy deployment should automatically recover after about 30 seconds. Run the following command until you see the **READY** state return to **1/1**.

`kubectl get pods`


Once it has recovered you are going to make both demo pods unhealthy by making a POST request to each deployment.

`curl -X POST http://$nameIP:$namePort/api/name/unhealthy`

`curl -X POST http://$nameIP:$namePort/api/name/unhealthy`

 If the response from the second request has the same pod name as the first, wait 5 seconds and run the command again. This is because the readiness probe has not noticed the microservice has become unhealthy.

 Now check that both pods are no longer in a ready state:

 `kubectl get pods`

 You should soon notice that the ping microservice has also changed state. This is because the readiness probe for that pod has realized the demo pod is no longer receiving requests and as such the ping microservice no longer works.

After a small amount of time, if you keep running the previous command you will notice the demo pods recover and change state to ready. Following this, the ping microservice will also become available after a short amount of time.
 
Clean up the cluster by running the following command:

`kubectl delete -f kubernetes.yaml`
 
## Great Work! You're done!

You have used MicroProfile Health to create an endpoint that reports on your Java microservice’s status. Then, you observed how Kubernetes uses the /health endpoint with the readiness probe to keep your microservices running smoothly.

If you would like to look at the code for these microservices follow the link to the github repository. For more information about the MicroProfile specification used in this tutorial, visit the MicroProfile website with the link below.

[Github repository](https://github.com/OpenLiberty/guide-kubernetes-microprofile-health)
[MicroProfile.io](https://microprofile.io)

