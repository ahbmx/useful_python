# Python Script for Unisphere for VMAX REST API Integration

Below is a comprehensive Python script that connects to the Unisphere for VMAX REST API, dynamically discovers REST versions and arrays, and collects capacity, health, host, WWN, and storage group information, storing it in pandas DataFrames.

```python
import requests
import pandas as pd
from requests.auth import HTTPBasicAuth
from requests.packages.urllib3.exceptions import InsecureRequestWarning
import warnings

# Suppress SSL warnings (not recommended for production)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
warnings.filterwarnings('ignore')

class UnisphereVMAX:
    def __init__(self, host, username, password, verify_ssl=False):
        """
        Initialize the Unisphere VMAX API client
        
        Args:
            host (str): Unisphere server hostname or IP
            username (str): Unisphere username
            password (str): Unisphere password
            verify_ssl (bool): Whether to verify SSL certificates
        """
        self.host = host
        self.username = username
        self.password = password
        self.verify_ssl = verify_ssl
        self.base_url = f"https://{self.host}/univmax"
        self.session = requests.Session()
        self.session.auth = HTTPBasicAuth(self.username, self.password)
        self.session.verify = self.verify_ssl
        self.rest_versions = []
        self.arrays = []
        
    def discover_rest_versions(self):
        """Discover available REST API versions"""
        try:
            response = self.session.get(f"{self.base_url}/restapi/version")
            response.raise_for_status()
            versions = response.json()['version']
            self.rest_versions = [v for v in versions if v.startswith('9')]  # Filter for VMAX relevant versions
            return self.rest_versions
        except Exception as e:
            print(f"Error discovering REST versions: {str(e)}")
            return []
    
    def discover_arrays(self):
        """Discover available VMAX arrays"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        if not self.rest_versions:
            return []
            
        # Use the latest REST version
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/system/symmetrix"
            )
            response.raise_for_status()
            self.arrays = response.json()['symmetrixId']
            return self.arrays
        except Exception as e:
            print(f"Error discovering arrays: {str(e)}")
            return []
    
    def get_array_capacity(self, array_id):
        """Get capacity information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}"
            )
            response.raise_for_status()
            data = response.json()
            
            # Create DataFrame for capacity data
            capacity_df = pd.DataFrame({
                'ArrayID': [array_id],
                'TotalCapacityGB': [data['system_capacity']['usable_total_tb'] * 1024],
                'UsedCapacityGB': [data['system_capacity']['usable_used_tb'] * 1024],
                'FreeCapacityGB': [data['system_capacity']['usable_free_tb'] * 1024],
                'SubscribedCapacityGB': [data['system_capacity']['subscribed_total_tb'] * 1024]
            })
            return capacity_df
        except Exception as e:
            print(f"Error getting capacity for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_array_health(self, array_id):
        """Get health status for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/system/symmetrix/{array_id}/health"
            )
            response.raise_for_status()
            data = response.json()
            
            # Create DataFrame for health data
            health_df = pd.DataFrame({
                'ArrayID': [array_id],
                'HealthStatus': [data['health']['health_score']['health_score']],
                'HealthDescription': [data['health']['health_score']['description']],
                'NumCriticalAlerts': [len(data['health']['alerts']['critical'])],
                'NumWarningAlerts': [len(data['health']['alerts']['warning'])],
                'NumInfoAlerts': [len(data['health']['alerts']['info'])]
            })
            return health_df
        except Exception as e:
            print(f"Error getting health for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_hosts(self, array_id):
        """Get host information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/host"
            )
            response.raise_for_status()
            hosts = response.json()['hostId']
            
            host_data = []
            for host in hosts:
                host_response = self.session.get(
                    f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/host/{host}"
                )
                host_response.raise_for_status()
                host_info = host_response.json()
                
                # Get host WWNs
                initiators = host_info.get('initiator', [])
                wwns = [i['initiatorId'] for i in initiators if i['initiatorId'].startswith('5')]
                
                host_data.append({
                    'ArrayID': array_id,
                    'HostID': host,
                    'HostType': host_info.get('type', ''),
                    'NumInitiators': len(initiators),
                    'WWNs': ', '.join(wwns),
                    'NumPorts': len(wwns)
                })
            
            hosts_df = pd.DataFrame(host_data)
            return hosts_df
        except Exception as e:
            print(f"Error getting hosts for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_storage_groups(self, array_id):
        """Get storage group information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/storagegroup"
            )
            response.raise_for_status()
            sgs = response.json()['storageGroupId']
            
            sg_data = []
            for sg in sgs:
                sg_response = self.session.get(
                    f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/storagegroup/{sg}"
                )
                sg_response.raise_for_status()
                sg_info = sg_response.json()
                
                sg_data.append({
                    'ArrayID': array_id,
                    'StorageGroup': sg,
                    'Hosts': ', '.join(sg_info.get('host', [])),
                    'NumVolumes': len(sg_info.get('volumeId', [])),
                    'ServiceLevel': sg_info.get('slo', ''),
                    'Workload': sg_info.get('workload', ''),
                    'CapGB': sg_info.get('cap_gb', 0)
                })
            
            sg_df = pd.DataFrame(sg_data)
            return sg_df
        except Exception as e:
            print(f"Error getting storage groups for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def collect_all_data(self):
        """Collect all data for all discovered arrays"""
        if not self.arrays:
            self.discover_arrays()
            
        if not self.arrays:
            return {}
            
        all_data = {
            'capacity': pd.DataFrame(),
            'health': pd.DataFrame(),
            'hosts': pd.DataFrame(),
            'storage_groups': pd.DataFrame()
        }
        
        for array in self.arrays:
            all_data['capacity'] = pd.concat([all_data['capacity'], self.get_array_capacity(array)])
            all_data['health'] = pd.concat([all_data['health'], self.get_array_health(array)])
            all_data['hosts'] = pd.concat([all_data['hosts'], self.get_hosts(array)])
            all_data['storage_groups'] = pd.concat([all_data['storage_groups'], self.get_storage_groups(array)])
        
        return all_data

# Example usage
if __name__ == "__main__":
    # Configuration - replace with your actual credentials
    UNISPHERE_HOST = "unisphere_host"
    USERNAME = "username"
    PASSWORD = "password"
    
    # Create client and collect data
    client = UnisphereVMAX(UNISPHERE_HOST, USERNAME, PASSWORD)
    
    # Discover REST versions and arrays
    print("Discovering REST versions...")
    versions = client.discover_rest_versions()
    print(f"Available REST versions: {versions}")
    
    print("\nDiscovering arrays...")
    arrays = client.discover_arrays()
    print(f"Available arrays: {arrays}")
    
    if arrays:
        print("\nCollecting all data...")
        data = client.collect_all_data()
        
        # Print sample data from each DataFrame
        print("\nCapacity Data:")
        print(data['capacity'].head())
        
        print("\nHealth Data:")
        print(data['health'].head())
        
        print("\nHosts Data:")
        print(data['hosts'].head())
        
        print("\nStorage Groups Data:")
        print(data['storage_groups'].head())
        
        # You can now save these DataFrames to CSV, Excel, or a database
        # Example: data['capacity'].to_csv('vmax_capacity.csv', index=False)
```

