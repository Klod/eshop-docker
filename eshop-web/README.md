# eShopOnWeb - running locally with Docker and in Azure with AKS

Welcome to eShop on Web!

This source code was scaffolded by a [Docker Application Template](https://github.com/sixeyed/eshoponweb-template) which also generated:

- a [Dockerfile](./src/Web/Dockerfile) to build, test and package the web app;
- a [Docker Compose](../docker-compose.yaml) file to run the app locally in containers;
- a [Docker Compose](../docker-compose.production.yaml) file to run the app in Azure.

> The output includes GitHub actions for creating an Azure Kubernetes Service cluster and a SQL Azure database, and for deploying the app to the cluster.

## Push this repo to create your AKS cluster & SQL Azure database

You should have a [GitHub repo set up](TODO) with all the secrets stored for the pipeline to create Azure resources.

Open a terminal for this folder and set up the GitHub remote:

```
git init
git add --all
git commit -m "A template built all this!"
git remote add origin https://github.com/kirill/eshop-docker.git
```

Push the code to GitHub. This will start workflows to create your Azure resources:

```
git push -u origin master
```

[You can check on the workflows here](https://github.com/klod/eshop-docker/actions). It make take a minute or so before the actions start. The workflows use secrets from your repo to connect to Azure. Then they:

- create an AKS cluster
- deploy [Helm](https://helm.sh) on the cluster
- deploy [Compose on Kubernetes](https://github.com/docker/compose-on-kubernetes) on the cluster
- create a SQL Server instance with two databases

Setting up the AKS cluster can take 10 minutes or more, but it's a one-off job that only happens when you first push your code.

## Meanwhile... run the app locally

Docker Application Designer uses the [docker-compose.yaml](../docker-compose.yaml) manifest to build and run the app locally.

Click _Start_ in Application Designer in the app isn't already running. Then browse to:

- http://localhost:5100

And you can also connect to your local SQL Server container using a SQL client - [Sqlectron](https://sqlectron.github.io) is a good choice:

- server address: `localhost:5433`
- username: `sa`
- password: `Pass@word`

## Check the Code

The app is an ASP .NET Core MVC web app, with some core best practices for running in Docker and Kubernetes:

- the [Dockerfile](./src/Web/Dockerfile) is a multi-stage build which compiles, tests and packages the app
- [configuration files](./src/Web/Program.cs) are loaded from multiple locations, so they can be provided by the platform
- there are [healthchecks](./src/Web/HealthChecks/ApiHealthCheck.cs) in the code which tell the container platform if the app is healthy

You can edit the source code and click _Restart_ from Docker Application Designer to see your changes - try a simple UI change in [\_Layout.cshtml](./src/Web/Views/Shared/_Layout.cshtml).

## Deploy to Kubernetes

Your AKS cluster has been provisioned with [Compose on Kubernetes](https://github.com/docker/compose-on-kubernetes), so you can use **the exact same** Docker Compose format to deploy to AKS.

And your GitHub repo has another Action which deploys your app using the generated [docker-compose.production.yaml](../docker-compose.production.yaml) file when you push changes.

Now that you've edited the UI, push your changes to deploy the app to AKS:

```
git add --all
git commit -m 'I edited this myself'
git push origin master
```

[Check on the GitHub actions](https://github.com/kirill/eshop-docker/actions) to see when the _Deploy to AKS_ workflow has completed.

## Browse to the app on AKS

Your app is running now in Azure and you can browse to the Kubernetes dashboard on AKS to see the deployment.

Launch an [Azure Cloud Shell](https://shell.azure.com) session and connect to your AKS cluster:

```
az aks get-credentials --resource-group klod-eshop-docker --name kirill-aks
```

Then you can use the `kubectl` command line to work with Kubernetes, or launch the web dashboard if you prefer to use the UI.

Using the command line you'll find the IP address for your application by fetching the service details:

```
kubectl get service eshopwebmvc-published
```

> Browse to the IP address in the `EXTERNAL_IP` field and you'll see your live app, running in Kubernetes and connected to a SQL Azure database.

Or you can run the Kubernetes dashboard:

```
az aks browse --resource-group klod-eshop-docker --name kirill-aks
```

> A new browser tab opens here showing the Kubernetes dashboard. Sometimes the tab opens before the dashboard is ready and you'll see a connection error - just refresh that tab and it will be fine.

From here you can see:

- the deployment generated from the Docker Compose file
- the secret containing the CosmosDB connection string
- the pod running the demo app
- the service with an external endpoint to access the app

> You can continue to work on the code locally, push to GitHub and your changes will be deployed to AKS

### Credits

Original source code: [dotnet-architecture/eShopOnWeb](https://github.com/dotnet-architecture/eShopOnWeb).
