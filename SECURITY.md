# Security Policy

The SkyWarden team takes security, model integrity, and safe deployment seriously. This policy outlines how vulnerabilities and security concerns should be reported and handled.

---

## 1. Supported Versions

Security updates and patches are actively applied to the following release lines:

| Version | Supported |
|---|---|
| `0.2.x` | :white_check_mark: Yes |
| `0.1.x` | :x: No (Please upgrade to latest) |
| `< 0.1.0` | :x: No |

---

## 2. Reporting a Vulnerability

If you discover a security vulnerability or exploit in SkyWarden, **please do not open a public issue.** Publicly disclosing vulnerabilities puts deployments at risk before a remediation is available.

Instead, please report vulnerabilities by contacting the maintainers directly:
- **Email:** `alok.20233040@mnnit.ac.in` or via GitHub Private Vulnerability Reporting on `ashishsps20/SkyWarden`.

### What to Include in Your Report
To help us triage and investigate efficiently, please include:
- A clear description of the vulnerability and its potential impact.
- Step-by-step instructions or minimal proof-of-concept (PoC) code to reproduce the issue.
- Details of your runtime environment (OS, Python version, PyTorch version, dependencies).
- Any proposed mitigations or workarounds, if known.

---

## 3. Vulnerability Response Timeline

- **Initial Response:** Within 48 hours of receiving the report.
- **Triage & Status Update:** Within 5 business days detailing whether the report is confirmed and outlining next steps.
- **Remediation Release:** A patched release or mitigating advisory will be published as soon as verification is complete.

---

## 4. Machine Learning & Application Security Best Practices

When deploying SkyWarden into production or network-accessible environments:
1. **Model Weights Integrity:** Only load PyTorch checkpoints (`.pt`, `.pth`, `.bin`) from trusted origins. Loading untrusted pickle-based PyTorch weights can lead to arbitrary code execution.
2. **Input Sanitization:** Validate image payloads (MIME types, dimension bounds, and maximum file size) before passing them to OpenCV or PIL decoding pipelines to guard against decompression bombs.
3. **Environment Isolation:** Run the application within unprivileged containers (non-root user) and avoid exposing Streamlit debug ports directly to the public internet without an authenticating reverse proxy (e.g., NGINX, Cloudflare).