## Script Explanation

### 1. Initialization and Setup

- **Class Initialization (`__init__` method)**:
  - Sets up connection parameters (host, username, password)
  - Configures SSL verification (disabled by default for simplicity)
  - Creates a requests Session with basic authentication
  - Initializes lists for REST versions and arrays

### 2. REST API Version Discovery

- **`discover_rest_versions()`**:
  - Queries the Unisphere REST API version endpoint
  - Filters for VMAX-relevant versions (starting with '9')
  - Stores discovered versions for future API calls

### 3. Array Discovery

- **`discover_arrays()`**:
  - Uses the latest discovered REST version
  - Queries the symmetrix endpoint to get all VMAX arrays managed by this Unisphere
  - Stores array IDs for subsequent operations

### 4. Data Collection Methods

#### Capacity Information
- **`get_array_capacity()`**:
  - Retrieves capacity metrics (total, used, free, subscribed) for a specific array
  - Returns data in a pandas DataFrame with columns:
    - ArrayID, TotalCapacityGB, UsedCapacityGB, FreeCapacityGB, SubscribedCapacityGB

#### Health Status
- **`get_array_health()`**:
  - Gets health score and alert information for an array
  - Returns DataFrame with columns:
    - ArrayID, HealthStatus, HealthDescription, NumCriticalAlerts, NumWarningAlerts, NumInfoAlerts

#### Host Information
- **`get_hosts()`**:
  - Retrieves all hosts connected to the array
  - For each host, gathers:
    - Host ID, type, number of initiators, WWNs, and number of ports
  - WWNs are filtered to only include Fibre Channel initiators (starting with '5')

