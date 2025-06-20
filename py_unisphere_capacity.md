# Enhanced Unisphere VMAX Capacity Data Extraction with API Version Check and Pandas Conversion

Here's a complete Python solution that:
1. Automatically checks for the available API versions
2. Uses the current API version
3. Retrieves capacity metrics for the last year
4. Converts the JSON response to a pandas DataFrame
5. Includes data cleaning and transformation

```python
import requests
import pandas as pd
from datetime import datetime, timedelta
import warnings
from urllib3.exceptions import InsecureRequestWarning

# Suppress SSL warnings (only for testing with self-signed certs)
warnings.filterwarnings('ignore', category=InsecureRequestWarning)

class UnisphereVMAXClient:
    def __init__(self, host, username, password, verify_ssl=False):
        self.base_url = f"https://{host}/univmax/restapi"
        self.username = username
        self.password = password
        self.verify_ssl = verify_ssl
        self.session_token = None
        self.api_version = None
        
    def _get_api_version(self):
        """Discover and set the current API version"""
        try:
            response = requests.get(
                f"{self.base_url}/version",
                auth=(self.username, self.password),
                verify=self.verify_ssl
            )
            response.raise_for_status()
            
            versions = response.json()
            current_version = next(
                (v['version'] for v in versions['version'] if v['isCurrentVersion']),
                None
            )
            
            if not current_version:
                raise ValueError("No current API version found")
                
            self.api_version = current_version
            return current_version
            
        except Exception as e:
            raise ConnectionError(f"Failed to get API version: {str(e)}")
    
    def _authenticate(self):
        """Establish an authenticated session"""
        if not self.api_version:
            self._get_api_version()
            
        auth_url = f"{self.base_url}/{self.api_version}/security/session"
        headers = {
            "Content-Type": "application/json",
            "Accept": "application/json",
            "X-EMC-REST-CLIENT": "true"
        }
        auth_data = {"username": self.username, "password": self.password}
        
        try:
            response = requests.post(
                auth_url,
                headers=headers,
                json=auth_data,
                verify=self.verify_ssl
            )
            response.raise_for_status()
            self.session_token = response.headers.get("EMCSESSION")
            return True
        except Exception as e:
            raise ConnectionError(f"Authentication failed: {str(e)}")
    
    def _logout(self):
        """Terminate the session"""
        if self.session_token:
            logout_url = f"{self.base_url}/{self.api_version}/security/session"
            headers = {
                "Accept": "application/json",
                "EMCSESSION": self.session_token,
                "X-EMC-REST-CLIENT": "true"
            }
            requests.delete(logout_url, headers=headers, verify=self.verify_ssl)
            self.session_token = None
    
    def get_capacity_metrics(self, symmetrix_id, days=365):
        """
        Get capacity metrics for specified time period and return as DataFrame
        Args:
            symmetrix_id: The VMAX array ID
            days: Number of days of history to retrieve (default 365)
        Returns:
            pandas.DataFrame containing capacity metrics
        """
        if not self.session_token:
            self._authenticate()
            
        # Calculate date range
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days)
        
        # Get capacity data
        capacity_url = (
            f"{self.base_url}/{self.api_version}/sloprovisioning/symmetrix/"
            f"{symmetrix_id}/performance/metrics"
        )
        
        params = {
            "startDate": start_date.strftime("%Y-%m-%d"),
            "endDate": end_date.strftime("%Y-%m-%d"),
            "metrics": "HostMBs,AllocatedCapacity,EffectiveUsedCapacity,SubscribedCapacity",
            "dataFormat": "Average"
        }
        
        headers = {
            "Accept": "application/json",
            "EMCSESSION": self.session_token,
            "X-EMC-REST-CLIENT": "true"
        }
        
        try:
            response = requests.get(
                capacity_url,
                headers=headers,
                params=params,
                verify=self.verify_ssl
            )
            response.raise_for_status()
            capacity_data = response.json()
            
            # Convert to DataFrame and clean data
            df = self._process_capacity_data(capacity_data)
            return df
            
        except Exception as e:
            raise RuntimeError(f"Failed to get capacity metrics: {str(e)}")
        finally:
            self._logout()
    
    def _process_capacity_data(self, capacity_data):
        """Convert JSON response to cleaned DataFrame"""
        # Extract the relevant metrics data
        metrics = capacity_data.get('resultList', {}).get('result', [])
        
        if not metrics:
            raise ValueError("No capacity metrics found in response")
        
        # Create DataFrame
        df = pd.json_normalize(metrics)
        
        # Convert timestamps to datetime
        if 'timestamp' in df.columns:
            df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
            df.set_index('timestamp', inplace=True)
        
        # Convert bytes to TB (assuming metrics are in MB)
        byte_columns = [col for col in df.columns if 'MB' in col or 'Capacity' in col]
        for col in byte_columns:
            df[col] = df[col] / (1024 * 1024)  # Convert MB to TB
        
        # Clean column names
        df.columns = df.columns.str.replace('.', '_').str.lower()
        
        return df

# Example usage
if __name__ == "__main__":
    # Configuration
    HOST = "your_unisphere_host"
    USERNAME = "your_username"
    PASSWORD = "your_password"
    SYMMETRIX_ID = "your_array_id"
    
    try:
        # Create client and get data
        client = UnisphereVMAXClient(HOST, USERNAME, PASSWORD, verify_ssl=False)
        capacity_df = client.get_capacity_metrics(SYMMETRIX_ID, days=365)
        
        # Display and analyze the data
        print("\nCapacity Metrics (TB):")
        print(capacity_df.head())
        
        # Optional: Save to CSV
        capacity_df.to_csv("vmax_capacity_metrics.csv")
        print("\nData saved to vmax_capacity_metrics.csv")
        
    except Exception as e:
        print(f"Error: {str(e)}")
```

## Key Features of This Solution:

1. **Automatic API Version Detection**:
   - The client automatically discovers and uses the current API version
   - No need to manually specify version numbers

2. **Comprehensive Data Processing**:
   - Converts timestamps to proper datetime objects
   - Transforms capacity metrics from MB to TB (adjustable)
   - Cleans column names for easier analysis

3. **Error Handling**:
   - Robust error handling for API requests
   - Automatic session cleanup

4. **Pandas Integration**:
   - Returns a properly formatted DataFrame
   - Time-based indexing for easy time series analysis
   - Includes example of saving to CSV

5. **Security**:
   - Proper session management (login/logout)
   - Configurable SSL verification

## Customization Options:

1. To change the capacity units (e.g., GB instead of TB), modify the conversion factor in `_process_capacity_data()`
2. To add more metrics, extend the `metrics` parameter in the `get_capacity_metrics()` method
3. For different time periods, adjust the `days` parameter when calling the method

Would you like me to add any specific additional analysis capabilities to the DataFrame output?
