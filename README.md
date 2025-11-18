# Frontend App CI/CD Example

build example

## CI/CD Features

This project includes a GitHub Actions workflow (`.github/workflows/build.yml`) to build the project. This would act as a Continuous Integration (CI) check to verify the latest code changes (git commit) have not broken the code base.

Make a small code change, `git commit`, `git push` and then observe the GitHub Action running in the GitHub website (within the Actions tab of the code repo).

> There may be other branches in this repo demonstrating additional CI/CD features.

### Configuring CI/CD

Single Page Apps (SPAs) must bake env vars in at build time. This means they must be built to target a specific environment and must be rebuilt for different environments. The only env var in this base project is `VITE_API_BASE_URL` which must be set to point at the backend products web service. For the CI/CD build, this env var is currently set in the workflow file.

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
