---
layout: post
title: Split On Trigger Property in Logic App
description: How does SplitOn Trigger work?
date: 2018-04-28
categories: saas azure logic app rest microsoft api 
img: 2018/spliton-logic-app.jpg
author: Carlos Raffellini
---

# SplitOn Trigger Property

- [SplitOn Trigger Property](#spliton-trigger-property)
    - [Create a logic app with SplitOn Trigger](#create-a-logic-app-with-spliton-trigger)
    - [Invoke the trigger with an array](#invoke-the-trigger-with-an-array)
    - [Limits to SplitOn](#limits-to-spliton)
    - [Conclusion](#conclusion)
- [References](#references)

**SplitOn** splits up the array items and starts a new logic app instance that runs for each array item. This approach is useful, for example, when you want to poll an endpoint that might return multiple new items between polling intervals.

You can add **SplitOn** only to triggers by manually defining or overriding in code view for your logic app's JSON definition. **You can't use SplitOn** when you want to implement a synchronous response pattern. Any workflow that uses **SplitOn** and includes a response action runs asynchronously and immediately sends a `202 ACCEPTED` response.

The trigger we use for split on could be any of the triggers which receive a body that contains an array.
 
## Create a logic app with SplitOn Trigger

```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "HTTP": {
                "inputs": {
                    "body": "@triggerBody()",
                    "method": "POST",
                    "uri": "your-function-app-URL"
                },
                "runAfter": {},
                "type": "Http"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {},
        "triggers": {
            "manual": {
                "inputs": {
                    "method": "POST",
                    "schema": {}
                },
                "kind": "Http",
                "splitOn": "@triggerBody()?.Rows",
                "type": "Request"
            }
        }
    }
}
```

## Invoke the trigger with an array

```http
POST https://prod-1.centralus.logic.azure.com:443/workflows/{workflow-id}/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig={signature} HTTP/1.1

{
    "Status": "Succeeded",
    "Rows": [ 
        { 
            "id": 1938109380,
            "name": "customer-name-one"
        },
        {
            "id": 1938109381,
            "name": "customer-name-two"
        },
        ...
    ]
}
```

The HTTP trigger will split the `Rows` array and will execute the workflow actions for each of the elements of the array. In this case, it will send one `POST` to  `your-function-app-URL` for each element of the array.


```http
POST http://your-function-app-URL HTTP/1.1
X-Ms-Workflow-Run-Tracking-Id: 55165dbb-bjf9-4a2d-852c-9fc2a7aed4f1
X-Ms-Client-Tracking-Id: 08586770293115571446874726667

{"id":1938109380,"name":"customer-name-one"}
```

```http
POST http://your-function-app-URL HTTP/1.1
X-Ms-Workflow-Run-Tracking-Id: 55165dbb-bjf9-4a2d-852c-9fc2a7aed4f1
X-Ms-Client-Tracking-Id: 08586770293115571446874726667

{"id":1938109381,"name":"customer-name-two"}
```

Here we can see one instance of the workflow per each element of the array:


![workflows](/assets/images/2018/workflows_with_same_id.jpg)

Also, looking at the logs you can see the body of each invocation:

![logs](/assets/images/2018/splited_request.jpg)

## Limits to SplitOn

**Name**|**Limit**|**Notes**
:-----:|:-----:|:-----:
ForEach items|100,000|You can use the query action to filter larger arrays as needed.
Until iterations|5,000| 
SplitOn items|100,000| 
ForEach Parallelism|50|The default is 20.


## Conclusion

Logic App is good for small management activities. And **SplitOn** is one of the coolest things I saw in the Service. Also, Logic App could save a lot of hassle with logging and visualization for simple tasks. However, Logic App isn't one of my favorites Azure Services. It lacks the flexibility of a programming language and it is too verbose for the long time or even medium development cycle.

---

# References

[Triggers: Process an array with multiple runs](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-workflow-actions-triggers#split-on-debatch)

[Logic Apps limits and configuration](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config)