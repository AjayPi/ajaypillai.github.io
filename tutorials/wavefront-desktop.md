# Adding Observability Using Python in a Kubernetes Environment with Wavefront

## Introduction

**Wavefront** is a cloud-based monitoring and analytics platform. It allows you to collect, visualize, and analyze metrics, traces, and histograms from your applications and infrastructure. To use Wavefront, you need to have a **Wavefront account**. As "the Company" has a corporate account, you should be able to sign in to Wavefront using your corporate credentialshas a subscription.

You also need to have a <a href="" title="A Wavefront cluster is a dedicated instance of Wavefront that hosts your data and dashboards."><sup>&#9432;</sup></a>**Wavefront cluster** and an <a href="" title="An API token is a secret key that allows you to send data to and query data from your Wavefront cluster."><sup>&#9432;</sup></a>**API token**. 

To send data to Wavefront, you need to use a <a href="" title="A Wavefront sender is a library or a tool that collects and formats data from your sources and sends it to Wavefront."><sup>&#9432;</sup></a>**Wavefront sender**. Wavefront supports various types of senders, such as SDKs, agents, proxies, and integrations. You can choose the sender that best suits your needs and environment.

This document provides a guide on how to add observability for desktop users using Python to connect to a Kubernetes environment. The observability portion will be handled by Wavefront.

## Prerequisites
- Python 3.x installed on your desktop
- Access to a Kubernetes environment
- Wavefront account

## Setting Up Your Environment

### Python Setup
Ensure that Python is correctly installed on your desktop. You can verify the installation by running the following command in your terminal:

```
python --version
```

### AWS Setup

To connect to AWS, you need to install and configure the AWS CLI (Command Line Interface). Here’s a basic guide:
1.	**Install the AWS CLI**: You can download and install the [AWS CLI](https://aws.amazon.com/cli/) from the official AWS website.
2.	**Configure the AWS CLI**: Once the AWS CLI is installed, you can configure it by running the `aws configure` command in your terminal. You’ll be prompted to enter your AWS Access Key ID, Secret Access Key, default region name, and default output format.
   
```
aws configure
AWS Access Key ID [None]: YOUR_ACCESS_KEY
AWS Secret Access Key [None]: YOUR_SECRET_KEY
Default region name [None]: YOUR_REGION
Default output format [None]: json
```
Replace `YOUR_ACCESS_KEY`, `YOUR_SECRET_KEY`, and `YOUR_REGION` with your actual AWS Access Key ID, Secret Access Key, and default region name.

### Kubernetes Setup

To connect to a Kubernetes environment hosted in AWS, you can use the AWS CLI and kubectl. Here’s a basic guide:
1.	**Install kubectl**: You can download and install [kubectl](https://kubernetes.io/docs/tasks/tools/) from the official Kubernetes website. The installation process varies depending on your operating system.
2.	**Connect to your Kubernetes cluster**: After configuring the AWS CLI, you can connect to your Kubernetes cluster using kubectl. First, you need to update the kubeconfig file for your cluster. You can do this with the following command:

```
aws eks --region region update-kubeconfig --name cluster_name
```

Replace `region` with your AWS region and `cluster_name` with the name of your Kubernetes cluster.

### Wavefront Setup

To set up your Wavefront account, follow the steps described below:

1.	**Sign in to Wavefront** : As we have a corporate account, you should be able to sign in to Wavefront using your corporate credentials.
2.	**Create a new user and assign roles**: Once signed in, you can create a new user for [yourself](https://docs.wavefront.com/user-accounts.html). N

## Connecting to Kubernetes with Python
In this section, you will learn how to connect to Kubernetes using Python. For example, you can use the following code snippet to list the pods with their IPs:

```python
from kubernetes import client, config

# Load config from default location
config.load_kube_config()

v1 = client.CoreV1Api()
print("Listing pods with their IPs:")
ret = v1.list_pod_for_all_namespaces(watch=False)
for i in ret.items:
    print(f"{i.status.pod_ip}, {i.metadata.namespace}, {i.metadata.name}")
```

## Example of usage: Monitoring CPU Usage of Pods

In this section, you will learn how to monitor the CPU usage of your pods using Wavefront. First, we need to fetch the CPU usage data from the Kubernetes cluster. We can use the Kubernetes Python client for this. Here’s an example:

```python
from kubernetes import client, config

# Load config from default location
config.load_kube_config()

v1 = client.CoreV1Api()

# Get the CPU usage of each pod
print("Listing pods with their CPU usage:")
ret = v1.list_pod_for_all_namespaces(watch=False)
for i in ret.items:
    cpu_usage = i.status.container_statuses[0].usage.cpu
    print(f"{i.metadata.namespace}, {i.metadata.name}, CPU usage: {cpu_usage}")
```

Next, we need to send the CPU usage data to Wavefront. We can use the Wavefront Python SDK for this: 

```python
import wavefront_sdk

# Create a Wavefront sender
sender = wavefront_sdk.WavefrontDirectClient(
    server="https://YOUR_CLUSTER.wavefront.com",
    token="YOUR_API_TOKEN"
)

# Send the CPU usage data as metrics
for i in ret.items:
    cpu_usage = i.status.container_statuses[0].usage.cpu
    namespace = i.metadata.namespace
    name = i.metadata.name
    sender.send_metric(
        name="pod.cpu.usage",
        value=cpu_usage,
        timestamp=None,
        source="python",
        tags={"namespace": namespace, "name": name}
    )

# Close the sender
sender.close()
```

Replace `YOUR_CLUSTER` and `YOUR_API_TOKEN` with your actual Wavefront cluster and API token.

Finally, we can visualize the CPU usage data in Wavefront. We can use the Wavefront UI or the Wavefront Query Language for this:

```python
pod.cpu.usage{source="python"}
```

This query will show the CPU usage of all the pods that we sent from Python. We can also filter by namespace or name using the tags. For example:

```python
pod.cpu.usage{source="python", namespace="default"}
```

### Create a line chart

Once the data is in Wavefront, you can create a line chart to visualize it. In Wavefront, navigate to Browse > Chart Builder. Select Line Chart as the chart type. In the Source field, enter the metric name `cpu.usage`. You can also add filters based on the tags we added earlier `namespace` and `pod`.

This query will show the CPU usage of the pods in the default namespace.

1. Create a Line Chart in Wavefront

- Log in to your Wavefront instance.
- Click on `Browse` > `Chart`.
- In the `Query Editor`, enter your query. For CPU usage, it might look something like this: `ts("cpu.usage")`.
- Press `Enter` or click `Run Query` to see the line chart.

2. Customize the Line Chart

- Fine-tune the metrics displayed on the chart by customizing the query and applying filters and functions.
- Use chart variables to set the time window, create alerts, and optimize display speed.
- Change the legend to customize how the chart looks.
- Set the time window on a chart to include metrics that stopped reporting.
- Use a logarithmic Y axis for skewed data.
- Filter out lines with minimum and maximum values.
- Use IEC/Binary prefixes in the Y axis and legends.
- Use dynamic units to optimize the display.

Remember to replace `cpu.usage` with the actual metric name you’re using. 

