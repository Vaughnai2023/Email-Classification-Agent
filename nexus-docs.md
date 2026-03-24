# Nexus Integrations — Product Documentation

*Last updated: March 2026*

---

## 1. Product Overview

Nexus Integrations is an integration platform built for mid-sized businesses that need to connect their core systems — CRMs, email platforms, billing tools, and internal APIs — without hiring a dedicated integration engineering team.

Nexus sits between your existing tools and keeps data flowing reliably. Rather than building point-to-point connections, you configure integrations through our visual dashboard and let the sync engine handle the rest.

### Key Features

- **API Connector Builder** — Pre-built connectors for 40+ popular platforms, plus a visual builder for custom REST and SOAP APIs (Enterprise plan only).
- **Data Sync Engine** — Bi-directional sync with configurable field mapping, deduplication, and conflict resolution. Supports batch and real-time modes.
- **Workflow Automation** — Trigger-based workflows that execute when data changes, new records are created, or on a schedule. Supports conditional logic, branching, and multi-step chains.
- **Monitoring Dashboard** — Real-time visibility into sync status, error rates, API call usage, and data throughput. Alerting via email and Slack.
- **Error Handling & Retry Logic** — Automatic retries with exponential backoff for transient failures. Dead letter queue for persistent errors. Configurable retry policies per integration.

### Supported Data Formats

- JSON (default for all REST-based integrations)
- XML (SOAP APIs and legacy systems)
- CSV (bulk imports/exports, scheduled file transfers)
- EDI (X12 and EDIFACT for supply chain and B2B integrations)

---

## 2. Pricing Plans

### Starter — $49/month

Best for small teams getting started with integration.

| Feature | Detail |
|---|---|
| Integrations | Up to 5 active integrations |
| API calls | 10,000 per month |
| Support | Email support (Mon-Fri 9am-6pm EST) |
| Response time | Within 24 hours |
| Monitoring | Basic monitoring dashboard |
| Community forum | Yes |
| Sync interval | Minimum 5 minutes |
| Audit logs | No |
| SSO/SAML | No |
| Custom connectors | No |
| Dedicated account manager | No |
| IP whitelisting | No |
| Webhook support | No |
| SLA guarantee | No |

### Professional — $149/month

For growing teams that need more capacity and faster support.

| Feature | Detail |
|---|---|
| Integrations | Up to 25 active integrations |
| API calls | 100,000 per month |
| Support | Priority email + live chat (Mon-Fri 9am-6pm EST) |
| Response time | Within 4 hours |
| Monitoring | Advanced monitoring with custom alerts |
| Community forum | Yes |
| Sync interval | Minimum 1 minute |
| Audit logs | Yes |
| Custom field mapping | Yes |
| Webhook support | Yes |
| IP whitelisting | Yes |
| SSO/SAML | No |
| Custom connectors | No |
| Dedicated account manager | No |
| SLA guarantee | No |

### Enterprise — Custom Pricing (Contact Sales)

For organizations with complex requirements, high volume, or compliance needs.

| Feature | Detail |
|---|---|
| Integrations | Unlimited |
| API calls | Unlimited |
| Support | 24/7 priority phone, email, and chat |
| Response time | Within 1 hour |
| Monitoring | Advanced monitoring with custom alerts |
| Sync interval | Real-time |
| Audit logs | Yes |
| Custom field mapping | Yes |
| Webhook support | Yes |
| IP whitelisting | Yes |
| SSO/SAML | Yes |
| Custom API connectors | Yes (visual builder included) |
| Dedicated account manager | Yes |
| SLA guarantee | 99.9% uptime SLA |
| On-premise deployment | Available |
| Data residency | US (default) or EU |
| Custom data retention | Yes (default 90 days, configurable) |
| Penetration test results | Shared under NDA upon request |

Contact sales@nexusintegrations.com for a custom quote. Annual billing discounts are available.

---

## 3. Supported Integrations

### CRM
- **Salesforce** — OAuth 2.0 connection. Requires API access enabled in your Salesforce org. Supports contacts, leads, accounts, opportunities, and custom objects.
- **HubSpot** — API key or OAuth connection. Supports contacts, companies, deals, and tickets.
- **Pipedrive** — API token connection. Supports persons, organizations, deals, and activities.
- **Zoho CRM** — OAuth 2.0 connection. Supports leads, contacts, accounts, deals, and custom modules.

### Email
- **Gmail** — OAuth 2.0 via Google Workspace. Supports reading, sending, labeling, and thread tracking.
- **Outlook / Microsoft 365** — OAuth 2.0 via Azure AD. Supports mail, calendar events, and contacts.
- **SendGrid** — API key connection. Supports transactional email sending, template management, and event webhooks.
- **Mailchimp** — API key connection. Supports audience management, campaign data, and automation triggers.

