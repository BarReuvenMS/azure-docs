---
title: Indexer access to protected resources
titleSuffix: Azure Cognitive Search
description: Learn import concepts and requirements related to network-level security options for outbound requests made by indexers in Azure Cognitive Search.

manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/18/2022
---

# Indexer access to content protected by Azure network security features

Azure Cognitive Search indexers can make outbound calls to various Azure resources during execution. This article explains the concepts behind indexer access to content that's protected by IP firewalls, private endpoints, or other Azure network-level security mechanisms. 

An indexer makes outbound calls in two situations:

- Connecting to external data sources during indexing
- Connecting to external, encapsulated code through a skillset

A list of all possible resource types that an indexer might access in a typical run are listed in the table below.

| Resource | Purpose within indexer run |
| --- | --- |
| Azure Storage (blobs, tables, ADLS Gen 2) | Data source |
| Azure Storage (blobs, tables) | Skillsets (caching enriched documents, and storing knowledge store projections) |
| Azure Cosmos DB (various APIs) | Data source |
| Azure SQL Database | Data source |
| SQL Server on Azure virtual machines | Data source |
| SQL Managed Instance | Data source |
| Azure Functions | Attached to a skillset and used to host for custom web api skills |
| Cognitive Services | Attached to a skillset and used to bill enrichment beyond the 20 free documents limit |

> [!NOTE]
> A Cognitive Service resource attached to a skillset is used for billing, based on the enrichments performed and written into the search index or a knowledge store. It isn't used for accessing the Cognitive Services APIs. Access from an indexer's enrichment pipeline to Cognitive Services APIs occurs via an internal secure communication channel, where data is strongly encrypted in transit and is never stored at rest.

Your Azure resources could be protected using any number of the network isolation mechanisms offered by Azure. Depending on the resource and region, Cognitive Search indexers can make outbound connections through IP firewalls and private endpoints, subject to the limitations indicated in the following table.

| Resource | IP Restriction | Private endpoint |
| --- | --- | ---- |
| Azure Storage (blobs, tables, ADLS Gen 2) | Supported only if the storage account and search service are in different regions | Supported |
| Azure Cosmos DB - SQL API | Supported | Supported |
| Azure Cosmos DB - MongoDB API | Supported | Unsupported |
| Azure Cosmos DB - Gremlin API | Supported | Unsupported |
| Azure SQL Database | Supported | Supported |
| SQL Server on Azure virtual machines | Supported | N/A |
| SQL Managed Instance | Supported | N/A |
| Azure Functions | Supported | Supported, only for certain tiers of Azure functions |

