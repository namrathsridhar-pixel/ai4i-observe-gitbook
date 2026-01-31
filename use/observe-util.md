# observe-util

## observe-util package



> **Enterprise-grade monitoring and metrics collection for FastAPI applications**

***

### 📍 Quick Info

| Property                    | Value                                      |
| --------------------------- | ------------------------------------------ |
| **Package**                 | `observe`                                  |
| **Location**                | `observe/`                                 |
| **Framework**               | FastAPI (native support)                   |
| **Supported Organizations** | `irctc`, `kisanmitra`, `bashadaan`, `beml` |
| **Metrics Endpoint**        | `/enterprise/metrics`                      |
| **Health Endpoint**         | `/enterprise/health`                       |

***

### 🎯 What Is This?

The Dhruva Observability Package provides comprehensive monitoring, metrics tracking, and real-time analytics for FastAPI applications. It enables multi-tenant metrics collection, SLA tracking, and business analytics through Prometheus and Grafana.

### ✨ Key Features

* **Multi-tenant metrics collection** with organization-level isolation
* **Automatic service detection** (Translation, TTS, ASR, NER, etc.)
* **Real-time monitoring** via Prometheus and Grafana
* **Business metrics** - characters, audio seconds processed
* **Easy integration** - 3 lines of code to get started
* **Plug-and-play** - Works with any FastAPI application

***

### 📋 Prerequisites

#### ⚠️ CRITICAL: Organization Identification Required



**Before using this package, you MUST implement organization extraction.** The system needs to identify which organization/tenant each request belongs to.

**Option 1: JWT Token with Organization Claims (Recommended)**



Add organization information to your JWT tokens:

```
payload = {
    "sub": "user@example.com",
    "organization": "your-organization-name",  # Required!
    "name": "Organization Display Name",
    "exp": 1234567890
}
```

The middleware checks for these fields in order: `organization`, `org`, `name`, `company`

**Option 2: Database Mapping**

Store organization with API keys in your database and modify the middleware's organization extraction logic in `middleware.py`.

**⚠️ Current Mock Implementation**

The package includes a **mock organization extractor** that uses hash-based mapping to assign organizations. **You MUST replace this** with proper implementation:

```
# In observe/middleware.py (lines 126-135)
def _get_organization_from_api_key(api_key: str) -> str:
    """Map API key to organization name using consistent hashing."""
    organizations = ["irctc", "kisanmitra", "bashadaan", "beml"]
    hash_value = int(hashlib.md5(api_key.encode()).hexdigest(), 16)
    org_index = hash_value % len(organizations)
    return organizations[org_index]
```

**⚠️ CRITICAL REQUIREMENT**:

* **Any API request MUST have an organization** that matches one of: `irctc`, `kisanmitra`, `bashadaan`, or `beml`
* If using JWT tokens, ensure the `organization` field contains one of these values
* To add new organizations, update the list in `observability/middleware.py` (line 129)

#### System Requirements

* **Python 3.8+**
* **FastAPI** application
* **Prometheus** (for metrics storage)
* **Grafana** (for visualization)

***

### 🚀 Installation

#### Option 1: Use as Local Module (Current Setup)

Copy the `observe/` folder into your FastAPI project:

```
your-project/
├── observe/           # Copy this entire folder
│   ├── __init__.py
│   ├── plugin.py
│   ├── middleware.py
│   ├── metrics.py
│   ├── config.py
│   └── dashboard-templates/
└── main.py                  # Your FastAPI app
```

**Install Dependencies:**

```
pip install -r requirements.txt
```

#### Option 2: Install from PyPI (Future)

```
pip install observe
```

***

### 🔧 Integration with FastAPI

#### Step 1: Import the Plugin

Add the observability plugin to your FastAPI application:

```
from fastapi import FastAPI
from observe import ObservabilityPlugin

app = FastAPI()

# Initialize and register observability
enterprise = ObservabilityPlugin()
enterprise.register_plugin(app)

# Your existing routes work unchanged
@app.get("/")
def read_root():
    return {"message": "Hello World"}

# All requests are now automatically tracked!
```

