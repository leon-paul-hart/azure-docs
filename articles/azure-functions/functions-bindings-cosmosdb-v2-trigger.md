---
title: Azure Cosmos DB trigger for Functions 2.x and higher
description: Learn to use the Azure Cosmos DB trigger in Azure Functions.
ms.topic: reference
ms.date: 03/04/2022
ms.devlang: csharp, java, javascript, powershell, python
ms.custom: devx-track-csharp, devx-track-python, ignite-2022
zone_pivot_groups: programming-languages-set-functions-lang-workers
---

# Azure Cosmos DB trigger for Azure Functions 2.x and higher

The Azure Cosmos DB Trigger uses the [Azure Cosmos DB Change Feed](../cosmos-db/change-feed.md) to listen for inserts and updates across partitions. The change feed publishes inserts and updates, not deletions.

For information on setup and configuration details, see the [overview](./functions-bindings-cosmosdb-v2.md).

## Example

::: zone pivot="programming-language-csharp"

The usage of the trigger depends on the extension package version and the C# modality used in your function app, which can be one of the following:

# [In-process](#tab/in-process)

An in-process class library is a compiled C# function runs in the same process as the Functions runtime.
 
# [Isolated process](#tab/isolated-process)

An isolated worker process class library compiled C# function runs in a process isolated from the runtime.   
   
# [C# script](#tab/csharp-script)

C# script is used primarily when creating C# functions in the Azure portal.

---

The following examples depend on the extension version for the given C# mode.

# [Functions 2.x+](#tab/functionsv2/in-process)

