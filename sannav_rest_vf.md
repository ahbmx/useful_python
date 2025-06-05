# Enhanced Python Script for Brocade/Broadcom SANnav with Virtual Fabrics Support

Here's an expanded version of the script that includes Virtual Fabrics (VF) functionality for Brocade SAN switches:

```python
import requests
import pandas as pd
from requests.auth import HTTPBasicAuth
from requests.packages.urllib3.exceptions import InsecureRequestWarning
import warnings
from typing import Dict, List, Optional
import json
from datetime import datetime, timedelta

# Suppress SSL warnings (not recommended for production)
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
warnings.filterwarnings('ignore')

class BrocadeSANnavAPI:
    def __init__(self, host: str, username: str, password: str, verify_ssl: bool = False):
        """
        Initialize the Brocade SANnav API client with Virtual Fabrics support
        
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
    
    def get_switches(self, include_vf_details: bool = True) -> pd.DataFrame:
        """Get all switches managed by SANnav with optional Virtual Fabric details"""
        try:
            switches = self._get_paginated_results('switches')
            
            switch_data = []
            for switch in switches:
                switch_info = {
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
                    'LastUpdated': switch.get('lastUpdated'),
                    'VirtualFabricsEnabled': switch.get('virtualFabricsEnabled'),
                    'BaseSwitchEnabled': switch.get('baseSwitchEnabled')
                }
                
                # Get additional Virtual Fabric details if enabled
                if include_vf_details and switch.get('virtualFabricsEnabled'):
                    vf_details = self.get_virtual_fabrics(switch.get('wwn'))
                    if not vf_details.empty:
                        switch_info.update({
                            'VFCount': len(vf_details),
                            'DefaultVFID': vf_details[vf_details['IsDefault'] == True]['VFID'].iloc[0] if True in vf_details['IsDefault'].values else None
                        })
                
                switch_data.append(switch_info)
            
            switches_df = pd.DataFrame(switch_data)
            return switches_df
        except Exception as e:
            print(f"Error getting switches: {str(e)}")
            return pd.DataFrame()
    
    def get_virtual_fabrics(self, switch_wwn: str) -> pd.DataFrame:
        """Get Virtual Fabrics configuration for a specific switch"""
        try:
            vfs = self._get_paginated_results(f'switches/{switch_wwn}/virtual-fabrics')
            
            vf_data = []
            for vf in vfs:
                vf_data.append({
                    'SwitchWWN': switch_wwn,
                    'VFID': vf.get('vfId'),
                    'VFName': vf.get('name'),
                    'IsDefault': vf.get('isDefault'),
                    'FabricID': vf.get('fabricId'),
                    'Principal': vf.get('principal'),
                    'DomainID': vf.get('domainId'),
                    'PortCount': len(vf.get('portMembers', [])),
                    'PortMembers': ', '.join([str(p) for p in vf.get('portMembers', [])]),
                    'InterconnectPorts': ', '.join([str(p) for p in vf.get('interconnectPorts', [])]),
                    'LastUpdated': vf.get('lastUpdated')
                })
            
            vfs_df = pd.DataFrame(vf_data)
            return vfs_df
        except Exception as e:
            print(f"Error getting Virtual Fabrics for switch {switch_wwn}: {str(e)}")
            return pd.DataFrame()
    
    def get_vf_ports(self, switch_wwn: str) -> pd.DataFrame:
        """Get port-to-Virtual Fabric mappings for a switch"""
        try:
            ports = self._get_paginated_results(f'switches/{switch_wwn}/ports')
            
            port_data = []
            for port in ports:
                port_info = {
                    'SwitchWWN': switch_wwn,
                    'PortIndex': port.get('portIndex'),
                    'PortName': port.get('portName'),
                    'PortWWN': port.get('portWWN'),
                    'PortType': port.get('portType'),
                    'PortState': port.get('portState'),
                    'OperationalStatus': port.get('operationalStatus'),
                    'Speed': port.get('speed'),
                    'VirtualFabricID': port.get('virtualFabricId'),
                    'VirtualFabricName': port.get('virtualFabricName'),
                    'BaseFabricPort': port.get('baseFabricPort')
                }
                
                # Get additional Virtual Fabric details if available
                if port.get('virtualFabricId'):
                    vf_details = self.get_virtual_fabric_details(switch_wwn, port.get('virtualFabricId'))
                    if vf_details:
                        port_info.update({
                            'VFFabricID': vf_details.get('fabricId'),
                            'VFPrincipal': vf_details.get('principal'),
                            'VFDomainID': vf_details.get('domainId')
                        })
                
                port_data.append(port_info)
            
            ports_df = pd.DataFrame(port_data)
            return ports_df
        except Exception as e:
            print(f"Error getting VF ports for switch {switch_wwn}: {str(e)}")
            return pd.DataFrame()
    
    def get_virtual_fabric_details(self, switch_wwn: str, vf_id: int) -> Optional[Dict]:
        """Get details for a specific Virtual Fabric"""
        try:
            vfs = self._get_paginated_results(f'switches/{switch_wwn}/virtual-fabrics')
            for vf in vfs:
                if vf.get('vfId') == vf_id:
                    return vf
            return None
        except Exception as e:
            print(f"Error getting Virtual Fabric details: {str(e)}")
            return None
    
    def get_fabrics(self, include_vf: bool = True) -> pd.DataFrame:
        """Get all fabric information including Virtual Fabrics"""
        try:
            fabrics = self._get_paginated_results('fabrics')
            
            fabric_data = []
            for fabric in fabrics:
                fabric_info = {
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
                    'LastUpdated': fabric.get('lastUpdated'),
                    'IsVirtualFabric': False
                }
                fabric_data.append(fabric_info)
            
            # Get Virtual Fabrics if requested
            if include_vf:
                switches = self.get_switches(include_vf_details=False)
                for switch_wwn in switches[switches['VirtualFabricsEnabled'] == True]['SwitchWWN']:
                    vfs = self.get_virtual_fabrics(switch_wwn)
                    for _, vf in vfs.iterrows():
                        fabric_data.append({
                            'FabricID': vf['FabricID'],
                            'FabricName': f"{vf['VFName']} (VFID:{vf['VFID']})",
                            'PrincipalSwitchWWN': switch_wwn,
                            'PrincipalSwitchName': switches[switches['SwitchWWN'] == switch_wwn]['SwitchName'].iloc[0],
                            'SwitchCount': 1,  # VF is specific to one switch
                            'EdgeSwitchCount': 0,
                            'DomainCount': 1,
                            'HealthStatus': 'UNKNOWN',
                            'FabricState': 'ACTIVE',
                            'FabricOSVersion': switches[switches['SwitchWWN'] == switch_wwn]['FirmwareVersion'].iloc[0],
                            'LastUpdated': vf['LastUpdated'],
                            'IsVirtualFabric': True,
                            'VFID': vf['VFID'],
                            'BaseSwitchWWN': switch_wwn
                        })
            
            fabrics_df = pd.DataFrame(fabric_data)
            return fabrics_df
        except Exception as e:
            print(f"Error getting fabrics: {str(e)}")
            return pd.DataFrame()
    
    def get_zones(self, fabric_id: str, is_virtual_fabric: bool = False) -> pd.DataFrame:
        """Get all zones for a specific fabric (including Virtual Fabrics)"""
        try:
            if is_virtual_fabric:
                # For Virtual Fabrics, we need to get the base switch WWN
                fabrics = self.get_fabrics()
                base_switch = fabrics[fabrics['FabricID'] == fabric_id]['BaseSwitchWWN'].iloc[0]
                zones = self._get_paginated_results(f'switches/{base_switch}/virtual-fabrics/zones?fabricId={fabric_id}')
            else:
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
                    'LastUpdated': zone.get('lastUpdated'),
                    'IsVirtualFabric': is_virtual_fabric
                })
            
            zones_df = pd.DataFrame(zone_data)
            return zones_df
        except Exception as e:
            print(f"Error getting zones for fabric {fabric_id}: {str(e)}")
            return pd.DataFrame()
    
    def collect_all_data(self) -> Dict[str, pd.DataFrame]:
        """Collect all data from SANnav including Virtual Fabrics"""
        print("Collecting switches with Virtual Fabric details...")
        switches = self.get_switches(include_vf_details=True)
        
        if switches.empty:
            return {}
            
        print("Collecting fabrics including Virtual Fabrics...")
        fabrics = self.get_fabrics(include_vf=True)
        
        print("Collecting zones for all fabrics...")
        zones = pd.DataFrame()
        for _, fabric in fabrics.iterrows():
            zones = pd.concat([zones, self.get_zones(
                fabric['FabricID'], 
                fabric.get('IsVirtualFabric', False)
            )])
        
        print("Collecting Virtual Fabric ports...")
        vf_ports = pd.DataFrame()
        for switch_wwn in switches[switches['VirtualFabricsEnabled'] == True]['SwitchWWN']:
            vf_ports = pd.concat([vf_ports, self.get_vf_ports(switch_wwn)])
        
        print("Collecting Virtual Fabrics configurations...")
        virtual_fabrics = pd.DataFrame()
        for switch_wwn in switches[switches['VirtualFabricsEnabled'] == True]['SwitchWWN']:
            virtual_fabrics = pd.concat([virtual_fabrics, self.get_virtual_fabrics(switch_wwn)])
        
        all_data = {
            'switches': switches,
            'fabrics': fabrics,
            'zones': zones,
            'virtual_fabrics': virtual_fabrics,
            'vf_ports': vf_ports
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
    
    print("Collecting all data including Virtual Fabrics...")
    data = client.collect_all_data()
    
    # Print sample data from each DataFrame
    if data:
        print("\nSwitches:")
        print(data['switches'].head())
        
        print("\nFabrics (including Virtual Fabrics):")
        print(data['fabrics'].head())
        
        print("\nZones:")
        print(data['zones'].head())
        
        print("\nVirtual Fabrics Configurations:")
        print(data['virtual_fabrics'].head())
        
        print("\nVirtual Fabric Port Mappings:")
        print(data['vf_ports'].head())
        
        # Save data to CSV files
        data['switches'].to_csv('brocade_switches_vf.csv', index=False)
        data['fabrics'].to_csv('brocade_fabrics_vf.csv', index=False)
        data['zones'].to_csv('brocade_zones_vf.csv', index=False)
        data['virtual_fabrics'].to_csv('brocade_virtual_fabrics.csv', index=False)
        data['vf_ports'].to_csv('brocade_vf_ports.csv', index=False)
        print("\nData saved to CSV files.")
    else:
        print("Failed to collect data from SANnav.")
```