### Billing
- **Stripe** — API key connection (use restricted keys in production). Supports customers, invoices, subscriptions, and payment events.
- **QuickBooks Online** — OAuth 2.0 connection. Supports customers, invoices, payments, and chart of accounts.
- **Xero** — OAuth 2.0 connection. Supports contacts, invoices, bank transactions, and reports.
- **FreshBooks** — OAuth 2.0 connection. Supports clients, invoices, expenses, and time entries.

### Communication
- **Slack** — OAuth 2.0 via Slack App. Supports posting messages, channel management, and event subscriptions.
- **Microsoft Teams** — OAuth 2.0 via Azure AD. Supports posting messages, channel notifications, and adaptive cards.

### Storage
- **Google Drive** — OAuth 2.0 connection. Supports file upload, download, folder management, and change notifications.
- **Dropbox** — OAuth 2.0 connection. Supports file sync, shared folders, and event webhooks.
- **AWS S3** — IAM access key connection. Supports bucket operations, object management, and event notifications.

### Databases
- **PostgreSQL** — Direct connection via connection string. Supports read/write, schema introspection, and change data capture.
- **MySQL** — Direct connection via connection string. Supports read/write and schema introspection.
- **MongoDB** — Connection string (SRV supported). Supports read/write and change streams.

> **Note:** Custom API connectors (for systems not listed above) are available on the Enterprise plan only. Contact sales@nexusintegrations.com to discuss your requirements.

---

## 4. Setup & Configuration Guides

### Getting Started

1. **Create your account** at app.nexusintegrations.com/signup. You'll need a work email address.
2. **Verify your email** and set your password.
3. **Choose your plan** — you can start with a 14-day free trial of the Professional plan.
4. **Dashboard overview** — After login, you'll see the main dashboard with:
   - **Integrations** — Add, configure, and manage your connected systems.
   - **Workflows** — Build and manage automation workflows.
   - **Monitoring** — View sync status, error logs, and API call usage.
   - **Settings** — Account settings, team management, billing, and security.

### Connecting Your First Integration

Nexus supports two authentication methods:

**OAuth 2.0 Flow (recommended)**
1. Go to **Integrations → Add New** and select the platform.
2. Click **Connect with OAuth**.
3. You'll be redirected to the third-party platform to authorize Nexus.
4. After authorization, you're redirected back to Nexus with the connection active.
5. Click **Test Connection** to verify.

**API Key Method**
1. Go to **Integrations → Add New** and select the platform.
2. Select **API Key** as the authentication method.
3. Generate an API key in the target platform's settings.
4. Paste the API key into the Nexus dashboard.
5. Click **Test Connection** to verify.

### Salesforce Setup (Step-by-Step)

Salesforce requires both OAuth and a security token for API access. Follow these steps:

1. **Enable API access in Salesforce:**
   - Log in to Salesforce → go to **Setup** (gear icon, top right).
   - Navigate to **Settings → API → API Access**.
   - Ensure the "API Enabled" permission is active for your user profile.

2. **Generate your security token:**
   - In Salesforce, go to **My Settings → Personal → Reset My Security Token**.
   - Click **Reset Security Token**. A new token is sent to your email.
   - Save this token — you'll need it in the next step.

3. **Connect in Nexus:**
   - In Nexus, go to **Integrations → Add New → Salesforce**.
   - Enter your Salesforce username.
   - Enter your password with the security token appended (e.g., if password is `MyPass123` and token is `ABCDEF`, enter `MyPass123ABCDEF`).
   - Select your Salesforce environment (Production or Sandbox).
   - Click **Test Connection**.

4. **Verify:** You should see a green "Connected" status. If not, see troubleshooting below.

### Troubleshooting Common Connection Errors

| Error Code | Meaning | Resolution |
|---|---|---|
| `AUTH_FAILED` | Authentication credentials are invalid or expired. | Double-check your API credentials. For Salesforce, ensure your security token is appended to your password (not entered separately). If you recently reset your password, you need a new security token too. |
| `RATE_LIMIT_EXCEEDED` | You've hit the API call limit for your current plan. | Check your usage in **Monitoring → API Usage**. Starter plans are limited to 10,000 calls/mo and Professional to 100,000 calls/mo. You can upgrade your plan or wait for the monthly reset (1st of each month, midnight UTC). |
| `SYNC_CONFLICT` | Duplicate or conflicting records were detected during sync. | Go to **Settings → Sync Rules** and enable conflict resolution. Options: "Source wins" (overwrites target), "Target wins" (keeps target), or "Flag for review" (pauses sync for that record). |
| `TIMEOUT_ERROR` | The target system did not respond within the expected time. | Check the target system's status page for outages. Wait 5 minutes and retry. If persistent, check firewall rules and ensure Nexus IP addresses are whitelisted (see Settings → Network). |
| `MAPPING_ERROR` | A required field mapping is missing or invalid. | Go to **Integrations → [Your Integration] → Field Mapping** and ensure all required fields in the target system have a mapped source field. |
| `WEBHOOK_FAILED` | Webhook delivery failed (HTTP 4xx or 5xx from your endpoint). | Verify your webhook URL is correct and publicly accessible. Check that your endpoint returns a 200 response within 30 seconds. |

