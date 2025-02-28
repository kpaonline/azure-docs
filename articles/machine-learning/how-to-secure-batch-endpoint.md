---
title: "Network isolation in batch endpoints"
titleSuffix: Azure Machine Learning
description: Learn how to deploy Batch Endpoints in private networks with isolation.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: santiagxf
ms.author: fasantia
ms.date: 10/10/2022
ms.reviewer: larryfrare configured
ms.custom: devplatv2
---

# Network isolation in batch endpoints

When deploying a machine learning model to a batch endpoint, you can secure their communication using private networks. This article explains the requirements to use batch endpoint in an environment secured by private networks.

## Securing batch endpoints

Batch endpoints inherent the networking configuration from the workspace where they are deployed. All the batch endpoints created inside of secure workspace are deployed as private batch endpoints by default. In order to have fully operational batch endpoints working with private networking, follow the following steps:

1. You have configured your Azure Machine Learning workspace for private networking. For more details about how to achieve it read [Create a secure workspace](tutorial-create-secure-workspace.md).

2. For Azure Container Registry in private networks, there are [some prerequisites about their configuration](how-to-secure-workspace-vnet.md#prerequisites).

    > [!WARNING]
    > Azure Container Registries with Quarantine feature enabled are not supported by the moment.

3. Ensure blob, file, queue, and table private endpoints are configured for the storage accounts as explained at [Secure Azure storage accounts](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts). Batch deployments require all the 4 to properly work.

4. Create the batch endpoint as regularly done. 

The following diagram shows how the networking looks like for batch endpoints when deployed in a private workspace:

:::image type="content" source="./media/how-to-secure-batch-endpoint/batch-vnet-peering.png" alt-text="Diagram that shows the high level architecture of a secure Azure Machine Learning workspace deployment.":::


## Securing batch deployment jobs

Azure Machine Learning batch deployments run on compute clusters. To secure batch deployment jobs, those compute clusters have to be deployed in a virtual network too.

1. Create an Azure Machine Learning [computer cluster in the virtual network](how-to-secure-training-vnet.md).
2. Ensure all related services have private endpoints configured in the network. Private endpoints are used for not only Azure Machine Learning workspace, but also its associated resources such as Azure Storage, Azure Key Vault, or Azure Container Registry. Azure Container Registry is a required service. While securing the Azure Machine Learning workspace with virtual networks, please note that there are [some prerequisites about Azure Container Registry](how-to-secure-workspace-vnet.md#prerequisites).
4. If your compute instance uses a public IP address, you must [Allow inbound communication](how-to-secure-training-vnet.md#compute-instancecluster-with-public-ip) so that management services can submit jobs to your compute resources.
    
    > [!TIP]
    > Compute cluster and compute instance can be created with or without a public IP address. If created with a public IP address, you get a load balancer with a public IP to accept the inbound access from Azure batch service and Azure Machine Learning service. You need to configure User Defined Routing (UDR) if you use a firewall. If created without a public IP, you get a private link service to accept the inbound access from Azure batch service and Azure Machine Learning service without a public IP.

1. Extra NSG may be required depending on your case. For more information, see [How to secure your training environment](how-to-secure-training-vnet.md).

For more information, see the [Secure an Azure Machine Learning training environment with virtual networks](how-to-secure-training-vnet.md) article.

## Using two-networks architecture

There are cases where the input data is not in the same network as in the Azure Machine Learning resources. In those cases, your Azure Machine Learning workspace may need to interact with more than one VNet. You can achieve this configuration by adding an extra set of private endpoints to the VNet where the rest of the resources are located.

The following diagram shows the high level design:

:::image type="content" source="./media/how-to-secure-batch-endpoint/batch-vnet-two-networks.png" alt-text="Diagram that shows the high level architecture of an Azure Machine Learning workspace interacting with two networks.":::

### Considerations

Have the following considerations when using such architecture:

* Put the second set of private endpoints in a different resource group and hence in different private DNS zones. It prevents a name resolution conflict between the set of IPs used for the workspace and the ones used by the client VNets. Azure Private DNS provides a reliable, secure DNS service to manage and resolve domain names in a virtual network without the need to add a custom DNS solution. By using private DNS zones, you can use your own custom domain names rather than the Azure-provided names available today. Note that the DNS resolution against a private DNS zone works only from virtual networks that are linked to it. For more details, see [recommended zone names for Azure services](../private-link/private-endpoint-dns.md#azure-services-dns-zone-configuration).
* For your storage accounts, add 4 private endpoints in each VNet for blob, file, queue, and table as explained at [Secure Azure storage accounts](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts).

## Limitations

Consider the following limitations when working on batch endpoints deployed regarding networking:

- If you change the networking configuration of the workspace from public to private, or from private to public, such doesn't affect existing batch endpoints networking configuration. Batch endpoints rely on the configuration of the workspace at the time of creation. You can recreate your endpoints if you want them to reflect changes you made in the workspace.

- When working on a private link-enabled workspace, batch endpoints can be created and managed using Azure Machine Learning studio. However, they can't be invoked from the UI in studio. Use the Azure Machine Learning CLI v2 instead for job creation. For more details about how to use it see [Run batch endpoint to start a batch scoring job](how-to-use-batch-endpoint.md#run-batch-endpoints-and-access-results).

## Recommended read

* [Secure Azure Machine Learning workspace resources using virtual networks (VNets)](how-to-network-security-overview.md)
