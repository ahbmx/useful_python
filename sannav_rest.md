# Python Script for Brocade/Broadcom SANnav REST API Integration

Below is a comprehensive Python script that connects to the Brocade/Broadcom SANnav REST API to collect switch information, zones, fabrics, performance metrics, and more, storing everything in pandas DataFrames with pagination support.

```python
import requests
import pandas as pd
from requests.auth import HTTPBasicAuth
from requests.packages.urllib3.exceptions import InsecureRequestWarning
import warnings
from typing import Dict, List, Optional
import json
from datetime import datetime

# Suppress SSL warnings (not recommended for production)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
warnings.filterwarnings('ignore')

class BrocadeSANnavAPI:
    def __init__(self, host: str, username: str, password: str, verify_ssl: bool = False):
        """
        Initialize the Brocade SANnav API client
        
        Args:
            host: SANnav server hostname or IP
            username: SANnav username
            password: SANnav password
            verify_ssl: Whether to verify SSL certificates
        """
        self.host = host
        self.username = username
        self.password = password
        self.verify_ssl = verify_ssl
        self.base_url = f"https://{self.host}/api/v2"
        self.session = requests.Session()
        self.session.auth = HTTPBasicAuth(self.username, self.password)
        self.session.verify = self.verify_ssl
        self.session.headers.update({
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        })
        self.page_size = 100  # Default page size for paginated requests
        self.auth_token = None
        self.authenticate()
        
    def authenticate(self):
        """Authenticate and get token"""
        try:
            auth_url = f"https://{self.host}/api/v2/oauth/token"
            payload = {
                'grant_type': 'password',
                'username': self.username,
                'password': self.password
            }
            response = self.session.post(auth_url, data=payload)
            response.raise_for_status()
            self.auth_token = response.json().get('access_token')
            self.session.headers.update({
                'Authorization': f'Bearer {self.auth_token}'
            })
        except Exception as e:
            print(f"Authentication failed: {str(e)}")
            raise
            
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
        params = params or {}
        params['limit'] = self.page_size
        offset = 0
        
        while True:
            try:
                params['offset'] = offset
                response = self.session.get(
                    f"{self.base_url}/{endpoint}",
                    params=params
                )
                response.raise_for_status()
                data = response.json()
                
                if isinstance(data, list):
                    results.extend(data)
                    if len(data) < self.page_size:
                        break
                elif isinstance(data, dict):
                    items = data.get('data', [])
                    if isinstance(items, list):
                        results.extend(items)
                        if len(items) < self.page_size:
                            break
                    else:
                        # Single item response
                        results.append(data)
                        break
                
                offset += self.page_size
                
            except Exception as e:
                print(f"Error in paginated request to {endpoint}: {str(e)}")
                break
        
        return results
    
    def get_switches(self) -> pd.DataFrame:
        """Get all switches managed by SANnav"""
        try:
            switches = self._get_paginated_results('switches')
            
            switch_data = []
            for switch in switches:
                switch_data.append({
                    'SwitchName': switch.get('name'),
                    'SwitchWWN': switch.get('wwn'),
                    'Model': switch.get('model'),
                    'FirmwareVersion': switch.get('firmwareVersion'),
                    'IPAddress': switch.get('ipAddress'),
                    'Status': switch.get('status'),
                    'PortCount': switch.get('numberOfPorts'),
                    'Principal': switch.get('principal'),
                    'FabricID': switch.get('fabricId'),
                    'FabricName': switch.get('fabricName'),
                    'DomainID': switch.get('domainId'),
                    'HealthStatus': switch.get('healthStatus'),
                    'LastUpdated': switch.get('lastUpdated')
                })
            
            switches_df = pd.DataFrame(switch_data)
            return switches_df
        except Exception as e:
            print(f"Error getting switches: {str(e)}")
            return pd.DataFrame()
    
    def get_switch_ports(self, switch_wwn: str) -> pd.DataFrame:
        """Get all ports for a specific switch"""
        try:
            ports = self._get_paginated_results(f'switches/{switch_wwn}/ports')
            
            port_data = []
            for port in ports:
                port_data.append({
                    'SwitchWWN': switch_wwn,
                    'PortIndex': port.get('portIndex'),
                    'PortName': port.get('portName'),
                    'PortWWN': port.get('portWWN'),
                    'PortType': port.get('portType'),
                    'PortState': port.get('portState'),
                    'OperationalStatus': port.get('operationalStatus'),
                    'Speed': port.get('speed'),
                    'ConnectedWWN': port.get('connectedWWN'),
                    'ConnectedPortWWN': port.get('connectedPortWWN'),
                    'ConnectedSwitchWWN': port.get('connectedSwitchWWN'),
                    'ConnectedSwitchName': port.get('connectedSwitchName'),
                    'ZoneStatus': port.get('zoneStatus'),
                    'PortHealth': port.get('portHealth'),
                    'RxPower': port.get('rxPower'),
                    'TxPower': port.get('txPower')
                })
            
            ports_df = pd.DataFrame(port_data)
            return ports_df
        except Exception as e:
            print(f"Error getting ports for switch {switch_wwn}: {str(e)}")
            return pd.DataFrame()
    
    def get_fabrics(self) -> pd.DataFrame:
        """Get all fabric information"""
        try:
            fabrics = self._get_paginated_results('fabrics')
            
            fabric_data = []
            for fabric in fabrics:
                fabric_data.append({
                    'FabricID': fabric.get('fabricId'),
                    'FabricName': fabric.get('fabricName'),
                    'PrincipalSwitchWWN': fabric.get('principalSwitchWWN'),
                    'PrincipalSwitchName': fabric.get('principalSwitchName'),
                    'SwitchCount': fabric.get('switchCount'),
                    'EdgeSwitchCount': fabric.get('edgeSwitchCount'),
                    'DomainCount': fabric.get('domainCount'),
                    'HealthStatus': fabric.get('healthStatus'),
                    'FabricState': fabric.get('fabricState'),
                    'FabricOSVersion': fabric.get('fabricOSVersion'),
                    'LastUpdated': fabric.get('lastUpdated')
                })
            
            fabrics_df = pd.DataFrame(fabric_data)
            return fabrics_df
        except Exception as e:
            print(f"Error getting fabrics: {str(e)}")
            return pd.DataFrame()
    
    def get_zones(self, fabric_id: str) -> pd.DataFrame:
        """Get all zones for a specific fabric"""
        try:
            zones = self._get_paginated_results(f'fabrics/{fabric_id}/zones')
            
            zone_data = []
            for zone in zones:
                zone_data.append({
                    'FabricID': fabric_id,
                    'ZoneName': zone.get('name'),
                    'ZoneType': zone.get('type'),
                    'MemberCount': len(zone.get('members', [])),
                    'Members': ', '.join([m.get('wwn') for m in zone.get('members', [])]),
                    'AliasMembers': ', '.join([m.get('alias') for m in zone.get('members', []) if m.get('alias')]),
                    'PrincipalSwitchWWN': zone.get('principalSwitchWWN'),
                    'LastUpdated': zone.get('lastUpdated')
                })
            
            zones_df = pd.DataFrame(zone_data)
            return zones_df
        except Exception as e:
            print(f"Error getting zones for fabric {fabric_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_aliases(self, fabric_id: str) -> pd.DataFrame:
        """Get all aliases for a specific fabric"""
        try:
            aliases = self._get_paginated_results(f'fabrics/{fabric_id}/aliases')
            
            alias_data = []
            for alias in aliases:
                alias_data.append({
                    'FabricID': fabric_id,
                    'AliasName': alias.get('name'),
                    'MemberCount': len(alias.get('members', [])),
                    'Members': ', '.join(alias.get('members', [])),
                    'PrincipalSwitchWWN': alias.get('principalSwitchWWN'),
                    'LastUpdated': alias.get('lastUpdated')
                })
            
            aliases_df = pd.DataFrame(alias_data)
            return aliases_df
        except Exception as e:
            print(f"Error getting aliases for fabric {fabric_id}: {str(e)}")
            return pd.DataFrame()
    
    def get_performance_metrics(self, switch_wwn: str, metric_type: str = 'port') -> pd.DataFrame:
        """
        Get performance metrics for a switch or ports
        
        Args:
            switch_wwn: Switch WWN
            metric_type: 'port' or 'switch' metrics
        """
        try:
            if metric_type == 'port':
                metrics = self._get_paginated_results(f'switches/{switch_wwn}/port-performance-metrics')
            else:
                metrics = self._get_paginated_results(f'switches/{switch_wwn}/switch-performance-metrics')
            
            metric_data = []
            for metric in metrics:
                metric_data.append({
                    'SwitchWWN': switch_wwn,
                    'MetricType': metric_type,
                    'Timestamp': metric.get('timestamp'),
                    'Interval': metric.get('interval'),
                    'PortIndex': metric.get('portIndex') if metric_type == 'port' else None,
                    'RxFrames': metric.get('rxFrames'),
                    'TxFrames': metric.get('txFrames'),
                    'RxBytes': metric.get('rxBytes'),
                    'TxBytes': metric.get('txBytes'),
                    'RxUtilization': metric.get('rxUtilization'),
                    'TxUtilization': metric.get('txUtilization'),
                    'CrcErrors': metric.get('crcErrors'),
                    'LinkFailures': metric.get('linkFailures'),
                    'SignalLoss': metric.get('signalLoss'),
                    'InvalidTxWords': metric.get('invalidTxWords'),
                    'Discards': metric.get('discards')
                })
            
            metrics_df = pd.DataFrame(metric_data)
            return metrics_df
        except Exception as e:
            print(f"Error getting {metric_type} performance metrics for switch {switch_wwn}: {str(e)}")
            return pd.DataFrame()
    
    def get_events(self, days: int = 1) -> pd.DataFrame:
        """Get events from SANnav"""
        try:
            end_time = datetime.utcnow().isoformat() + 'Z'
            start_time = datetime.utcfromtimestamp(
                datetime.utcnow().timestamp() - days * 24 * 60 * 60
            ).isoformat() + 'Z'
            
            params = {
                'startTime': start_time,
                'endTime': end_time
            }
            events = self._get_paginated_results('events', params=params)
            
            event_data = []
            for event in events:
                event_data.append({
                    'EventID': event.get('eventId'),
                    'EventType': event.get('eventType'),
                    'Severity': event.get('severity'),
                    'Timestamp': event.get('timestamp'),
                    'Source': event.get('source'),
                    'Description': event.get('description'),
                    'SwitchWWN': event.get('switchWWN'),
                    'SwitchName': event.get('switchName'),
                    'FabricID': event.get('fabricId'),
                    'FabricName': event.get('fabricName'),
                    'PortIndex': event.get('portIndex'),
                    'PortWWN': event.get('portWWN')
                })
            
            events_df = pd.DataFrame(event_data)
            return events_df
        except Exception as e:
            print(f"Error getting events: {str(e)}")
            return pd.DataFrame()
    
    def collect_all_data(self) -> Dict[str, pd.DataFrame]:
        """Collect all data from SANnav"""
        print("Collecting switches...")
        switches = self.get_switches()
        
        if switches.empty:
            return {}
            
        print("Collecting fabrics...")
        fabrics = self.get_fabrics()
        
        print("Collecting zones...")
        zones = pd.DataFrame()
        for fabric_id in fabrics['FabricID'].unique():
            zones = pd.concat([zones, self.get_zones(fabric_id)])
        
        print("Collecting aliases...")
        aliases = pd.DataFrame()
        for fabric_id in fabrics['FabricID'].unique():
            aliases = pd.concat([aliases, self.get_aliases(fabric_id)])
        
        print("Collecting ports...")
        ports = pd.DataFrame()
        for switch_wwn in switches['SwitchWWN'].unique():
            ports = pd.concat([ports, self.get_switch_ports(switch_wwn)])
        
        print("Collecting performance metrics...")
        perf_metrics = pd.DataFrame()
        for switch_wwn in switches['SwitchWWN'].unique()[:3]:  # Limit to 3 switches for demo
            perf_metrics = pd.concat([
                perf_metrics,
                self.get_performance_metrics(switch_wwn, 'port'),
                self.get_performance_metrics(switch_wwn, 'switch')
            ])
        
        print("Collecting events...")
        events = self.get_events(days=1)
        
        all_data = {
            'switches': switches,
            'fabrics': fabrics,
            'zones': zones,
            'aliases': aliases,
            'ports': ports,
            'performance_metrics': perf_metrics,
            'events': events
        }
        
        return all_data

# Example usage
if __name__ == "__main__":
    # Configuration - replace with your actual credentials
    SANNAV_HOST = "sannav.example.com"
    USERNAME = "admin"
    PASSWORD = "password"
    
    # Create client and collect data
    print("Connecting to SANnav...")
    client = BrocadeSANnavAPI(SANNAV_HOST, USERNAME, PASSWORD)
    
    print("Collecting all data from SANnav...")
    data = client.collect_all_data()
    
    # Print sample data from each DataFrame
    if data:
        print("\nSwitches:")
        print(data['switches'].head())
        
        print("\nFabrics:")
        print(data['fabrics'].head())
        
        print("\nZones:")
        print(data['zones'].head())
        
        print("\nAliases:")
        print(data['aliases'].head())
        
        print("\nPorts:")
        print(data['ports'].head())
        
        print("\nPerformance Metrics:")
        print(data['performance_metrics'].head())
        
        print("\nEvents:")
        print(data['events'].head())
        
        # Save data to CSV files
        data['switches'].to_csv('brocade_switches.csv', index=False)
        data['fabrics'].to_csv('brocade_fabrics.csv', index=False)
        data['zones'].to_csv('brocade_zones.csv', index=False)
        data['aliases'].to_csv('brocade_aliases.csv', index=False)
        data['ports'].to_csv('brocade_ports.csv', index=False)
        data['performance_metrics'].to_csv('brocade_performance.csv', index=False)
        data['events'].to_csv('brocade_events.csv', index=False)
        print("\nData saved to CSV files.")
    else:
        print("Failed to collect data from SANnav.")
```

