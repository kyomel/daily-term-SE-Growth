day - 2

## Cloud Systems Drift

### Definition:

Cloud Systems Drift (also called Configuration Drift or Infrastructure Drift) occurs when the actual state of your cloud infrastructure gradually diverges from its intended or documented configuration over time. It happens when manual changes, emergency fixes, undocumented updates, or inconsistent deployments cause your systems to become different from what they should be.

**Key Concept:**

- Intended State: What your infrastructure should look like (documented, planned)
- Actual State: What your infrastructure really looks like (after changes accumulate)
- Drift: The gap between these two states
- Problem: Systems become unpredictable, unreproducible, and fragile

### Example:

AWS Cloud Infrastructure

```
Initial State (Documented)
# infrastructure-plan.yaml (What SHOULD exist)

production:
  region: us-east-1

  web_servers:
    type: t3.medium
    count: 3
    ami: ami-12345 (Ubuntu 20.04)
    security_groups:
      - allow-http-443
      - allow-ssh-from-vpn

  database:
    type: db.t3.large
    engine: PostgreSQL 13.4
    backup: enabled
    multi_az: true

  load_balancer:
    type: application
    listeners:
      - port: 443
        protocol: HTTPS
        certificate: arn:aws:acm:cert-123

  s3_buckets:
    - name: prod-assets
      versioning: enabled
      encryption: AES256

  iam_roles:
    - ec2-web-server-role
    - lambda-processor-role

Actual State After 6 Months (Reality)
# actual-infrastructure.yaml (What ACTUALLY exists)

production:
  region: us-east-1

  web_servers:
    server_1:
      type: t3.medium ✅
      ami: ami-12345 ✅
      security_groups:
        - allow-http-443 ✅
        - allow-ssh-from-vpn ✅
        - allow-mysql-3306 ⚠️ (Added manually for debugging)
        - allow-all-from-office ⚠️ (Emergency access)
      notes: "Original server"

    server_2:
      type: t3.large ❌ (Upgraded during Black Friday)
      ami: ami-67890 ❌ (Different AMI! Someone rebuilt it)
      security_groups:
        - allow-http-443 ✅
        - custom-sg-12345 ❌ (Wrong security group!)
      notes: "Manually patched, never documented"

    server_3:
      type: t3.medium ✅
      ami: ami-12345 ✅
      security_groups:
        - allow-http-443 ✅
        - allow-ssh-from-anywhere ❌ (Security risk!)
      custom_packages:
        - ffmpeg ❌ (Installed manually)
        - imagemagick ❌ (Undocumented dependency)
      notes: "Bob's special configuration"

    server_4: ❌ (Extra server! Not in plan!)
      type: t3.small
      ami: ami-11111
      notes: "Created for testing, never removed"

  database:
    type: db.t3.xlarge ❌ (Upgraded, not documented)
    engine: PostgreSQL 14.2 ❌ (Upgraded without testing)
    backup: disabled ❌❌❌ (CRITICAL! Turned off to save money)
    multi_az: false ❌ (Downgraded to save costs)
    manual_changes:
      - max_connections increased to 500
      - shared_buffers modified
      - Custom performance tuning

  load_balancer:
    type: application ✅
    listeners:
      - port: 443 ✅
        protocol: HTTPS ✅
        certificate: arn:aws:acm:cert-999 ❌ (Different cert!)
      - port: 8080 ❌ (Added for internal API)
      - port: 9000 ❌ (Debug endpoint, still exposed!)

  s3_buckets:
    - name: prod-assets ✅
      versioning: disabled ❌ (Turned off!)
      encryption: none ❌❌❌ (REMOVED! Security issue!)

    - name: prod-backups ❌ (Not in original plan)
      public_access: true ❌❌❌ (PUBLICLY ACCESSIBLE!)

    - name: temp-test-bucket ❌ (Should have been deleted)

  iam_roles:
    - ec2-web-server-role ✅
    - lambda-processor-role ✅
    - admin-emergency-role ❌ (Too many permissions!)
    - bob-test-role ❌ (Personal role, never removed)
    - contractor-access ❌ (Contractor left 3 months ago!)
    - temp-role-123 ❌ (No idea what this is for)

The Discovery
// Engineer trying to rebuild production environment

console.log("Attempting to recreate production...");

// Step 1: Use documented configuration
const documentedConfig = loadConfig('infrastructure-plan.yaml');
createInfrastructure(documentedConfig);

// Result:
console.log("❌ FAILED: Application doesn't start!");
console.log("❌ Database connection refused");
console.log("❌ Missing custom packages");
console.log("❌ Security groups don't match");

// Step 2: Compare actual vs documented
const actualConfig = scanActualInfrastructure();
const drift = compareConfigs(documentedConfig, actualConfig);

console.log("🔍 DRIFT DETECTED:");
console.log(drift);

/*
Output:
{
  servers: {
    count_mismatch: "Expected 3, Found 4",
    instance_types: "2 of 3 don't match expected",
    ami_drift: "2 servers using wrong AMI",
    security_groups: "15 unauthorized rules found"
  },
  database: {
    size_drift: "Upgraded from large to xlarge",
    version_drift: "13.4 → 14.2 (undocumented)",
    backup_status: "CRITICAL: Backups disabled!",
    availability: "No longer multi-AZ"
  },
  storage: {
    bucket_count: "Expected 1, Found 3",
    public_buckets: "1 bucket publicly accessible!",
    encryption: "Missing on production bucket!"
  },
  security: {
    excessive_iam_roles: "3 unauthorized roles",
    contractor_access: "Still has access!",
    open_ports: "Debug endpoint exposed to internet"
  }
}
*/

console.log("💥 Cannot safely recreate production environment!");
console.log("⚠️  Documentation is worthless!");

Consequences of Drift
❌ REPRODUCIBILITY ISSUES:
├─ Can't rebuild servers from documentation
├─ Disaster recovery plans don't work
├─ Scaling fails (new servers don't match old ones)
└─ Testing environments don't match production

❌ SECURITY RISKS:
├─ Unknown security groups/firewall rules
├─ Unpatched systems
├─ Forgotten debugging endpoints exposed
└─ Old credentials still active

❌ OPERATIONAL PROBLEMS:
├─ "Works on my machine" but not others
├─ Unpredictable behavior
├─ Difficult to debug issues
└─ Can't identify what changed

❌ COMPLIANCE VIOLATIONS:
├─ Can't prove system state for audits
├─ Changes not tracked
├─ No change management
└─ Regulatory requirements not met

❌ COST ISSUES:
├─ Zombie resources running (forgotten servers)
├─ Over-provisioned systems
├─ Duplicate resources
└─ Can't optimize without knowing actual state
```