**This creates these endpoints automatically:**

* `/enterprise/metrics` - Enterprise observability metrics
* `/enterprise/health` - Health check
* `/enterprise/config` - Configuration

#### Optional: Add Standard Prometheus Metrics

If you want **both** standard Prometheus metrics AND enterprise metrics:

```
from fastapi import FastAPI
from prometheus_client import make_asgi_app
from observe import ObservabilityPlugin

app = FastAPI()

# Initialize observability plugin
enterprise = ObservabilityPlugin()
enterprise.register_plugin(app)

# Optional: Mount standard Prometheus metrics endpoint
metrics_app = make_asgi_app()
app.mount("/metrics", metrics_app)
```

**This gives you two metrics endpoints:**

* `/metrics` - Standard Prometheus metrics (if using prometheus\_client)
* `/enterprise/metrics` - Enterprise observability metrics (organization-filtered)

#### Step 2: Configure Environment Variables



Set these environment variables before starting your application:

```
# Required
export OBSERVE_UTIL_ENABLED=true

# Optional
export OBSERVE_UTIL_DEBUG=true
export OBSERVE_UTIL_METRICS_PATH=/enterprise/metrics
export OBSERVE_UTIL_HEALTH_PATH=/enterprise/health
export OBSERVE_UTIL_COLLECT_SYSTEM_METRICS=true
export OBSERVE_UTIL_COLLECT_GPU_METRICS=true
export OBSERVE_UTIL_COLLECT_DB_METRICS=true
```

#### Step 3: Verify Installation



Start your FastAPI application and test the endpoints:

```
# Start your FastAPI app (example)
uvicorn main:app --host 0.0.0.0 --port 8000

# Verify metrics endpoint
curl http://localhost:8000/enterprise/metrics

# Check plugin status
curl http://localhost:8000/enterprise/health

# View configuration
curl http://localhost:8000/enterprise/config
```

***

### 📊 What Gets Tracked Automatically



Once integrated, the middleware automatically tracks:

#### Request Metrics



* Total requests by organization/app/endpoint
* Request duration (histograms)
* Error counts and rates
* Status code distribution

#### Service Metrics



* Service-specific requests (Translation, TTS, ASR, etc.)
* Component latency tracking
* Service availability

#### Business Metrics



* **TTS**: Characters synthesized
* **Translation**: Characters translated
* **ASR**: Audio seconds processed
* **OCR**: Image Size Processed (KBs)
* **NER**: Tokens Processed
* **Transliteration**: Characters Transliterated
* **Text Language Detection**: Characters Processed
* **Speaker Diarization**: Audio Seconds Processed
* **Language Diarization**: Audio Seconds Processed
* **Audio Language Detection**: Audio Seconds Processed

#### System Metrics



* CPU usage
* Memory usage
* Peak throughput
* Active connections

***

### 🎓 How It Works



#### Example Request Flow

```
# Client makes a request with JWT token
curl -X POST "http://localhost:8000/translation/v1" \
  -H "Authorization: Bearer eyJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{"input": [{"source": "Hello World"}]}'
```

#### What Happens Automatically



1. **Middleware intercepts** the request at `/translation/v1`
2. **Extracts organization** from JWT token → reads `organization` field (e.g., `irctc`)
3. **Validates organization** → must be one of: `irctc`, `kisanmitra`, `bashadaan`, `beml`
4. **Detects service type** → URL contains `/translation/` → service = `translation`
5. **Counts characters** → "Hello World" = 11 characters
6. **Processes request** → passes to your actual route handler
7. **Tracks metrics** → request count, duration, characters translated
8. **Updates Prometheus** → metrics available at `/enterprise/metrics`
9. **Grafana displays** → organization-filtered dashboard shows the data

***

### 🔧 Configuration

#### Environment Variables Reference