## Script Explanation

### 1. Initialization and Authentication

- **Class Initialization (`__init__` method)**:
  - Sets up connection parameters (host, username, password)
  - Configures SSL verification (disabled by default)
  - Creates a requests Session with basic authentication
  - Uses OAuth token authentication after initial login

- **`authenticate()`**:
  - Obtains OAuth token using username/password
  - Updates session headers with bearer token

### 2. Pagination Support

- **`_get_paginated_results()`**:
  - Handles SANnav's offset/limit pagination
  - Automatically follows pagination until all results are collected
  - Supports optional query parameters
  - Returns all items combined in a single list

### 3. Data Collection Methods

#### Switch Information
- **`get_switches()`**:
  - Collects all switches managed by SANnav
  - Includes model, firmware, IP, status, and health
  - Captures fabric membership and domain IDs

#### Port Information
- **`get_switch_ports()`**:
  - Gets all ports for a specific switch
  - Includes port state, speed, connected devices
  - Captures optical power levels (Rx/Tx) where available
  - Shows zoning status and port health

#### Fabric Information
- **`get_fabrics()`**:
  - Retrieves all fabric configurations
  - Identifies principal switches and fabric state
  - Includes switch counts and Fabric OS version

#### Zoning Configuration
- **`get_zones()`**:
  - Gets all zones for a specific fabric
  - Shows zone type and member WWNs
  - Identifies alias members if present

