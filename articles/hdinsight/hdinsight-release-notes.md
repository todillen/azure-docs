---
title: Release notes for Azure HDInsight 
description: Latest release notes for Azure HDInsight. Get development tips and details for Hadoop, Spark, R Server, Hive, and more.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
ms.date: 01/24/2019
---
# Release notes

This article provides information about the **most recent** Azure HDInsight release updates. For information on earlier releases, see [HDInsight Release Notes Archive](hdinsight-release-notes-archive.md).

## Summary

Azure HDInsight is one of the most popular services among enterprise customers for open-source analytics on Azure.

## Release date: 01/09/2019

This release applies both for HDInsight 3.6 and 4.0. HDInsight release is made available to all regions over several days. The release date here indicates the first region release date. If you don't see below changes, please wait for the release being live in your region in several days.

> [!IMPORTANT]  
> Linux is the only operating system used on HDInsight version 3.4 or greater. For more information, see [HDInsight versioning article](hdinsight-component-versioning.md).

## New features
### TLS 1.2 enforcement
Transport Layer Security (TLS) and Secure Sockets Layer (SSL) are cryptographic protocols that provide communications security over a computer network. Learn more about [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0.2C_2.0_and_3.0). HDInsight uses TLS 1.2 on public HTTPs endpoints but TLS 1.1 is still supported for backward compatibility. 

With this release, customers can opt into TLS 1.2 only for all connections through the public cluster endpoint. To support this, the new property **minSupportedTlsVersion** is introduced and can be specified during cluster creation. If the property is not set, the cluster still supports TLS 1.0, 1.1 and 1.2, which is the same as today's behavior. Customers can set the value for this property to "1.2", which means that the cluster only supports TLS 1.2 and above. For more information, see [Plan a virtual network - Transport Layer Security](https://docs.microsoft.com/azure/hdinsight/hdinsight-plan-virtual-network-deployment#transport-layer-security).

### Bring your own key for disk encryption
All managed disks in HDInsight are protected with Azure Storage Service Encryption (SSE). Data on those disks is encrypted by Microsoft-managed keys by default. Starting from this release, you can Bring Your Own Key (BYOK) for disk encryption and manage it using Azure Key Vault. BYOK encryption is a one-step configuration during cluster creation with no additional cost. Just register HDInsight as a managed identity with Azure Key Vault and add the encryption key when you create your cluster. For more information, see [Customer-managed key disk encryption](https://docs.microsoft.com/azure/hdinsight/disk-encryption).

## Deprecation
No deprecations for this release. To get ready for upcoming deprecations, see [Upcoming changes](#upcoming-changes).

## Behavior changes
No behavior changes for this release. To get ready for upcoming changes, see [Upcoming changes](#upcoming-changes).

## Upcoming changes
The following changes will happen in upcoming releases. 

### A minimum 4-core VM is required for Head Node 
A minimum 4-core VM is required for Head Node to ensure the high availability and reliability of HDInsight clusters. Starting from April 6th 2020, customers can only choose 4-core or above VM as Head Node for the new HDInsight clusters. Existing clusters will continue to run as expected. 

### ESP Spark cluster node size change 
In the upcoming release, the minimum allowed node size for ESP Spark cluster will be changed to Standard_D13_V2. 
A-series VMs could cause ESP cluster issues because of relatively low CPU and memory capacity. A-series VMs will be deprecated for creating new ESP clusters.

### Moving to Azure virtual machine scale sets
HDInsight now uses Azure virtual machines to provision the cluster. In the upcoming release, HDInsight will use Azure virtual machine scale sets instead. See more about Azure virtual machine scale sets.

### HBase 2.0 to 2.1
In the upcoming HDInsight 4.0 release, HBase version will be upgraded from version 2.0 to 2.1.

## Bug fixes
HDInsight continues to make cluster reliability and performance improvements. 

## Component version change
No component version change for this release. You could find the current component versions for HDInsight 4.0 ad HDInsight 3.6 here.

## Known issues

As of January 24, 2020, there is an active issue in which you may receive an error when attempting to use a Jupyter notebook. Use the steps below to fix the issue. You can also refer to this [MSDN post](https://social.msdn.microsoft.com/Forums/en-us/8c763fb4-79a9-496f-a75c-44a125e934ac/hdinshight-create-not-create-jupyter-notebook?forum=hdinsight) or this [StackOverflow post](https://stackoverflow.com/questions/59687614/azure-hdinsight-jupyter-notebook-not-working/59831103) for up-to-date information, or to ask additional questions. This page will be updated when the issue is fixed.

**Errors**

* ValueError: Cannot convert notebook to v5 because that version doesn't exist
* Error loading notebook An unknown error occurred while loading this notebook. This version can load notebook formats v4 or earlier

**Cause** 

The _version.py file on the cluster was updated to 5.x.x instead of 4.4.x.##.

**Solution**

If you create a new Jupyter notebook and receive one of the errors listed above, perform the following steps to fix the issue.

1. Open Ambari in a web browser by going to https://CLUSTERNAME.azurehdinsight.net, where CLUSTERNAME is the name of your cluster.
1. In Ambari, on the left menu, click **Jupyter**, then on **Service Actions**, click **Stop**.
1. ssh into the cluster headnode where the Jupyter service is running.
1. Open the following file /usr/bin/anaconda/lib/python2.7/site-packages/nbformat/_version.py in sudo mode.
1. The existing entry should show something similar to the following code: 

    version_info = (5, 0, 3)

    Modify the entry to: 
    
    version_info = (4, 4, 0)
1. Save the file.
1. Go back to Ambari, and in **Service Actions**, click **Restart All**.
