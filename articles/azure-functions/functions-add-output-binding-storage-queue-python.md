---
title: Add an Azure Storage queue binding to your Python function 
description: Integrate an Azure Storage queue with a Python function using an output binding.
ms.date: 01/15/2020
ms.topic: quickstart
---

# Add an Azure Storage queue binding to your Python function

In this article, you integrate an Azure Storage queue with the function and storage account you created in [Create an HTTP triggered Python function](functions-create-first-function-python.md). You achieve this integration by using an *output binding* that writes data from an HTTP request to a message in the queue. Completing this article incurs no additional costs beyond the few USD cents of the previous quickstart.

## Prerequisites

- Complete the quickstart, [Create an HTTP triggered Python function](functions-create-first-function-python.md). If you already cleaned up resources at the end of that article, go through the steps again to recreate the Functions app in Azure, but leave the resources in place.

## Retrieve the Azure Storage connection string

When you created a function app in Azure in the previous quickstart, you also created a Storage account. The connection string for this account is stored securely in app settings in Azure. By downloading the setting into the *local.settings.json* file, you can use that connection write to a Storage queue in the same account when running the function locally. 

1. From the root of the project, run the following command, replacing `<app_name>` with the name of your function app from the previous quickstart. This command will overwrite any existing values in the file.

    ```
    func azure functionapp fetch-app-settings <app_name>
    ```
    
1. Open *local.settings.json* and locate the value named `AzureWebJobsStorage`, which is the Storage account connection string. You use the name `AzureWebJobsStorage` and the connection string in other sections of this article.

> [!IMPORTANT]
> Because *local.settings.json* contains secrets downloaded from Azure, always exclude this file from source control. The *.gitignore* file created with a local functions project excludes the file by default.

## Add an output binding to function.json

Although a function can have only one trigger, it can have multiple input and output bindings, which let you connect to other Azure services and resources without writing custom integration code. You declare these bindings in the *function.json* file in your function folder as appropriate for the language you're using for the function.

From the previous quickstart, your *function.json* file in the *HttpExample* folder contains two bindings in the `bindings` collection:

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

Each binding has at least a type, a direction, and a name. In the example above, the first binding is of type `httpTrigger` with the direction `in`. For the `in` direction, `name` specifies the name of an input parameter that's sent to the function when invoked by the trigger.

The second binding is of type `http` with the direction `out`, in which case the special `name` of `$return` indicates that this binding uses the function's return value rather than providing an input parameter.

To write to an Azure Storage queue from this function, add an `out` binding of type `queue` with the name `msg`, as shown in the code below:

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "msg",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

In this case, `msg` is given to the function as an output argument. For a `queue` type, you must also specify the name of the queue in `queueName` and provide the *name* of the Azure Storage connection (from *local.settings.json*) in `connection`.

