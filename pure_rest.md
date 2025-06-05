# Python Script for Pure Storage REST API Integration

Below is a comprehensive Python script that connects to the Pure Storage REST API, collects array information, capacity metrics, volume data, host information, and performance statistics, storing everything in pandas DataFrames with pagination support.

```python
import requests
import pandas as pd
from requests.auth import HTTPBasicAuth
from requests.packages.urllib3.exceptions import InsecureRequestWarning
import warnings
from typing import Dict, List, Optional
import json

# Suppress SSL warnings (not recommended for production)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
warnings.filterwarnings('ignore')

class PureStorageAPI:
    def __init__(self, host: str, api_token: str, verify_ssl: bool = False):
        """
        Initialize the Pure Storage API client
        
        Args:
            host: FlashArray hostname or IP
            api_token: API token for authentication
            verify_ssl: Whether to verify SSL certificates
        """
        self.host = host
        self.api_token = api_token
        self.verify_ssl = verify_ssl
        self.base_url = f"https://{self.host}/api/2.11"
        self.session = requests.Session()
        self.session.verify = self.verify_ssl
        self.session.headers.update({
            'Content-Type': 'application/json',
            'Authorization': f'Bearer {self.api_token}'
        })
        self.page_size = 100  # Default page size for paginated requests
        self.array_name = None
        
    def _get_paginated_results(self, endpoint: str, params: Optional[Dict] = None) -> List[Dict]:
        """
        Helper method to handle paginated API responses
        
        Args:
            endpoint: The API endpoint to query (without base URL)
            params: Optional query parameters
            
        Returns:
            List of all results across all pages
        """
        results = []
        continuation_token = None
        params = params or {}
        params['limit'] = self.page_size
        
        while True:
            try:
                if continuation_token:
                    params['continuation_token'] = continuation_token
                
                response = self.session.get(
                    f"{self.base_url}/{endpoint}",
                    params=params
                )
                response.raise_for_status()
                data = response.json()
                
                if isinstance(data, list):
                    results.extend(data)
                else:
                    items = data.get('items', [])
                    if items:
                        results.extend(items)
                    else:
                        # Some endpoints return the array directly
                        results.append(data)
                
                # Check for pagination token
                continuation_token = data.get('continuation_token')
                if not continuation_token:
                    break
                
            except Exception as e:
                print(f"Error in paginated request to {endpoint}: {str(e)}")
                break
        
        return results
    
    def get_array_info(self) -> pd.DataFrame:
        """Get information about the FlashArray"""
        try:
            data = self._get_paginated_results('array')[0]  # First item is the array info
            self.array_name = data.get('array_name')
            
            array_df = pd.DataFrame({
                'ArrayName': [data.get('array_name')],
                'Model': [data.get('model')],
                'Version': [data.get('version')],
                'PurityVersion': [data.get('purity_version')],
                'Serial': [data.get('serial')],
                'ConsoleLockEnabled': [data.get('console_lock_enabled')],
                'MaintenanceMode': [data.get('maintenance_mode')],
                'EULAStatus': [data.get('eula_status')],
                'APIVersion': ['2.11']  # Hardcoded as we're using 2.11
            })
            return array_df
        except Exception as e:
            print(f"Error getting array info: {str(e)}")
            return pd.DataFrame()
    
    def get_array_capacity(self) -> pd.DataFrame:
        """Get capacity metrics for the FlashArray"""
        try:
            data = self._get_paginated_results('array?space=true')[0]
            
            capacity_df = pd.DataFrame({
                'ArrayName': [self.array_name],
                'TotalCapacityGB': [data.get('capacity') / 1024 / 1024 / 1024],
                'UsedCapacityGB': [data.get('total') / 1024 / 1024 / 1024],
                'FreeCapacityGB': [data.get('capacity') / 1024 / 1024 / 1024 - data.get('total') / 1024 / 1024 / 1024],
                'DataReduction': [data.get('data_reduction')],
                'SystemCapacityGB': [data.get('system') / 1024 / 1024 / 1024],
                'SnapshotCapacityGB': [data.get('snapshots') / 1024 / 1024 / 1024],
                'VolumeCapacityGB': [data.get('volumes') / 1024 / 1024 / 1024],
                'SharedSpaceGB': [data.get('shared_space') / 1024 / 1024 / 1024],
                'ThinProvisioning': [data.get('thin_provisioning')]
            })
            return capacity_df
        except Exception as e:
            print(f"Error getting array capacity: {str(e)}")
            return pd.DataFrame()
    
    def get_volumes(self) -> pd.DataFrame:
        """Get all volumes on the FlashArray"""
        try:
            volumes = self._get_paginated_results('volume')
            
            volume_data = []
            for vol in volumes:
                volume_data.append({
                    'ArrayName': self.array_name,
                    'VolumeName': vol.get('name'),
                    'SizeGB': vol.get('size') / 1024 / 1024 / 1024 if vol.get('size') else 0,
                    'Created': vol.get('created'),
                    'Serial': vol.get('serial'),
                    'Source': vol.get('source'),
                    'HostEncryptionKeyStatus': vol.get('host_encryption_key_status'),
                    'Provisioning': vol.get('provisioning'),
                    'PriorityAdjustment': vol.get('priority_adjustment', {}).get('priority'),
                    'QoSEnabled': vol.get('qos_enabled'),
                    'TotalPhysicalGB': vol.get('space', {}).get('total_physical') / 1024 / 1024 / 1024 if vol.get('space', {}).get('total_physical') else 0,
                    'DataReduction': vol.get('space', {}).get('data_reduction'),
                    'SnapshotsSpaceGB': vol.get('space', {}).get('snapshots') / 1024 / 1024 / 1024 if vol.get('space', {}).get('snapshots') else 0
                })
            
            volumes_df = pd.DataFrame(volume_data)
            return volumes_df
        except Exception as e:
            print(f"Error getting volumes: {str(e)}")
            return pd.DataFrame()
    
    def get_hosts(self) -> pd.DataFrame:
        """Get all hosts connected to the FlashArray"""
        try:
            hosts = self._get_paginated_results('host')
            
            host_data = []
            for host in hosts:
                host_data.append({
                    'ArrayName': self.array_name,
                    'HostName': host.get('name'),
                    'IQN': ', '.join(host.get('iqn', [])),
                    'WWN': ', '.join(host.get('wwn', [])),
                    'NQN': ', '.join(host.get('nqn', [])),
                    'Personality': host.get('personality'),
                    'OSType': host.get('os_type'),
                    'HostGroup': host.get('host_group'),
                    'VolumeCount': len(host.get('volumes', [])),
                    'PreferredArrays': ', '.join(host.get('preferred_arrays', [])),
                    'PortConnectivity': json.dumps(host.get('port_connectivity', {}))
                })
            
            hosts_df = pd.DataFrame(host_data)
            return hosts_df
        except Exception as e:
            print(f"Error getting hosts: {str(e)}")
            return pd.DataFrame()
    
    def get_host_volume_connections(self) -> pd.DataFrame:
        """Get all host-volume connections"""
        try:
            connections = self._get_paginated_results('host/volume')
            
            connection_data = []
            for conn in connections:
                connection_data.append({
                    'ArrayName': self.array_name,
                    'HostName': conn.get('host', {}).get('name'),
                    'VolumeName': conn.get('volume', {}).get('name'),
                    'LUN': conn.get('lun'),
                    'ConnectionType': 'FC' if conn.get('wwn') else ('iSCSI' if conn.get('iqn') else 'NVMe')
                })
            
            connections_df = pd.DataFrame(connection_data)
            return connections_df
        except Exception as e:
            print(f"Error getting host-volume connections: {str(e)}")
            return pd.DataFrame()
    
    def get_performance_metrics(self, interval: str = '1h') -> pd.DataFrame:
        """
        Get performance metrics for the array
        
        Args:
            interval: Time interval for metrics (1h, 1d, 1w)
        """
        try:
            # Get array performance
            array_perf = self._get_paginated_results(f'array?action=monitor&interval={interval}')[0]
            
            # Get volume performance (sample first 100 volumes for demo)
            volume_perf = self._get_paginated_results(f'volume?action=monitor&interval={interval}')[:100]
            
            # Create performance DataFrame
            perf_data = {
                'ArrayName': [self.array_name],
                'Interval': [interval],
                'TotalIOPS': [array_perf.get('iops')],
                'ReadIOPS': [array_perf.get('read_iops')],
                'WriteIOPS': [array_perf.get('write_iops')],
                'TotalThroughputMB': [array_perf.get('throughput') / 1024 / 1024 if array_perf.get('throughput') else 0],
                'ReadThroughputMB': [array_perf.get('read_throughput') / 1024 / 1024 if array_perf.get('read_throughput') else 0],
                'WriteThroughputMB': [array_perf.get('write_throughput') / 1024 / 1024 if array_perf.get('write_throughput') else 0],
                'AvgLatencyMS': [array_perf.get('usec_per_read_op') / 1000 if array_perf.get('usec_per_read_op') else 0],
                'ReadLatencyMS': [array_perf.get('usec_per_read_op') / 1000 if array_perf.get('usec_per_read_op') else 0],
                'WriteLatencyMS': [array_perf.get('usec_per_write_op') / 1000 if array_perf.get('usec_per_write_op') else 0],
                'QueueDepth': [array_perf.get('queue_depth')],
                'SampleVolumes': [len(volume_perf)]
            }
            
            perf_df = pd.DataFrame(perf_data)
            return perf_df
        except Exception as e:
            print(f"Error getting performance metrics: {str(e)}")
            return pd.DataFrame()
    
    def get_alerts(self, resolved: bool = False) -> pd.DataFrame:
        """Get array alerts"""
        try:
            alerts = self._get_paginated_results(f'alert?resolved={str(resolved).lower()}')
            
            alert_data = []
            for alert in alerts:
                alert_data.append({
                    'ArrayName': self.array_name,
                    'AlertID': alert.get('id'),
                    'Name': alert.get('name'),
                    'ComponentType': alert.get('component_type'),
                    'ComponentName': alert.get('component_name'),
                    'Severity': alert.get('severity'),
                    'Category': alert.get('category'),
                    'Summary': alert.get('summary'),
                    'State': alert.get('state'),
                    'Created': alert.get('created'),
                    'Updated': alert.get('updated'),
                    'Resolved': alert.get('resolved'),
                    'Notified': alert.get('notified')
                })
            
            alerts_df = pd.DataFrame(alert_data)
            return alerts_df
        except Exception as e:
            print(f"Error getting alerts: {str(e)}")
            return pd.DataFrame()
    
    def collect_all_data(self) -> Dict[str, pd.DataFrame]:
        """Collect all data from the FlashArray"""
        print("Collecting array information...")
        array_info = self.get_array_info()
        
        if array_info.empty:
            return {}
            
        self.array_name = array_info.iloc[0]['ArrayName']
        
        print("Collecting capacity metrics...")
        capacity = self.get_array_capacity()
        
        print("Collecting volumes...")
        volumes = self.get_volumes()
        
        print("Collecting hosts...")
        hosts = self.get_hosts()
        
        print("Collecting host-volume connections...")
        connections = self.get_host_volume_connections()
        
        print("Collecting performance metrics...")
        performance = self.get_performance_metrics()
        
        print("Collecting alerts...")
        alerts = self.get_alerts()
        
        all_data = {
            'array_info': array_info,
            'capacity': capacity,
            'volumes': volumes,
            'hosts': hosts,
            'connections': connections,
            'performance': performance,
            'alerts': alerts
        }
        
        return all_data

# Example usage
if __name__ == "__main__":
    # Configuration - replace with your actual credentials
    FLASHARRAY_HOST = "purestorage-array.example.com"
    API_TOKEN = "your-api-token-here"
    
    # Create client and collect data
    client = PureStorageAPI(FLASHARRAY_HOST, API_TOKEN)
    
    print("Collecting all data from Pure Storage array...")
    data = client.collect_all_data()
    
    # Print sample data from each DataFrame
    if data:
        print("\nArray Information:")
        print(data['array_info'].head())
        
        print("\nCapacity Metrics:")
        print(data['capacity'].head())
        
        print("\nVolumes:")
        print(data['volumes'].head())
        
        print("\nHosts:")
        print(data['hosts'].head())
        
        print("\nHost-Volume Connections:")
        print(data['connections'].head())
        
        print("\nPerformance Metrics:")
        print(data['performance'].head())
        
        print("\nAlerts:")
        print(data['alerts'].head())
        
        # Save data to CSV files
        data['array_info'].to_csv('pure_array_info.csv', index=False)
        data['capacity'].to_csv('pure_capacity.csv', index=False)
        data['volumes'].to_csv('pure_volumes.csv', index=False)
        data['hosts'].to_csv('pure_hosts.csv', index=False)
        data['connections'].to_csv('pure_connections.csv', index=False)
        data['performance'].to_csv('pure_performance.csv', index=False)
        data['alerts'].to_csv('pure_alerts.csv', index=False)
        print("\nData saved to CSV files.")
    else:
        print("Failed to collect data from the array.")
```

