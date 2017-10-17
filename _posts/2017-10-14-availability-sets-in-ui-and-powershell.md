---
layout: post
title: Differences between azure availability sets in portal ui and powershell
description: Automation of availability sets in azure
date: 2017-10-14
categories: automation infrastructure 
img: 2017/load_balancer_availability_set_relational_model.png
author: Carlos Raffellini
---

This week I was learning about availability sets in azure. I was spinning load balancers up and VMs and attaching them to an availability set. To do that the portal UI backend pool creation view asks you to choose the availability set you want to associate with the load balancer. After that, any other machine or backend pool you create for that load balancer should be associated with the same availability set.

##### Definition: Back-end address pool – these are IP addresses associated with the virtual machine Network Interface Card (NIC) to which load is distributed.

![ui](/assets/images/2017/backend_pool_portal_ui.JPG)

However, I do not like to do the same thing twice through the UI. So, I created a [powershell script](#ps) to do that. Suddenly I realized, there is no place to associate a backend pool with an availability set in powershell - none of next cmdlet shows an explicit association between a load balancer and an availability set [New-AzureRmLoadBalancer](https://docs.microsoft.com/en-us/powershell/module/azurerm.network/new-azurermloadbalancer?view=azurermps-4.4.1), [New-AzureRmLoadBalancerRuleConfig](https://docs.microsoft.com/en-us/powershell/module/azurerm.network/new-azurermloadbalancerruleconfig?view=azurermps-4.4.1),  [New-AzureRmLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/en-us/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig?view=azurermps-4.4.1), [New-AzureRmAvailabilitySet](https://docs.microsoft.com/en-us/powershell/module/azurerm.compute/new-azurermavailabilityset?view=azurermps-4.4.1).

I prefer graphical ways to see relationships so I created a graphical view of azure cmdlet relations to create a load balancer with VMs and an availability set. This graph match with the cmdlet's of the script showed in the bottom of this post.

![relations](/assets/images/2017/load_balancer_availability_set_relational_model.png)

After adding a new VM to a load balancer using the next comlet the portal show the association between the backen pool and the availability set:


{% gist charlyraffellini/f55e3dc5a73c13e0e632186f15fb35d7 %}


![ui](/assets/images/2017/lb_with_availability_set.JPG)


I tried to add a new virtual machine in a different availability set to the same load balancer. This is what I got from the test:

![availability set error](/assets/images/2017/availability_set_error.JPG)

This last error shows that in spite of the PowerShell script does not associate directly an availability set with a load balancer. The load balancer is associated with the availability set of the first machine that we attach to the load balancer.

All of this associations are maintained internally by the Azure resources. However, the UI and the PowerShell cmdlet's show this association in a different way.

---

## References:

[Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)

[Azure Load Balancer Support](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-arm)

[Availability Set](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)

[New-AzureRmLoadBalancer](https://docs.microsoft.com/en-us/powershell/module/azurerm.network/new-azurermloadbalancer?view=azurermps-4.4.1)

[New-AzureRmLoadBalancerRuleConfig](https://docs.microsoft.com/en-us/powershell/module/azurerm.network/new-azurermloadbalancerruleconfig?view=azurermps-4.4.1)

[New-AzureRmLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/en-us/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig?view=azurermps-4.4.1)

[New-AzureRmAvailabilitySet](https://docs.microsoft.com/en-us/powershell/module/azurerm.compute/new-azurermavailabilityset?view=azurermps-4.4.1)

<h2><a name="ps">Powershell Script</a></h2>

{% gist charlyraffellini/0cabf6c975b15ed4463c006eb1660aa6 %}