---

day - 3

## The Heartbeat (Distributed System)

### Definition:

The Heartbeat is a pattern in distributed systems where nodes (servers, services, or processes) send periodic signals (heartbeats) to indicate they are alive and functioning properly. If a heartbeat stops, other nodes in the system assume the silent node has failed and take appropriate action (like routing traffic elsewhere or triggering failover).

**Key Concept:**

- Periodic Signal: "I'm alive!" message sent at regular intervals
- Health Check: Proves the node is responsive and working
- Failure Detection: Absence of heartbeat indicates failure
- Automated Response: System reacts without human intervention

### Example:

Web Server Cluster

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    │  (HAProxy)      │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼─────┐      ┌─────▼─────┐     ┌─────▼─────┐
    │ Server 1  │      │ Server 2  │     │ Server 3  │
    │ (Node)    │      │ (Node)    │     │ (Node)    │
    └───────────┘      └───────────┘     └───────────┘

Each server sends heartbeat to load balancer every 3 seconds.
Load balancer expects heartbeat within 10 seconds (3 missed = dead).

Heartbeat Implementation

// server.js - Web Server with Heartbeat

const express = require('express');
const axios = require('axios');

const app = express();
const SERVER_ID = process.env.SERVER_ID || 'server-1';
const SERVER_PORT = process.env.PORT || 3000;
const LOAD_BALANCER_URL = 'http://load-balancer:8080';
const HEARTBEAT_INTERVAL = 3000; // 3 seconds

// Main application logic
app.get('/', (req, res) => {
  res.json({
    message: 'Hello from ' + SERVER_ID,
    timestamp: new Date().toISOString()
  });
});

// Health check endpoint (alternative to active heartbeat)
app.get('/health', (req, res) => {
  // Check if server is actually healthy
  const isHealthy = checkSystemHealth();

  if (isHealthy) {
    res.status(200).json({
      status: 'healthy',
      server: SERVER_ID,
      uptime: process.uptime(),
      memory: process.memoryUsage(),
      timestamp: Date.now()
    });
  } else {
    // Report unhealthy state
    res.status(503).json({
      status: 'unhealthy',
      server: SERVER_ID,
      error: 'System degraded'
    });
  }
});

function checkSystemHealth() {
  // Check critical components
  const memoryOK = process.memoryUsage().heapUsed < 500 * 1024 * 1024; // < 500MB
  const dbConnected = checkDatabaseConnection();
  const cpuOK = getCurrentCPU() < 90; // < 90% CPU

  return memoryOK && dbConnected && cpuOK;
}

// HEARTBEAT: Active push to load balancer
function sendHeartbeat() {
  const heartbeatData = {
    server_id: SERVER_ID,
    timestamp: Date.now(),
    status: 'alive',
    metrics: {
      cpu: getCurrentCPU(),
      memory: process.memoryUsage().heapUsed,
      requests_per_sec: getRequestRate(),
      response_time_ms: getAvgResponseTime()
    }
  };

  axios.post(`${LOAD_BALANCER_URL}/heartbeat`, heartbeatData)
    .then(response => {
      console.log(`✅ [${SERVER_ID}] Heartbeat sent at ${new Date().toISOString()}`);
    })
    .catch(error => {
      console.error(`❌ [${SERVER_ID}] Failed to send heartbeat:`, error.message);
    });
}

// Start heartbeat loop
setInterval(sendHeartbeat, HEARTBEAT_INTERVAL);
console.log(`🔄 [${SERVER_ID}] Heartbeat started (every ${HEARTBEAT_INTERVAL}ms)`);

// Start server
app.listen(SERVER_PORT, () => {
  console.log(`🚀 [${SERVER_ID}] Server running on port ${SERVER_PORT}`);

  // Send initial heartbeat
  sendHeartbeat();
});

// Graceful shutdown - send final heartbeat
process.on('SIGTERM', () => {
  console.log(`🛑 [${SERVER_ID}] Shutting down gracefully...`);

  // Notify load balancer we're going down
  axios.post(`${LOAD_BALANCER_URL}/shutdown`, {
    server_id: SERVER_ID,
    reason: 'graceful_shutdown'
  }).then(() => {
    process.exit(0);
  });
});
```

---
