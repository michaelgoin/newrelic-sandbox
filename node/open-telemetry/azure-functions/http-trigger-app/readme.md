# Azure Functions: HTTP Trigger Example

An example Azure Function App that includes an HTTP trigger Azure Function instrumented using Open Telemetry.

When the URL for the function is requested, the function will make a call to `wwwexample.com` and then return either a 200 status, return the status passed in for `forceStatus` or throw an error if `throws` is true in the request params or POST body. Distributed Tracing (DT) data is extracted from the request headers and applied to the function span.

When debugging locally, the URL to invoke will be output to the terminal. For example: `[GET,POST] http://localhost:7071/api/HttpTriggerExample`. This can then be invoked via command-line, web browser, etc. Please keep in mind the VS Code Azure debugging tools don't always process things correctly. Sometimes query params don't get processed, for example.

Once you've created an Azure Function App in Azure and deployed, you can grab the Azure URL via the VS Code command pallet: `Command + Shift + P` -> `Azure Functions: Copy Function URL`. See the [Deployment](#deployment) section below.

## Telemetry

This example currently only exports trace data and does not have metric examples.

Traces are buffered using a `BatchSpanProcessor`. The batch span processor will export when either the timeout is hit, the buffer size limit is hit, or the function completes execution via a call to force flush.

A minimal amount of useful HTTP specific data has been captured for this example. You may wish to capture additional details and add according to the [HTTP Semantic Conventions](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/semantic_conventions/http.md).

## Required/Recommended Configuration

The following settings are recommended to be set in your Azure Function App configuration and local settings (`local.settings.json`) or ENV vars. The first time you debug an example, VS Code will automatically create `local.settings.json` for you. This file is not committed to source control as it may contain secrets.

* `API_KEY` [required]: Must be set to allow data to be sent to new relic. Requires a new relic ingest license key: https://one.newrelic.com/launcher/api-keys-ui.launcher.

## Optional Configuration

* `SERVICE_NAME`: Set to override the name of the service which will map to an entity in New Relic. The service name will default to the Azure Function App name from the `WEBSITE_SITE_NAME` ENV populated by Azure.

* `EXPORT_CONSOLE`: Set to any casing of 'true' or '1' to output Open Telemetry data to the console, via `ConsoleSpanExporter`, for viewing in Azure log streams.

* `EXPORT_URL`: Set to override the URL for OTLP data export.

* `EXPORT_BUFFER_TIMEOUT`: Set to override the default buffer timeout that triggers flushing data prior to function completing invocation. 5000ms is the default in this example.

* `EXPORT_BUFFER_SIZE`: Set to override the default buffer size that triggers flushing data prior to function completing invocation. 500 spans is the default in this example.

## Local Execution

You'll likely need to setup/point-to an Azure Storage account your first time debugging. Multiple files in `.vscode` have been committed to source control to aid in debugging the examples using VS Code, which includes the debug launch settings.

Use `npm run dev` to run locally, without a debugger, with multiple azure functions apps running at the same time. otherwise, they will conflict on port when launching.

## Deployment

### Create a new Azure Function App.

Creating a new Azure Function App can be done through the website or via the Azure extension. Below describes creating through the Azure extension in VS Code.

[`Resources` -> `Function App` -> right-click] OR [command pallet (`Command + Shift + P`)] -> `Azure Functions: Create Function App in Azure....`

Enter a name, a Node.js runtime and a region. This current version of the example has been tested with Node 16 in Azure and Node 18 locally.

### Add Application Settings

Add required application settings as well as any desired optional settings. These will become ENV variables available to the the function.

An easy way to get started is through the Azure plugin. Select the function app -> `Application Settings` -> right-click -> `Upload local settings` OR use the command pallet (`Command + Shift + P`) -> `Azure Functions: Upload Local Settings`... And then modify any different settings you need in Azure. Modification can also be done through the Azure extension.

## Deploy to the Azure Function App

Now in your primary application folder, deploy the application to the Azure Function App.

[right-click] OR [command pallet (`Command + Shift + P`)] -> `Azure Functions: Deploy to Function App...` and choose the Function App you created earlier.

## Get the URL

Once deployed, to execute from another function within Azure, you'll need to get the proper URL.

This can be done inside VS Code by leveraging the command pallet. `Command + Shift + P` -> `Azure Functions: Copy Function URL`. Choose the deployed Azure Function App and then the specific function. Once selected, the URL will be on your clip-board. Paste into the appropriate location for usage, such as the `EXTERNAL_URL` setting of the queue trigger example.

This URL will likely take the form `https://<azure function app>.azurewebsites.net/api/<functionname>?code=<some code>`.

You can also get the URL through the Azure UI. Navigate to your Azure Function App -> `Functions` (left nav) -> `HttpTriggerExample` -> `Get Function URL` (tool bar) -> copy the default function key.
