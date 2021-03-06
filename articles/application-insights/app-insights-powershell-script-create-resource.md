---
title: PowerShell script to create an Application Insights resource | Microsoft Docs
description: Automate creation of Application Insights resources.
services: application-insights
documentationcenter: windows
author: alancameronwills
manager: douge

ms.assetid: f0082c9b-43ad-4576-a417-4ea8e0daf3d9
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 11/19/2016
ms.author: awills

---
# PowerShell script to create an Application Insights resource


When you want to monitor a new application - or a new version of an application - with [Azure Application Insights](https://azure.microsoft.com/services/application-insights/), you set up a new resource in Microsoft Azure. This resource is where the telemetry data from your app is analyzed and displayed. 

You can automate the creation of a new resource by using PowerShell.

For example, if you are developing a mobile device app, it's likely that, at any time, there will be several published versions of your app in use by your customers. You don't want to get the telemetry results from different versions mixed up. So you get your build process to create a new resource for each build.

> [!NOTE]
> If you want to create a set of resources all at the same time, consider [creating the resources using an Azure template](app-insights-powershell.md).
> 
> 

## Script to create an Application Insights resource
See the relevant cmdlet specs:

* [New-AzureRmResource](https://msdn.microsoft.com/library/mt652510.aspx)
* [New-AzureRmRoleAssignment](https://msdn.microsoft.com/library/mt678995.aspx)

*PowerShell Script*  

```PowerShell


###########################################
# Set Values
###########################################

# If running manually, uncomment before the first 
# execution to login to the Azure Portal:

# Add-AzureRmAccount

# Set the name of the Application Insights Resource

$appInsightsName = "TestApp"

# Set the application name used for the value of the Tag "AppInsightsApp" 

$applicationTagName = "MyApp"

# Set the name of the Resource Group to use.  
# Default is the application name.
$resourceGroupName = "MyAppResourceGroup"

###################################################
# Create the Resource and Output the name and iKey
###################################################

#Select the azure subscription
Select-AzureSubscription -SubscriptionName "MySubscription"

# Create the App Insights Resource


$resource = New-AzureRmResource `
  -ResourceName $appInsightsName `
  -ResourceGroupName Fabrikam `
  -Tag @{ applicationType = "web", applicationName = $applicationTagName} `
  -ResourceType "Microsoft.Insights/components" `
  -Location "East US" `  // or North Europe, West Europe, South Central US
  -PropertyObject @{"Application_Type"="web"} `
  -Force

# Give owner access to the team

New-AzureRmRoleAssignment `
  -SignInName "myteam@fabrikam.com" `
  -RoleDefinitionName Owner `
  -Scope $resource.ResourceId 


#Display iKey
Write-Host "App Insights Name = " $resource.Name
Write-Host "IKey = " $resource.Properties.InstrumentationKey

```

## What to do with the iKey
Each resource is identified by its instrumentation key (iKey). The iKey is an output of the resource creation script. Your build script should provide the iKey to the Application Insights SDK embedded in your app.

There are two ways to make the iKey available to the SDK:

* In [ApplicationInsights.config](app-insights-configuration-with-applicationinsights-config.md): 
  * `<instrumentationkey>`*ikey*`</instrumentationkey>`
* Or in [initialization code](app-insights-api-custom-events-metrics.md): 
  * `Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey = "`*iKey*`";`

## See also
* [Create Application Insights and web test resources from templates](app-insights-powershell.md)
* [Set up monitoring of Azure diagnostics with PowerShell](app-insights-powershell-azure-diagnostics.md) 
* [Set alerts by using PowerShell](app-insights-powershell-alerts.md)

