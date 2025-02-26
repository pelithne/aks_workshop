## Using Azure Monitor with Managed Prometheus, Grafana, and Container Insights


### Introduction
Welcome to this tutorial on using Azure Monitor with managed Prometheus, Grafana, and Container Insights. This guide is designed for inexperienced AKS users and should take around 45 minutes to complete. By the end of this tutorial, you will have a basic understanding of how to monitor your AKS cluster using these tools.

### Objectives
Enable Container Insights
Set up managed Prometheus
Integrate Grafana with Prometheus
Perform hands-on exercises to create and view dashboards



### Enable monitoring (if not done in previsous step)

````az aks enable-addons --resource-group <myResourceGroup> --name <myAKSCluster> --addons monitoring````

Verify monitoring is enabled:

````az aks show --resource-group <myResourceGroup> --name <myAKSCluster> --query addonProfiles.omsagent.enabled````


#### Setting Up Managed Prometheus
Now, let's set up managed Prometheus for monitoring.

Create a Log Analytics workspace:

````az monitor log-analytics workspace create --resource-group myResourceGroup --workspace-name myWorkspace````

#### Enable Prometheus metrics:

````az aks update --resource-group myResourceGroup --name myAKSCluster --enable-managed-prometheus````


#### Integrating Grafana
We'll integrate Grafana with Prometheus and display dashboards.

Create a Grafana instance:

````az monitor grafana create --resource-group myResourceGroup --name myGrafana````

#### Configure Grafana to use Prometheus:

Open the Azure portal and navigate to your Grafana instance.
Go to Configuration > Data Sources.
Add Prometheus as a data source and provide the necessary details.

#### Create a dashboard:

In Grafana, go to Dashboards > New Dashboard.
Add a new panel and select Prometheus as the data source.
Create visualizations based on your metrics.

#### Create a CPU Usage Dashboard

Create a new dashboard in Grafana.
Add a panel to display CPU usage metrics from Prometheus.


#### Monitor Pod Memory Usage

Add another panel to your dashboard to monitor memory usage of pods.
