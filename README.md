# Infrastructure & API Sanity Check Workflow (n8n)

This repository contains a **generic n8n workflow** designed for DevOps teams to validate:

- On-prem IP/Port connectivity
- SaaS/API endpoint availability
- Proxy vs. non‑proxy routing
- Internal service-to-service connectivity inside Kubernetes
- Automated reporting via internal broadcast/notification service

It serves as a **portable “sanity‑checker”** that can be dropped into any n8n environment running inside Kubernetes.

---

## What This Workflow Does

### 1. **On‑Prem IP/Port Connectivity**
The workflow dynamically tests TCP connectivity across any number of internal endpoints.

## How it Works

- A Python Code node defines targets (name, IP, ports).
- Each port is tested using:  
  `nc -zvnw 3 <ip> <port>`

### Note for macOS Users
macOS uses a different version of `nc` that **does not support `-w`**.

Use this instead:

```bash
nc -zv -G 3 <ip> <port>
```

- Results are summarized into:
    - **OPEN**
    - **CLOSED**
    - **Unreachable / Timeout**

Useful for testing:
- Databases  
- LDAP  
- Internal GitLab  
- Proxies  
- Storage controllers  
- Any internal infra reachable from the pod

---

### 2. **SaaS/API Connectivity Tests**
This section tests external services like:
- PagerDuty
- Corporate portals
- External APIs
- Any service where authentication or proxy routing matters

Each API test:
- Has its own URL, headers, method, and body
- Supports proxy or no‑proxy rules
- Returns full JSON, headers, or status code

The workflow automatically evaluates success by checking:
- HTTP error codes  
- Known error fields  
- API-specific "success": false  
- Empty responses  

---

### 3. **Merging the Results**
Both sections are merged into one payload:
- `port_results` → internal connectivity
- `api_responses` → external/API connectivity  

This gives you a **complete health snapshot** of your environment.

---

### 4. **Email / Broadcast Reporting**
The workflow sends a JSON payload to any broadcast or notification microservice, for example:

```
http://<broadcast-service>/api/v1/Email/Send
```

Payload includes:
- All API test results
- All connectivity results

This can then be routed to:
- Email  
- Teams / Slack  
- PagerDuty  
- Logs  

---

## Architecture Overview

```
n8n Pod
│
├── SaaS/API Connectivity Tests
│
├── On‑Prem Connectivity (nc)
│
├── Summary (Python)
│
└── Broadcast → Internal Email/Notification MS
```

Everything runs **inside the cluster**, giving real service‑level observability.

---

## Deployment Notes

This workflow is designed to run **inside Kubernetes**.

Requirements:
- n8n running in cluster  
- netcat (nc) inside container  
- Correct `NO_PROXY`:  
  ```
  localhost,127.0.0.1,.svc,.svc.cluster.local
  ```
- Proxy for external traffic  
- Internal DNS working  

---

## Customization

### On‑Prem Targets
```python
targets = [
    {"name": "Internal Batman", "ip": "127.0.0.1", "ports": [389, 636]},
    {"name": "Internal GitLab", "ip": "127.0.0.2", "ports": [443]},
]
```

### SaaS/API Targets
```python
{
  "name": "PagerDuty - OAuth Test",
  "method": "GET",
  "url": "https://api.pagerduty.com/...",
  "proxy": "http://127.0.0.1:8080"
}
```

---

## Use Cases

Perfect for:
- Platform teams  
- DevOps  
- SRE  
- Infra automation  
- Kubernetes operations  

Use it to:
- Validate connectivity before deployments  
- Detect DNS/proxy issues  
- Confirm login endpoints are live  
- Slack/email daily health reports  

---

## Automation

The workflow can run via Cron in n8n:
- Hourly  
- Daily  

---

## Repo Structure

```
README.md         # Documentation  
healthcheck.json     # n8n health check export  
```

## License

This project is licensed under the **MIT License** — free for personal and commercial use.

---

## Author
**Jawwad Ahmed Abbasi**  
Senior Software Developer  
[GitHub](https://github.com/jawwadabbasi) | [YouTube](https://www.youtube.com/@jawwad_abbasi)