| Variable                              | Required | Default               | Description                          |
| ------------------------------------- | -------- | --------------------- | ------------------------------------ |
| `OBSERVE_UTIL_ENABLED`                | Yes      | `false`               | Enable/disable the plugin            |
| `OBSERVE_UTIL_DEBUG`                  | No       | `false`               | Enable debug logging                 |
| `OBSERVE_UTIL_METRICS_PATH`           | No       | `/enterprise/metrics` | Metrics endpoint path                |
| `OBSERVE_UTIL_HEALTH_PATH`            | No       | `/enterprise/health`  | Health check endpoint                |
| `OBSERVE_UTIL_COLLECT_SYSTEM_METRICS` | No       | `true`                | Collect system metrics (CPU, memory) |
| `OBSERVE_UTIL_COLLECT_GPU_METRICS`    | No       | `true`                | Collect GPU usage metrics            |
| `OBSERVE_UTIL_COLLECT_DB_METRICS`     | No       | `true`                | Collect database connection metrics  |

***

### 📡 Available Endpoints



Once registered, the plugin automatically creates these endpoints:

| Endpoint              | Method | Description                                |
| --------------------- | ------ | ------------------------------------------ |
| `/enterprise/metrics` | GET    | Prometheus metrics (scraped by Prometheus) |
| `/enterprise/health`  | GET    | Health check and plugin status             |
| `/enterprise/config`  | GET    | Current plugin configuration               |

***

### 🔍 Prometheus Configuration



Add this scrape configuration to your `prometheus.yml`:

```
scrape_configs:
  - job_name: "dhruva-enterprise-observability"
    scrape_interval: 5s
    static_configs:
      - targets: ["your-app-host:8000"]  # Update with your host
    metrics_path: "/enterprise/metrics"
```

Restart Prometheus to start collecting metrics.

***

### 📊 Grafana Dashboard Setup

#### Prerequisites

You should already have:

* Grafana running and accessible
* Prometheus configured to scrape `/enterprise/metrics`

#### Setup Grafana Dashboard (Per Organization)

**1. Login to Grafana**

Navigate to your Grafana instance (e.g., `http://your-grafana-url:3000`)

**2. Create Organization**

* Go to **Server Admin → Organizations → New Organization**
* Enter organization name (e.g., "IRCTC", "KisanMitra")
* Click **Create**

**3. Switch to the Organization**

* **Server Admin → Organizations → \[Your Org] → Switch To**

**4. Create Prometheus Data Source**

* Go to **Configuration → Data Sources → Add data source**
* Select **Prometheus**
* Configure:
  * **Name**: `Prometheus-[OrgName]` (e.g., `Prometheus-IRCTC`)
  * **URL**: Your Prometheus URL (e.g., `http://prometheus:9090`)
* Click **Save & Test**
* **Important**: Note the **UID** from the browser URL
  * Example: `http://grafana:3000/datasources/edit/P1809F7CD0C75ACF3`
  * UID = `P1809F7CD0C75ACF3`

**5. Import Dashboard from Package**

The observability package includes pre-built dashboard templates in the `dashboard-templates/` folder.

**Locate Dashboard JSON:**

```
observe/dashboard-templates/
├── devops_operational_dashboard_template.json
└── ...
```

**Update Dashboard JSON:**

1. Open the dashboard JSON file in a text editor
2. Find and replace all instances of: `"uid": "OLD_UID"`
3. With your actual data source UID: `"uid": "P1809F7CD0C75ACF3"`

**Using Find & Replace in text editor:**

```
Find:    "uid": "OLD_UID_HERE"
Replace: "uid": "P1809F7CD0C75ACF3"
```

**Using command line:**

```
# Linux/Mac
sed -i 's/"uid": "OLD_UID_HERE"/"uid": "P1809F7CD0C75ACF3"/g' "devops_operational_dashboard_template.json"

# Windows PowerShell
(Get-Content "dashboard.json") -replace '"uid": "OLD_UID_HERE"', '"uid": "P1809F7CD0C75ACF3"' | Set-Content "dashboard.json"
```

**6. Import Dashboard to Grafana**