#### Alias Information
- **`get_aliases()`**:
  - Collects all aliases for a fabric
  - Lists member WWNs for each alias
  - Useful for understanding zone configurations

#### Performance Metrics
- **`get_performance_metrics()`**:
  - Collects both port-level and switch-level metrics
  - Includes frame/byte counts and utilization
  - Captures error counters (CRC, signal loss, etc.)
  - Supports different time intervals

#### Events and Alerts
- **`get_events()`**:
  - Retrieves recent events (configurable time window)
  - Includes severity, source, and affected components
  - Useful for troubleshooting and monitoring

### 4. Comprehensive Data Collection

- **`collect_all_data()`**:
  - Orchestrates collection of all data types
  - Handles iteration across fabrics and switches
  - Returns a dictionary of DataFrames for each category
  - Provides progress feedback during collection

### 5. Example Usage

The script demonstrates:
1. Client initialization with SANnav hostname and credentials
2. Complete data collection
3. Sample output display
4. Automatic export to CSV files

## Key Features for SANnav

1. **Fabric Focus**: Emphasizes fabric, switch, and zone relationships
2. **Port-Level Detail**: Captures detailed port configurations and metrics
3. **Zoning Support**: Collects both zones and aliases
4. **Performance Metrics**: Gathers both historical and real-time performance data
5. **Event Monitoring**: Provides visibility into SAN events and alerts
6. **Comprehensive Coverage**: Combines configuration and operational data

## Usage Notes

1. Requires SANnav 2.0 or later (API v2)
2. For large environments, adjust the page_size parameter
3. Performance metrics collection can be resource-intensive
4. Event collection is limited to 1 day by default (adjustable)
5. All data is exported to CSV files for easy analysis
6. Optical power metrics are only available for optical ports



