Sure! Below is a step-by-step explanation and the corresponding Python script that:

1. Defines a class to connect to the **Broadcom SANnav REST API**.
2. Retrieves various information such as capacity, health, hosts, WWNs, port info, and utilization.
3. Stores each data type in separate **pandas DataFrames**.

---

### 📌 Prerequisites

Install required packages (if not already installed):

```bash
pip install requests pandas
```

---

### ✅ Step-by-Step Plan

1. **Import required modules.**
2. **Create a class `SANnavClient`:**

   * `__init__`: Store API URL and credentials.
   * `login`: Authenticate and store the session token.
   * Methods like `get_capacity`, `get_health`, `get_hosts`, etc., to fetch relevant data.
3. **Store each response into a pandas DataFrame.**
4. **Main function** to run the client and display or manipulate the data.

---

### 🧠 Assumptions

* SANnav is accessible via HTTPS.
* API responses are in JSON format.
* Endpoints like `/capacity`, `/health`, etc., are illustrative; you may need to adapt to actual SANnav API routes.

---

### 🧾 Python Script

```python
import requests
import pandas as pd

class SANnavClient:
    def __init__(self, base_url, username, password, verify_ssl=False):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        self.username = username
        self.password = password
        self.verify_ssl = verify_ssl
        self.token = None

    def login(self):
        url = f"{self.base_url}/rest/login"
        payload = {
            "userName": self.username,
            "password": self.password
        }
        response = self.session.post(url, json=payload, verify=self.verify_ssl)
        response.raise_for_status()
        self.token = response.json().get('Authorization')
        self.session.headers.update({'Authorization': self.token})

    def get_data(self, endpoint):
        url = f"{self.base_url}/rest/{endpoint}"
        response = self.session.get(url, verify=self.verify_ssl)
        response.raise_for_status()
        return response.json()

    def get_capacity(self):
        return pd.DataFrame(self.get_data("capacity"))

    def get_health(self):
        return pd.DataFrame(self.get_data("health"))

    def get_hosts(self):
        return pd.DataFrame(self.get_data("hosts"))

    def get_wwns(self):
        return pd.DataFrame(self.get_data("wwns"))

    def get_ports(self):
        return pd.DataFrame(self.get_data("ports"))

    def get_utilization(self):
        return pd.DataFrame(self.get_data("utilization"))

def main():
    # Update with actual values
    scli = SANnavClient(
        base_url="https://sannav.example.com",
        username="admin",
        password="password",
        verify_ssl=False  # Set to True with proper certs in production
    )

    try:
        scli.login()
        print("Logged in successfully.")

        # Fetch data
        capacity_df = scli.get_capacity()
        health_df = scli.get_health()
        hosts_df = scli.get_hosts()
        wwns_df = scli.get_wwns()
        ports_df = scli.get_ports()
        utilization_df = scli.get_utilization()

        # Display sample data
        print("Capacity:\n", capacity_df.head())
        print("Health:\n", health_df.head())
        print("Hosts:\n", hosts_df.head())
        print("WWNs:\n", wwns_df.head())
        print("Ports:\n", ports_df.head())
        print("Utilization:\n", utilization_df.head())

    except requests.HTTPError as e:
        print(f"HTTP Error: {e}")
    except Exception as e:
        print(f"Error: {e}")

if __name__ == "__main__":
    main()
```

---

### 🧾 Explanation

* **`SANnavClient` class**: Manages connection and API interactions.
* **Authentication**: `login()` sends username/password, stores the `Authorization` token.
* **Generic method** `get_data(endpoint)`: Reduces redundancy.
* **Each API call** (`get_capacity`, `get_health`, etc.) returns a DataFrame.
* **Main script**:

  * Logs in.
  * Calls each method.
  * Displays sample data from the DataFrames.

---

### 📎 Notes

* Replace the placeholder endpoints (`capacity`, `health`, etc.) with actual SANnav REST API paths.
* Use `print(df.to_csv())` or `df.to_excel()` to save outputs.
* This is a skeleton to be expanded based on the actual API documentation and data structure.

Would you like me to tailor this further to your SANnav version or API documentation?
