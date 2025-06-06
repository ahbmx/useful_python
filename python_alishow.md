# Improved `alishow` Parser for Multi-line Alias-WWN Entries

Here's an enhanced Python script that handles cases where the alias name and WWN aren't on the same line in the `alishow` output:

```python
import re
from collections import defaultdict

def parse_alishow_multiline(output):
    """
    Parses alishow output where alias and WWN may be on separate lines.
    Returns: 
        - aliases: dict of {alias_name: [WWNs]}
        - zones: dict of {zone_name: [members]}
        - configs: dict of {config_name: [zones]}
    """
    data = {
        'aliases': defaultdict(list),
        'zones': defaultdict(list),
        'configs': defaultdict(list),
        'current_config': None,
        'current_zone': None,
        'pending_alias': None  # For multi-line alias definitions
    }
    
    for line in output.splitlines():
        line = line.strip()
        
        # Skip empty lines and headers
        if not line or line.startswith('Effective'):
            continue
            
        # Configuration definition
        if line.startswith('cfg:'):
            data['current_config'] = line.split('cfg:')[1].strip()
            continue
            
        # Zone definition
        if line.startswith('zone:'):
            data['current_zone'] = line.split('zone:')[1].strip()
            if data['current_config']:
                data['configs'][data['current_config']].append(data['current_zone'])
            continue
            
        # Alias definition start (may be followed by WWN on next line)
        if line.startswith('alias:'):
            alias_part = line.split('alias:')[1].strip()
            if ' ' in alias_part:  # Both alias and WWN on same line
                parts = alias_part.split()
                alias_name = parts[0]
                wwn = parts[1].rstrip(';').lower()
                data['aliases'][alias_name].append(wwn)
            else:  # Only alias name on this line
                data['pending_alias'] = alias_part.rstrip(';')
            continue
            
        # WWN line following an alias definition
        if data['pending_alias'] and re.match(r'^[0-9a-fA-F:]+;?$', line):
            wwn = line.strip().rstrip(';').lower()
            data['aliases'][data['pending_alias']].append(wwn)
            data['pending_alias'] = None
            continue
            
        # Zone member (could be alias or WWN)
        if data['current_zone'] and line.strip(';'):
            member = line.strip(';').strip()
            if member:
                data['zones'][data['current_zone']].append(member)
                
    # Convert defaultdict to regular dict for cleaner output
    data['aliases'] = dict(data['aliases'])
    data['zones'] = dict(data['zones'])
    data['configs'] = dict(data['configs'])
    
    return data

# Example usage with multi-line alias definition
if __name__ == "__main__":
    sample_output = """
    Effective configuration:
    cfg:    PRODUCTION_CONFIG
            zone:   SERVER1_ZONE
                    SERVER1_ALIAS;
            zone:   STORAGE_ZONE
                    ARRAY1_PORT1;
                    ARRAY1_PORT2;
                    
    alias: SERVER1_ALIAS
           10:00:00:00:c9:29:2f:9d;
           
    alias: ARRAY1_PORT1 10:00:00:05:1e:e7:02:1e
    alias: ARRAY1_PORT2
           10:00:00:05:1e:e7:02:1f;
    """
    
    parsed_data = parse_alishow_multiline(sample_output)
    
    print("Aliases and WWNs:")
    for alias, wwns in parsed_data['aliases'].items():
        print(f"  {alias}: {', '.join(wwns)}")
    
    print("\nZone Memberships:")
    for zone, members in parsed_data['zones'].items():
        print(f"  {zone}: {', '.join(members)}")
    
    print("\nConfiguration to Zone Mapping:")
    for config, zones in parsed_data['configs'].items():
        print(f"  {config}: {', '.join(zones)}")
```

## Key Improvements:

1. **Handles multi-line alias definitions** where:
   - The alias name appears on one line after `alias:`
   - The WWN appears on the following line

2. **Tracks three types of information**:
   - Aliases and their WWNs
   - Zone memberships
   - Configuration to zone mappings

3. **Robust parsing** that handles:
   - Different line endings (with or without semicolons)
   - Case-insensitive WWN matching
   - Both compact and expanded format alias definitions

4. **State machine approach** that remembers:
   - Current configuration being processed
   - Current zone being processed
   - Pending alias definitions waiting for WWNs

## How to Use with Real Switch Data:

```python
# Using Paramiko to get real switch output
import paramiko

def get_switch_output(host, username, password, command):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(host, username=username, password=password)
    stdin, stdout, stderr = ssh.exec_command(command)
    output = stdout.read().decode()
    ssh.close()
    return output

# Get and parse real data
switch_output = get_switch_output('switch_ip', 'admin', 'password', 'alishow')
parsed_data = parse_alishow_multiline(switch_output)

# Example: Find all zones containing a specific WWN
target_wwn = '10:00:00:00:c9:29:2f:9d'
zones_with_wwn = [
    zone for zone, members in parsed_data['zones'].items()
    if target_wwn in members or 
    any(target_wwn in parsed_data['aliases'].get(alias, []) 
    for alias in members
]
print(f"Zones containing WWN {target_wwn}: {', '.join(zones_with_wwn)}")
```

This solution provides comprehensive parsing of the `alishow` output regardless of whether the alias and WWN appear on the same line or different lines.
