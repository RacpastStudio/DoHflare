# Security Policy

## Supported Versions

DoHflare is designed as a continuously deployed Serverless Edge application. Security updates and patches are exclusively provided for the latest commit on the main branch.

| Version | Supported          | Description                                      |
| ------- | ------------------ | ------------------------------------------------ |
| Latest  | :white_check_mark: | The current `main` branch deployed to the edge.  |
| Older   | :x:                | Historical commits and branches are unsupported. |

## Reporting a Vulnerability

We take the security of DoHflare and the privacy of its DNS routing mechanisms very seriously. 

**DO NOT report security vulnerabilities through public GitHub issues, discussions, or pull requests.** Public disclosure before a patch is deployed puts the infrastructure and its users at risk.

If you believe you have discovered a vulnerability (e.g., cache poisoning vectors, memory out-of-bounds reads during binary parsing, or EDNS0 client IP leakage), please report it directly via email to:

📧 **racpast@gmail.com**

### What to Include in Your Report
To facilitate a rapid investigation, please ensure your report contains the following:
* **Description:** A detailed summary of the vulnerability and its potential impact.
* **Steps to Reproduce:** Step-by-step instructions. A malicious `.bin` query payload or a specific `curl` command is highly appreciated.
* **Suggested Mitigation:** Any insights you have on how to patch the issue within the Cloudflare Workers environment.

### Response Service Level Agreement (SLA)
* **Acknowledgment:** We will acknowledge receipt of your vulnerability report within **48 hours**.
* **Triage & Assessment:** We will verify the vulnerability and provide an initial assessment within **5 business days**.
* **Resolution:** We strive to release a patch or apply edge mitigations within **15 to 30 days**, depending on the complexity of the issue.

## Out of Scope

The following vectors are explicitly out of scope and will not be accepted as valid security vulnerabilities:
* **Volumetric DDoS Attacks:** Layer 3/4 or Layer 7 volumetric attacks are mitigated by the underlying Cloudflare infrastructure, not the DoHflare application logic.
* **Theoretical Vulnerabilities:** Claims without a demonstrable Proof of Concept (PoC).
* **Upstream Resolver Issues:** Vulnerabilities belonging to the upstream DNS providers (e.g., Google DNS, Quad9) rather than the proxy itself.
* **Configuration Errors:** Misconfigurations of Cloudflare Dashboard settings (e.g., insecure Environment Variables or missing Custom Domain TLS policies) by the end-user.

## Safe Harbor

We support safe harbor for security researchers who:
* Make a good faith effort to avoid privacy violations, destruction of data, and interruption or degradation of our services.
* Comply with this policy and provide us a reasonable amount of time to resolve the issue before disclosing it publicly.

We will not recommend or pursue legal action related to your research if you adhere to these guidelines.