For more information on the details of bindings, see [Azure Functions triggers and bindings concepts](functions-triggers-bindings.md) and [queue output configuration](functions-bindings-storage-queue.md#output---configuration).

## Add code to use the output binding

With the queue binding specified in *function.json*, you can now update your function to receive the `msg` output parameter and write messages to the queue.

Update *HttpExample\\\_\_init\_\_.py* to match the following code, adding the `msg` parameter to the function definition and `msg.set(name)` under the `if name:` statement.

```python
import logging

import azure.functions as func


def main(req: func.HttpRequest, msg: func.Out[func.QueueMessage]) -> str:

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        msg.set(name)
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )
```

The `msg` parameter is an instance of the [`azure.functions.InputStream class`](/python/api/azure-functions/azure.functions.httprequest). Its `set` method writes a string message to the queue, in this case the name passed to the function in the URL query string.

Observe that you *don't* need to write any code for authentication, getting a queue reference, or writing data. All these integration tasks are conveniently handled in the Azure Functions runtime and queue output binding.

## Run and test the function locally

1. In a terminal or command prompt, navigate to your function project folder, *LocalFunctionProj*.

1. Start the Azure Functions runtime host by using the following command.

    ```
    func host start
    ```

1. Once startup is complete and you see the URL for the `HttpExample` endpoint, copy its URL to a browser and append the query string `?name=<your-name>`, making the full URL like `http://localhost:7071/api/HttpExample?name=Guido`. The browser should display a message like `Hello Guido` as in the previous article.

    If you don't see the `HttpExample` endpoint appear, stop the host with **Ctrl**+**C** and check the output for errors. For example, the host won't activate the endpoint if there's an error in *function.json*. Also check that you are running `func host start` from the functions project folder and not the *HttpExample* folder.

1. When you're done, stop the host with **Ctrl**+**C**.

> [!TIP]
> During startup, the host downloads and installs the [Storage binding extension](functions-bindings-storage-blob.md#packages---functions-2x-and-higher) and other Microsoft binding extensions. This installation happens because binding extensions are enabled by default in the *host.json* file with the following properties:
>
> ```json
> {
>     "version": "2.0",
>     "extensionBundle": {
>         "id": "Microsoft.Azure.Functions.ExtensionBundle",
>         "version": "[1.*, 2.0.0)"
>     }
> }
> ```
>
> If you encounter any errors related to binding extensions, check that the above properties are present in *host.json*.

## View the message in the Azure Storage queue

When your function generates an HTTP response for the web browser, it also calls `msg.set(name)`, which writes a message to an Azure Storage queue named `outqueue`, as specified in the queue binding. You can view the queue in the [Azure portal](../storage/queues/storage-quickstart-queues-portal.md) or in the  [Microsoft Azure Storage Explorer](https://storageexplorer.com/). You can also view the queue in the Azure CLI as described in the following steps:

1. Open the function project's *local.setting.json* file and copy the connection string value. In a terminal or command window, run the following command to create an environment variable named `AZURE_STORAGE_CONNECTION_STRING`, pasting your specific connection string in place of  `<connection_string>`. (This environment variable means you don't need to supply the connection string to each subsequent command using the `--connection-string` argument.)

    # [bash](#tab/bash)
    
    ```bash
    AZURE_STORAGE_CONNECTION_STRING="<connection_string>"
    ```
    
    # [PowerShell](#tab/powershell)
    
    ```powershell
    $env:AZURE_STORAGE_CONNECTION_STRING = "<connection_string>"
    ```
    
    # [Cmd](#tab/cmd)
    
    ```cmd
    set AZURE_STORAGE_CONNECTION_STRING="<connection_string>"
    ```
    
    ---
    
1. (Optional) Use the [`az storage queue list`](/cli/azure/storage/queue#az-storage-queue-list) command to view the Storage queues in your account. The output from this command should include a queue named `outqueue`, which was created when the function wrote its first message to that queue.
    
    # [bash](#tab/bash)
    
    ```bash
    az storage queue list --output tsv
    ```
    
    # [PowerShell](#tab/powershell)
    
    ```powershell
    az storage queue list --output tsv
    ```
    
    # [Cmd](#tab/cmd)
    
    ```cmd
    az storage queue list --output tsv
    ```
    
    ---


1. Use the [`az storage message peek`](/cli/azure/storage/message#az-storage-message-peek) command to view the messages in this queue, which should be the first name you used when testing the function earlier. The command retrieves the first message in the queue in [base64 encoding](functions-bindings-storage-queue.md#encoding), so you must also decode the message to view as text.

    # [bash](#tab/bash)
    
    ```bash
    echo `echo $(az storage message peek --queue-name outqueue -o tsv --query '[].{Message:content}') | base64 --decode`
    ```
    
    # [PowerShell](#tab/powershell)
    
    ```powershell
    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($(az storage message peek --queue-name outqueue -o tsv --query '[].{Message:content}')))
    ```
    
    # [Cmd](#tab/cmd)
    
    Because you need to dereference the message collection and decode from base64, run PowerShell and use the PowerShell command.

    ---
    
## Redeploy the project to Azure

Now that you've tested the function locally and verified that it wrote a message to the Azure Storage queue, you can redeploy your project to update the endpoint running on Azure.

1. In the *LocalFunctionsProj* folder, use the [`func azure functionapp publish`](functions-run-local.md#project-file-deployment) command to redeploy the project, replacing`<app_name>` with the name of your app.

    ```
    func azure functionapp publish <app_name>
    ```
    
1. As in the previous quickstart, use a browser or CURL to test the redeployed function.

    # [Browser](#tab/browser)
    
    Copy the complete **Invoke url** shown in the output of the publish command into a browser address bar, appending the query parameter `&name=Azure`. The browser should display similar output as when you ran the function locally.

    ![The output of the function run on Azure in a browser](./media/functions-create-first-function-python/function-test-cloud-browser.png)

    # [curl](#tab/curl)
    
    Run [curl](https://curl.haxx.se/) with the **Invoke url**, appending the parameter `&name=Azure`. The output of the command should be the text, "Hello Azure".
    
    ![The output of the function run on Azure using curl](./media/functions-create-first-function-python/function-test-cloud-curl.png)

    --- 

1. Examine the Storage queue again, as described in the previous section, to verify that it contains the new message written to the queue.

    If you're using the Azure CLI to examine the queue, the `az storage message peek` command shows only the first message in the queue. To simulate processing the messages, use `az storage message get` instead with all the same arguments. The `get` command returns the message and removes it from the queue. You can then repeat the same command until the queue is empty (and the command gives an error).

## Clean up resources

If you continue to the next step, [Enable Application Insights integration](functions-monitoring.md#manually-connect-an-app-insights-resource), keep all your resources in place as you'll build on what you've already done.

Otherwise, use the following command to delete the resource group and all its contained resources to avoid incurring further costs.

```azurecli
az group delete --name AzureFunctionsQuickstart-rg
```

## Next steps

> [!div class="nextstepaction"]
> [Enable Application Insights integration with an Azure Functions app](functions-monitoring.md#manually-connect-an-app-insights-resource)

Other resources:

- [Examples of complete Function projects in Python](/samples/browse/?products=azure-functions&languages=python).
- [Azure Functions Python developer guide](functions-reference-python.md)
- [Azure Functions triggers and bindings](functions-triggers-bindings.md).
- [Functions pricing page](https://azure.microsoft.com/pricing/details/functions/)
- [Estimating Consumption plan costs](functions-consumption-costs.md) article.
