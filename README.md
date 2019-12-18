This repository is a sample project which demonstrates the fundamentals of Kubernetes deployment.

Here I used the Gcloud Kubernetes instance to setup Kubernetes nodes and access it using the Ubuntu instance on the smae gcloud account.

I assume that you are running 3 Kubernetes nodes and have access to those nodes using `kubectl`.

Firstly, clone this repository onto your local machine using the command `git clone https://github.com/jugnumisal/Kubernetes-test-project.git`

Then, go to the directory
`cd Kubernetes-test-project`

# Rolling Update Deployment demo

run `kubectl create -f rolling-update.yml`

Once you enter this command, the kubernetes will create a replica set of the image mentioned in the yaml file. For this case, Jenkins. To see the pods spun up, enter the command

`kubectl get pods`

You will be able to see 5 replicas of the Jenkins image running on your Kubernetes cluster.

Now the pods are spun-up and running but you won't be able to access them as we haven't exposed the ports to our local machine yet.

To do that, we need to create a service that would expose the port 80 to the port 80 of outside world and attach that service to our pods.

run `kubectl create -f service1.yml`

Our service gets created and you can see the service `svc` when you run the command `kubectl get svc`.

Wait for some time and when there is an IP address available in the external IP column, enter that IP address in your browser and you shall be able to access your Jenkins container.

Now, go to the `rolling-update.yml` file using vi editor `sudo vim rolling-update.yml` and change the image name from Jenkins to something else. Let's say `nginx`.

Now save the file and run the file again but this time using the command

`kubectl apply -f rolling-update.yml`

We use create only for the creation of the replicaset for the first time. Later, whenever there is an update to the previous version, we apply the changes to the current running replica set.

When you refresh the page on your browser, you will observe that the page has switched to Nginx homepage.

When you apply the changes, a new replica set is created one pod at a time. How this works is, for every new pod created, one old pod is disable from the old replica set. This way, the new updates are applied to one container at a time. This is called **Rolling Update Deployment**.

# Blue-green Deployment demo

Now, since we already have one update rolling onto our Kubernetes cluster, we create a new node that create a new replica set on our cluster.

Run `kubectl create -f blue-green.yml`

When you run this command, a new node is spun up with the Jenkins image. Run `kubectl get nodes` and you shall see 2 deployments in the list.

One is our already running nginx container and the other one is our new Jenkins container.

Now how the blue-green deployment works is that we have 2 consequent deployments running on our cluster. We create a new service for this new deployment using the service 2.

`kubectl create -f service2.yml`

This will create a new service and it will be attached to the new deployment.

Now in the live environment, a new deployment environment is create just like this one which is called `green` environment. And the old one is called `blue` environment.

All the final testings are performed on this green environment and if everything seems alright, the existing service of the blue deployment is shifted to the green deployment.

Go to the `service1.yml` file and change the selector app from `app-v1` to `app-v2`.
Now run `kubectl apply -f service`.yml`

This will divert the traffic on the same nginx homepage to Jenkins page.

This is process of creating a new deployment environment with all it's binaries and libraries, performing testings on it and then switching the traffic to this new deployment is called **Blue-Green Deployment**.
