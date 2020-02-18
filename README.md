## The goal of this interactive scenario is to deploy two Java microservices to Kubernetes and check their health.

The online terminal is a pre-configured Linux environment that can be used as a regular console (you can type commands).For information regarding setting up a Kubernetes cluster please look at the following tutorial:
[Setting up a cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/) 

## Deploy Java microservices

Let's begin by ensuring your Kubernetes environment is set up by running by executing the following command in the terminal:

`kubectl version`

Clone the repository with the required configuration files by entering the following command in the terminal and then navigate to the correct directory:

`git clone https://github.com/yasmin-aumeeruddy/snl-kube-health.git`

`cd snl-kube-health`

Now that you have your Kubernetes cluster up and running, you can deploy your microservices using the following command:

`kubectl apply -f kubernetes.yaml`

While you are waiting for your services to start up, take a look at the provided kubernetes.yaml file that you have used to deploy your two Java microservices, by issuing the command below:

`cat kubernetes.yaml`

Here you can see configuration for two deployments and two services. The first deployment `name-deployment` has two replicas which means it will have two pods running. We also specify the Docker image name and the container ports we would like to map outside the container, `9080`. This deployment contains a simple Java microservice that displays a brief greeting and the name of the container it runs in.

The second deployment `ping-deployment` does not specify any replicas as we only require one pod for this tutorial. This deployment pings the Kubernetes service that encapsulates the pod running the name microservice. This demonstrates how communication can be established between pods in your cluster.

For each deployment, you can find information relating to the readiness probe, provided by Kubernetes, underneath the ‘readinessProbe’ attribute. We have specified a delay of 15 seconds that will give the deployment sufficient time to start up. The polling period is set to 5 seconds so it will check the pods health every 5 seconds and if it gets one bad request it will mark that pod as unhealthy.

The Kubernetes readiness probes in these services are implemented using MicroProfile health. The two Docker images that are being used for this tutorial have classes annotated with `@Health` that are integrated with CDI. Run the following command to have a look inside one of the classes used in this tutorial. This is just a simple class that contains a method `setUnhealthy()` that will make the service unhealthy for 60 seconds that allows the tutorial to demonstrate how useful this can be with Kubernetes. Once you have run the following command and had a look at the code behind the service please move on to the next step.

`cat system/src/main/java/io/openliberty/guides/system/SystemReadinessCheck.java`

Issue the following command to check the health of your pods:

`kubectl get pods`

The microservices are fully deployed and ready for requests when the `READY` column indicates 1/1 for each deployment. Repeat the previous command until all deployments are ready before continuing. Now that your microservices are deployed and running, you are ready to send some requests.

Firstly check the IP address of your Kubernetes cluster by running the following command:

minikube ip{{execute}}

You need to set the variable IP to the IP address of your Kubernetes cluster by running the following command:

IP=$(minikube ip){{execute}}

When you run the following command it will use the IP address of your cluster (This may take several minutes).

`curl http://$IP:31000/api/name`

You should see a response similar to the following:

`Hello! I'm container [container name]`

Similarly, navigate to `curl http://$IP:32000/api/ping/name-service` and observe a response with the content pong.

## Turn one of your Microservices Unhealthy

Now that your microservices are up and running and you have made sure that your requests are working, we can monitor the microservices’ health. Let’s start by making one of our microservices unhealthy. To do this, you need to make a POST request to a specific URL endpoint provided by the MicroProfile specification, which allows you to make a service unhealthy:

`curl -X POST http://$IP:31000/api/name/unhealthy`

If you now check the health of your pods you should notice it is not ready as shown by `0/1`.

`kubectl get pods`

Now if you send a request to the endpoint again you will notice it will not fail as your other microservice will now handle the request:

`curl http://$IP:31000/api/name`

## Test out the readiness Probe

The unhealthy deployment should automatically recover after about 1 minute. Run the following command until you see the `READY` state return to `1/1`.

`kubectl get pods`

Once it has recovered you are going to make both demo pods unhealthy by making a POST request to each deployment.

`curl -X POST http://$IP:31000/api/name/unhealthy`

`curl -X POST http://$IP:31000/api/name/unhealthy`

 If the response from the second request has the same pod name as the first, wait 5 seconds and run the command again. This is because the readiness probe has not noticed the microservice has become unhealthy.

 Now check that both pods are no longer in a ready state:

 `kubectl get pods`

 You should soon notice that the ping microservice has also changed state. This is because the readiness probe for that pod has realized the demo pod is no longer receiving requests and as such the ping microservice no longer works.

 After a small amount of time, if you keep running the previous command you will notice the demo pods recover and change state to ready. Following this, the ping microservice will also become available after a short amount of time.
