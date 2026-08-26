# sample-azure-function

# Azure HTTP Function — Sample Code, Folder Structure, Deploy & Test

Python v2 programming model, HTTP-triggered Azure Function.

## Folder Structure

```
my-http-function/
├── .funcignore
├── .gitignore
├── host.json
├── local.settings.json
├── requirements.txt
└── function_app.py
```

## `function_app.py`

```python
import azure.functions as func
import logging
import json

app = func.FunctionApp(http_auth_level=func.AuthLevel.ANONYMOUS)

@app.route(route="hello")
def hello(req: func.HttpRequest) -> func.HttpResponse:
    logging.info("HTTP trigger function processed a request.")

    name = req.params.get("name")
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            req_body = None
        if req_body:
            name = req_body.get("name")

    if name:
        return func.HttpResponse(
            json.dumps({"message": f"Hello, {name}!"}),
            mimetype="application/json",
            status_code=200
        )
    else:
        return func.HttpResponse(
            json.dumps({"error": "Pass a name in the query string or request body"}),
            mimetype="application/json",
            status_code=400
        )
```

## `host.json`

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "excludedTypes": "Request"
      }
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  }
}
```

## `local.settings.json`

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "python"
  }
}
```

## `requirements.txt`

```
azure-functions
```

## `.funcignore`

```
.venv/
local.settings.json
test/
.vscode/
__pycache__/
.git*
```

---

## Local Test (before deploying)

```bash
# create venv and install deps
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# run locally (requires Azure Functions Core Tools)
func start
```

Test it:

```bash
curl "http://localhost:7071/api/hello?name=Ram"

# or POST
curl -X POST http://localhost:7071/api/hello \
  -H "Content-Type: application/json" \
  -d '{"name": "Ram"}'
```

## Deploy to Azure (CLI)

```bash
# variables
RESOURCE_GROUP="rg-func-demo"
LOCATION="eastus"
STORAGE_ACCOUNT="stfuncdemo$RANDOM"
FUNCTION_APP="func-http-demo-$RANDOM"

# resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# storage account (required by Functions)
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS

# function app (Linux, Consumption plan, Python 3.11)
az functionapp create \
  --resource-group $RESOURCE_GROUP \
  --consumption-plan-location $LOCATION \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --os-type Linux \
  --name $FUNCTION_APP \
  --storage-account $STORAGE_ACCOUNT

# deploy code
func azure functionapp publish $FUNCTION_APP
```

## Test in Azure

```bash
Content-Type: application/json
{"name": "Ram"}
```

```bash
curl "https://$FUNCTION_APP.azurewebsites.net/api/hello?name=Ram"
```

---

**Notes:**
- Auth level is set to `ANONYMOUS` for easy testing — switch to `FUNCTION` for production and pass the key: `?code=<function_key>`.
- Requires Azure Functions Core Tools and Azure CLI logged in (`az login`).
