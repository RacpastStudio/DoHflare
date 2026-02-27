![DoHflare Banner](./assets/banner.jpg)
<p align="right">
  <a href="./README.md">简体中文</a> | <b>English</b>
</p>

# DoHflare

![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-F38020?logo=cloudflare&logoColor=white) ![License: AGPL v3](https://img.shields.io/badge/License-AGPL%20v3-blue.svg)

A DNS over HTTPS (DoH) proxy designed specifically for deployment on Cloudflare Workers. 

DoHflare proxies client DNS queries to upstream DoH resolvers while enforcing TTL constraints, preserving EDNS0 Client Subnet (ECS) data, and managing traffic via a dual-layer caching mechanism (Isolate Memory + Cloudflare Cache API).

## 📑 Table of Contents

- [📁 Project Structure](#-project-structure)
- [📦️ Deployment](#%EF%B8%8F-deployment)
  - [Method 1: Cloudflare Dashboard (Quick Deploy)](#method-1-cloudflare-dashboard-quick-deploy)
  - [Method 2: GitHub Integration (CI/CD)](#method-2-github-integration-cicd)
  - [Method 3: Wrangler CLI (Local Development)](#method-3-wrangler-cli-local-development)
  - [Method 4: Cloudflare Pages (Direct Upload)](#method-4-cloudflare-pages-direct-upload)
- [⚙️ Configuration](#%EF%B8%8F-configuration)
  - [Environment Variables Reference](#environment-variables-reference)
- [📖 Client Setup & Usage Examples](#-client-setup--usage-examples)
- [🚨 Error Codes & Status Mapping](#-error-codes--status-mapping)
- [⚖️ Platform Limits & Constraints](#%EF%B8%8F-platform-limits--constraints)
- [📜 License and Compliance](#-license-and-compliance)

---

## 📁 Project Structure

```text
.
├── assets/
│   └── banner.jpg         # Repository hero banner image
├── src/
│   └── index.js           # Main Worker script
├── .gitignore             # Git ignore rules
├── LICENSE                # GNU AGPLv3 License file
├── NOTICE                 # Additional terms and attribution requirements
├── package.json           # Project metadata and npm deployment scripts
├── README.md              # Project documentation (Chinese)
├── README.en.md           # Project documentation (English)
├── SECURITY.md            # Security policy and vulnerability reporting
└── wrangler.jsonc         # Cloudflare Workers configuration and env vars
```

## 📦️ Deployment

DoHflare can be deployed via multiple methods depending on your infrastructure preferences and CI/CD requirements.

*(Note: The Cloudflare Dashboard is updated frequently. The exact menu paths and button names described in the dashboard methods below are for reference and may vary slightly over time.)*

### Method 1: Cloudflare Dashboard (Quick Deploy)

Ideal for rapid deployment without a local development environment.

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/) and navigate to **Workers & Pages**.
2. Click **Create application**.
3. Under the "Ship something new" menu, select **Start with Hello World!**, assign a name for your Worker, and click **Deploy**.
4. Once the starter template is deployed, click **Edit code**, replace the default script with the raw contents of `src/index.js`, and click **Deploy**.
5. Navigate to the Worker's **Settings** -> **Variables and Secrets** to configure your environment variables (refer to the Configuration section below).
6. Navigate to **Settings** -> **Domains & Routes** and click **Add** to bind your proxy to a specific custom domain.

### Method 2: GitHub Integration (CI/CD)

Recommended for automated deployments and version control tracking.

1. **Fork** this repository to your personal GitHub account.
2. In the Cloudflare Dashboard, navigate to **Workers & Pages** -> **Create application**.
3. Select **Continue with GitHub** and authorize Cloudflare to access your forked repository.
4. Follow the prompts to configure the build settings. Ensure all necessary environment variables are defined in the Cloudflare Dashboard settings to override the default runtime fallbacks.

### Method 3: Wrangler CLI (Local Development)

The standard approach for developers preferring terminal operations and local configurations. This project requires the Cloudflare Workers CLI  (`wrangler`).

1. Authenticate your local environment with Cloudflare:
```bash
wrangler login
```
2. Deploy the Worker using the configuration defined in `wrangler.jsonc`:
```bash
wrangler deploy
```

### Method 4: Cloudflare Pages (Direct Upload)

If you prefer to bypass the Workers pipeline and use the Cloudflare Pages UI, you can deploy using Advanced Mode with a single script.

1. Create a new, empty directory on your local machine.
2. Copy `src/index.js` into this new directory and rename it strictly to `_worker.js`.
3. In the Cloudflare Dashboard, navigate to **Workers & Pages** -> **Create application**.
4. Click **Get started** next to "Looking to deploy Pages?" at the bottom of the page.
5. Under the **Drag and drop your files** section, click **Get started**.
6. On the "Deploy a site by uploading your project" screen, enter your desired project name in the text box and click **Create project**.
7. Under "Upload your project assets", choose to upload a **folder** and select the directory containing *only* your `_worker.js` file.
8. Once the upload finishes, click **Deploy site**.
9. After successful deployment, click **Continue to project** to enter the project dashboard.
10. Navigate to the **Custom domains** tab to bind your proxy to a specific custom domain and follow the provided DNS setup instructions.
11. **Configuration Warning**: The `wrangler.jsonc` file is completely ignored in this method. You must navigate to **Settings** -> **Variables and Secrets** to manually configure all necessary environment variables to override the hardcoded runtime defaults.

## ⚙️ Configuration

All runtime behaviors are controlled via environment variables defined in the `vars` block of `wrangler.jsonc` or via the Cloudflare Dashboard. The script includes hardcoded fallback defaults; you only need to assign a variable if you intend to override the default value.

### Environment Variables Reference

| Variable                        | Default Value                                                | Description                                                  |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `UPSTREAM_URLS`                 | `["https://dns.google/dns-query", "https://dns11.quad9.net/dns-query"]` | JSON array of upstream DoH resolvers.                        |
| `DOH_PATH`                      | `/dns-query`                                                 | The listening URI path for DoH requests.                     |
| `DOH_CONTENT_TYPE`              | `application/dns-message`                                    | Expected MIME type for DNS payloads.                         |
| `ROOT_CONTENT`                  | *(Generated HTML)*                                           | Custom HTML content to serve at the `/` path.                |
| `ROOT_CONTENT_TYPE`             | `text/html; charset=utf-8`                                   | MIME type for the root path response.                        |
| `ROOT_CACHE_TTL`                | `86400`                                                      | Time-to-live (in seconds) for the root landing page.         |
| `MAX_CACHEABLE_BYTES`           | `65536`                                                      | Maximum byte size for caching a DNS response.                |
| `MAX_POST_BODY_BYTES`           | `8192`                                                       | Maximum byte size allowed for incoming POST requests.        |
| `FETCH_TIMEOUT_MS`              | `2500`                                                       | Timeout limit (in milliseconds) for upstream fetch operations. |
| `MAX_RETRIES`                   | `2`                                                          | Maximum number of retry attempts for failed upstream requests. |
| `DEFAULT_POSITIVE_TTL`          | `60`                                                         | Default TTL (in seconds) applied to successful resolutions if none is provided. |
| `DEFAULT_NEGATIVE_TTL`          | `15`                                                         | Default TTL (in seconds) applied to NXDOMAIN or failed resolutions. |
| `FALLBACK_ECS_IP`               | `119.29.29.0`                                                | Default IP address injected for EDNS Client Subnet if the client IP is unresolvable. |
| `CF_CACHE_WRITE_THRESHOLD`      | `500`                                                        | Probability threshold (out of 10000) for writing responses to the Cloudflare global cache. |
| `GLOBAL_WRITE_COOLDOWN_MS`      | `300000`                                                     | Minimum interval (in milliseconds) required between global cache writes for the same key. |
| `GLOBAL_WRITE_PER_MINUTE_LIMIT` | `200`                                                        | Maximum number of global cache writes allowed per minute across the isolate. |
| `GLOBAL_CACHE_NAMESPACE`        | `https://dohflare.local/cache/`                              | The dummy URL namespace used for isolating objects in the Cloudflare Cache API. |
| `HOT_WINDOW_MS`                 | `60000`                                                      | Time window (in milliseconds) to track request frequency for L1 hot cache promotion. |
| `HOT_HIT_THRESHOLD`             | `20`                                                         | Minimum number of hits within the hot window to trigger an immediate global cache write. |
| `STALE_WINDOW_FACTOR`           | `0.5`                                                        | Multiplier applied to the TTL to determine the acceptable stale-while-revalidate window. |
| `EXTREME_STALE_FALLBACK_MS`     | `86400000`                                                   | Maximum time (in milliseconds) to serve stale cache data if all upstream requests fail. |
| `JITTER_PCT`                    | `10`                                                         | Percentage of deterministic jitter applied to TTLs to mitigate cache stampedes. |

## 📖 Client Setup & Usage Examples

Once deployed, your DoHflare proxy is fully compliant with modern Secure DNS standards. You can configure your browsers to route DNS traffic through your edge node or test it via CLI tools.

### 1. Browser Configuration

Modern web browsers natively support DNS over HTTPS. Here is how you can set up your private DoHflare instance:

* **Microsoft Edge**:
  1. Navigate to `edge://settings/privacy/security`.
  2. Scroll down to find and turn on **Use secure DNS**.
  3. Select **Choose a service provider**, and enter your Endpoint URL in the text box.
* **Google Chrome**:
  1. Navigate to `chrome://settings/security`.
  2. Turn on **Use secure DNS**.
  3. In the **Select DNS provider** dropdown, choose **Add custom DNS service provider**, and enter your Endpoint URL in the text box.

*(Note: The steps above are for reference only, as browser interfaces may change with updates. For other browsers like Firefox, Safari, or Brave, please search online for specific DoH configuration guides.)*

### 2. Using [natesales/q](https://github.com/natesales/q)

`q` is recommended for testing DoH endpoints. It handles DoH encapsulation natively and produces `dig`-like output.

```bash
# Test a standard A record query
q A cloudflare.com @https://<YOUR_WORKER_DOMAIN>/dns-query
```

## 🚨 Error Codes & Status Mapping

DoHflare relies on standard HTTP status codes combined with custom headers (`X-DOHFLARE-Code`) for precise debugging without breaking the DNS payload format.

| HTTP Status                  | Internal Code    | Description                                                  |
| ---------------------------- | ---------------- | ------------------------------------------------------------ |
| `200 OK`                     | N/A              | Successful resolution (Cache HIT or MISS).                   |
| `400 Bad Request`            | N/A              | Missing `dns` parameter, invalid Base64URL, or malformed DNS query. |
| `405 Method Not Allowed`     | N/A              | Request method is not `GET`, `POST`, or `OPTIONS`.           |
| `413 Payload Too Large`      | N/A              | POST body exceeds `MAX_POST_BODY_BYTES`.                     |
| `415 Unsupported Media Type` | N/A              | Missing or invalid `Content-Type` header in POST request.    |
| `502 Bad Gateway`            | `UPSTREAM_ERR`   | Upstream resolvers are unreachable or returned an invalid response. |
| `500 Internal Server Error`  | `INTERNAL_FATAL` | Unhandled exception during payload parsing or cache writing. |

**Internal Parsing Errors (Captured in logs):**

* `OOB_READ`: Attempted to read out-of-bounds memory in the ArrayBuffer.
* `MALFORMED_LOOP`: Detected an infinite pointer loop in the DNS name compression scheme.
* `PAYLOAD_TOO_LARGE`: Upstream response exceeded `MAX_CACHEABLE_BYTES`.

## ⚖️ Platform Limits & Constraints

DoHflare's performance is bound by the execution limits of the Cloudflare Workers platform. Please ensure your deployment tier aligns with your expected traffic volume.

| Resource                  | Workers Free   | Workers Paid      | Impact on DoHflare                                           |
| ------------------------- | -------------- | ----------------- | ------------------------------------------------------------ |
| **Requests per day**      | 100,000        | No limit          | Proxy will return HTTP 429 once limit is reached.            |
| **CPU Time per request**  | 10 ms          | 5 min             | DNS parsing is lightweight; 10ms is typically more than sufficient. |
| **Subrequests (fetch)**   | 50 per request | 10000 per request | DoHflare strictly manages upstream retries, staying well below limits. |
| **Cache API Object Size** | 512 MB         | 512 MB            | Safely capped by `MAX_CACHEABLE_BYTES` (default 64KB).       |

*For complete and up-to-date limits, refer to the [Cloudflare Workers Limits Documentation](https://developers.cloudflare.com/workers/platform/limits).*

## 📜 License and Compliance

This software is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**.

**ADDITIONAL NOTICE**: Pursuant to Section 7 of the GNU AGPLv3, additional terms apply to this work. Refer to the `NOTICE` file in the repository for specific attribution requirements and compliance policies.

---

*Copyright © 2026 Racpast. All rights reserved.*

