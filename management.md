# 1. Testing, Upgrade, Management and Scaling
Welcome to this technical tutorial on Azure Load Testing Service (ALT), Scalability and Upgrade of AKS cluster. In this chapter, we will focus on making our application scalable, generate artificial load against the AKS cluster utilizing ALT and upgrade an AKS cluster. 

Let’s begin!

# 2. Validating the environment

## 2.1 Validate Auto Cluster is configured on the AKS cluster.

1) Navigate to the Azure portal at https://portal.azure.com and enter your login credentials
> [!IMPORTANT]
> Ensure you do not login with your company mail address.
2) Click on **Resource groups** under **Azure Services**.
   
3) Click on the **Resource group name** that you created earlier in previous chapter.
   
4) Click on the Azure Kubernetes Service called **k8s**.
   
5) On your left hand side menu expand **Settings** and click **Node pools**.
   
6) Validate that Scale method is set to **AutoScale**. **Target nodes** is set to **1** and **Ready nodes** is set to **1**.

# 3. Hands-On Exercises

## 3.1 Configure Horizontal Pod Auto Scaler.

Horizontal Pod Autoscaler (HPA) automatically adjusts the number of pod replicas based on observed metrics like CPU utilization. This ensures efficient handling of varying loads, optimizing resource usage, and maintaining application performance. In this tutorial, we will focus on CPU utilization. If the CPU utilization exceeds 50%, the HPA will scale out. Note that in the manifest file, the CPU request is 150 milli-cores, which means the average CPU usage is 75 milli-cores. Refer to the previous deployment file for further analysis: [Deployment Manifest](https://github.com/pelithne/aks_workshop/blob/main/manifests/deployment.yaml)

1) Before enabling HPA, we need to identify the deployment name for our pod. Launch Azure Cloud Shell if not already opened, and execute the following commands.

````bash
kubectl get deployments
````
The output is similar to:

````bash
 kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
azure-vote-back    1/1     1            1           3h29m
azure-vote-front   1/1     1            1           3h29m
````

2) Enable **HPA** for both **azure-vote-back** and **azure-vote-front** with an average CPU utilization of **50%**, and the pods should scale out to a max of **10 pods**, and scale back to a minimum of **1 pod**.

````bash
kubectl autoscale deployment azure-vote-back --cpu-percent=50 --min=1 --max=10

kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=1 --max=10
````

3) Validate that HPA has been enabled for both deployments.

````bash
kubectl get hpa
````

The output is similar to:

````bash

kubectl get hpa
NAME               REFERENCE                     TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
azure-vote-back    Deployment/azure-vote-back    cpu: 1%/50%   1         10        1          57s
azure-vote-front   Deployment/azure-vote-front   cpu: 0%/50%   1         10        1          48s
````
## 3.2 Configure Azure Load Testing Service.

Great job reaching this point! We now have the Horizontal Pod Autoscaler (HPA) configured and cluster autoscaling enabled on the AKS cluster. This means that your AKS cluster is set up to automatically adjust the number of pods based on CPU usage, and it can also scale the number of nodes as needed. Now, you need to create simulated traffic to test the scalability and performance of your setup utilizing ALT. Before we jump into to nitty gritty, we will conduct a scenario based load test which will help us to define a Test criteria and Load Pattern design.

## 3.2.1 Scenario

As a tester, you are responsible for running a load test on a company's internet-facing landing page web application, which is deployed on an Azure Kubernetes Service (AKS) cluster. The application owner expects the web application to provide fast and consistent responses to visitors, with an average latency of no more than 800 milliseconds. The application owner also wants to keep the cost of running the web application as low as possible, utilizing HPA and cluster autoscaler to optimize resource usage. You will use Azure Load Testing Service to generate realistic and scalable load on the web application. Based on the requirements, we will use Azure Load Testing Service to create the testing criteria.

### 3.2.2 Test Criteria

1. **Test Duration**: 5 minutes
2. **Request Type**: HTTP GET
3. **Target URL**: The IP of the `azure-vote-front` service
4. **Success Criteria**:
   - Response time should be less than 800 milliseconds for 95% of the requests
   - Error rate should be less than 1%

### 3.2.3 Load Pattern Design

1. **Initial Users**: 10 users
2. **Ramp-Up**: Increase the number of users by 100 every minute until reaching 500 users
3. **Steady State**: Maintain 500 users for the remaining duration of the test

### 3.2.4 Configure Azure Load Testing Service

1) We need to identify the endpoint of the AKS cluster, where we will generate load towards, copy the public IP address.
   
````bash
kubectl get service azure-vote-front -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
````
2) In the Azure Portal, type **Azure Load Testing** in the search field.
3) From the drop down menu choose **Azure Load Testing**.
4) Click on the button called **Create Azure load testing resource**.
   
> [!Note]
> Azure Load Test resource is essentially just a container which holds your different tests.

5) Select your **Subscription** from the drop down menu.
6) Select your existing **resource group**. I.e the same where you have deployed your AKS instance.
7) Provide a name for your **Azure Load Testing Resource**.
8) Select **Sweden Central** as your region for deployment.
9) Click on **Review + create**.
10) Click **Create**.
11) Once the deployment is finished, click on the **Home >** breadcrumb on the top left hand side of the portal.
12) Click on **Resource groups**.
13) Click on your **Resource group**
14) Within the resource group, you should be able to locate your **Azure Load Testing resource** Click on the resource.
15) On your left hand side menu, expand **Tests**
16) Click on **Tests**
17) Click on **Create** drop down menu and choose **Create a URL-based test**
18)  Provide a **test name** for your test environment
19)  Provide a test description for your test environment for example: **Testing load against my app in Azure Kubernetes Service**
20)  Check the box **Run test after creation**
21)  Click **Next** You should now be in Test plan view.
22)  Click **Add request**.
23)  In the Request name type **AksEndPoint**
24) In the URL field, provide the IP for your exposed pod in AKS, as this is the endpoint we will be generating load for, in the following format "http://xxx.xxx.xxx"

