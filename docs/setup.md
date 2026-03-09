# Detailed Azure Setup Guide

Step-by-step guide for deploying your site with Azure Static Web Apps.

## Prerequisites

- A [GitHub](https://github.com) account
- An [Azure](https://azure.microsoft.com/free/) account (free tier works)
- This repo cloned or forked to your GitHub account

## Step 1: Create the Azure Resource

1. Go to [portal.azure.com](https://portal.azure.com)
2. Click **Create a resource**
3. Search for **Static Web App** and select it
4. Click **Create**

### Configuration

| Field | Value |
|-------|-------|
| Subscription | Your Azure subscription |
| Resource Group | Create new or use existing |
| Name | e.g., `my-app-prod` |
| Plan type | **Free** (Standard for custom auth providers) |
| Region | Choose closest to your users |
| Source | **GitHub** |
| Organization | Your GitHub org/account |
| Repository | Your forked/cloned repo |
| Branch | `main` |
| Build Preset | **Custom** |
| App location | `site` |
| API location | `api` |
| Output location | (leave empty) |

5. Click **Review + Create** > **Create**

Azure will:
- Add a deployment secret (`AZURE_STATIC_WEB_APPS_API_TOKEN`) to your repo
- May also create a workflow file — you can delete it since this template already has one

## Step 2: Verify the Secret Name

1. Go to your GitHub repo > **Settings** > **Secrets and variables** > **Actions**
2. Confirm a secret exists. If it's named something like `AZURE_STATIC_WEB_APPS_API_TOKEN_GRAY_FLOWER_0C1374110`, either:
   - **Rename it** to `AZURE_STATIC_WEB_APPS_API_TOKEN`, or
   - **Update** `.github/workflows/azure-swa.yml` to use the Azure-generated name

## Step 3: Push and Deploy

```bash
git push origin main
```

Go to **Actions** tab in your repo to watch the deployment. First deploy takes 1-2 minutes.

## Step 4: Verify Security Headers

After deployment, check your headers:

```bash
curl -I https://your-app.azurestaticapps.net
```

You should see:
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Security-Policy: default-src 'self'; ...
```

## Step 5: Configure Authentication (Optional)

### Built-in Providers (Free Plan)

Microsoft (Entra ID) and GitHub work out of the box. No configuration needed.

### Google Auth (Standard Plan)

Requires custom auth configuration:

1. Create a Google OAuth app at [console.cloud.google.com](https://console.cloud.google.com)
2. In Azure Portal > Your SWA > **Authentication**
3. Add Google as a custom provider with your client ID and secret

See: [Azure SWA Custom Authentication](https://learn.microsoft.com/en-us/azure/static-web-apps/authentication-custom)

### Role Assignment

1. Azure Portal > Your SWA > **Role management**
2. Click **Invite**
3. Enter the user's email and assign the `approved` role
4. They'll receive an invitation link

Alternatively, use the [invitation API](https://learn.microsoft.com/en-us/azure/static-web-apps/authentication-authorization#role-management) for programmatic role assignment.

## Step 6: Custom Domain (Optional)

1. Azure Portal > Your SWA > **Custom domains**
2. Click **Add**
3. Enter your domain (e.g., `app.example.com`)
4. Add the CNAME or TXT record to your DNS
5. Azure provisions a free SSL certificate automatically (takes ~15 minutes)

## Step 7: Add API Functions (Optional)

Create serverless API endpoints with Azure Functions:

```bash
mkdir -p api/hello
```

`api/host.json`:
```json
{
  "version": "2.0"
}
```

`api/hello/function.json`:
```json
{
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

`api/hello/index.js`:
```javascript
module.exports = async function (context, req) {
  context.res = {
    body: { message: "Hello from Azure Functions!" }
  };
};
```

Access at: `https://your-app.azurestaticapps.net/api/hello`

## Troubleshooting

### Deployment fails with "token not found"

- Verify the secret name matches between the workflow file and GitHub Secrets
- Re-generate the token: Azure Portal > Your SWA > **Manage deployment token**

### Auth redirects loop

- Check that `staticwebapp.config.json` is in the `site/` directory (the `app_location`)
- Verify the `post_login_redirect_uri` in your auth links matches an actual route

### 404 on page refresh (SPA)

The template includes a SPA fallback rule:
```json
{
  "responseOverrides": {
    "404": {
      "rewrite": "/index.html",
      "statusCode": 404
    }
  }
}
```

If you're using a client-side router, change `statusCode` to `200`.

### CSP blocking resources

Update the `Content-Security-Policy` in `staticwebapp.config.json` to allow your CDN or API domains:

```json
"Content-Security-Policy": "default-src 'self'; connect-src 'self' https://your-api.com; ..."
```