## Script Explanation

### 1. Initialization and Setup

- **Class Initialization (`__init__` method)**:
  - Sets up connection parameters (host, API token)
  - Configures SSL verification (disabled by default)
  - Creates a requests Session with API token authentication
  - Sets default page size for paginated requests (100 items)

### 2. Pagination Support

- **`_get_paginated_results()`**:
  - Handles Pure Storage's continuation token pagination
  - Automatically follows pagination links until all results are collected
  - Supports optional query parameters
  - Returns all items combined in a single list

### 3. Data Collection Methods

#### Array Information
- **`get_array_info()`**:
  - Collects basic array details (model, version, serial number)
  - Sets the array_name property for use in other methods

#### Capacity Metrics
- **`get_array_capacity()`**:
  - Retrieves total, used, and free capacity
  - Includes data reduction ratios and space breakdown
  - All sizes converted to GB for consistency

#### Volume Information
- **`get_volumes()`**:
  - Gets all volumes with detailed space usage
  - Includes provisioning type, encryption status, and QoS settings
  - Calculates physical space usage after data reduction

#### Host Information
- **`get_hosts()`**:
  - Collects all configured hosts
  - Captures connection identifiers (WWN, IQN, NQN)
  - Includes host personality and OS type

#### Host-Volume Connections
- **`get_host_volume_connections()`**:
  - Maps which volumes are presented to which hosts
  - Includes LUN numbers and connection type (FC, iSCSI, NVMe)

