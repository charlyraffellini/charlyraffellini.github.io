---
layout: post
title: Azure availability sets in portal ui and powershell
description: Automation of availability sets in azure
date: 2017-10-14
categories: automation infrastructure 
img: 2017/load_balancer_availability_set_relational_model.png
author: Carlos Raffellini
---

This week I was learning about availability sets in azure. I was spinning load balancers and VMs and attaching them to an availability set. To do that the portal UI backend pool creation view asks you to choose the availability set you want to associate with the load balancer. After that, any other machine or backend pool you create for that load balancer should be associated with the same availability set.

##### Definition: Back-end address pool – these are IP addresses associated with the virtual machine Network Interface Card (NIC) to which load is distributed.

![ui](/assets/images/2017/backend_pool_portal_ui.JPG)

However, I do not like to do the same thing twice through the UI. So, I created a [powershell script](#ps) to do that. However, there was no place to associate a backend pool with an availability set. I prefer graphical ways to see relationships so I created a graphical view of azure cmdlet relations to create a load balancer with VMs and an availability set. This graph match with the cmdlet's of the script showed in the bottom of this post.

![relations](/assets/images/2017/load_balancer_availability_set_relational_model.png)

I tried to add a virtual machine in a different availability set from the one than the one associated with the load balancer. This is what I got from the test:

![availability set error](/assets/images/2017/availability_set_error.JPG)

This last error shows that in spite of the PowerShell script does not associate directly an availability set with a load balancer. The load balancer gets associated with the availability set of the first machine that we attach to the load balancer.

All of this associations are maintained internally by in the Azure resources. However, the UI and the PowerShell cmdlet's show this association in a different way.

---

## References:

[Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)

[Azure Load Balancer Support](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-arm)

[Availability Set](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)

<h2><a name="ps">Powershell Script</a></h2>

{% gist charlyraffellini/0cabf6c975b15ed4463c006eb1660aa6 %}