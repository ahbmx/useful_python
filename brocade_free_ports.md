To **calculate free ports** on a **Brocade X7-8 Director** (especially in a Virtual Fabrics setup), you’ll want to:

1. **List all physical ports.**
2. **Check how many are licensed.**
3. **Check which are in use.**
4. **Subtract in-use from licensed to get free.**

Here’s how you do it step-by-step:

---

### ✅ **Step-by-Step to Calculate Free Ports**

#### 🔹 1. **List All Licensed Ports**

```bash
licenseportshow
```

* Look for ports that are **Licensed**.
* Count the number of licensed ports per blade/slot.

#### 🔹 2. **Check Port Status**

```bash
switchshow
```

* Ports in **"No\_Light"**, **"Disabled"**, or **"Offline"** but licensed may be considered free (if not assigned or intentionally down).
* Ports in **"Online"** or **"In-Use"** are active.

You can filter with:

```bash
switchshow | grep -i "no_light\|disabled\|offline"
```

#### 🔹 3. **Check Logical Switch Port Assignment (for VF)**

If using Virtual Fabrics:

```bash
setcontext <LS_ID>
switchshow
```

Repeat for each logical switch to track ports allocated per switch.

Or use:

```bash
lscfg --show
```

To see which ports are assigned to which LS.

#### 🔹 4. **Optional – Port Usage in Summary**

Use:

```bash
portshow | grep -i "state"
```

To quickly tally states across all ports.

---

### 🧮 **Formula**

```
Free Ports = Licensed Ports - Ports in Use (Online or Assigned)
```

---

### 🔍 **Example**

Let’s say:

* `licenseportshow` shows 256 licensed ports.
* `switchshow` shows 180 ports are online or assigned to a LS.
* Then:

```bash
Free Ports = 256 - 180 = 76 free ports
```

---

Would you like a shell script to automate this check across all logical switches?