#### Storage Groups
- **`get_storage_groups()`**:
  - Collects information about all storage groups on the array
  - For each storage group, captures:
    - Name, associated hosts, number of volumes, service level, workload, and capacity

### 5. Comprehensive Data Collection

- **`collect_all_data()`**:
  - Orchestrates data collection for all discovered arrays
  - Returns a dictionary of DataFrames for each data type (capacity, health, hosts, storage groups)

### 6. Example Usage

The `if __name__ == "__main__":` block demonstrates how to:
1. Create a client instance
2. Discover REST versions and arrays
3. Collect all data
4. Print sample data from each DataFrame

## Important Notes

1. **Security**: The script disables SSL verification for simplicity. In production, you should properly configure SSL certificates.

2. **Error Handling**: Basic error handling is included, but you may want to enhance it for production use.

3. **Performance**: For environments with many arrays/hosts/storage groups, consider adding pagination or async requests.

4. **Data Storage**: The script shows how to collect data into DataFrames. You can easily extend it to save data to CSV, Excel, or databases.

5. **API Version**: The script automatically uses the latest discovered REST API version for compatibility.



# Enhanced Python Script for Unisphere for VMAX REST API with Pagination and Devices

Here's an improved version of the script that adds pagination support and device information collection:

```python
import requests
import pandas as pd
from requests.auth import HTTPBasicAuth
from requests.packages.urllib3.exceptions import InsecureRequestWarning
import warnings
from typing import Dict, List, Optional

# Suppress SSL warnings (not recommended for production)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
warnings.filterwarnings('ignore')

class UnisphereVMAX:
    def __init__(self, host: str, username: str, password: str, verify_ssl: bool = False):
        """
        Initialize the Unisphere VMAX API client
        
        Args:
            host: Unisphere server hostname or IP
            username: Unisphere username
            password: Unisphere password
            verify_ssl: Whether to verify SSL certificates
        """
        self.host = host
        self.username = username
        self.password = password
        self.verify_ssl = verify_ssl
        self.base_url = f"https://{self.host}/univmax"
        self.session = requests.Session()
        self.session.auth = HTTPBasicAuth(self.username, self.password)
        self.session.verify = self.verify_ssl
        self.rest_versions = []
        self.arrays = []
        self.page_size = 1000  # Default page size for paginated requests
        
    def _get_paginated_results(self, url: str, result_key: str) -> List[Dict]:
        """
        Helper method to handle paginated API responses
        
        Args:
            url: The initial URL to query
            result_key: The key in the JSON response that contains the results
            
        Returns:
            List of all results across all pages
        """
        results = []
        next_page_url = url
        
        while next_page_url:
            try:
                response = self.session.get(next_page_url)
                response.raise_for_status()
                data = response.json()
                
                if result_key in data:
                    results.extend(data[result_key])
                
                # Check for pagination links
                links = data.get('links', [])
                next_link = next((link for link in links if link['rel'] == 'next'), None)
                next_page_url = next_link['href'] if next_link else None
                
            except Exception as e:
                print(f"Error in paginated request: {str(e)}")
                break
        
        return results
    
    def discover_rest_versions(self) -> List[str]:
        """Discover available REST API versions"""
        try:
            response = self.session.get(f"{self.base_url}/restapi/version")
            response.raise_for_status()
            versions = response.json()['version']
            self.rest_versions = [v for v in versions if v.startswith('9')]  # Filter for VMAX relevant versions
            return self.rest_versions
        except Exception as e:
            print(f"Error discovering REST versions: {str(e)}")
            return []
    
    def discover_arrays(self) -> List[str]:
        """Discover available VMAX arrays"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        if not self.rest_versions:
            return []
            
        # Use the latest REST version
        latest_version = sorted(self.rest_versions)[-1]
        try:
            url = f"{self.base_url}/restapi/{latest_version}/system/symmetrix?page_size={self.page_size}"
            self.arrays = self._get_paginated_results(url, 'symmetrixId')
            return self.arrays
        except Exception as e:
            print(f"Error discovering arrays: {str(e)}")
            return []
    
    def get_array_capacity(self, array_id: str) -> pd.DataFrame:
        """Get capacity information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}"
            )
            response.raise_for_status()
            data = response.json()
            
            # Create DataFrame for capacity data
            capacity_df = pd.DataFrame({
                'ArrayID': [array_id],
                'TotalCapacityGB': [data['system_capacity']['usable_total_tb'] * 1024],
                'UsedCapacityGB': [data['system_capacity']['usable_used_tb'] * 1024],
                'FreeCapacityGB': [data['system_capacity']['usable_free_tb'] * 1024],
                'SubscribedCapacityGB': [data['system_capacity']['subscribed_total_tb'] * 1024],
                'PhysicalCapacityGB': [data['system_capacity']['physical_total_tb'] * 1024]
            })
            return capacity_df
        except Exception as e:
            print(f"Error getting capacity for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_array_health(self, array_id: str) -> pd.DataFrame:
        """Get health status for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            response = self.session.get(
                f"{self.base_url}/restapi/{latest_version}/system/symmetrix/{array_id}/health"
            )
            response.raise_for_status()
            data = response.json()
            
            # Create DataFrame for health data
            health_df = pd.DataFrame({
                'ArrayID': [array_id],
                'HealthStatus': [data['health']['health_score']['health_score']],
                'HealthDescription': [data['health']['health_score']['description']],
                'NumCriticalAlerts': [len(data['health']['alerts']['critical'])],
                'NumWarningAlerts': [len(data['health']['alerts']['warning'])],
                'NumInfoAlerts': [len(data['health']['alerts']['info'])]
            })
            return health_df
        except Exception as e:
            print(f"Error getting health for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_hosts(self, array_id: str) -> pd.DataFrame:
        """Get host information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            url = f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/host?page_size={self.page_size}"
            hosts = self._get_paginated_results(url, 'hostId')
            
            host_data = []
            for host in hosts:
                host_response = self.session.get(
                    f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/host/{host}"
                )
                host_response.raise_for_status()
                host_info = host_response.json()
                
                # Get host WWNs
                initiators = host_info.get('initiator', [])
                wwns = [i['initiatorId'] for i in initiators if i['initiatorId'].startswith('5')]
                
                host_data.append({
                    'ArrayID': array_id,
                    'HostID': host,
                    'HostType': host_info.get('type', ''),
                    'NumInitiators': len(initiators),
                    'WWNs': ', '.join(wwns),
                    'NumPorts': len(wwns),
                    'HostFlags': str(host_info.get('host_flags', {}))
                })
            
            hosts_df = pd.DataFrame(host_data)
            return hosts_df
        except Exception as e:
            print(f"Error getting hosts for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_storage_groups(self, array_id: str) -> pd.DataFrame:
        """Get storage group information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            url = f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/storagegroup?page_size={self.page_size}"
            sgs = self._get_paginated_results(url, 'storageGroupId')
            
            sg_data = []
            for sg in sgs:
                sg_response = self.session.get(
                    f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/storagegroup/{sg}"
                )
                sg_response.raise_for_status()
                sg_info = sg_response.json()
                
                sg_data.append({
                    'ArrayID': array_id,
                    'StorageGroup': sg,
                    'Hosts': ', '.join(sg_info.get('host', [])),
                    'NumVolumes': len(sg_info.get('volumeId', [])),
                    'ServiceLevel': sg_info.get('slo', ''),
                    'Workload': sg_info.get('workload', ''),
                    'CapGB': sg_info.get('cap_gb', 0),
                    'SRP': sg_info.get('srp', ''),
                    'ChildSGs': ', '.join(sg_info.get('child_storage_group', []))
                })
            
            sg_df = pd.DataFrame(sg_data)
            return sg_df
        except Exception as e:
            print(f"Error getting storage groups for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_devices(self, array_id: str) -> pd.DataFrame:
        """Get device information for a specific array"""
        if not self.rest_versions:
            self.discover_rest_versions()
            
        latest_version = sorted(self.rest_versions)[-1]
        try:
            url = f"{self.base_url}/restapi/{latest_version}/sloprovisioning/symmetrix/{array_id}/volume?page_size={self.page_size}"
            devices = self._get_paginated_results(url, 'resultList')
            
            device_data = []
            for device in devices:
                device_data.append({
                    'ArrayID': array_id,
                    'DeviceID': device['volumeId'],
                    'CapacityGB': device['cap_gb'],
                    'Status': device['status'],
                    'Type': device['type'],
                    'StorageGroup': device.get('storageGroupId', ''),
                    'WWN': device.get('wwn', ''),
                    'AllocatedPercent': device.get('allocated_percent', 0),
                    'Emulation': device.get('emulation', ''),
                    'RDFGroupID': device.get('rdfGroupId', '')
                })
            
            devices_df = pd.DataFrame(device_data)
            return devices_df
        except Exception as e:
            print(f"Error getting devices for array {array_id}: {str(e)}")
            return pd.DataFrame()
    
    def collect_all_data(self) -> Dict[str, pd.DataFrame]:
        """Collect all data for all discovered arrays"""
        if not self.arrays:
            self.discover_arrays()
            
        if not self.arrays:
            return {}
            
        all_data = {
            'capacity': pd.DataFrame(),
            'health': pd.DataFrame(),
            'hosts': pd.DataFrame(),
            'storage_groups': pd.DataFrame(),
            'devices': pd.DataFrame()
        }
        
        for array in self.arrays:
            print(f"Collecting data for array {array}...")
            all_data['capacity'] = pd.concat([all_data['capacity'], self.get_array_capacity(array)])
            all_data['health'] = pd.concat([all_data['health'], self.get_array_health(array)])
            all_data['hosts'] = pd.concat([all_data['hosts'], self.get_hosts(array)])
            all_data['storage_groups'] = pd.concat([all_data['storage_groups'], self.get_storage_groups(array)])
            all_data['devices'] = pd.concat([all_data['devices'], self.get_devices(array)])
        
        return all_data

# Example usage
if __name__ == "__main__":
    # Configuration - replace with your actual credentials
    UNISPHERE_HOST = "unisphere_host"
    USERNAME = "username"
    PASSWORD = "password"
    
    # Create client and collect data
    client = UnisphereVMAX(UNISPHERE_HOST, USERNAME, PASSWORD)
    
    # Discover REST versions and arrays
    print("Discovering REST versions...")
    versions = client.discover_rest_versions()
    print(f"Available REST versions: {versions}")
    
    print("\nDiscovering arrays...")
    arrays = client.discover_arrays()
    print(f"Available arrays: {arrays}")
    
    if arrays:
        print("\nCollecting all data...")
        data = client.collect_all_data()
        
        # Print sample data from each DataFrame
        print("\nCapacity Data:")
        print(data['capacity'].head())
        
        print("\nHealth Data:")
        print(data['health'].head())
        
        print("\nHosts Data:")
        print(data['hosts'].head())
        
        print("\nStorage Groups Data:")
        print(data['storage_groups'].head())
        
        print("\nDevices Data:")
        print(data['devices'].head())
        
        # Save data to CSV files
        data['capacity'].to_csv('vmax_capacity.csv', index=False)
        data['health'].to_csv('vmax_health.csv', index=False)
        data['hosts'].to_csv('vmax_hosts.csv', index=False)
        data['storage_groups'].to_csv('vmax_storage_groups.csv', index=False)
        data['devices'].to_csv('vmax_devices.csv', index=False)
        print("\nData saved to CSV files.")
```

