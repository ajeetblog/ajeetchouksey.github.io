---
layout: post
title: "Using VSTS to provision multi server environments in Azure Dev Test Labs"
date: 2017-01-04
author: tarun
tags: ["DevOps", "Azure", "AzureDevTestLabs"]
categories:
- "DevOps"
img: "/images/screenshots/tarun/AzureDTL/AzureDtl_DevOps_InfrastructureIsCode.png"
description: "Azure Dev Test Labs support single server environments, previously we've seen how to use the VSTS Azure Dev Test Labs extension from the Visual Studio Marketplace to automate the provisioing of virtaual machines in the Azure Dev Test Labs. In this blog post we'll see how to deploy multi server environments in Azure Dev Test Labs using Arm deployment task in VSTS."
permalink: /DevOps/multiserver-environments-in-azure-devtestlabs
keywords: "DevOps, Azure, AzureDevTestLabs, TeamBuild, Continuos Deployments, Release Management"
---

At launch Azure Dev Test Labs only supported single server environments. While this was great for development and standalone testing, this limited the use of Dev Test Labs for any meaningful testing. If you are truly following DevOps you would like your automated environment provisioing to resemble production like environments throughout from development all the way to production. After all it's all about pushing left and testing your automation from the outset ;) 
<!--more-->
# Introduction
As one of the most voted features, it came as no surprise when @ Connect() 2016 the Microsoft Azure team announced the availability of _**Bring Your Own Template (BYOT)**_ support in Azure Dev Test Labs. BYOT basically means support for custom ARM templates, an ARM template could be used to create a single server or complex multi server environment that could comprise of both IaaS and PaaS resources. [Xiaoying Guo](https://social.msdn.microsoft.com/profile/Xiaoying+Guo) a program manager in the Azure Dev Test Labs team has a great blogpost on [How to set up custom multi server ARM based environments in Azure Dev Test Labs?](https://blogs.msdn.microsoft.com/devtestlab/2016/11/16/connect-2016-news-for-azure-devtest-labs-azure-resource-manager-template-based-environments-vm-auto-shutdown-and-more/). 

> In this blogpost I am going to take this even a step further and show you how to leverage Visual Studio Team Services or Team Foundation Server Release Pipelines to orchestrate the deployment of this complex environment to DevTestLabs using the Azure Resource Group Deployment task.

# Goal 
Create a release pipeline in VSTS to push an ARM to provision an test environment, staging environment and production environment in Azure Dev Test Labs. 

# Target Environment
The target environment that we wish to deploy in the Azure Dev Test Labs is a multi server environment that comprises of,
+ 1 or 2 Windows 2012R2 IIS Web Servers.
+ 1 SQL Server 2014 running on premium or standard storage.
+ 1 virtual network with 2 subnets with NSG rules.
+ 1 storage account for the VHD files.
+ 1 Availability Set for IIS servers.
+ 1 Load balancer with NATing rules.   

![AzureDTL-ComplexArmTemplateIntroduction](/images/screenshots/tarun/AzureDTL/AzureDTL-ComplexArmTemplateIntroduction.png)

The code for the ARM template is available in my [GitHub repo](https://github.com/tarunaroraonline/azure-devtestlabsdemo) you are welcome to fork this repository... 

# How to make your ARM templates work with Azure Dev Test Labs?
Your standard ARM templates will need an additional json file in the parent folder in order to make your ARM template work with Azure Dev Test Labs. For example, in the below screen shot - in order to make my ARM template 'iis-2vm-sql-1vm' work with Azure DTL i've had to add a new file 'labdeploy.json' in the root directory of the template. 

![AzureDTL-AdditionalLabDeployJsonFile](/images/screenshots/tarun/AzureDTL/AzureDTL-AdditionalLabDeployJsonFile.png)


The additional file defines the parameters, resources and output generated by this ARM template.  