* Go to **Dashboards → Import**
* Click **Upload JSON file**
* Select the modified dashboard JSON file (from `observe/dashboard-templates/`)
* Click **Import**
* Dashboard should now show metrics filtered for your organization

**7. Repeat for Each Organization**

For each client organization:

1. Create new Grafana organization
2. Create new Prometheus data source
3. Note the UID
4. **Make a copy** of the dashboard JSON
5. Update the copy with the new UID
6. Import into the organization

**8. Creating Custom Dashboards**

If you need to create a new custom dashboard with different visualizations:

1. **View Available Metrics**: Check `observe/metrics.py` to see all base metrics exposed by the package
2. **Write PromQL Queries**: Create PromQL queries based on these base metrics
3. **Create Dashboard in Grafana**:
   * Go to **Dashboards → New Dashboard**
   * Add panels and configure with your PromQL queries
   * Save the dashboard

**Example PromQL Queries:**

```
# Total requests by organization
sum(enterprise_requests_total{customer="irctc"}) by (endpoint)

# Average request duration
rate(enterprise_request_duration_seconds_sum[5m]) / rate(enterprise_request_duration_seconds_count[5m])

# Characters processed by service
sum(rate(enterprise_characters_total[5m])) by (service_type, customer)

# Error rate
rate(enterprise_errors_total[5m])
```

**Note**: All available metrics are defined in `observe/metrics.py`. Refer to that file to see the exact metric names, labels, and descriptions when building custom queries.

***

### ✅ Verification Checklist

#### Plugin Verification

```
# Check metrics endpoint (should return Prometheus format metrics)
curl http://localhost:8000/enterprise/metrics

# Check health endpoint
curl http://localhost:8000/enterprise/health

# Check configuration
curl http://localhost:8000/enterprise/config
```

#### Prometheus Verification

```
# Check Prometheus targets (adjust URL to your Prometheus instance)
curl http://your-prometheus-url:9090/api/v1/targets
```

Look for the `dhruva-enterprise-observability` job showing as **UP**.

#### Grafana Verification

**In Grafana:**

* ✅ Organization created for each client
* ✅ Data source connected (green checkmark on "Save & Test")
* ✅ Dashboard imported successfully
* ✅ Dashboard shows data (not "No data")
* ✅ Metrics filtered for correct organization

***

### 📦 Package Structure

```
observe/
├── __init__.py              # Package initialization
├── plugin.py                # Main ObservabilityPlugin class
├── middleware.py            # FastAPI middleware (request tracking)
├── metrics.py               # MetricsCollector (50+ metrics)
├── config.py                # PluginConfig (environment variables)
├── dashboard-templates/              # Pre-built Grafana dashboard Templates
│   └── *.json              # Dashboard JSON files
└── README.md                # This file
```

**Key Files:**

* `plugin.py` - Main entry point, register with FastAPI
* `middleware.py` - Intercepts requests, tracks metrics (UPDATE ORG LIST HERE)
* `metrics.py` - Defines all Prometheus metrics
* `config.py` - Configuration from environment variables
* `dashboard-templates/*.json` - Import these into Grafana (update UIDs first!)

***

### 🎯 Quick Start Checklist

1. ✅ **Copy observability folder** to your FastAPI project
2. ✅ **Install dependencies**: `pip install -r requirements.txt`
3.  ✅ **Add 3 lines to your FastAPI app**:

    ```
    from observe import ObservabilityPlugin
    enterprise = ObservabilityPlugin()
    enterprise.register_plugin(app)
    ```
4. ✅ **Set environment variables** (at minimum: `OBSERVE_UTIL_ENABLED=true`)
5. ✅ **Implement organization extraction** in JWT tokens
6. ✅ **Update organization list** in `middleware.py` if needed
7. ✅ **Start your application**
8. ✅ **Verify endpoints**: `/enterprise/metrics` and `/enterprise/health`
9. ✅ **Configure Prometheus** to scrape `/enterprise/metrics`
10. ✅ **Setup Grafana**: Create orgs, data sources, import dashboards