> [!NOTE]
> In addition to the options listed above, for network-secured Azure Storage accounts, you can make Azure Cognitive Search a [trusted Microsoft service](../storage/common/storage-network-security.md#trusted-microsoft-services). This means that a specific search service can bypass virtual network or IP restrictions on the storage account and can access data in the storage account, if the appropriate role-based access control is enabled on the storage account. For more information, see [Indexer connections using the trusted service exception](search-indexer-howto-access-trusted-service-exception.md). This option can be utilized instead of the IP restriction route, in case either the storage account or the search service can't be moved to a different region.

## Indexer execution environment

Azure Cognitive Search indexers are capable of efficiently extracting content from data sources, adding enrichments to the extracted content, optionally generating projections before writing the results to the search index.

For optimum processing, a search service will determine an internal execution environment to set up the operation. You can't control or configure the environment, but it's important to know they exist so that you can account for them when setting up IP firewall rules.

Depending on the number and types of tasks assigned, the indexer will run in one of two environments:

- An environment private to a specific search service. Indexers running in such environments share resources with other workloads (such as other customer-initiated indexing or querying workloads). Typically, only indexers that perform text-based indexing (for example, do not use a skillset) run in this environment.

- A multi-tenant environment hosting indexers that are resource intensive, such as those with skillsets. This environment is used to offload computationally intensive processing, leaving service-specific resources available for routine operations. This multi-tenant environment is managed and secured by Microsoft, at no extra cost to the customer.

For any given indexer run, Azure Cognitive Search determines the best environment in which to run the indexer. If you're using an IP firewall to control access to Azure resources, knowing about execution environments will help you set up an IP range that is inclusive of both, as discussed in the next section.

## Granting access to indexer IP ranges

If the resource that your indexer pulls data from exists behind a firewall, you'll need [inbound rules that admit indexer connections](search-indexer-howto-access-ip-restricted.md). Make sure that the IP ranges in inbound rules include all of the IPs from which an indexer request can originate. As stated above, there are two possible environments in which indexers run and from which access requests can originate. You'll need to add the IP addresses of **both** environments for indexer access to work.

- To obtain the IP address of the search service private environment, use `nslookup` (or `ping`) the fully qualified domain name (FQDN) of your search service. The FQDN of a search service in the public cloud would be `<service-name>.search.windows.net`.

- To obtain the IP addresses of the multi-tenant environments within which an indexer might run, use the `AzureCognitiveSearch` service tag. [Azure service tags](../virtual-network/service-tags-overview.md) have a published range of IP addresses for each service. You can find these IPs using the [discovery API](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api) or a [downloadable JSON file](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files). In either case, IP ranges are broken down by region. You should specify only those IP ranges assigned to the region in which your search service is provisioned.

For certain data sources, the service tag itself can be used directly instead of enumerating the list of IP ranges (the IP address of the search service still needs to be used explicitly). These data sources restrict access by means of setting up a [Network Security Group rule](../virtual-network/network-security-groups-overview.md), which natively support adding a service tag, unlike IP rules such as the ones offered by Azure Storage, Cosmos DB, Azure SQL, and so forth. The data sources that support the ability to utilize the `AzureCognitiveSearch` service tag directly in addition to search service IP address are:

- [SQL Server on Azure virtual machines](./search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md#restrict-access-to-the-azure-cognitive-search)

- [SQL Managed Instances](./search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers.md#verify-nsg-rules)

## Granting access via private endpoints

When integrating Azure Cognitive Search into a solution that runs on a virtual network, consider the following constraints:

- An indexer can't make a direct connection to a [virtual network service endpoint](../virtual-network/virtual-network-service-endpoints-overview.md). Public endpoints with credentials, private endpoints, trusted service, and IP addressing are the only supported methodologies for indexer connections.
- A search service always runs in the cloud and can't be provisioned into a specific virtual network, running natively on a virtual machine. This functionality will not be offered by Azure Cognitive Search.

To achieve integration, you can use [private endpoints](../private-link/private-endpoint-overview.md) on outbound connections to resources that are locked down (running on a protected virtual network, or just not available over a public connection). 

The mechanism by which a search service connects to your protected resource is through a shared private link. A shared private link is [Azure Private Link](../private-link/private-link-overview.md) resource that's created, managed, and used from within Cognitive Search.

### Billing impact 

- A shared private link requires a billable search service, where the minimum tier is either Basic for text-based indexing or Standard 2 (S2) for skills-based indexing. See [tier limits on the number of private endpoints](search-limits-quotas-capacity.md#shared-private-link-resource-limits) for details. 

- Inbound and outbound connections are subject to [Azure Private Link pricing](https://azure.microsoft.com/pricing/details/private-link/).

### Step 1: Create a private endpoint to the secure resource

In Azure Cognitive Search, you can create a shared private link using either the portal or a [management API](/rest/api/searchmanagement/2021-04-01-preview/shared-private-link-resources/create-or-update). 

Traffic that goes over this (outbound) private endpoint connection will originate only from the virtual network that's in the search service specific "private" indexer execution environment.

Azure Cognitive Search will validate that callers of this API have Azure RBAC role permissions to approve private endpoint connection requests to the secure resource. For example, if you request a private endpoint connection to a storage account with read-only permissions, this call will be rejected.

### Step 2: Approve the private endpoint connection

When the (asynchronous) operation that creates a shared private link resource completes, a private endpoint connection will be created in a "Pending" state. No traffic flows over the connection yet.

The customer is then expected to locate this request on their secure resource and "Approve" it. Typically, this can be done either via the Azure portal or via the [REST API](/rest/api/virtualnetwork/privatelinkservices/updateprivateendpointconnection).

### Step 3: Force indexers to run in the "private" environment

An approved private endpoint allows outgoing calls from the search service to a resource that has some form of network level access restrictions (for example a storage account data source that is configured to only be accessed from certain virtual networks) to succeed.

This means any indexer that is able to reach out to such a data source over the private endpoint will succeed.
If the private endpoint isn't approved, or if the indexer doesn't utilize the private endpoint connection then the indexer run will end up in `transientFailure`.

To enable indexers to access resources via private endpoint connections, it's mandatory to set the `executionEnvironment` of the indexer to `"Private"` to ensure that all indexer runs will be able to utilize the private endpoint. This is because private endpoints are provisioned within the private search service-specific environment.

```json
    {
      "name" : "myindexer",
      ... other indexer properties
      "parameters" : {
          ... other parameters
          "configuration" : {
            ... other configuration properties
            "executionEnvironment": "Private"
          }
        }
    }
```

These steps are described in greater detail in [Indexer connections through a private endpoint](search-indexer-howto-access-private.md).
Once you have an approved private endpoint to a resource, indexers that are set to be *private* attempt to obtain access via the private endpoint connection.

## Next steps

- [Indexer connections through IP firewalls](search-indexer-howto-access-ip-restricted.md)
- [Indexer connections using the trusted service exception](search-indexer-howto-access-trusted-service-exception.md)
- [Indexer connections to a private endpoint](search-indexer-howto-access-private.md)
