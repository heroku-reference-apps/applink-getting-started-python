# Heroku AppLink Python App Template

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://www.heroku.com/deploy?template=https://github.com/heroku-reference-apps/applink-getting-started-python)

The Heroku AppLink Python app template is a [FastAPI](https://fastapi.tiangolo.com/) web application that demonstrates how to build APIs for Salesforce integration using Heroku AppLink. This template includes authentication, authorization, and API specifications for seamless integration with Salesforce and Data Cloud.

## Table of Contents

- [Quick Start](#quick-start)
- [Local Development](#local-development)
- [Testing with invoke.py](#testing-with-invokepy)
- [Running Automated Tests](#running-automated-tests)
- [Manual Heroku Deployment](#manual-heroku-deployment)
- [Heroku AppLink Setup](#heroku-applink-setup)
- [Additional Resources](#additional-resources)

## Quick Start

### Prerequisites

- Python 3.8+
- `pip` for package management
- Git
- Heroku CLI (for deployment)
- Salesforce org (for AppLink integration)

### Deploy to Heroku (One-Click)

Click the Deploy button above to deploy this app directly to Heroku with the AppLink add-on pre-configured.

## Local Development

### 1. Clone and Install

```bash
git clone https://github.com/heroku-reference-apps/applink-getting-started-python.git
cd applink-getting-started-python
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. Start the Development Server

```bash
uvicorn app.main:app --reload
```

Your app will be available at `http://localhost:8000`. When you visit this URL, you will see a welcome message.

#### A Note on Local Errors
If you try to access an endpoint under the `/api` prefix directly in your browser (e.g., `http://localhost:8000/api/accounts/`), you will see a `ValueError: x-client-context not set` error. This is expected behavior. The AppLink middleware is protecting these endpoints, and they require Salesforce context headers to be present. The `invoke.py` script is designed to provide these headers for local testing.

### 3. API Endpoints

- **GET /** - A public welcome page.
- **GET /health** - A public health check endpoint.
- **POST /handleDataCloudDataChangeEvent/** - A public webhook for Data Cloud events.
- **GET /docs** - Interactive Swagger UI for API documentation.
- **GET /api/accounts/** - (Protected) Retrieve Salesforce accounts from the invoking org.
- **POST /api/unitofwork/** - (Protected) Create a unit of work for Salesforce.

### 4. View API Documentation

Visit `http://localhost:8000/docs` to explore the interactive API documentation powered by Swagger UI.

## Testing with invoke.py

The `bin/invoke.py` script allows you to test your locally running, protected API endpoints with the proper Salesforce client context headers.

### Usage

```bash
./bin/invoke.py ORG_DOMAIN ACCESS_TOKEN ORG_ID USER_ID [METHOD] [API_PATH] [--data DATA]
```

### Examples

```bash
# Test the accounts endpoint (note the /api/ prefix)
./bin/invoke.py mycompany.my.salesforce.com TOKEN_123 00D123456789ABC 005123456789ABC GET /api/accounts/

# Test with POST data
./bin/invoke.py mycompany.my.salesforce.com TOKEN_123 00D123456789ABC 005123456789ABC POST /api/unitofwork/ --data '{"data":{"accountName":"Test Account", "lastName":"Test", "subject":"Test Case"}}'
```

### Getting Salesforce Credentials

To get the required Salesforce credentials for testing:

1.  **Access Token**: Use Salesforce CLI to generate a session token (`sf org display --target-org <alias> --json | jq .result.accessToken -r`).
2.  **Org ID**: Found in Setup → Company Information or by running `sf org a display --target-org <alias> --json | jq .result.id -r`.
3.  **User ID**: Found in your user profile or Setup → Users or by running `sf org display --target-org <alias> --json | jq .result.userId -r`.

## Running Automated Tests

This project uses `pytest` for unit testing. To run the tests:

```bash
pytest
```

## Manual Heroku Deployment

### 1. Prerequisites

- [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) installed
- Git repository initialized
- Heroku account with billing enabled (for add-ons)

### 2. Create Heroku App

```bash
# Let Heroku generate a name for your app
heroku create
```
Heroku will respond with the unique name for your new application (e.g., `salty-sierra-55555`). **Take note of this app name, as you will need it in later steps.**

### 3. Add Required Buildpacks

The app requires two buildpacks in the correct order:

```bash
# Add the AppLink Service Mesh buildpack first
heroku buildpacks:add heroku/heroku-applink-service-mesh

# Add the Python buildpack second
heroku buildpacks:add heroku/python
```

### 4. Provision the AppLink Add-on

```bash
# Provision the Heroku AppLink add-on
heroku addons:create heroku-applink --app YOUR_APP_NAME
```
Heroku will provision the add-on and give it a unique name (e.g., `applink-slippery-54321`). **You will need this name in the setup steps below.** If you forget it, you can find it by running `heroku addons`.

### 5. Deploy the Application

```bash
# Deploy to Heroku
git push heroku main

# Check deployment status
heroku ps:scale web=1
heroku open
```

### 6. Verify Deployment

```bash
# Check app logs
heroku logs --tail
```

## Heroku AppLink Setup

In the following steps, you will need the **Heroku app name** (from step 2) and the **AppLink add-on name** (from step 4).

### 1. Install AppLink CLI Plugin

```bash
# Install the AppLink CLI plugin
heroku plugins:install @heroku-cli/plugin-applink
```

### 2. Connect to Salesforce Org

This command securely connects your app to Salesforce. The `CONNECTION_NAME` is a friendly name **that you define yourself** to identify this link. You will reuse this name in the `publish` step.

```bash
# For a production or developer org. Replace CONNECTION_NAME with a name of your choice (e.g., "production_org").
heroku salesforce:connect CONNECTION_NAME --addon YOUR_ADDON_NAME -a YOUR_APP_NAME

# For a sandbox org
heroku salesforce:connect CONNECTION_NAME --addon YOUR_ADDON_NAME -a YOUR_APP_NAME --login-url https://test.salesforce.com
```
This will open a browser for you to log in to Salesforce and approve the connection.

### 3. Authorize a User

```bash
# Authorize a Salesforce user for API access
heroku salesforce:authorizations:add auth-user --addon YOUR_ADDON_NAME -a YOUR_APP_NAME
```

### 4. Publish Your App

This final step publishes your API to Salesforce and creates the necessary components.

```bash
# Publish the app to Salesforce as an External Service
heroku salesforce:publish api-spec.yaml \
  --client-name MyAPI \
  --connection-name CONNECTION_NAME \
  --authorization-external-client-app-name MyAppLinkExternalClientApp \
  --authorization-permission-set-name MyAppLinkPermSet \
  --addon YOUR_ADDON_NAME
```

**Parameter Breakdown:**
- `--client-name`: The name for the External Service and its generated Apex classes (e.g., `MyAPI`).
- `--connection-name`: The friendly name you gave the AppLink connection in the previous step (e.g., `production_org`).
- `--authorization-external-client-app-name`: The name for the **new** External Client App that Heroku will create. **Note:** This value must match the `externalClientApp` value defined in `api-spec.yaml`.
- `--authorization-permission-set-name`: The name for the **new** Permission Set that Heroku will create. **Note:** This value must match the `permissionSet` value defined in `api-spec.yaml`.
- `--addon`: The unique name of your AppLink add-on (e.g., `applink-slippery-54321`).

### 5. Required Salesforce Permissions

Users need the "Manage Heroku AppLink" permission in Salesforce:
1.  Go to Setup → Permission Sets
2.  Create a new permission set or edit an existing one
3.  Add "Manage Heroku AppLink" system permission
4.  Assign the permission set to users

## Additional Resources

### Documentation

- [Getting Started with Heroku AppLink](https://devcenter.heroku.com/articles/getting-started-heroku-applink)
- [Heroku AppLink CLI Plugin](https://devcenter.heroku.com/articles/heroku-applink-cli)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---

**Note**: This template is designed for educational purposes and as a starting point for building Salesforce-integrated applications. For production use, ensure proper error handling, security measures, and testing practices are implemented.
