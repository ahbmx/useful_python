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




# SymCLI Solutions Enabler Commands for PowerMax 8000 (v9.2.3.6)

Here are the essential SymCLI commands to check health, capacity, audit logs, and events on your PowerMax 8000 array:

## Health Status Commands

1. **Array Health Summary**:
   ```
   symcfg list -v
   ```

2. **Detailed Health Status**:
   ```
   symhealth show -sid <SID>
   ```

3. **Component Health**:
   ```
   symcfg list -dir all -v
   symcfg list -fa all -v
   symcfg list -sa all -v
   symcfg list -rdfa all -v
   ```

## Capacity Information

1. **Array Capacity Overview**:
   ```
   symcfg list -sid <SID> -capacity
   ```

2. **Detailed Storage Pools**:
   ```
   symcfg list -sid <SID> -pool -detail
   ```

3. **Thin Pool Utilization**:
   ```
   symcfg list -sid <SID> -thin -detail
   ```

4. **Device Capacity**:
   ```
   symdev list -sid <SID> -cap
   ```

## Audit Log Commands

1. **View Audit Log**:
   ```
   symaudit list -sid <SID>
   ```

2. **Filter Audit Log by Time**:
   ```
   symaudit list -sid <SID> -start "MM/DD/YYYY HH:MM:SS" -end "MM/DD/YYYY HH:MM:SS"
   ```

3. **Filter by User**:
   ```
   symaudit list -sid <SID> -user <username>
   ```

4. **Filter by Command**:
   ```
   symaudit list -sid <SID> -command <command_pattern>
   ```

## Event Monitoring

1. **Recent Array Events**:
   ```
   symevent list -sid <SID>
   ```

2. **Filter Events by Severity**:
   ```
   symevent list -sid <SID> -severity <INFO|WARNING|CRITICAL|FATAL>
   ```

3. **Filter Events by Time**:
   ```
   symevent list -sid <SID> -start "MM/DD/YYYY HH:MM:SS" -end "MM/DD/YYYY HH:MM:SS"
   ```

4. **Detailed Event Information**:
   ```
   symevent show <event_id> -sid <SID>
   ```

## Additional Useful Commands

1. **Array Version Information**:
   ```
   symcfg -sid <SID> list -version
   ```

2. **Performance Metrics**:
   ```
   symstat list -sid <SID>
   ```

3. **RDF Status (if configured)**:
   ```
   symrdf list -sid <SID>
   ```

Remember to replace `<SID>` with your actual PowerMax 8000 array serial number. For more detailed information on any command, you can use the `-help` option (e.g., `symcfg -help`).