Let me guess... you forgot the Public IP, didn't you? No worries, we've got your back! 😄

````bash
kubectl get service azure-vote-front -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
````
25)  keep the HTTP method as is, it should be set to **GET** by default.
26)  Click on **Add**.
27)  Click **Next**.
28)  Click **Next**.
29)  You should now be in the Load view,lets base it on the load pattern design. in the engine instances set it to **2**
30)  Given the requirement, a linear load pattern is suitable as it gradually increases the number of users over time, thus keep the load pattern as is, it should be set to **Linear**.
31)  In the concurrent users per engine set it's value to **250**.
32)  Test Duration should be set to **5**. 
33)  Ramp-up time set it to **5**.
34)  Now we are finished with our load pattern design, Click **Next**.
35)  In the Monitoring view, lets add our AKS cluster and also our nodepools as well, so we can monitor the server side, during performance test. Click **Add/Modify**.
36)  Add the **Virtual Machine Scale Set** and **Kubernetes Service** and then press **Apply**
37)  Press **Next**
38)  In the Test criteria view, lets define the passing criteria as described in the Test criteria chapter. Response time should be less then 800 milli seconds. Under Metric drop down menu choose **Response time**.
39)  Under Aggregation drop down menu choose **Average**.
40)  Under Condition drop down menu choose **Greater than**.
41)  Under Request name choose your request name called **AksEndPoint**.
42)  Under Threshold choose **800** and the unit will automatically set to **ms**.
43)  We also need to define the Error rate, create a new condition, under metric choose **Error**.
44)  Under Aggregation choose **Percentage**.
45)  Under Condition choose **Greater than**.
46)  Under Request name choose your request name called **AksEndPoint**.
47)  Set the Threshold to **1** and the unit should automatically be set to **%**
48)  Click on **Review + Create**
49)  Click on **Create**
50)  The Azure load test will now create the necessary compute engines to generate load towards your endpoint:
    
We have now configured:

- **Response Time Condition**: Ensures that the average response time is less than 800 milliseconds. If the response time exceeds this threshold, the test fails.
- **Error Rate Condition**: Ensures that the error rate is less than 1%. If the error rate exceeds this threshold, the test fails.
  
51)   Monitor the metrics in Azure Load testing service.   
52)   In Azure Cloud Shell Monitor pods scaling and also your nodes as well.
    
    ````bash
    kubectl get hpa --watch 
    kubectl get nodes --watch
    ````
53) Once the actual load test is complete, wait approximately 5 minutes. You should see that the HPA has started to scale down the pods from 10 to 1. After the pods have scaled down, wait an additional 10 minutes for the nodes to scale back down to 1.
    
    ````bash
    kubectl get hpa --watch 
    kubectl get nodes --watch
    ````

### 3.2.5 Upgrading AKS Cluster

Upgrading your AKS cluster ensures that you are running the latest version of Kubernetes, which includes new features, security patches, and performance improvements. Follow these steps to upgrade your AKS cluster:

1) Before upgrading, check the current Kubernetes version of your AKS cluster.

```bash
   kubectl get nodes
```
2) Check available Kubernetes Versions:
   
```bash
   az aks get-upgrades --resource-group <resource-group-name> --name k8s --output table
```
3) Take the latest version from the list and conduct the upgrade:
   
```bash
   az aks upgrade --resource-group <resource-group-name> --name k8s --kubernetes-version <desired-version> --yes &
```
> [!Note]
> Upgrade can take up to 10 mins.

4) Monitor the uprgade process:
   
```bash 
   az aks show --resource-group <resource-group-name> --name k8s --query "provisioningState"
```
5) Verify the Upgrade:
   
```bash
   kubectl get nodes
```

### 3.2.6 Reflecting on Test Results (Optional)

After running the load test, take some time to reflect on the results and consider how you can improve your application's performance.

1. **Analyze the Test Results**:
   - Review the response time and error rate metrics in the Azure Load Testing Service.
   - Check if the response time was consistently below 800 milliseconds for 95% of the requests.
   - Verify if the error rate was below 1%.

2. **Monitor Server-Side Metrics**:
   - Use the Azure Portal to monitor the server-side metrics for your AKS cluster, including CPU and memory usage for both pods and nodes.
   - Check the Horizontal Pod Autoscaler (HPA) and node autoscaler metrics to see if they scaled as expected.

3. **Reflect on the Observations**:
   - Did the application meet the performance criteria? If not, what were the main issues?
   - Were there any unexpected spikes in response time or error rates?
   - Did the HPA and node autoscaler work as expected to handle the load?

4. **Identify Areas for Improvement**:
   - **Optimize Resource Requests and Limits**: Ensure that the CPU and memory requests and limits for your pods are set appropriately to avoid over or under-provisioning.
   - **Improve Application Code**: Identify any bottlenecks in your application code that could be optimized for better performance.
   - **Enhance Autoscaling Policies**: Adjust the HPA and node autoscaler policies to better handle varying loads.
   - **Use Caching**: Implement caching strategies to reduce the load on your backend services.
   - **Optimize Database Queries**: Ensure that your database queries are optimized for performance.

5. **Plan for Future Tests**:
   - Based on the insights gained, plan for additional load tests with different scenarios to further validate and improve your application's performance.

By reflecting on these aspects, you can gain valuable insights into your application's performance and identify actionable steps to enhance its scalability and reliability.
