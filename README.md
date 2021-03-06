# azure-kubernetes-couchbase

This is a walkthrough of setting the [Couchbase Operator](https://blog.couchbase.com/introducing-couchbase-operator/) up on [Azure Container Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/).

## Deploy an AKS Cluster

AKS is currently in public preview.  There are a bunch of [nice tutorials](https://docs.microsoft.com/en-us/azure/aks/) on how to use it.

For this walkthrough we're going to use the Azure 2.0 CLI.  If you haven't already, you'll need to [install that and login](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli).  Even if you have the CLI installed already, you might want to update it.  An alternative would be to do all this in a [cloud shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview), though you'll eventually need to get kubectl working locally in order to set up a tunnel to open a web browser to your Couchbase cluster.

To make sure the Azure 2.0 CLI is working properly try running:

    az group list

With that all set, we can register the AKS provider.  Presumably once the service goes GA this won't be necessary.

    az provider register -n Microsoft.ContainerService

If that runs well, you should see:

![provider](/images/provider.png)

Now you're ready to create a resource group and a cluster inside that:

    az group create --name myResourceGroup --location eastus
    az aks create --resource-group myResourceGroup --name myAKSCluster

Note that it's way easier to do this from the CLI because the CLI creates the service principal you need automatically.  If you do this in the portal you'll need to create the service principal manually.

![deploying](/images/deployed.png)

When that's done, or even while it's running, you can login to the [Azure Portal](https://portal.azure.com) and take a look at your new cluster there as well:

![portal](/images/portal.png)

## Configure kubectl

Now that we have a cluster, the next step is to install and set up kubectl up so it can connect to the cluster.

    az aks install-cli
    az aks get-credentials --resource-group=myResourceGroup --name=myAKSCluster

You should see something like this:

![getcreds](/images/getcreds.png)

You might need to set your KUBECONFIG too:

    export KUBECONFIG=~/.kube/config

With all that set up we can make sure our kubectl is working by running:

    kubectl get nodes

That should show three nodes:

![getnodes](/images/getnodes.png)

## Deploying the Operator

Once you have an AKS cluster deployed and a running kubectl, you're ready to deploy the Operator.  The documentation on that is [here](http://docs.couchbase.com/prerelease/couchbase-operator/beta/overview.html).

To create the deployment and check it deployed, run this:

    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/operator.yaml
    kubectl get deployments

You should see something like this:

![operatordeployed](/images/operatordeployed.png)

## Deploying a Couchbase Cluster

We're there!  Time to get a live cluster.  Run this:

    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/secret.yaml
    kubectl create -f https://s3.amazonaws.com/packages.couchbase.com/kubernetes/beta/couchbase-cluster.yaml

That should give this:

![couchbasecreated](/images/couchbasecreated.png)

You can view the Couchbase and operator pods by running:

    kubectl get pods

## Accessing the Couchbase Web UI

You've now got a cluster.  But to use it you probably want to set up port forwarding.  To do that run:

    kubectl port-forward cb-example-0000 8091:8091

Leave that command running:

![portforward](/images/portforward.png)

Now open up a browser to http://localhost:8091

![loginscreen](/images/loginscreen.png)

The username is `Administrator` and password is `password`.  And now you're in!

![webui](/images/webui.png)