***

### 🚨 Important Notes

#### Before Using in Production



1. **MUST IMPLEMENT** organization extraction from JWT tokens
2. **MUST ENSURE** API requests include organization matching one of: `irctc`, `kisanmitra`, `bashadaan`, `beml`
3. **MUST CONFIGURE** proper JWT secret and verification
4. **MUST CREATE** Grafana organizations for each client
5. **MUST CREATE** Prometheus data sources in each Grafana organization
6. **MUST UPDATE** dashboard JSON files with correct data source UIDs
7. **MUST CONFIGURE** Prometheus to scrape `/enterprise/metrics` endpoint

#### Known Limitations

* **Manual Grafana setup required** for each organization
* **Dashboard JSON must be updated** with data source UIDs before import
* **Mock organization extractor** must be replaced with real implementation

***

### 🔐 Security Considerations

#### Production Requirements

1. ✅ **JWT Signature Verification**: Implement proper token verification (don't use `verify_signature=False`)
2. ✅ **JWT Secret Security**: Use a secure secret key for JWT validation
3. ✅ **Organization Isolation**: Verify metrics don't leak between organizations
4. ✅ **Access Control**: Ensure proper RBAC in Grafana organizations
5. ✅ **Network Security**: Restrict access to metrics endpoint if needed

***

### 💡 Common Commands

```
# View all metrics
curl http://localhost:8000/enterprise/metrics

# View observability plugin status
curl http://localhost:8000/enterprise/health

# View observability configuration
curl http://localhost:8000/enterprise/config
```

***

### 🐛 Troubleshooting

#### Issue: "No data" in Grafana

**Solution**: Verify data source UID in dashboard JSON matches your Prometheus data source UID.

#### Issue: Metrics endpoint returns 404



**Solution**:

1. Set `OBSERVE_UTIL_ENABLED=true`
2. Verify plugin is registered in your FastAPI app
3. Restart application

#### Issue: Wrong organization data showing



**Solution**:

1. Enable debug mode: `export OBSERVE_UTIL_DEBUG=true`
2. Check logs for organization extraction
3. Implement proper JWT-based organization extraction

#### Enable Debug Mode

```
export OBSERVE_UTIL_DEBUG=true
```

Check logs for detailed information about:

* Organization extraction: `🔑 Extracted customer from JWT: ...`
* Request tracking: `🔍 Request: GET /path -> Service: ..., Organization: ...`
* Plugin initialization: `✅ Dhruva Observability Plugin initialized successfully`

#### Walkthrough Videos

* v1.0 : [https://youtu.be/i7Tv5sLzic8](https://youtu.be/i7Tv5sLzic8)
* v1.1 : [https://youtu.be/sABEDKOrO-Q](https://youtu.be/sABEDKOrO-Q)
* v1.2 : [https://youtu.be/7DicoN3wa8o](https://youtu.be/7DicoN3wa8o)

#### Test Cases

* [https://docs.google.com/spreadsheets/d/1TWlaZ9ADzfIME8f4SiNyq1kzc33j1i7q-RQ9zm0WscY/edit?gid=506454981#gid=506454981](https://docs.google.com/spreadsheets/d/1TWlaZ9ADzfIME8f4SiNyq1kzc33j1i7q-RQ9zm0WscY/edit?gid=506454981#gid=506454981)

***

### 📄 License

Part of the Dhruva Platform

***

### 🔄 Version Information



* **Package Version**: 1.0.3
* **Last Updated**: October 2025
* **Framework**: FastAPI (native support)
* **Python**: 3.8+

***

### 🎯 Summary



This observability package provides **plug-and-play monitoring** for FastAPI applications:

1. **Copy** the `observe/` folder to your project
2. **Install** dependencies: `pip install -r requirements.txt`
3. **Add** 3 lines of code to your FastAPI app
4. **Set** environment variables
5. **Start** your app and verify `/enterprise/metrics` works
6. **Configure** Prometheus and Grafana
7. **Done!** All requests are now tracked with organization-level isolation