## Key Enhancements

### 1. Pagination Support

Added a new helper method `_get_paginated_results()` that:
- Takes a URL and result key as input
- Follows pagination links until all results are collected
- Handles the 'links' section in API responses to find the 'next' page
- Uses a configurable page size (default 1000 items per page)

This method is now used for:
- Array discovery
- Host listing
- Storage group listing
- Device listing

### 2. Device Information Collection

Added a new method `get_devices()` that:
- Retrieves all volumes/devices for a given array
- Collects key device information including:
  - Device ID
  - Capacity
  - Status
  - Type
  - Associated storage group
  - WWN
  - Allocation percentage
  - Emulation type
  - RDF group (for replicated devices)

### 3. Additional Improvements

1. **Type Hints**: Added Python type hints for better code clarity and IDE support
2. **Enhanced Storage Group Data**: Added SRP and child storage group information
3. **Enhanced Host Data**: Added host flags information
4. **Capacity Metrics**: Added physical capacity to capacity metrics
5. **Progress Feedback**: Added print statements during data collection
6. **CSV Export**: Added automatic CSV export for all data types

### 4. Data Structure

The collected data is now organized in a dictionary with these DataFrames:
- `capacity`: Array-level capacity metrics
- `health`: Array health status
- `hosts`: Connected host information
- `storage_groups`: Storage group configurations
- `devices`: Volume/device details

### Usage Notes

1. For large environments, you may need to adjust the `page_size` parameter
2. The script now handles pagination automatically for all list operations
3. Device collection provides detailed storage volume information
4. All data is exported to CSV files for easy analysis

This enhanced version provides complete visibility into your VMAX storage environment with proper handling of large datasets through pagination.