The following example shows a [C# function](functions-dotnet-class-library.md) that is invoked when there are inserts or updates in the specified database and collection.

```cs
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class CosmosTrigger
    {
        [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "ToDoItems",
            collectionName: "Items",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]IReadOnlyList<Document> documents,
            ILogger log)
        {
            if (documents != null && documents.Count > 0)
            {
                log.LogInformation($"Documents modified: {documents.Count}");
                log.LogInformation($"First document Id: {documents[0].Id}");
            }
        }
    }
}
```

# [Extension 4.x+ (preview)](#tab/extensionv4/in-process)

Apps using [Azure Cosmos DB extension version 4.x](./functions-bindings-cosmosdb-v2.md?tabs=extensionv4) or higher will have different attribute properties, which are shown below. This example refers to a simple `ToDoItem` type.

```cs
namespace CosmosDBSamplesV2
{
    public class ToDoItem
    {
        public string Id { get; set; }
        public string Description { get; set; }
    }
}
```

```cs
using System.Collections.Generic;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace CosmosDBSamplesV2
{
    public static class CosmosTrigger
    {
        [FunctionName("CosmosTrigger")]
        public static void Run([CosmosDBTrigger(
            databaseName: "databaseName",
            containerName: "containerName",
            Connection = "CosmosDBConnectionSetting",
            LeaseContainerName = "leases",
            CreateLeaseContainerIfNotExists = true)]IReadOnlyList<ToDoItem> input, ILogger log)
        {
            if (input != null && input.Count > 0)
            {
                log.LogInformation("Documents modified " + input.Count);
                log.LogInformation("First document Id " + input[0].Id);
            }
        }
    }
}
```

# [Functions 2.x+](#tab/functionsv2/isolated-process)

The following code defines a `MyDocument` type:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/CosmosDB/CosmosDBFunction.cs" range="37-46":::

This document type is the type of the [`IReadOnlyList<T>`](/dotnet/api/system.collections.generic.ireadonlylist-1) used as the Azure Cosmos DB trigger binding parameter in the following example:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/CosmosDB/CosmosDBFunction.cs" range="4-35":::


# [Extension 4.x+ (preview)](#tab/extensionv4/isolated-process)

Example pending.

# [Functions 2.x+](#tab/functionsv2/csharp-script)

The following example shows an Azure Cosmos DB trigger binding in a *function.json* file and a [C# script function](functions-reference-csharp.md) that uses the binding. The function writes log messages when Azure Cosmos DB records are added or modified.

Here's the binding data in the *function.json* file:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseCollectionName": "leases",
    "connectionStringSetting": "<connection-app-setting>",
    "databaseName": "Tasks",
    "collectionName": "Items",
    "createLeaseCollectionIfNotExists": true
}
```

Here's the C# script code:

```cs
    #r "Microsoft.Azure.DocumentDB.Core"

    using System;
    using Microsoft.Azure.Documents;
    using System.Collections.Generic;
    using Microsoft.Extensions.Logging;

    public static void Run(IReadOnlyList<Document> documents, ILogger log)
    {
      log.LogInformation("Documents modified " + documents.Count);
      log.LogInformation("First document Id " + documents[0].Id);
    }
```

# [Extension 4.x+ (preview)](#tab/extensionv4/csharp-script)

The following example shows an Azure Cosmos DB trigger binding in a *function.json* file and a [C# script function](functions-reference-csharp.md) that uses the binding. The function writes log messages when Azure Cosmos DB records are added or modified.

Here's the binding data in the *function.json* file:

```json
{
    "type": "cosmosDBTrigger",
    "name": "documents",
    "direction": "in",
    "leaseContainerName": "leases",
    "connection": "<connection-app-setting>",
    "databaseName": "Tasks",
    "containerName": "Items",
    "createLeaseContainerIfNotExists": true
}
```

Here's the C# script code:

```cs
    #r "Microsoft.Azure.DocumentDB.Core"

    using System;
    using Microsoft.Azure.Documents;
    using System.Collections.Generic;
    using Microsoft.Extensions.Logging;

    public static void Run(IReadOnlyList<Document> documents, ILogger log)
    {
      log.LogInformation("Documents modified " + documents.Count);
      log.LogInformation("First document Id " + documents[0].Id);
    }
```

---

::: zone-end
::: zone pivot="programming-language-java"

This function is invoked when there are inserts or updates in the specified database and collection.

# [Functions 2.x+](#tab/functionsv2)

```java
    @FunctionName("cosmosDBMonitor")
    public void cosmosDbProcessor(
        @CosmosDBTrigger(name = "items",
            databaseName = "ToDoList",
            collectionName = "Items",
            leaseCollectionName = "leases",
            createLeaseCollectionIfNotExists = true,
            connectionStringSetting = "AzureCosmosDBConnection") String[] items,
            final ExecutionContext context ) {
                context.getLogger().info(items.length + "item(s) is/are changed.");
            }
```
# [Extension 4.x+ (preview)](#tab/extensionv4)

[!INCLUDE [functions-cosmosdb-extension-java-note](../../includes/functions-cosmosdb-extension-java-note.md)]

---

In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `@CosmosDBTrigger` annotation on parameters whose value would come from Azure Cosmos DB.  This annotation can be used with native Java types, POJOs, or nullable values using `Optional<T>`.

::: zone-end  
::: zone pivot="programming-language-javascript"  

The following example shows an Azure Cosmos DB trigger binding in a *function.json* file and a [JavaScript function](functions-reference-node.md) that uses the binding. The function writes log messages when Azure Cosmos DB records are added or modified.

Here's the binding data in the *function.json* file:

[!INCLUDE [functions-cosmosdb-trigger-attributes](../../includes/functions-cosmosdb-trigger-attributes.md)]

Here's the JavaScript code:

```javascript
    module.exports = async function (context, documents) {
      context.log('First document Id modified : ', documents[0].id);
    }
```

::: zone-end  
::: zone pivot="programming-language-powershell"  

The following example shows how to run a function as data changes in Azure Cosmos DB.

[!INCLUDE [functions-cosmosdb-trigger-attributes](../../includes/functions-cosmosdb-trigger-attributes.md)]

In the _run.ps1_ file, you have access to the document that triggers the function via the `$Documents` parameter.

```powershell
param($Documents, $TriggerMetadata) 

Write-Host "First document Id modified : $($Documents[0].id)" 
```

::: zone-end  
::: zone pivot="programming-language-python"  

The following example shows an Azure Cosmos DB trigger binding in a *function.json* file and a [Python function](functions-reference-python.md) that uses the binding. The function writes log messages when Azure Cosmos DB records are modified.

Here's the binding data in the *function.json* file:

[!INCLUDE [functions-cosmosdb-trigger-attributes](../../includes/functions-cosmosdb-trigger-attributes.md)]

Here's the Python code:

```python
    import logging
    import azure.functions as func


    def main(documents: func.DocumentList) -> str:
        if documents:
            logging.info('First document Id modified: %s', documents[0]['id'])
```

::: zone-end  
::: zone pivot="programming-language-csharp"
## Attributes

Both [in-process](functions-dotnet-class-library.md) and [isolated process](dotnet-isolated-process-guide.md) C# libraries use the [CosmosDBTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions.CosmosDB/Trigger/CosmosDBTriggerAttribute.cs) to define the function. C# script instead uses a function.json configuration file.

# [Functions 2.x+](#tab/functionsv2/in-process)

[!INCLUDE [functions-cosmosdb-attributes-v3](../../includes/functions-cosmosdb-attributes-v3.md)] 

# [Extension 4.x+ (preview)](#tab/extensionv4/in-process)

[!INCLUDE [functions-cosmosdb-attributes-v4](../../includes/functions-cosmosdb-attributes-v4.md)]

# [Functions 2.x+](#tab/functionsv2/isolated-process)

[!INCLUDE [functions-cosmosdb-attributes-v3](../../includes/functions-cosmosdb-attributes-v3.md)]

# [Extension 4.x+ (preview)](#tab/extensionv4/isolated-process)

[!INCLUDE [functions-cosmosdb-attributes-v4](../../includes/functions-cosmosdb-attributes-v4.md)]

# [Functions 2.x+](#tab/functionsv2/csharp-script)

[!INCLUDE [functions-cosmosdb-settings-v3](../../includes/functions-cosmosdb-settings-v3.md)]

# [Extension 4.x+ (preview)](#tab/extensionv4/csharp-script)

[!INCLUDE [functions-cosmosdb-settings-v4](../../includes/functions-cosmosdb-settings-v4.md)]

---

::: zone-end  
::: zone pivot="programming-language-java"  
## Annotations

# [Functions 2.x+](#tab/functionsv2)

From the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `@CosmosDBInput` annotation on parameters that read data from Azure Cosmos DB. The annotation supports the following properties:

+ [name](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.name)
+ [connectionStringSetting](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.connectionstringsetting)
+ [databaseName](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.databasename)
+ [collectionName](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.collectionname)
+ [leaseConnectionStringSetting](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leaseconnectionstringsetting)
+ [leaseDatabaseName](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leasedatabasename)
+ [leaseCollectionName](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leasecollectionname)
+ [createLeaseCollectionIfNotExists](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.createleasecollectionifnotexists)
+ [leasesCollectionThroughput](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leasescollectionthroughput)
+ [leaseCollectionPrefix](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leasecollectionprefix)
+ [feedPollDelay](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.feedpolldelay)
+ [leaseAcquireInterval](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leaseacquireinterval)
+ [leaseExpirationInterval](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leaseexpirationinterval)
+ [leaseRenewInterval](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.leaserenewinterval)
+ [checkpointInterval](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.checkpointinterval)
+ [checkpointDocumentCount](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.checkpointdocumentcount)
+ [maxItemsPerInvocation](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.maxitemsperinvocation)
+ [startFromBeginning](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.startfrombeginning)
+ [preferredLocations](/java/api/com.microsoft.azure.functions.annotation.cosmosdbtrigger.preferredlocations)

# [Extension 4.x+ (preview)](#tab/extensionv4)

[!INCLUDE [functions-cosmosdb-extension-java-note](../../includes/functions-cosmosdb-extension-java-note.md)]

---

::: zone-end  
::: zone pivot="programming-language-javascript,programming-language-powershell,programming-language-python"  
## Configuration

The following table explains the binding configuration properties that you set in the *function.json* file, where properties differ by extension version:  

# [Functions 2.x+](#tab/functionsv2)

[!INCLUDE [functions-cosmosdb-settings-v3](../../includes/functions-cosmosdb-settings-v3.md)]

# [Extension 4.x+ (preview)](#tab/extensionv4)

[!INCLUDE [functions-cosmosdb-settings-v4](../../includes/functions-cosmosdb-settings-v4.md)]

---

::: zone-end  

See the [Example section](#example) for complete examples.

## Usage

The parameter type supported by the Azure Cosmos DB trigger depends on the Functions runtime version, the extension package version, and the C# modality used.

The trigger requires a second collection that it uses to store _leases_ over the partitions. Both the collection being monitored and the collection that contains the leases must be available for the trigger to work.

::: zone pivot="programming-language-csharp"  
>[!IMPORTANT]
> If multiple functions are configured to use an Azure Cosmos DB trigger for the same collection, each of the functions should use a dedicated lease collection or specify a different `LeaseCollectionPrefix` for each function. Otherwise, only one of the functions is triggered. For information about the prefix, see the [Attributes section](#attributes).
::: zone-end
::: zone pivot="programming-language-java"  
>[!IMPORTANT]
> If multiple functions are configured to use an Azure Cosmos DB trigger for the same collection, each of the functions should use a dedicated lease collection or specify a different `leaseCollectionPrefix` for each function. Otherwise, only one of the functions is triggered. For information about the prefix, see the [Annotations section](#annotations).
::: zone-end
::: zone pivot="programming-language-javascript,programming-language-powershell,programming-language-python"  
>[!IMPORTANT]
> If multiple functions are configured to use an Azure Cosmos DB trigger for the same collection, each of the functions should use a dedicated lease collection or specify a different `leaseCollectionPrefix` for each function. Otherwise, only one of the functions will be triggered. For information about the prefix, see the [Configuration section](#configuration).
::: zone-end

The trigger doesn't indicate whether a document was updated or inserted, it just provides the document itself. If you need to handle updates and inserts differently, you could do that by implementing timestamp fields for insertion or update.

[!INCLUDE [functions-cosmosdb-connections](../../includes/functions-cosmosdb-connections.md)]

## Next steps

- [Read an Azure Cosmos DB document (Input binding)](./functions-bindings-cosmosdb-v2-input.md)
- [Save changes to an Azure Cosmos DB document (Output binding)](./functions-bindings-cosmosdb-v2-output.md)

[version 4.x of the extension]: ./functions-bindings-cosmosdb-v2.md?tabs=extensionv4
