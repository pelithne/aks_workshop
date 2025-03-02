# Using Azure Monitor with Managed Prometheus and Grafana

In this section you will be using Azure Monitor with managed Prometheus and Grafana. By the end of this tutorial, you will have a basic understanding of how to monitor your AKS cluster using these tools.



## Objectives
* Enable managed Prometheus and Managed Grafana
* Create and view dashboards


## Introduction
This section describes how to enable complete monitoring of your Kubernetes clusters using the following Azure Monitor features:

* Managed Prometheus for metric collection
* Managed Grafana for visualization.


Using the Azure portal, you can enable all of the features at the same time. This is very simple, but we are here to learn! :-)

You can also enable them individually by using the Azure CLI, Azure Resource Manager template, Terraform, or Azure Policy. In these instructions, we use mainly the azure CLI.

### Add metrics add-on to scrape Prometheus metrics

Use the ````--enable-azure-monitor-metrics```` option for ````az aks update````  to install the metrics add-on that scrapes Prometheus metrics. 



````
az aks update --enable-azure-monitor-metrics --name <cluster-name> --resource-group <cluster-resource-group>
````
 
This will install a number of pods in your kubernetes cluster, with names similar to ````ama-metrics-85c8894994-lkp5d ```` which are responsible for scraping metrics from the AKS applications and infrastructure, and forwarding it to Azure Monitor. The command also installs the retina-agent, which you can read more about here: https://github.com/microsoft/retina

You can check this using ````kubectl get pods```` and look specifically for pods in the ````kube-system```` namespace. 

````
kubectl get pods -n kube-system
````

This should give an output similar to this (note how some pods are "younger" than the others)

````
NAME                                            READY   STATUS    RESTARTS      AGE
ama-metrics-85c8894994-lkp5d                    2/2     Running   0             63s
ama-metrics-85c8894994-rfc8k                    2/2     Running   0             63s
ama-metrics-ksm-5bd68b9c-m9fjj                  1/1     Running   0             63s
ama-metrics-node-qgpb5                          2/2     Running   0             63s
ama-metrics-operator-targets-79cd8c5446-j59n6   2/2     Running   2 (54s ago)   63s
azure-cns-pf265                                 1/1     Running   0             32m
azure-ip-masq-agent-rfqz5                       1/1     Running   0             32m
azure-policy-64fb7f857b-gcwg5                   1/1     Running   0             23m
azure-policy-webhook-5bc64ff56d-dh7xw           1/1     Running   0             23m
cloud-node-manager-vb5j8                        1/1     Running   0             32m
coredns-659fcb469c-hhzzm                        1/1     Running   0             33m
coredns-659fcb469c-mcjgd                        1/1     Running   0             32m
coredns-autoscaler-bfcb7c74c-sq9bd              1/1     Running   0             33m
csi-azuredisk-node-z4h5v                        3/3     Running   0             32m
csi-azurefile-node-w7kfq                        3/3     Running   0             32m
konnectivity-agent-5d8cb46ff4-p46gw             1/1     Running   0             12m
konnectivity-agent-5d8cb46ff4-qlxk5             1/1     Running   0             12m
kube-proxy-gwl7x                                1/1     Running   0             32m
metrics-server-5dfc656944-hn9nv                 2/2     Running   0             32m
metrics-server-5dfc656944-jfd9n                 2/2     Running   0             32m
retina-agent-qdhdw                              0/1     Running   0             55s

````

### Work with Grafana

Now that we have configured the log and metrics collection, we can go ahead and create a grafana dashboard. 

#### Create Managed Grafana Instance

In the search field in the Azure Portal, write **grafana**. In your search results, you should see **Azure Managed Grafana**

<img src="./media/search-managed-grafana.png" alt="Search" width="50%">

When you click on **Azure Managed Grafana** you will be presented with the option to create a managed grafana instance. 

<img src="./media/no-managed-grafana.png" alt="Search" width="50%">



Click on **Create Azure Managed Grafana**

Now, you need to configure your Grafana, just like when creating any Azure resource with the portal. 

You only have to care about the **Basics**

| Property           | Value                                                                |
|--------------------|----------------------------------------------------------------------|
| Subscription       | Use the subscription that you received for this workshop             |
| Resource group name| Use the one you created earlier for the AKS cluster                  |
| Location           | Sweden Central                                                       |
| Name               | Globally unique name (for example based on your corporate signum)    |
| Pricing Plan       | Standard                                                             |
| Grafana Version    | 10     

Then select **Create** and after validation click on the **Create** button.

#### Link Grafana workspace 
When you configured the Prometheus scraping, an **Azure Monitor Workspace** was automatically created for you. For **Grafana** to be able to use the metrics from Prometheus, a connection to that workspace needs to be configured. 

The easiest way to find the workspace is to search in the Azure Portal. Search for **Azure Monitor Workspace**

<img src="./media/search-azure-monitor-workspace.png" alt="Search" width="50%">


Click on **Azure Monitor Workspace** in the search results. Then select **Linked Grafana Workspaces** in the left hand navigation bar.

<img src="./media/linked-grafana-workspaces.png" alt="Search" width="30%">

There will be no linked Grafana Workspaces available, so select **Link**.

When you click to expand the **Grafana Workspace** field, you will find the Grafana workspace previosly created. 

<img src="./media/select-grafana-workspace-to-link.png" alt="Search" width="50%">

Select it, and click on the **Link** button.



#### Create your first dashboard

When the **Managed Grafana** instance was created, you probably saw something like this **Go to resource**


<img src="./media/grafana-go-to-resource.png" alt="Search" width="50%">

If you navigated away from this, you can simply search for **Grafana** again in the search field of the Azure Portal.


In the **Overview** of your Grafana resource, there will be a link to your **Grafana Endpoint** which will look similar to ````https://k8s-ws-grafana-98765-hhh6gsfgbmaacjba.cse.grafana.azure.com````. 

Click on that link to go to your Grafana instance. You will be able to log into Grafana using the same username and credentials that you used for the Azure Portal.

In the left hand navigation pane, select **Dashboards**

<img src="./media/grafana-navigation.png" alt="Search" width="30%">

You should see three categories, Azure Managed Prometheus, Azure Monitor and Microsoft Defender for Cloud. The one you are interested in today, is the Azure Managed Prometheus. 


<img src="./media/grafana-dashboards.png" alt="Search" width="50%">

Expand **Azure Managed Prometheus** to find the preconfigured dashboards underneath. Select (for example) the one named **Kubernetes / Compute Resources / Node (Pods)**

This dashboard container metrics for CPU and Memory utilization in the pods. In the lists on the right hand side, next to the graphs you can select for which pod you want to see metrics.

<img src="./media/grafana-node-pod-dasboard.png" alt="Search" width="50%">


#### If you feel that you have the time, play around with the different dashboards or create a custom dashboard if you are already familiar with Grafana and Prometheus.