#### Performance Metrics
- **`get_performance_metrics()`**:
  - Collects array-wide IOPS, throughput, and latency
  - Supports different time intervals (1h, 1d, 1w)
  - Samples volume performance data

#### Alerts
- **`get_alerts()`**:
  - Retrieves active or resolved alerts
  - Includes severity, category, and component information

### 4. Comprehensive Data Collection

- **`collect_all_data()`**:
  - Orchestrates collection of all data types
  - Returns a dictionary of DataFrames for each category
  - Provides progress feedback during collection

### 5. Example Usage

The script demonstrates:
1. Client initialization with array hostname and API token
2. Complete data collection
3. Sample output display
4. Automatic export to CSV files

## Key Differences from VMAX Script

1. **Authentication**: Uses API tokens instead of username/password
2. **Pagination**: Uses Pure's continuation token approach
3. **Data Model**: Reflects Pure's simpler architecture (single array focus)
4. **Terminology**: Uses Pure's naming (volumes instead of devices, etc.)
5. **Performance Metrics**: Includes more detailed performance collection
6. **Data Reduction**: Highlights Pure's efficiency capabilities

## Usage Notes

1. Generate an API token in the Pure Storage GUI under Settings > Users > API Tokens
2. For large environments, adjust the page_size parameter
3. Performance metrics can be collected for different time intervals
4. The script handles both active and resolved alerts
5. All capacity values are converted to GB for consistency