## Key Virtual Fabric Enhancements

### 1. Virtual Fabric Discovery

- **`get_virtual_fabrics()`**: Retrieves all Virtual Fabrics for a specific switch including:
  - VF ID and name
  - Port members and interconnect ports
  - Default VF status
  - Associated fabric and domain IDs

### 2. Enhanced Switch Information

- **`get_switches()`** now includes:
  - VirtualFabricsEnabled flag
  - BaseSwitchEnabled flag
  - Count of Virtual Fabrics per switch
  - Default Virtual Fabric ID

### 3. Port-to-VF Mapping

- **`get_vf_ports()`**: Shows which ports belong to which Virtual Fabrics including:
  - Virtual Fabric ID and name
  - Base fabric port status
  - Additional VF details when available

### 4. Virtual Fabric Zones

- **`get_zones()`** enhanced to:
  - Handle both regular fabrics and Virtual Fabrics
  - Include IsVirtualFabric flag in output
  - Properly query zones for Virtual Fabrics

### 5. Comprehensive Fabric View

- **`get_fabrics()`** now:
  - Includes both physical and Virtual Fabrics
  - Clearly marks Virtual Fabrics
  - Shows base switch information for Virtual Fabrics
  - Maintains consistent data structure

## Usage Examples

1. **Get all Virtual Fabrics for a switch**:
```python
vf_config = client.get_virtual_fabrics("10:00:00:05:1e:e5:6a:00")
```

2. **See which ports belong to which VF**:
```python
vf_ports = client.get_vf_ports("10:00:00:05:1e:e5:6a:00")
```

3. **Get zones for a Virtual Fabric**:
```python
vf_zones = client.get_zones("fabric_vf_128", is_virtual_fabric=True)
```

4. **Collect complete SAN topology**:
```python
full_data = client.collect_all_data()
```

This enhanced script provides complete visibility into both physical and Virtual Fabric configurations in your Brocade/Broadcom SAN environment.
