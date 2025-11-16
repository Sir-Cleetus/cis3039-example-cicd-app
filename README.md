# Frontend App CI/CD Example

## CI/CD Features

This project includes a GitHub Actions workflow (`.github/workflows/build.yml`) to build the project. This would act as a Continuous Integration (CI) check to verify the latest code changes (git commit) have not broken the code base.

Make a small code change, `git commit`, `git push` and then observe the GitHub Action running in the GitHub website (within the Actions tab of the code repo).

> There may be other branches in this repo demonstrating additional CI/CD features.

### Configuring CI/CD

Single Page Apps (SPAs) must bake env vars in at build time. This means they must be built to target a specific environment and must be rebuilt for different environments. The only env var in this base project is `VITE_API_BASE_URL` which must be set to point at the backend products web service. See the **Azure Setup** section below on configuring automated deployments to Azure.

## Local Setup

### Install dependencies

```bash
npm install
```

### Configure local environment variables

```bash
cp .env.example .env.local
```

Update/add values within `.env.local`

> Note: Never commit your `.env.local` to source control. The example is safe to share.

### Compile and Hot-Reload for Development

```bash
npm run dev
```

### Type-Check, Compile and Minify for Production

```bash
npm run build
```

## Azure Setup

### Publish the app to an Azure Storage Account (static website)

1. Create a Storage Account suited for static sites:

```bash
az storage account create \
  --name <productappstoragename> \
  --resource-group <your-resource-group> \
  --location <permitted-location> \
  --sku Standard_LRS \
  --kind StorageV2
```

2. Enable static website and set the index page:

```bash
az storage blob service-properties update \
  --account-name <productappstoragename> \
  --static-website \
  --index-document index.html \
  --404-document index.html
```

3. Build the SPA and upload the contents of dist to the `$web` container:

```bash
# build
npm run build

# publish
az storage blob upload-batch \
  --account-name <productappstoragename> \
  --source ./dist \
  --destination '$web' \
  --overwrite

# get URL
az storage account show \
  --name <productappstoragename> \
  --resource-group <your-resource-group> \
  --query "primaryEndpoints.web" \
  --output tsv
```

4. Add your storage site to CORS on the Function App (so the deployed web app can access the web service):

```bash
az functionapp cors add \
  --name <your-backend-products-func> \
  --resource-group <your-resource-group> \
  --allowed-origins https://<app-store-endpoint>
```

### Configure Automated Deployment

1. Get the Connection String for your Storage Account:

   ```bash
   az storage account show-connection-string \
   --name <productappstoragename> \
   --resource-group <your-resource-group> \
   --query connectionString \
   --output tsv
   ```

2. Create a GitHub repo secret with the Connection String:

   From the GitHub repo website: `Settings → Secrets and Variables → Actions → New Repository Secret`

   Name the secret `AZURE_STORAGE_CONNECTION_STRING` and copy in the contents of the publishing profile from the previous step.

3. Update the workflow with the base URL for your deployed Function App (backend service, e.g. a deployed version of `cis3039-example-cicd-svc`):

   Modify `.github/workflows/build-and-deploy.yml` with the base url for your test environment's backend service.

   Merge the changes into the main branch to become permenent.

4. Make a small code change (e.g. some wording on a page), commit it and you should see the GitHub Action build and deploy to your test environment.

## How the project was made

This section explains how you could create a new project in the same style as this one.

Initialise the project files:

```bash
npm create vue@3 . -- \
  --force \
  --bare \
  --typescript \
  --router \
  --prettier
```

Restore any delete devcontainer configuration:

```bash
git restore .devcontainer/devcontainer.json
```

Add extensions to devcontainer:

- `"esbenp.prettier-vscode"`,
- `"Vue.volar"`

Update preferences in `.prettierrc.json`:

- `"semi": true`
- `"singleQuote": true`
- `"printWidth": 80`
