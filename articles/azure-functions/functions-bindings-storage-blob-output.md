---
title: Azure Blob storage output binding for Azure Functions
description: Learn how to provide Azure Blob storage output binding data to an Azure Function.
ms.topic: reference
ms.date: 03/04/2022
ms.devlang: csharp, java, javascript, powershell, python
ms.custom: devx-track-csharp, devx-track-python, ignite-2022
zone_pivot_groups: programming-languages-set-functions-lang-workers
---

# Azure Blob storage output binding for Azure Functions

The output binding allows you to modify and delete blob storage data in an Azure Function.

For information on setup and configuration details, see the [overview](./functions-bindings-storage-blob.md).

## Example

::: zone pivot="programming-language-csharp"

[!INCLUDE [functions-bindings-csharp-intro](../../includes/functions-bindings-csharp-intro.md)]

# [In-process](#tab/in-process)

The following example is a [C# function](functions-dotnet-class-library.md) that runs in-process and uses a blob trigger and two output blob bindings. The function is triggered by the creation of an image blob in the *sample-images* container. It creates small and medium size copies of the image blob.

```csharp
using System.Collections.Generic;
using System.IO;
using Microsoft.Azure.WebJobs;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.PixelFormats;
using SixLabors.ImageSharp.Processing;

public class ResizeImages
{
    [FunctionName("ResizeImage")]
    public static void Run([BlobTrigger("sample-images/{name}")] Stream image,
        [Blob("sample-images-sm/{name}", FileAccess.Write)] Stream imageSmall,
        [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageMedium)
    {
        IImageFormat format;

        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageSmall, ImageSize.Small, format);
        }

        image.Position = 0;
        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageMedium, ImageSize.Medium, format);
        }
    }

    public static void ResizeImage(Image<Rgba32> input, Stream output, ImageSize size, IImageFormat format)
    {
        var dimensions = imageDimensionsTable[size];

        input.Mutate(x => x.Resize(dimensions.Item1, dimensions.Item2));
        input.Save(output, format);
    }

    public enum ImageSize { ExtraSmall, Small, Medium }

    private static Dictionary<ImageSize, (int, int)> imageDimensionsTable = new Dictionary<ImageSize, (int, int)>() {
        { ImageSize.ExtraSmall, (320, 200) },
        { ImageSize.Small,      (640, 400) },
        { ImageSize.Medium,     (800, 600) }
    };

}
```

# [Isolated process](#tab/isolated-process)

The following example is a [C# function](dotnet-isolated-process-guide.md) that runs in an isolated worker process and uses a blob trigger with both blob input and blob output blob bindings. The function is triggered by the creation of a blob in the *test-samples-trigger* container. It reads a text file from the *test-samples-input* container and creates a new text file in an output container based on the name of the triggered file.

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/Blob/BlobFunction.cs" range="4-26":::

# [C# Script](#tab/csharp-script)

The following example shows blob input and output bindings in a *function.json* file and [C# script (.csx)](functions-reference-csharp.md) code that uses the bindings. The function makes a copy of a text blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

The [configuration](#configuration) section explains these properties.

Here's the C# script code:

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

---

::: zone-end
::: zone pivot="programming-language-java"

This section contains the following examples:

- [HTTP trigger, using OutputBinding](#http-trigger-using-outputbinding-java)
- [Queue trigger, using function return value](#queue-trigger-using-function-return-value-java)

#### HTTP trigger, using OutputBinding (Java)

 The following example shows a Java function that uses the `HttpTrigger` annotation to receive a parameter containing the name of a file in a blob storage container. The `BlobInput` annotation then reads the file and passes its contents to the function as a `byte[]`. The `BlobOutput` annotation binds to `OutputBinding outputItem`, which is then used by the function to write the contents of the input blob to the configured storage container.

```java
  @FunctionName("copyBlobHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage copyBlobHttp(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    @BlobOutput(
      name = "target", 
      path = "myblob/{Query.file}-CopyViaHttp")
    OutputBinding<String> outputItem,
    final ExecutionContext context) {
      // Save blob to outputItem
      outputItem.setValue(new String(content, StandardCharsets.UTF_8));

      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### Queue trigger, using function return value (Java)

 The following example shows a Java function that uses the `QueueTrigger` annotation to receive a message containing the name of a file in a blob storage container. The `BlobInput` annotation then reads the file and passes its contents to the function as a `byte[]`. The `BlobOutput` annotation binds to the function return value, which is then used by the runtime to write the contents of the input blob to the configured storage container.

```java
  @FunctionName("copyBlobQueueTrigger")
  @StorageAccount("Storage_Account_Connection_String")
  @BlobOutput(
    name = "target", 
    path = "myblob/{queueTrigger}-Copy")
  public String copyBlobQueue(
    @QueueTrigger(
      name = "filename", 
      dataType = "string",
      queueName = "myqueue-items") 
    String filename,
    @BlobInput(
      name = "file", 
      path = "samples-workitems/{queueTrigger}") 
    String content,
    final ExecutionContext context) {
      context.getLogger().info("The content of \"" + filename + "\" is: " + content);
      return content;
  }
```

 In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `@BlobOutput` annotation on function parameters whose value would be written to an object in blob storage.  The parameter type should be `OutputBinding<T>`, where `T` is any native Java type or a POJO.

::: zone-end  
::: zone pivot="programming-language-javascript"  

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [JavaScript code](functions-reference-node.md) that uses the bindings. The function makes a copy of a blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

The [configuration](#configuration) section explains these properties.

Here's the JavaScript code:

```javascript
module.exports = async function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
};
```

::: zone-end  
::: zone pivot="programming-language-powershell"  

The following example demonstrates how to create a copy of an incoming blob as the output from a [PowerShell function](functions-reference-powershell.md).

In the function's configuration file (*function.json*), the `trigger` metadata property is used to specify the output blob name in the `path` properties.

> [!NOTE]
> To avoid infinite loops, make sure your input and output paths are different.

```json
{
  "bindings": [
    {
      "name": "myInputBlob",
      "path": "data/{trigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in",
      "type": "blobTrigger"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "data/copy/{trigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Here's the PowerShell code:

```powershell
# Input bindings are passed in via param block.
param([byte[]] $myInputBlob, $TriggerMetadata)
Write-Host "PowerShell Blob trigger function Processed blob Name: $($TriggerMetadata.Name)"
Push-OutputBinding -Name myOutputBlob -Value $myInputBlob
```


::: zone-end  
::: zone pivot="programming-language-python"  

<!--Same example for input and output. -->

The following example shows blob input and output bindings in a *function.json* file and [Python code](functions-reference-python.md) that uses the bindings. The function makes a copy of a blob. The function is triggered by a queue message that contains the name of the blob to copy. The new blob is named *{originalblobname}-Copy*.

In the *function.json* file, the `queueTrigger` metadata property is used to specify the blob name in the `path` properties:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "dataType": "binary",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "outputblob",
      "type": "blob",
      "dataType": "binary",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

The [configuration](#configuration) section explains these properties.

Here's the Python code:

```python
import logging
import azure.functions as func


def main(queuemsg: func.QueueMessage, inputblob: bytes, outputblob: func.Out[bytes]):
    logging.info(f'Python Queue trigger function processed {len(inputblob)} bytes')
    outputblob.set(inputblob)
```

---

::: zone-end  
::: zone pivot="programming-language-csharp"
## Attributes

Both [in-process](functions-dotnet-class-library.md) and [isolated worker process](dotnet-isolated-process-guide.md) C# libraries use attribute to define the function. C# script instead uses a function.json configuration file.

# [In-process](#tab/in-process)

The [BlobAttribute](/dotnet/api/microsoft.azure.webjobs.blobattribute) attribute's constructor takes the following parameters:

|Parameter | Description|
|---------|----------------------|
|**BlobPath** | The path to the blob.|
|**Connection** | The name of an app setting or setting collection that specifies how to connect to Azure Blobs. See [Connections](#connections).|
|**Access** | Indicates whether you will be reading or writing.|

The following example sets the path to the blob and a `FileAccess` parameter indicating write for an output binding:

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
{
    ...
}
```

[!INCLUDE [functions-bindings-storage-attribute](../../includes/functions-bindings-storage-attribute.md)]

# [Isolated process](#tab/isolated-process)

The `BlobOutputAttribute` constructor takes the following parameters:

|Parameter | Description|
|---------|----------------------|
|**BlobPath** | The path to the blob.|
|**Connection** | The name of an app setting or setting collection that specifies how to connect to Azure Blobs. See [Connections](#connections).|


# [C# script](#tab/csharp-script)

The following table explains the binding configuration properties for C# script that you set in the *function.json* file. 

|function.json property | Description|
|---------|----------------------|
|**type** | Must be set to `blob`. |
|**direction** | Must be set to `in`. Exceptions are noted in the [usage](#usage) section. |
|**name** | The name of the variable that represents the blob in function code.|
|**path** | The path to the blob. |
|**connection** | The name of an app setting or setting collection that specifies how to connect to Azure Blobs. See [Connections](#connections).|
|**dataType**| For dynamically typed languages, specifies the underlying data type. Possible values are `string`, `binary`, or `stream`. For more detail, refer to the [triggers and bindings concepts](functions-triggers-bindings.md?tabs=python#trigger-and-binding-definitions). |

---

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

::: zone-end  
::: zone pivot="programming-language-java"  
## Annotations

The `@BlobOutput` attribute gives you access to the blob that triggered the function. If you use a byte array with the attribute, set `dataType` to `binary`. Refer to the [output example](#example) for details.
::: zone-end  
::: zone pivot="programming-language-javascript,programming-language-powershell,programming-language-python"  
## Configuration

The following table explains the binding configuration properties that you set in the *function.json* file. 

|function.json property | Attribute property |Description|
|---------|---------|----------------------|
|**type** | Must be set to `blob`. |
|**direction** | Must be set to `out` for an output binding. Exceptions are noted in the [usage](#usage) section. |
|**name** | The name of the variable that represents the blob in function code.  Set to `$return` to reference the function return value.|
|**path** | The path to the blob container. |
|**connection** | The name of an app setting or setting collection that specifies how to connect to Azure Blobs. See [Connections](#connections).|

::: zone-end  

See the [Example section](#example) for complete examples.

## Usage

::: zone pivot="programming-language-csharp"  
The usage of the Blob output binding depends on the extension package version, and the C# modality used in your function app, which can be one of the following:

[!INCLUDE [functions-bindings-blob-storage-usage-csharp](../../includes/functions-bindings-blob-storage-usage-csharp.md)]


::: zone-end  
<!--Any of the below pivots can be combined if the usage info is identical.-->
::: zone pivot="programming-language-java"

The `@BlobOutput` attribute gives you access to the blob that triggered the function. If you use a byte array with the attribute, set `dataType` to `binary`. Refer to the [output example](#example) for details.

::: zone-end  
::: zone pivot="programming-language-javascript" 
 
Access the blob data using `context.bindings.<BINDING_NAME>`, where the binding name is defined in the _function.json_ file.

::: zone-end  
::: zone pivot="programming-language-powershell"  

Access the blob data via a parameter that matches the name designated by binding's name parameter in the _function.json_ file.

::: zone-end  
::: zone pivot="programming-language-python"  

You can declare function parameters as the following types to write out to blob storage:

- Strings as `func.Out[str]`
- Streams as `func.Out[func.InputStream]`

Refer to the [output example](#example) for details.

::: zone-end  

[!INCLUDE [functions-storage-blob-connections](../../includes/functions-storage-blob-connections.md)]

## Exceptions and return codes

| Binding |  Reference |
|---|---|
| Blob | [Blob Error Codes](/rest/api/storageservices/fileservices/blob-service-error-codes) |
| Blob, Table, Queue |  [Storage Error Codes](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Blob, Table, Queue |  [Troubleshooting](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## Next steps

- [Run a function when blob storage data changes](./functions-bindings-storage-blob-trigger.md)
- [Read blob storage data when a function runs](./functions-bindings-storage-blob-input.md)
