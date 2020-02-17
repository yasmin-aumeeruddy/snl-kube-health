## The goal of this interactive scenario is to deploy two Java microservices to Kubernetes and check their health.

The online terminal is a pre-configured Linux environment that can be used as a regular console (you can type commands).For information regarding setting up a Kubernetes cluster please look at the following tutorial:
[Setting up a cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/) 

## Deploy Java microservices

Let's begin by ensuring your Kubernetes environment is set up by running by executing the following command in the terminal:

`kubectl version`

Clone the repository with the required configuration files by entering the following command in the terminal and then navigate to the correct directory:

`git clone https://github.com/OpenLiberty/guide-kubernetes-microprofile-health.git`
`cd guide-kubernetes-microprofile-health/finish`

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
