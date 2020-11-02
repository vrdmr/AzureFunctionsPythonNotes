# Intro to Azure Functions (Python)

Tags: #azurefunctions #azurefunctionspython #tutorial

- [Intro to Azure Functions (Python)](#intro-to-azure-functions-python)
  - [Functions in action](#functions-in-action)
    - [Using CLI](#using-cli)
    - [Using VSCode and Azure Functions extension](#using-vscode-and-azure-functions-extension)
    - [Structure of the Functions](#structure-of-the-functions)
    - [Programming model of Functions](#programming-model-of-functions)
  - [Worker Architecture (Azure Functions worker)](#worker-architecture-azure-functions-worker)
    - [Bigger picture](#bigger-picture)
  - [Azure Functions library for Python](#azure-functions-library-for-python)
  - [Use-cases](#use-cases)
  - [Closing thoughts](#closing-thoughts)

## Functions in action

Azure Function provides event-driven way to run "functions" in a "serverless" fashion. There are various supported triggers which can generate specific events which could trigger the functions. You can utilize SDKs to talk to different services, but AzFunc also provides a set of bindings and integrating with other services is streamlined by using these bindings.

### Using CLI

`func` - [Azure-functions-core-tools](https://github.com/Azure/azure-functions-core-tools)

*Usage*:

```bash
# Create Function app in Azure
$ az functionapp create --consumption-plan-location westus --name PythonFunctionAppDemo --os-type Linux --resource-group PythonFunctionsDemo --runtime python -s pyfuncdemostorageaccnt --functions-version 3

## Your Linux function app 'PythonFunctionAppDemo',
## that uses a consumption plan has been successfully
## created but is not active until content is published
## using Azure Portal or the Functions Core Tools.

# Create Function app locally
$ func init --worker-runtime python

# Create an Http Trigger Function inside the function app
$ func new --language python --template "Http Trigger" --name AzPyDemo

# Test locally by using the functions runtime
$ func host start 

# Deploy the function app to the cloud
$ func azure functionapp publish PythonFunctionAppDemo
```

Reference: [Work with Azure Functions Core Tools | Microsoft Docs](https://docs.microsoft.com/azure/azure-functions/functions-run-local)

### Using VSCode and Azure Functions extension

- VS Code extension - Azure Functions
  - Create New Project
  - Create New Functions
  - Add new bindings to an existing function app
- F5 Debugging
- Deploy to Azure

### Structure of the Functions

Recommended folder structure for a Python Functions project:

```text
__app__
 | - my_first_function
 | | - __init__.py
 | | - function.json
 | | - example.py
 | - my_second_function
 | | - __init__.py
 | | - function.json
 | - shared_code
 | | - my_first_helper_function.py
 | | - my_second_helper_function.py
 | - host.json
 | - requirements.txt
 | - Dockerfile
 tests
```

Example function.json

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },{
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

Example script file:

```python
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    return func.HttpResponse("NoOp", status_code=200)
```

*Extra notes:*

- *local.settings.json*: Used to store app settings and connection strings when running locally. This file doesn’t get published to Azure. To learn more, see  [local.settings.file](https://docs.microsoft.com/azure/azure-functions/functions-run-local#local-settings-file) .
- *requirements.txt*: Contains the list of packages the system installs when publishing to Azure.
- *host.json*: Contains global configuration options that affect all functions in a function app. This file does get published to Azure. Not all options are supported when running locally. To learn more, see  [host.json](https://docs.microsoft.comazure/azure-functions/functions-host-json) .
- *.funcignore*: (Optional) declares files that shouldn’t get published to Azure.
- *Dockerfile*: (Optional) used when publishing your project in a  [custom container](https://docs.microsoft.comazure/azure-functions/functions-create-function-linux-custom-image) .

### Programming model of Functions

Reference: [Python developer reference for Azure Functions | Microsoft Docs](https://docs.microsoft.com/azure/azure-functions/functions-reference-python)
By default, the runtime expects the method to be implemented as a global method called `main()` in the `__init__.py` file. You can change the entry point using the `“entryPoint”` key in the *function.json*.

Data from triggers and bindings is bound to the function via method attributes using the name property defined in the *function.json* file.

## Worker Architecture (Azure Functions worker)

The Python Worker is an asynchronous [gRPC](https://grpc.io/docs/guides/) client controlled by the WebHost. The worker manages loading and execution of Python functions, as well as providing convenient Python wrappers for [Azure Functions data types](https://github.com/Azure/azure-functions-python-library). Python worker is built on top of the  [asyncio](https://docs.python.org/3/library/asyncio.html)  module.

![handle_function_load](Intro%20to%20Azure%20Functions/image.png)

Loading request:

```bash
[2020-11-01T23:55:33.380] Switched to gRPC logging.
[2020-11-01T23:55:33.390] Received WorkerInitRequest, request ID 0709510b-6924-4251-ba25-b9dedd15422a
[2020-11-01T23:55:33.404] Worker process started and initialized.
[2020-11-01T23:55:33.409] Received FunctionLoadRequest, request ID: 0709510b-6924-4251-ba25-b9dedd15422a, function ID: 700372df-3cae-4760-9ea3-cd6fb6fdb554
[2020-11-01T23:55:33.505] Successfully processed FunctionLoadRequest, request ID: 0709510b-6924-4251-ba25-b9dedd15422a, function ID: 700372df-3cae-4760-9ea3-cd6fb6fdb554
```

![handle_function_invocation](Intro%20to%20Azure%20Functions/image%202.png)

Invocation Request:

```bash
[2020-11-01T23:57:43.095] Function is sync, request ID: 0709510b-6924-4251-ba25-b9dedd15422a,function ID: 700372df-3cae-4760-9ea3-cd6fb6fdb554, invocation ID: 61450e7b-3d7d-45ff-ab49-109def540752
[2020-11-01T23:57:43.099] Python HTTP trigger function processed a request.
[2020-11-01T23:57:43.099] Successfully processed FunctionInvocationRequest, request ID: 0709510b-6924-4251-ba25-b9dedd15422a, function ID: 700372df-3cae-4760-9ea3-cd6fb6fdb554, invocation ID: 61450e7b-3d7d-45ff-ab49-109def540752
[2020-11-01T23:57:43.155] Executed 'Functions.HttpTrigger1' (Succeeded, Id=61450e7b-3d7d-45ff-ab49-109def540752, Duration=151ms)
[2020-11-01T23:57:43.176] Executed HTTP request: {
[2020-11-01T23:57:43.176]   requestId: "a6af917a-0ab0-4805-ad32-7b08cfef05fe",
[2020-11-01T23:57:43.176]   identities: "",
[2020-11-01T23:57:43.176]   status: "200",
[2020-11-01T23:57:43.176]   duration: "275"
[2020-11-01T23:57:43.176] }
```

### Bigger picture

![inside_worker](Intro%20to%20Azure%20Functions/image%203.png)

![inside_worker_2x](Intro%20to%20Azure%20Functions/image%204.png)

![antares](Intro%20to%20Azure%20Functions/image%205.png)

For Linux Consumption, the worker pools are replaced by a Service Fabric mesh with different language containers running and ready.

## Azure Functions library for Python

[Azure Functions library](https://github.com/Azure/azure-functions-python-library) is an SDK that provides housing for data methods for triggers and bindings which developers use in functions.

Triggers / Bindings : HTTP, Blob, Queue, Timer, Cosmos DB, Event Grid, Event Hubs, Service Bus and Custom binding support

References:

- PyPI link: [azure-functions | PyPI](https://pypi.org/project/azure-functions)
- Reference link: [azure.functions package | Microsoft Docs](https://docs.microsoft.com/python/api/azure-functions/azure.functions?view=azure-python)

An example function -

```python
import azure.functions as func

def main(req: func.HttpRequest,
         obj: func.InputStream,
         msg: func.Out[func.QueueMessage]) -> func.HttpResponse:
    message = obj.read()
    msg.set(message)
    return func.HttpResponse("NoOp", status_code=200)
```

function.json

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous"
    },
    {
      "name": "obj",
      "direction": "in",
      "type": "blob",
      "path": "samples/{id}",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "msg",
      "direction": "out",
      "type": "queue",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "direction": "out",
      "type": "http"
    }
  ]
}
```

Inputs are divided into two categories in Azure Functions: trigger input and additional input.

## Use-cases

Some of the use-cases in the Python world -

1. Serverless APIs and design first features with `Autorest.AzFunc-Python`
  Python HTTP Triggers functions are used to build APIs. A good reference on why AzureFunctions are great for REST API specifications is available at [Going from 0 to 11 with REST APIs on Azure Functions | Dev.to](https://dev.to/azure/going-from-0-to-11-with-rest-apis-on-azure-functions-f2n).
  
    We facilitate the true\* design-first way of API development now, with the inclusion of the new feature - *HTTP Triggers from OpenAPI (v2/v3) Specification File*. Now developers only need an OpenAPI specification file for the web API they want to build and we would generate scaffolding to get you started and you only need to focus on the business logic.

    VSCode Example:

    ![autorest generation for azure function app in python](Intro%20to%20Azure%20Functions/StencilGenerationPython.gif)

    *GIF also located at [Azure Update: Generate a new function app from an OpenAPI specification](https://azure.microsoft.com/updates/generate-a-new-function-app-from-an-openapi-specification/)*

    CLI Example:

    ```bash
    autorest --azure-functions-python \
             --input-file:/path/to/spec.json \  
             --output-folder:./generated-azfunctions \
             --no-namespace-folders:true \
             --version:3.0.6320

    # Note: the `--input-file` could be a URL as well.
    ```

2. Machine Learning, Data Processing and ETL-lite
  Since ~2012 (don't quote me on this :)), Python has has displaced R as the preferred language of statisticians and data scientists, and there has been a significant rise in the number of modules and libraries available for doing data-related operations (NumPy, MatPlotLib, SciPy, PyTorch, Scikit-Learn, Tensorflow, ...). This rise has made Python to be the primary language for Machine Learning and Data Processing projects.

    Thus, there are many customers who build their Data Processing pipelines on Functions, where the data comes from different event sources, such as Service Bus, EventHub, and is transformed by the Functions. These new patterns have given rise to Serverless Data Processing pipelines.

3. Automation and Administration
  Since it's inception, Python has always been one of the primary tools in the arsenal of a system administrator for scripting complex workflows and administer large infrastructure set ups. And with the Timer and other event-triggers available in Functions, Python functions become the preferred way of doing system administration.

    An example tutorial on [Getting the list of Resource Groups using Azure Functions](https://docs.microsoft.com/samples/azure-samples/azure-functions-python-list-resource-groups/azure-functions-python-sample-list-resource-groups/) would help you setup and perform basic operations on your Azure Subscription.

## Closing thoughts

Python functions allow you to try from the latest in Machine Learning, to do all your infrastructure maintenance with the comfort of an easy-to-learn language. :)
