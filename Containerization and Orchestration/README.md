# Pre-Requisites

- Git
- Docker 
- Local Kubernetes Cluster (Kind, Minikube)
- kubectl
- DockerHub Account (https://hub.docker.com/)

## Optional

- kubectx and kubens (https://github.com/ahmetb/kubectx)

# The App!

![ToDo List](./assets/ToDoList.png "ToDo List")

[Front end](https://github.com/r-magalhaes/ws-frontend)

[Back end](https://github.com/r-magalhaes/ws-backend)

[Database](https://github.com/r-magalhaes/ws-database)

# The Problem...

We've been managing the deployment of this application in an on-premise. So far this app is been used inside of your organisation which made it pretty simples to manage all around. You've dockarized the Front-end, Back-end and even the Database. Now your organisation decided to launch the app online for other people to use it.

Some things to take into consideration:
- You're aware you'll need to have control over increasing/decreasing load when the App goes live
- The App is still somewhat raw which means new updates will arrive and you want to have a system in place to have automated deployments
- You also want to take advantage of the dockerized images you already have

You've chosen to use Kubernetes as the container orchestration tool!