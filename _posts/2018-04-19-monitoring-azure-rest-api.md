---
layout: post
title: Metrics Monitoring provider in Azure REST API
description: General Monitoring Endpoints and Storage Endpoints
date: 2018-04-19
categories: saas azure storage rest microsoft api cosmosdb monitoring
img: 2018/monitoring-provider-rest-api.jpg
author: Carlos Raffellini
---

# Metrics Monitoring provider in Azure

- [Metrics Monitoring provider in Azure](#metrics-monitoring-provider-in-azure)
    - [1 Authentication](#1-authentication)
    - [2 Metrics and Metric Definitions](#2-metrics-and-metric-definitions)
        - [2.1 Metric Definitions Endpoint](#21-metric-definitions-endpoint)
        - [2.2 Metrics Endpoint](#22-metrics-endpoint)
- [3 Monitoring REST Endpoint in Storage Account](#3-monitoring-rest-endpoint-in-storage-account)
    - [3.1 Storage Account Metrics](#31-storage-account-metrics)
        - [3.1.1 Metric Definitions](#311-metric-definitions)
    - [3.2 Blog Service Metrics](#32-blog-service-metrics)
        - [3.2.1 Metric Definitions](#321-metric-definitions)
        - [3.2.2 Metric](#322-metric)
    - [4 Conclusion](#4-conclusion)
- [References](#references)

## 1 Authentication

Monitor API Authentication is usual Resource Group Management REST Authentication.

## 2 Metrics and Metric Definitions

The monitor REST API has two endpoints directly related to metrics one retrieves *metric definitions* and the other retrieve a *metric time series*.

### 2.1 Metric Definitions Endpoint

`GET https://management.azure.com/{resourceUri}/providers/microsoft.insights/metricDefinitions?api-version=2018-01-01`

An example of `resourceUri` is `subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}`.

This endpoint returns the description of the metrics. For instance 

```json
    ...
    {
        "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}/providers/microsoft.insights/metricdefinitions/SVRBSY",
        "resourceId": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}",
        "namespace": "Microsoft.EventHub/namespaces",
        "name": {
            "value": "SVRBSY",
            "localizedValue": "Server Busy Errors"
        },
        "isDimensionRequired": false,
        "unit": "Count",
        "primaryAggregationType": "Total",
        "supportedAggregationTypes": [
            "None",
            "Average",
            "Minimum",
            "Maximum",
            "Total",
            "Count"
        ],
        "metricAvailabilities": [
            {
                "timeGrain": "PT1M",
                "retention": "P93D"
            },
            {
                "timeGrain": "PT5M",
                "retention": "P93D"
            },
            {
                "timeGrain": "PT15M",
                "retention": "P93D"
            },
            {
                "timeGrain": "PT30M",
                "retention": "P93D"
            },
            {
                "timeGrain": "PT1H",
                "retention": "P93D"
            },
            {
                "timeGrain": "PT6H",
                "retention": "P93D"
            },
            {
                "timeGrain": "PT12H",
                "retention": "P93D"
            },
            {
                "timeGrain": "P1D",
                "retention": "P93D"
            }
        ]
    },
    ...
```

### 2.2 Metrics Endpoint

`GET https://management.azure.com/{resourceUri}/providers/microsoft.insights/metrics?api-version=2018-01-01`

An example of `resourceUri` is `subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}`.

By default, this endpoint retrieves only one metric. Here is an example of the same resource we got the Metric Definitions before.

`GET https://management.azure.com/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace}/providers/microsoft.insights/metrics/?api-version=2018-01-01&metricnames=SVRBSY&interval=PT1H`

The interesting thing is in the last bit. We are retrieving the metric called `SVRBSY` and we aggregate by an hour (`SVRBSY`). This is not retrieved by default. Here is an example of the response.

```json
{
    "cost": 0,
    "timespan": "2018-04-18T13:36:16Z/2018-04-18T14:36:16Z",
    "interval": "PT1H",
    "value": [
        {
            "id": "subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace}/providers/microsoft.insights/metrics/SVRBSY",
            "type": "Microsoft.Insights/metrics",
            "name": {
                "value": "SVRBSY",
                "localizedValue": "Server Busy Errors"
            },
            "unit": "Count",
            "timeseries": [
                {
                    "metadatavalues": [],
                    "data": [
                        {
                            "timeStamp": "2018-04-18T13:36:00Z",
                            "total": 0
                        }
                    ]
                }
            ]
        }
    ],
    "namespace": "Microsoft.EventHub/namespaces",
    "resourceregion": "westeurope"
}
```

There are more parameters for this request we are not considering here. There are plenty of stuff you can do. Here is the [doc of the endpoint](https://docs.microsoft.com/en-us/rest/api/monitor/metrics/list). Here is the list of [metric names we can retrieve by resource](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-supported-metrics#microsofteventhubnamespaces).


# 3 Monitoring REST Endpoint in Storage Account

When we retrieve metrics from Storage Account we can retrieve from the Storage account or from any of the services (Blob, File, Table or Queue).

## 3.1 Storage Account Metrics

### 3.1.1 Metric Definitions

`GET https://management.azure.com/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/providers/microsoft.insights/metricDefinitions/?api-version=2018-01-01`

Example of the response:

```json
{
    "value": [
        {
            "id": "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/providers/microsoft.insights/metricdefinitions/UsedCapacity",
            "resourceId": "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
            "namespace": "Microsoft.Storage/storageAccounts",
            "category": "Capacity",
            "name": {
                "value": "UsedCapacity",
                "localizedValue": "Used capacity"
            },
            "isDimensionRequired": false,
            "unit": "Bytes",
            "primaryAggregationType": "Average",
            "supportedAggregationTypes": [
                "Total",
                "Average",
                "Minimum",
                "Maximum"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ]
        },
        ...
    ]
}
```

All the metrics supported metrics can be found in [here](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-supported-metrics#microsoftstoragestorageaccounts). They are, **UsedCapacity, Transactions, Ingress, Egress, SuccessServerLatency, SuccessE2ELatency, and Availability**.


## 3.2 Blog Service Metrics

### 3.2.1 Metric Definitions

`GET https://management.azure.com/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/providers/microsoft.insights/metricDefinitions/?api-version=2018-01-01`

Example of the Response:

```json
{
    "value": [
        {
            "id": "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/blobServices/default/providers/microsoft.insights/metricdefinitions/BlobCapacity",
            "resourceId": "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/blobServices/default",
            "namespace": "Microsoft.Storage/storageAccounts/blobServices",
            "category": "Capacity",
            "name": {
                "value": "BlobCapacity",
                "localizedValue": "Blob Capacity"
            },
            "isDimensionRequired": false,
            "unit": "Bytes",
            "primaryAggregationType": "Total",
            "supportedAggregationTypes": [
                "Total",
                "Average",
                "Minimum",
                "Maximum"
            ],
            "metricAvailabilities": [
                {
                    "timeGrain": "PT1H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT6H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "PT12H",
                    "retention": "P93D"
                },
                {
                    "timeGrain": "P1D",
                    "retention": "P93D"
                }
            ],
            "dimensions": [
                {
                    "value": "BlobType",
                    "localizedValue": "Blob type"
                }
            ]
        },
        ...
    ]
}
```

All the metrics supported metrics can be found in [here](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-supported-metrics#microsoftstoragestorageaccountsblobservices). They are, **BlobCapacity, BlobCount, ContainerCount, Transactions, Ingress, Egress, SuccessServerLatency, SuccessE2ELatency, and Availability**.

### 3.2.2 Metric

Now let's retrieve any of the metrics.

`GET https://management.azure.com/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/blobServices/default/providers/microsoft.insights/metrics/?api-version=2018-01-01&metricnames=Transactions&interval=PT6H`

Here we are retrieving the metric `Transactions` aggregated by 6 hours (`PT6H`). Also, I want to highlight the namespace we are using `/Microsoft.Storage/storageAccounts/{storage-account}/blobServices/default/`. This represents the blob service of the storage account.

Here an example of the response:

```json
{
    "cost": 0,
    "timespan": "2018-04-18T15:00:54Z/2018-04-18T16:00:54Z",
    "interval": "PT6H",
    "value": [
        {
            "id": "/subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{storage-account}/blobServices/default/providers/microsoft.insights/metrics/Transactions",
            "type": "Microsoft.Insights/metrics",
            "name": {
                "value": "Transactions",
                "localizedValue": "Transactions"
            },
            "unit": "Count",
            "timeseries": [
                {
                    "metadatavalues": [],
                    "data": [
                        {
                            "timeStamp": "2018-04-18T15:00:00Z"
                        }
                    ]
                }
            ]
        }
    ],
    "namespace": "Microsoft.Storage/storageAccounts/blobServices",
    "resourceregion": "westeurope"
}
```

## 4 Conclusion

Retrieve metrics from Azure is quite straightforward. The complexity lay in the number of services and metrics we can retrieve. There are tools that can help to visualize azure metrics. For instance, Grafana has a plugin you can install to visualize time series that are retrieved from Azure Monitoring.

---

# References

[Azure Storage metrics in Azure Monitor (preview)](https://docs.microsoft.com/en-us/azure/storage/common/storage-metrics-in-azure-monitor)

[Azure Monitoring REST API walkthrough](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-rest-api-walkthrough)

[Monitoring Overview](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview)

[Monitoring Supported Metrics](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-supported-metrics)

[Rest API Monitoring Reference](https://docs.microsoft.com/en-us/rest/api/monitor/)

[Metric Definitions Endpoint](https://docs.microsoft.com/en-us/rest/api/monitor/metricdefinitions/list)

[Metrics Endpoint](https://docs.microsoft.com/en-us/rest/api/monitor/metrics/list)

[Reacting to Blob storage events](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview)
