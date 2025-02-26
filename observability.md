## Using Azure Monitor with Managed Prometheus, Grafana, and Container Insights


In this section you will be using Azure Monitor with managed Prometheus, Grafana, and Container Insights. By the end of this tutorial, you will have a basic understanding of how to monitor your AKS cluster using these tools.



### Objectives
Enable Container Insights
Enable managed Prometheus and MAnaged Grafana
Create and view dashboards


### Introduction
This section describes how to enable complete monitoring of your Kubernetes clusters using the following Azure Monitor features:

* Managed Prometheus for metric collection
* Container insights for log collection
* Managed Grafana for visualization.

Using the Azure portal, you can enable all of the features at the same time. You can also enable them individually by using the Azure CLI, Azure Resource Manager template, Terraform, or Azure Policy. In these instructions, we use the azure CLI.

#### Add metrics add-on to scrape Prometheus metrics

Use the ````--enable-azure-monitor-metrics```` option for ````az aks update````  to install the metrics add-on that scrapes Prometheus metrics.

````
az aks update --enable-azure-monitor-metrics --name <cluster-name> --resource-group <cluster-resource-group>
````





In the first section of the workshop, you created a Kubernetes cluster using AKS. The cluster was created with the following options, which makes sure that **Managed Grafana** and **Managed Prometheus** is enabled:

````

````


#### Create a dashboard:

In Grafana, go to Dashboards > New Dashboard.
Add a new panel and select Prometheus as the data source.
Create visualizations based on your metrics.

#### Create a CPU Usage Dashboard

Create a new dashboard in Grafana.
Add a panel to display CPU usage metrics from Prometheus.


#### Monitor Pod Memory Usage

Add another panel to your dashboard to monitor memory usage of pods.
