For Brocade X7-8 Directors (running FOS – Fabric OS), especially in environments using **Virtual Fabrics**, you can use the following CLI commands to check **capacity**, **health**, **connectivity between fabrics and ISLs**, and **port errors**.

Make sure you’re logged into the correct logical switch if you're using Virtual Fabrics.

---

### 🧩 **1. Switch Capacity (Ports, Blades, Usage)**

**General port status:**

```bash
switchshow
```

**Show slot and port usage:**

```bash
slotshow
```

**Port licensing (to see licensed capacity):**

```bash
licenseportshow
```

**Show hardware blade information:**

```bash
bladeshow
```

**Check the number of ports used/free:**

```bash
portshow | grep "state"
```

---

### ❤️ **2. Switch Health**

**Basic health summary:**

```bash
switchstatusshow
```

**Power supplies, fans, temperature:**

```bash
fanshow
tempshow
psshow
```

**General diagnostics:**

```bash
diagshow
```

**Check for any issues (like errors, CRCs, or marginal ports):**

```bash
errdump
errshow
```

**RASlog (for deeper hardware/software events):**

```bash
raslog
```

---

### 🔗 **3. ISL Connectivity & Inter-Fabric Links (IFLs)**

**Display ISLs:**

```bash
islshow
```

**Check E-Ports (ISLs):**

```bash
portshow <port_number>
```

Look for:

* Port type: `E-Port`
* Link status: `Up`
* Distance/Buffer credits

**Show routing information between virtual fabrics:**

```bash
fabricshow
```

**Display all logical switches in VF-enabled system:**

```bash
lscfg --show
```

**Check inter-switch connectivity in VF environments:**

```bash
lsdbshow
```

---

### ❗ **4. Port Errors and Health**

**Per-port detailed stats:**

```bash
porterrshow
```

**Clears port error counters (good before/after comparison):**

```bash
statsclear
```

**Monitor live port performance:**

```bash
portperfshow
```

**Persistent port health history:**

```bash
porthealthshow
```

**Track error counts per physical port (good for ISLs, targets, HBAs):**

```bash
portstatsshow <port_number>
```

---

### 📦 **Bonus - Logical Switch Commands (Virtual Fabrics)**

**Check all logical switches:**

```bash
lscfg --show
```

**Switch context to another logical switch:**

```bash
setcontext <LS_ID>
```

**Check which logical switch you're in:**

```bash
fabricshow
```

**Show which ports are assigned to which logical switch:**

```bash
portcfgshow
```

---

Would you like a summarized script or checklist for periodic health checks or port auditing?



# Brocade X7-8 Director Commands for Virtual Fabrics

Here are the essential commands to check capacity, health, connectivity, and port errors on Brocade X7-8 directors with Virtual Fabrics:

## Basic System Information
```bash
# Show chassis information
chassisshow

# Show firmware versions
version

# Show system status
sysmon -show
```

## Capacity and Resource Utilization
```bash
# Show switch capacity and resource usage
switchshow -capacity

# Display port utilization statistics
portperfshow

# Show CPU and memory usage
statshow

# Show buffer utilization
buffershow
```

## Virtual Fabric Configuration
```bash
# List all virtual fabrics
fabricshow -v

# Show current virtual fabric membership
fabricshow

# Display virtual fabric configuration details
vfdump
```

## Health and Connectivity
```bash
# Show overall fabric health
fabrichealthshow

# Display switch health status
healthcheck

# Show all ISL connections
islshow

# Show extended ISL information
islshow -e

# Show trunking information
trunkshow
```

## Port Errors and Statistics
```bash
# Show port statistics and errors
porterrshow

# Display detailed port statistics
portstatsshow

# Show frame errors
framehistoryshow

# Show SFP diagnostics (for a specific port)
sfpshow <portnumber>
```

## Advanced Monitoring
```bash
# Continuous monitoring of port errors (Ctrl+C to stop)
porterrshow -iter 1

# Show performance statistics
perfshow

# Display fabric topology
topology -fabric
```

## Logs and Alerts
```bash
# Show recent error logs
errshow

# Display current alerts
alarmshow

# Show system log
logshow
```

Remember to:
1. Use the `-f <fabricID>` flag with commands when you need to query a specific virtual fabric
2. For detailed help on any command, append `-help` (e.g., `switchshow -help`)
3. Some commands may require admin privileges