### Data Mapping Configuration

1. After connecting an integration, go to **Integrations → [Your Integration] → Field Mapping**.
2. Nexus auto-detects fields from both source and target systems.
3. Drag source fields to target fields to create mappings.
4. Use **Transform** options to modify data in transit (e.g., date format conversion, string concatenation, value mapping).
5. Mark fields as **Required** to prevent sync when data is missing.
6. Click **Save & Validate** to test your mappings against sample data.

### Setting Up Webhook Endpoints

*Available on Professional and Enterprise plans.*

1. Go to **Integrations → Webhooks → Add Endpoint**.
2. Enter your receiving URL (must be HTTPS).
3. Select the events you want to receive (e.g., "record.created", "record.updated", "sync.completed").
4. Optionally set a signing secret for payload verification.
5. Click **Send Test Event** to verify your endpoint.
6. Webhook payloads are JSON and include event type, timestamp, and affected record data.

### Scheduling Sync Intervals

Sync frequency depends on your plan:

| Plan | Minimum Interval | Options |
|---|---|---|
| Starter | 5 minutes | 5 min, 15 min, 30 min, 1 hour, daily |
| Professional | 1 minute | 1 min, 5 min, 15 min, 30 min, 1 hour, daily |
| Enterprise | Real-time | Real-time, 1 min, 5 min, 15 min, 30 min, 1 hour, daily |

To configure: go to **Integrations → [Your Integration] → Sync Settings → Interval**.

Real-time sync (Enterprise) uses change data capture and webhooks to trigger syncs immediately when source data changes.

---

## 5. Security & Compliance

### Certifications & Compliance

- **SOC 2 Type II** — Nexus has completed SOC 2 Type II certification. The audit report is available upon request. Contact security@nexusintegrations.com or your dedicated account manager (Enterprise plan) to request a copy.
- **GDPR Compliant** — Nexus complies with the EU General Data Protection Regulation. An EU Data Processing Addendum (DPA) is available for customers who require one. Contact legal@nexusintegrations.com.

### Data Encryption

- **At rest:** AES-256 encryption for all stored data.
- **In transit:** TLS 1.3 for all data transfers between your systems and Nexus.
- API keys and credentials are encrypted using envelope encryption and stored in a dedicated secrets manager (not in the application database).

### Data Residency

- **Default:** United States (AWS us-east-1).
- **EU option:** Available on the Enterprise plan (AWS eu-west-1). Contact sales to configure.
- Data residency selection is made at account creation for Enterprise customers and cannot be changed after initial setup without a migration.

### Access Control

- **SSO/SAML:** Available on the Enterprise plan only. Supports Okta, Azure AD, OneLogin, and any SAML 2.0-compliant identity provider.
- **Audit Logs:** Available on Professional and Enterprise plans. Logs include user actions, integration changes, data access events, and admin operations. Retained for 1 year.
- **IP Whitelisting:** Available on Professional and Enterprise plans. Restrict dashboard access and API calls to specified IP addresses or CIDR ranges. Configure in **Settings → Security → IP Whitelisting**.
- **Role-Based Access Control (RBAC):** Three built-in roles — Admin, Editor, Viewer. Custom roles available on Enterprise.

### Security Assessments

- **Penetration Testing:** Nexus undergoes annual third-party penetration testing. Summary results are shared under NDA with Enterprise customers upon request.
- **Data Retention:** Default retention period is 90 days for sync logs and transient data. Enterprise customers can configure custom retention policies.
- **Bug Bounty:** Nexus maintains a responsible disclosure program. Report vulnerabilities to security@nexusintegrations.com.

> **Important:** For security questionnaires, vendor assessment forms, or compliance requests beyond what is documented here, please contact security@nexusintegrations.com directly. The general support team is not equipped to provide custom security assessments or complete third-party security questionnaires.

---

## 6. Support & Contact

| Channel | Address |
|---|---|
| General support | support@nexusintegrations.com |
| Security inquiries | security@nexusintegrations.com |
| Sales & Enterprise | sales@nexusintegrations.com |
| Legal | legal@nexusintegrations.com |
| Documentation | docs.nexusintegrations.com |
| Status page | status.nexusintegrations.com |
| Community forum | community.nexusintegrations.com |

### Support Hours & Response Times

| Plan | Hours | Response Time |
|---|---|---|
| Starter | Mon-Fri 9am-6pm EST | Within 24 hours |
| Professional | Mon-Fri 9am-6pm EST | Within 4 hours |
| Enterprise | 24/7 | Within 1 hour |

Enterprise customers also have access to:
- Direct phone support line (provided by your account manager)
- Dedicated Slack channel with the Nexus support engineering team
- Quarterly business reviews

### Escalation Path

If your issue is not resolved within the expected response time:
1. Reply to your existing support ticket with "ESCALATE" in the subject line.
2. Enterprise customers can call their dedicated support line.
3. For urgent production issues, email urgent@nexusintegrations.com (Enterprise only).
