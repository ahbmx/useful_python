Here are the main `symcli` (Solutions Enabler) commands you can use to get **health**, **capacity**, **audit logs**, and **events** for a **PowerMax 8000** array, assuming you're using **SymCLI v9.2.3.6**:

---

### 🔍 1. **Health Status of the Array**

To get the overall health and status of the array components:

```bash
symcfg -sid <SID> list -health
```

Or for detailed environment and system status:

```bash
symsys -sid <SID> list -detail
```

You can also check director or DA health:

```bash
symcfg -sid <SID> list -v -dir all
```

---

### 💽 2. **Capacity Reporting**

#### Overall Capacity Summary:

```bash
symcfg -sid <SID> list -capacity -gb
```

#### Detailed Capacity (including Used/Free):

```bash
symcfg -sid <SID> list -thin -pool -detail -gb
```

#### Specific Pool Capacity:

```bash
symcfg -sid <SID> list -pool <pool_name> -detail -gb
```

#### Snapshot Capacity:

```bash
symsnapvx -sid <SID> list -detail
```

---

### 🕵️ 3. **Audit Logs**

The audit logs are not directly extracted using basic `symcli` commands. They are typically accessed through:

```bash
symaccess -sid <SID> auditlog list
```

Or through the Solutions Enabler database logs if configured. If you're using Unisphere for PowerMax or SYMAPI logging, you’ll find logs in:

```bash
/var/symapi/log
```

Or use this to parse logs (Linux example):

```bash
grep 'AUDIT' /var/symapi/log/symapi.log
```

Note: Full audit trail may require viewing through **Unisphere for PowerMax** or **SE Management Server**.

---

### ⚠️ 4. **Events or Alerts**

To list recent system events:

```bash
symevent -sid <SID> list
```

Or view system status / faults:

```bash
symcfg -sid <SID> list -fa -detail
```

For environmental or fault events:

```bash
symsys -sid <SID> list -env
```

---

### ✅ Notes:

* Replace `<SID>` with your array’s Symmetrix ID.
* Use `-v` or `-detail` flags to get more verbose output.
* If running remotely, ensure `SYMCLI_CONNECT` or `SYMCLI_SERVER` environment variables are set correctly.

Would you like a script to run all these checks together for periodic monitoring or reporting?



To extract **`symaudit`** and **`symevent`** logs from the **last 24 hours**, here's how you can do it using **SymCLI v9.2.3.6** for a **PowerMax 8000**:

---

## ✅ 1. **`symaudit` — Audit Log (Last 24 Hours)**

```bash
symaudit -sid <SID> list -time_window 24:00
```

This retrieves all audit records for the last 24 hours. You can add `-detail` for extended information:

```bash
symaudit -sid <SID> list -time_window 24:00 -detail
```

---

## ✅ 2. **`symevent` — Event Log (Last 24 Hours)**

```bash
symevent -sid <SID> list -time_window 24:00
```

Add `-detail` if you want more verbose output:

```bash
symevent -sid <SID> list -time_window 24:00 -detail
```

---

### 🔁 Optional: Redirect Output to a File

If you want to save the output:

```bash
symaudit -sid <SID> list -time_window 24:00 > symaudit_24h.log
symevent -sid <SID> list -time_window 24:00 > symevent_24h.log
```

---

Let me know if you want a combined script or to filter by severity/type.
