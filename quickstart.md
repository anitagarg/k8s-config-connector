# Getting started with Kubernetes Config Connector

## Overview

This guide walks you through the basics of managing resources in Config Connector. 

Config Connector is a Kubernetes add-on that allows you to manage your GCP resources through Kubernetes configuration. 

In this tutorial, you manage a **Cloud Spanner** instance. Cloud Spanner is a scalable regional database service for regional and global application data. 

### Select a project

Select a GCP Console project to use for this tutorial.

<walkthrough-project-setup></walkthrough-project-setup>

## Setup

Every command requires a project ID. Set a default project ID so you do not need to provide it every time. 

```sh  
gcloud config set project {{project-id}}  
```

Enable the Kubernetes API, which you will need for this tutorial.

```sh  
gcloud services enable container.googleapis.com  
```

### Create an identity

A Config Connector cluster needs a GCP identity to communicate with other resources.

To set up the identity, first run the following command to create the `cnrm-system` Service Account:

```sh  
gcloud iam service-accounts create cnrm-system  
```

Next, give the IAM Service Account elevated permissions using the following command:

```sh  
gcloud projects add-iam-policy-binding {{project-id}}   
  --member serviceAccount:cnrm-system@{{project-id}}.iam.gserviceaccount.com   
  --role roles/owner  
```

Create a Service Account Key and export its credentials to a file:

```sh  
gcloud iam service-accounts keys create --iam-account   
 cnrm-system@{{project-id}}.iam.gserviceaccount.com key.json  
```

Create the `cnrm-system` namespace:

```sh  
kubectl create namespace cnrm-system  
```

Import the key's credentials as a **Secret**. In Kubernetes, Secrets are objects used to store sensitive data with encryption:

```sh  
 kubectl create secret generic gcp-key --from-file key.json --namespace cnrm-system  
```

Then, remove the credentials from your system:

```sh  
rm key.json  
```

### Install Config Connector

Install ConfigConnector on your cluster using `kubectl`:

```sh  
kubectl  
```

Enable the Cloud Spanner service using the following command:

```sh  
gcloud services enable spanner.googleapis.com  
```

## Organizing resources

Config Connector uses **Kubernetes Namespaces** to organize resources. 

Namespaces are virtual clusters that provide a way to divide cluster resources between multiple users.

To get started, you need a Namespace mapped to your GCP project. To create this Namespace, run the following command:

```sh  
kubectl create namespace {{project-id}}  
```

## Discovering available resources

To view available resources in Config Connector, run:

```sh  
kubectl get crds --selector cnrm.cloud.google.com/managed-by-kcc=true  
```

As an example, you can view the API description for the `SpannerInstance` resource with `kubectl describe`:

```sh  
kubectl describe crd spannerinstances.spanner.cnrm.cloud.google.com  
```

## Exploring `spannerinstance.yaml`

Many of the example tasks use a file named `spanner-instance.yaml`. To view this file, first use the following command:

```sh  
cd  
```

Then, run the following command to access the correct directory:

```sh  
cd k8s-config-connector/apps/bookstore/config/manifests  
```

To open the file, run the following command:  
```sh  
cloudshell edit spanner-instance.yaml  
```

## Creating a resource

When you create a Config Connector resource, an associated GCP resource is created. Config Connector manages this resource. However, if a GCP resource already exists with the same name, then Config Connector acquires the resource and manages it.

Use the `kubectl apply` command to create resources. To create the Cloud Spanner instance, use the following command:

```sh  
kubectl --namespace {{project-id}} describe spannerinstance spannerinstance-sample  
```

## Describing a resource

To view a resource, use `kubectl describe`.

To view the Cloud Spanner instance, run the following command:

```sh  
kubectl --namespace {{project-id}} describe spannerinstance spannerinstance-sample  
```

## Updating a resource

First, modify the `spanner-instance.yaml` file to change `spec.displayName` from "Bookstore" to "Display Name":

To open the file, run the following command:  
```sh  
cloudshell edit spanner-instance.yaml  
```

Then, run the following command to change `spec.displayName`: 

```sh  
sed -i -e 's/Bookstore/Display Name/g' spanner-instance.yaml  
```

To update the resource, use `kubectl apply`:

```sh  
kubectl --namespace {{project-id}} apply -f spanner-instance.yaml  
```

Check the Cloud Spanner instance for the change in name:

```sh  
kubectl --namespace {{project-id}} describe spannerinstance spannerinstance-sample  
```

## Deleting a resource

**If you delete a Config Connector resource, the associated GCP resource is deleted by default. **

To keep the GCP resource, add the following to the Config Connector resource YAML before deleting:

```  
...  
metadata:  
  annotations:  
    cnrm.cloud.google.com/deletion-policy: abandon  
...  
```

To delete resources, use `kubectl delete`:

```sh  
kubectl --namespace {{project-id}} delete -f spanner-instance.yaml  
```

## What's Next

+   Learn about how Config Connector models GCP resources with [Kubernetes constructs](https://cloud.google.com/config-connector/docs/concepts/resources).
+   See the [GCP resources](https://cloud.google.com/config-connector/docs/reference/resources) Config Connector can manage. 
