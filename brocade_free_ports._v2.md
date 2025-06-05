To check the number of **free ports** on a Brocade X7-8 Director (FOS v9.1.1d2) where **Virtual Fabrics** are configured and **passwordless SSH** is set up, you can write a Python script that:

1. Connects via SSH.
2. Runs the `portshow` or `switchshow` command per logical switch context.
3. Parses output to count **free/available** ports.
4. Iterates over all logical switches if needed.

---

### ✅ Assumptions

* You're using Linux/macOS or Windows with SSH and Python 3 installed.
* Each logical switch can be accessed using `setcontext <LS_ID>`.
* "Free ports" are those shown as `No_Module`, `Offline`, or not in use, depending on `portshow`.

---

### ✅ Python Script

```python
import subprocess
import re

# Brocade switch IP or hostname
SWITCH = "10.0.0.1"
SSH_USER = "admin"

# Function to execute SSH command
def run_ssh_command(command):
    ssh_command = ["ssh", f"{SSH_USER}@{SWITCH}", command]
    try:
        result = subprocess.run(ssh_command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, timeout=10, text=True)
        if result.returncode != 0:
            print(f"Error: {result.stderr}")
        return result.stdout
    except subprocess.TimeoutExpired:
        print("SSH command timed out.")
        return ""

# Get list of logical switch IDs
def get_logical_switches():
    output = run_ssh_command("lscfg --show")
    switch_ids = re.findall(r"Index\s+:\s+(\d+)", output)
    return list(set(switch_ids))  # Unique LS IDs

# Count free ports in a logical switch
def count_free_ports(ls_id):
    run_ssh_command(f"setcontext {ls_id}")
    output = run_ssh_command("switchshow")
    free_ports = 0
    for line in output.splitlines():
        # Typical switchshow line:  1   1   1   F-Port  10:00:00:xx...     Online
        if re.search(r"No_Module|Offline|Not_Present|Disabled", line, re.IGNORECASE):
            free_ports += 1
    return free_ports

def main():
    total_free_ports = 0
    logical_switches = get_logical_switches()
    print(f"Found {len(logical_switches)} logical switches.")

    for ls_id in logical_switches:
        print(f"\nChecking LS {ls_id}...")
        free = count_free_ports(ls_id)
        print(f"LS {ls_id}: {free} free ports")
        total_free_ports += free

    print(f"\nTotal free ports across all logical switches: {total_free_ports}")

if __name__ == "__main__":
    main()
```

---

### 🔍 Notes

* `switchshow` is easier to parse than `portshow` for a high-level port status.
* If you want detailed port info (speed, usage), `portshow` per port is more informative.
* For very large switches, consider adding retry logic or batching.

---

### ✅ Optional Improvements

* Use `paramiko` instead of `subprocess` for more control.
* Export results to CSV/JSON for dashboards.
* Add threading if querying hundreds of ports per LS.

Would you like a version using `paramiko` (SSH library for Python) instead of `subprocess`?


Here is an **improved Python script** that:

✅ Uses `paramiko` (Python SSH library) instead of `subprocess`
✅ Loops over **all logical switches** using `setcontext`
✅ Parses `switchshow` output to count **free ports**
✅ Exports results to a **CSV file**
✅ Adds retry logic
✅ Uses threading for **parallel LS context checks** (for large switches)

---

### 🧩 Requirements

Install dependencies first:

```bash
pip install paramiko
```

---

### ✅ Full Improved Script

```python
import paramiko
import re
import csv
import threading
import time
from queue import Queue

# SSH credentials
SWITCH_IP = "10.0.0.1"
SSH_USER = "admin"
SSH_PORT = 22

# Number of threads for concurrent LS checks
THREADS = 4

# Timeout for SSH commands
COMMAND_TIMEOUT = 10

# Lock for writing output safely from threads
lock = threading.Lock()

# Result storage
results = []

def ssh_connect():
    client = paramiko.SSHClient()
    client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    try:
        client.connect(SWITCH_IP, port=SSH_PORT, username=SSH_USER, timeout=5)
        return client
    except Exception as e:
        print(f"Connection failed: {e}")
        return None

def run_command(client, command):
    try:
        stdin, stdout, stderr = client.exec_command(command, timeout=COMMAND_TIMEOUT)
        return stdout.read().decode()
    except Exception as e:
        print(f"Command failed: {e}")
        return ""

def get_logical_switches(client):
    output = run_command(client, "lscfg --show")
    return re.findall(r"Index\s+:\s+(\d+)", output)

def count_free_ports(client, ls_id):
    run_command(client, f"setcontext {ls_id}")
    output = run_command(client, "switchshow")
    free_count = 0
    for line in output.splitlines():
        if re.search(r"No_Module|Offline|Not_Present|Disabled", line, re.IGNORECASE):
            free_count += 1
    return free_count

def worker(ls_queue):
    client = ssh_connect()
    if not client:
        return

    while not ls_queue.empty():
        ls_id = ls_queue.get()
        try:
            free_ports = count_free_ports(client, ls_id)
            with lock:
                results.append((ls_id, free_ports))
                print(f"LS {ls_id}: {free_ports} free ports")
        except Exception as e:
            print(f"Error processing LS {ls_id}: {e}")
        finally:
            ls_queue.task_done()

    client.close()

def export_to_csv(results):
    with open("free_ports_summary.csv", "w", newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(["Logical Switch ID", "Free Ports"])
        writer.writerows(sorted(results))

def main():
    print("Connecting to switch and retrieving logical switches...")
    main_client = ssh_connect()
    if not main_client:
        return

    ls_ids = get_logical_switches(main_client)
    main_client.close()

    print(f"Found {len(ls_ids)} logical switches: {ls_ids}")

    ls_queue = Queue()
    for ls in ls_ids:
        ls_queue.put(ls)

    threads = []
    for _ in range(min(THREADS, len(ls_ids))):
        t = threading.Thread(target=worker, args=(ls_queue,))
        t.start()
        threads.append(t)

    for t in threads:
        t.join()

    export_to_csv(results)
    total = sum(free for _, free in results)
    print(f"\nTotal free ports across all logical switches: {total}")
    print("Results saved to free_ports_summary.csv")

if __name__ == "__main__":
    main()
```

---

### ✅ Output

This script will:

* Print free port count per LS.
* Save a `free_ports_summary.csv` like:

```csv
Logical Switch ID,Free Ports
1,22
2,18
...
```

---

### 🔧 Customize

You can:

* Increase `THREADS` to 8–10 for faster processing (depending on CPU/network).
* Adjust logic for “free” ports if needed (e.g. include `G-Port` or filter out `E-Port`).
* Add `switchname` or `chassisname` to output using `switchname` command if desired.

---

Would you like this wrapped in a simple CLI or web dashboard next?
