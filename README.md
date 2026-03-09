# Azure SWA Starter

**Production-ready Azure Static Web Apps template with security, auth, and CI/CD pre-configured.**

> Deploy a secure static site to Azure in under 10 minutes. No build tools needed — just HTML, CSS, and JS with battle-tested defaults.

## What's Included

```
azure-swa-starter/
├── .github/workflows/
│   └── azure-swa.yml              # CI/CD: deploy on push, PR previews
├── site/
│   ├── index.html                  # Landing page with dark/light mode
│   ├── login.html                  # Auth page (Microsoft/GitHub/Google SSO)
│   ├── staticwebapp.config.json    # Route rules, security headers, auth
│   └── assets/                     # Static assets (images, fonts, etc.)
├── api/                            # Azure Functions (optional backend)
├── docs/
│   └── setup.md                    # Detailed Azure setup walkthrough
└── README.md
```

## Quick Start

### 1. Use this template

```bash
# Clone or use GitHub's "Use this template" button
git clone https://github.com/bonevisionlabs/azure-swa-starter.git my-site
cd my-site
```

### 2. Create an Azure Static Web App

In the [Azure Portal](https://portal.azure.com):

1. **Create a resource** > **Static Web App**
2. Choose your subscription and resource group
3. Name your app (e.g., `my-site-prod`)
4. Plan: **Free** (or Standard for custom auth)
5. Source: **GitHub** > select your repo
6. Build preset: **Custom**
   - App location: `site`
   - API location: `api`
   - Output location: (leave empty)
7. **Create**

Azure automatically adds the deployment secret to your repo.

### 3. Rename the secret (if needed)

The workflow expects `AZURE_STATIC_WEB_APPS_API_TOKEN`. If Azure created a differently-named secret, either:
- Rename the secret in your repo settings, or
- Update the secret name in `.github/workflows/azure-swa.yml`

### 4. Push and deploy

```bash
git push origin main
```

The GitHub Action triggers automatically and deploys your site.

## Security Features

### Headers (pre-configured in `staticwebapp.config.json`)

| Header | Value | Purpose |
|--------|-------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Force HTTPS |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `SAMEORIGIN` | Block clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referrer data |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Deny device APIs |
| `Content-Security-Policy` | Script/style/font allowlist | Prevent XSS |

### File Blocking

Sensitive file types return 404 automatically:
- `*.py` — Python scripts
- `*.env` — Environment files
- `*.pem` — Certificates/keys

Add more patterns in `staticwebapp.config.json` → `routes`.

### Authentication

Azure SWA provides built-in SSO with zero configuration:

| Provider | Auth URL | Setup |
|----------|----------|-------|
| Microsoft (Entra ID) | `/.auth/login/aad` | Works out of the box |
| GitHub | `/.auth/login/github` | Works out of the box |
| Google | `/.auth/login/google` | Requires [custom auth config](https://learn.microsoft.com/en-us/azure/static-web-apps/authentication-custom) |

Role-based access control:
```json
{
  "route": "/app",
  "allowedRoles": ["approved"]
}
```

Assign roles via the Azure Portal or [invitation system](https://learn.microsoft.com/en-us/azure/static-web-apps/authentication-authorization#role-management).

## Route Rules

| Route | Behavior |
|-------|----------|
| `/login` | Rewrites to `login.html` (clean URL) |
| `/app` | Protected — requires `approved` role |
| `/api/*` | Passes to Azure Functions backend |
| `*.py`, `*.env`, `*.pem` | Returns 404 |
| 401 (unauthorized) | Redirects to `/login` |
| 403 (forbidden) | Redirects to `/login?error=forbidden` |
| 404 (not found) | Falls back to `index.html` (SPA support) |

## CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/azure-swa.yml`) provides:

- **Deploy on push** to `main` (only when `site/`, `api/`, or workflow files change)
- **PR preview environments** (deployed on PR open, cleaned up on PR close)
- **Path-filtered triggers** so unrelated commits don't trigger deploys
- **No build step** by default (`skip_app_build: true`) — set to `false` if using a framework

## Customization

### Replace the branding

1. Edit `site/index.html` — update app name, hero text, feature cards
2. Edit `site/login.html` — update app name, badge text
3. Replace `YourApp` with your brand in both nav bars
4. Update the theme colors in the CSS custom properties (`:root` and `[data-theme="light"]`)

### Add a build step

For React, Vue, Svelte, etc., update the workflow:

```yaml
# In .github/workflows/azure-swa.yml
skip_app_build: false
output_location: "dist"    # or "build", depending on your framework
```

### Add API functions

Create Azure Functions in the `api/` directory:

```
api/
├── host.json
├── package.json
└── submit-form/
    ├── function.json
    └── index.js
```

They're automatically deployed and accessible at `/api/submit-form`.

### Custom domain

1. Azure Portal > Your SWA > **Custom domains**
2. Add your domain
3. Azure provisions a free SSL certificate automatically

## Requirements

- GitHub account
- Azure account ([free tier](https://azure.microsoft.com/free/) works)
- No local tooling required — deploy directly from GitHub

## License

MIT License. See [LICENSE](LICENSE).

## Origin

Extracted from a production deployment pattern that powers a medical imaging research platform. De-identified and generalized for any static site that needs secure hosting with authentication.
