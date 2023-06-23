# Kubernetes app workshop step by step

This is a walkthrough for the Kubernetes workshop that took place in IPV on the 19th of May of 2023.

**NOTE 1:** This walkthrough can be done using either a Kind cluster or a Minikube cluster. There's a slight difference when it comes to leveraging ingresses but it shouldn't cause any trouble.


**NOTE 2:** The kubernetes manifests used for this walkthough are "attached" to my Docker Hub account with working images. It would be a lot more insightful if you would build your own images along the way!

## The application

![ToDo List](../assets/todo_interface.png)

The application was split into three components: 

- the frontend (https://github.com/r-magalhaes/ws-frontend)
- the backend (https://github.com/r-magalhaes/ws-backend)
- the database (https://github.com/r-magalhaes/ws-database)

Every component had already been Dockerized. You can find the Dockerfile on the root of the repository.

The dockerfile setup for the app

![Docker Setup](../assets/workshop-apps.png)

## The objective

The objective of this workshop was to deploy this ToDo List fullstack application in a local Kubernetes cluster.


# Step by step

<br>

## The namespace

<br>
<br>

First lets create a namespace to deploy all the components of our app.

```kubectl create ns workshop```

Now make sure you switch to it.

If you downloaded kubens you can simply do

```kubens workshop```

Otherwise use

```kubectl config set-context –current –namespace=workshop```

<br>

## The frontend

Lets start with the frontend.

The simplest unit we can work on with Kubernetes is the *Pod* resource. Lets try it.

<br>

### Launching the frontend with a Pod resource

<br>

```cd "Containerization and Orchestration/walkthrough/resources"```

<br>

```kubectl apply -f frontend/pod.yaml```

<br>

Now lets get some visual feedback of our frontend.

```kubectl port-forward pods/workshop-frontend 3000:80```

Open your browser and check out:

```localhost:3000```

Cool, seems to be working. We need to go a little deeper though, the pod does not guarantee us the high-availability that Kubernetes so loudly promises, so we need something more robust, a *Deployment* resource.

<br>

### Deploying the frontend with a Deployment resource

<br>

Lets cleanup the pod we just created, we don't need it anymore.

```kubectl delete -f frontend/pod.yaml```

<br>

Now apply the deployment resource

```kubectl apply -f frontend/deployment.yaml```

This resource manages pods for us as well as another resource - the ReplicaSet. The ReplicaSet resource makes sure there's always an x number of pods running at any single time.

<br>

### Creating a Service for the frontend

Now we want to make sure our frontend is reachable from any other pods in the cluster that require so. For this we need to have a way to expose it via a resource that will give us a permanent way of getting to these pods. This is what the Service resource will give us. Also, by default, will load balance between multiple replicas of our pods.

```kubectl apply -f frontend/service.yaml```

<br>
<br>

## The backend

<br>

### Deploying the frontend with a Deployment resource

```kubectl apply -f backend/deployment.yaml```

### Troubleshooting the backend

We can take this oportunity to look into a couple of commands that will help us troubleshoot our applications in Kubernetes.

The backend requires a database to properly function and at least until now we haven't deployed any for it to connect to. This connection needs to be guaranteed by the backend so lets run the following command to check the backend's pods logs.

First get the name of the backend pod

```kubectl get pods```

<br>

Now lets get the logs

```kubectl logs [pod-name]```

We can see that the connection is not guaranteed yet. We'll fix it in a bit!

<br>

### Creating a Service for the backend

```kubectl apply -f backend/service.yaml```

<br>
<br>

## The database

<br>

For the database we're going to look into Helm. All of these resources we've been creating (Deployments, Services, etc), well some people/organisation have already packaged their applications using Helm for others to use for an easy instalation.

Besides already everything packaged for us to use, we can easily override these deployments with our our configuration. So, lets get to it.

For this workshop we used the Bitnami PostgreSQL [Helm Chart](https://artifacthub.io/packages/helm/bitnami/postgresql)

By taking a look into their documentation there are two fields we want to override.

- auth.existingSecret - Will allow us to feed Postgres with some configuration of our own. In this case we want to override the postgresPassword field.
- auth.database - Will allow us to create a database with the provided name

In order to do that we'll use a values.yaml file.

We can use the documentation to fetch the command we need to run to install Postgres in our cluster. In adition to it, we'll add a flag to allow us to pass the values we want to override the custem deployment. (the -f flag)

```helm install workshop-database oci://registry-1.docker.io/bitnamicharts/postgresql -f database/values.yaml```

<br>

If everything installed successfully we'll se a bunch of notes provided by Bitnami that tell us a bit how we can play around their Postgres. The most important one is knowing the Service name we can use to connect to it.

<br>

### Getting the database ready

<br>

There's still a couple of things we need to do to get our database ready to use. We already provided some values to it but if we check the pod we'll see that something is not quite right.

Lets look into another useful command to troubleshoot our apps, the describe command.

First get the name of the postgres pod name, then describe it.

```kubectl describe pod [pod-name]```

<br>

This gives us the pod configuration and a list of events. If we look at the event list we can see that postgres can't find a certain secret. Remember the auth.existingSecret value we overrided the Bitnami chart with? Yup, this secret does not exist yet. Lets create it.

```kubectl apply -f database/secret.yaml```

<br>

Now describe the postgres pod again...looks like it worked!

<br>

### Creating the table for the ToDo

<br>

Here's how we're approaching this:

- We'll port-forward the postgres Pod to a port on our machine.
- Use a postgres client docker image to run a create table command against it.

So, make sure you have at least two shells ready.

Port-forward the postgres Pod to our machine

```kubectl port-forward services/[postgres-service] 5432```

<br>

Now let's use a psql client image so that we can run commands against our database.

We're going to use the *jbergknoff/postgresql-client* for it.


```docker run -it --rm jbergknoff/postgresql-client postgresql://postgres:workshop#123@host.docker.internal:5432/wstodo```

Notice that "wstodo" has to match the auth.database field we used in our values.yaml that was utilized to deploy PostgreSQL. This is the database we created and want to connect to.

Now let's create a table:

```
CREATE TABLE todo(
  todo_id SERIAL PRIMARY KEY,
  description VARCHAR(255)
);
```

Now we can close this container, it served it's purpose! We can also stop the port-forward.

<br>

### Handling Database configuration and sensitive information

<br>

We need to take care of a couple of things. There are some necessary information we have to provide to PostgreSQL, and from that information some it's sensite, like user and password information for example and other not so much, like the port to use.

For the information that is considered sensitive we will use a Secret and for the rest we will use a ConfigMap. By using these resources we don't have to keep rebuilding our images everytime we need to update one of these variables.

<br>

Let's start with the ConfigMap (database/cm.yaml)

```kubectl apply -f backend/cm.yaml```

And now the sensitive information:

```kubectl apply -f backend/secret.yaml```

Rollout the backend deployment to check if we got a successful connection:

```kubectl rollout restart deployment workshop-backend-deployment```

It's working!!!


<br>

### Connecting the Frontend to the Backend

<br>

If we go to our frontend and try to add any items to our ToDo list, it won't work, the frontend and the backend are not communicating.

We need to have a way to test this setup completely locally, so, in order to do that we'll use and Ingress and use Hostname mapping to do so.

First let's install and Ingress Controller in our cluster that will be able to handle Ingress resources and internally route paths as we please.

<br>

**NOTE:** If you're using Minikube check the following url:

<https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/>

<br>

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml```

Now let's apply the ingress resource:

```kubectl apply -f backend/ing.yaml```

Check the ingress *host* field, this is the value we need to map both in our Frontend backend host environment variable and our host files for name resolution.

In the Frontend docker image update the variable *REACT_APP_BE_HOST* to the value "ws.backend.com"

Rebuild the image and push it to your registry. Then make sure your frontend pods are running this image.

<br>

Now let's work on the hosts file.

**Mac:** /etc/hosts

**Windows:** C:\Windows\System32\Drivers\etc\hosts

Create the following entry:

```127.0.0.1    ws.backend.com```

Time for the ultimate test! Add some items on your ToDo list.

It is done! We've fully deployed our ToDo app in a Kubernetes runtime environment!!




