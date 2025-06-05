Here's an enhanced version of the script that includes SAN switches in the diagram, showing the complete path from servers through switches to the PowerMax array:

```python
import xml.etree.ElementTree as ET
from xml.dom import minidom

def create_drawio_diagram(servers, switches, powermax_name, connections):
    """
    Creates a DrawIO diagram showing servers connected to a PowerMax array via SAN switches
    
    Args:
        servers (list): List of server hostnames
        switches (list): List of SAN switch hostnames
        powermax_name (str): Name of the PowerMax array
        connections (list of tuples): List of (server, switch, powermax_port) connections
    """
    # Create the root MXGraphModel element
    root = ET.Element("mxGraphModel")
    root.set("dx", "1426")
    root.set("dy", "857")
    root.set("grid", "1")
    root.set("gridSize", "10")
    root.set("guides", "1")
    root.set("tooltips", "1")
    root.set("connect", "1")
    root.set("arrows", "1")
    root.set("fold", "1")
    root.set("page", "1")
    root.set("pageScale", "1")
    root.set("pageWidth", "1100")
    root.set("pageHeight", "850")
    root.set("math", "0")
    root.set("shadow", "0")
    
    # Create the root element
    parent = ET.SubElement(root, "root")
    
    # Add the default parent cell
    default_cell = ET.SubElement(parent, "mxCell")
    default_cell.set("id", "0")
    
    # Add the default parent cell for layers
    default_layer = ET.SubElement(parent, "mxCell")
    default_layer.set("id", "1")
    default_layer.set("parent", "0")
    
    # Create a title cell
    title_cell = ET.SubElement(parent, "mxCell")
    title_cell.set("id", "title")
    title_cell.set("value", f"SAN Connectivity: Servers → Switches → {powermax_name}")
    title_cell.set("style", "text;html=1;strokeColor=none;fillColor=none;align=center;verticalAlign=middle;whiteSpace=wrap;rounded=0;fontSize=24;fontStyle=1")
    title_cell.set("vertex", "1")
    title_cell.set("parent", "1")
    geometry = ET.SubElement(title_cell, "mxGeometry")
    geometry.set("x", "350")
    geometry.set("y", "20")
    geometry.set("width", "400")
    geometry.set("height", "40")
    geometry.set("as", "geometry")
    
    # Create PowerMax array shape (centered right)
    powermax_cell = ET.SubElement(parent, "mxCell")
    powermax_cell.set("id", "powermax")
    powermax_cell.set("value", powermax_name)
    powermax_cell.set("style", "swimlane;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;rounded=1;fontSize=16;fontStyle=1")
    powermax_cell.set("vertex", "1")
    powermax_cell.set("parent", "1")
    geometry = ET.SubElement(powermax_cell, "mxGeometry")
    geometry.set("x", "800")
    geometry.set("y", "150")
    geometry.set("width", "150")
    geometry.set("height", "300")
    geometry.set("as", "geometry")
    
    # Add director ports to the PowerMax
    port_positions = {
        "A1": (800, 200),
        "A2": (800, 250),
        "A3": (800, 300),
        "A4": (800, 350),
        "B1": (950, 200),
        "B2": (950, 250),
        "B3": (950, 300),
        "B4": (950, 350),
    }
    
    port_cells = {}
    for port, (x, y) in port_positions.items():
        port_cell = ET.SubElement(parent, "mxCell")
        port_cell.set("id", f"port_{port}")
        port_cell.set("value", port)
        port_cell.set("style", "shape=ellipse;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#f8cecc;strokeColor=#b85450;fontSize=10")
        port_cell.set("vertex", "1")
        port_cell.set("parent", "1")
        geometry = ET.SubElement(port_cell, "mxGeometry")
        geometry.set("x", str(x))
        geometry.set("y", str(y))
        geometry.set("width", "20")
        geometry.set("height", "20")
        geometry.set("as", "geometry")
        port_cells[port] = port_cell
    
    # Create SAN switches (middle of diagram)
    switch_cells = {}
    switch_x = 500
    switch_y_start = 200
    switch_y_step = 100
    
    for i, switch in enumerate(switches):
        switch_cell = ET.SubElement(parent, "mxCell")
        switch_cell.set("id", f"switch_{switch}")
        switch_cell.set("value", switch)
        switch_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#fff2cc;strokeColor=#d6b656;rounded=1;fontSize=14")
        switch_cell.set("vertex", "1")
        switch_cell.set("parent", "1")
        geometry = ET.SubElement(switch_cell, "mxGeometry")
        geometry.set("x", str(switch_x))
        geometry.set("y", str(switch_y_start + i * switch_y_step))
        geometry.set("width", "120")
        geometry.set("height", "60")
        geometry.set("as", "geometry")
        switch_cells[switch] = switch_cell
        
        # Add switch ports (simplified - two per switch)
        for j in range(1, 3):
            port_id = f"{switch}_port{j}"
            port_cell = ET.SubElement(parent, "mxCell")
            port_cell.set("id", port_id)
            port_cell.set("value", f"Port {j}")
            port_cell.set("style", "shape=ellipse;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#d5e8d4;strokeColor=#82b366;fontSize=8")
            port_cell.set("vertex", "1")
            port_cell.set("parent", "1")
            geometry = ET.SubElement(port_cell, "mxGeometry")
            port_x = switch_x - 15 if j == 1 else switch_x + 120 - 5
            geometry.set("x", str(port_x))
            geometry.set("y", str(switch_y_start + i * switch_y_step + 20))
            geometry.set("width", "15")
            geometry.set("height", "15")
            geometry.set("as", "geometry")
    
    # Create server shapes (left column)
    server_cells = {}
    server_x = 100
    server_y_start = 150
    server_y_step = 80
    
    for i, server in enumerate(servers):
        server_cell = ET.SubElement(parent, "mxCell")
        server_cell.set("id", f"server_{server}")
        server_cell.set("value", server)
        server_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;rounded=1;fontSize=12")
        server_cell.set("vertex", "1")
        server_cell.set("parent", "1")
        geometry = ET.SubElement(server_cell, "mxGeometry")
        geometry.set("x", str(server_x))
        geometry.set("y", str(server_y_start + i * server_y_step))
        geometry.set("width", "120")
        geometry.set("height", "60")
        geometry.set("as", "geometry")
        server_cells[server] = server_cell
        
        # Add HBA ports to servers
        hba_cell = ET.SubElement(parent, "mxCell")
        hba_cell.set("id", f"hba_{server}")
        hba_cell.set("value", "HBA")
        hba_cell.set("style", "shape=ellipse;whiteSpace=wrap;html=1;aspect=fixed;fillColor=#e1d5e7;strokeColor=#9673a6;fontSize=8")
        hba_cell.set("vertex", "1")
        hba_cell.set("parent", "1")
        geometry = ET.SubElement(hba_cell, "mxGeometry")
        geometry.set("x", str(server_x + 120 - 10))
        geometry.set("y", str(server_y_start + i * server_y_step + 20))
        geometry.set("width", "15")
        geometry.set("height", "15")
        geometry.set("as", "geometry")
    
    # Create connections
    connection_id = 0
    for server, switch, powermax_port in connections:
        # Connection from server to switch
        edge1 = ET.SubElement(parent, "mxCell")
        edge1.set("id", f"conn_{connection_id}")
        edge1.set("style", "edgeStyle=none;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;endArrow=classic;endFill=1;strokeWidth=2;strokeColor=#666666")
        edge1.set("edge", "1")
        edge1.set("parent", "1")
        edge1.set("source", f"hba_{server}")
        edge1.set("target", f"{switch}_port1")
        geometry = ET.SubElement(edge1, "mxGeometry")
        geometry.set("relative", "1")
        geometry.set("as", "geometry")
        connection_id += 1
        
        # Connection from switch to PowerMax
        edge2 = ET.SubElement(parent, "mxCell")
        edge2.set("id", f"conn_{connection_id}")
        edge2.set("style", "edgeStyle=none;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;endArrow=classic;endFill=1;strokeWidth=2;strokeColor=#666666")
        edge2.set("edge", "1")
        edge2.set("parent", "1")
        edge2.set("source", f"{switch}_port2")
        edge2.set("target", f"port_{powermax_port}")
        geometry = ET.SubElement(edge2, "mxGeometry")
        geometry.set("relative", "1")
        geometry.set("as", "geometry")
        connection_id += 1
    
    # Convert to XML and prettify
    rough_xml = ET.tostring(root, 'utf-8')
    parsed_xml = minidom.parseString(rough_xml)
    pretty_xml = parsed_xml.toprettyxml(indent="  ")
    
    # Create the complete DrawIO file structure
    drawio_xml = f"""<mxfile>
  <diagram name="Page-1" id="0">
    {pretty_xml.split('<', 1)[1]}
</mxfile>"""
    
    return drawio_xml

# Example usage
if __name__ == "__main__":
    # Define your infrastructure
    servers = ["web01", "web02", "db01", "db02", "app01", "app02"]
    switches = ["SAN-A", "SAN-B"]
    powermax_name = "PowerMax-4567"
    
    # Define connections: (server, switch, powermax_port)
    connections = [
        ("web01", "SAN-A", "A1"),
        ("web02", "SAN-A", "A2"),
        ("db01", "SAN-A", "A3"),
        ("db02", "SAN-B", "B1"),
        ("app01", "SAN-B", "B2"),
        ("app02", "SAN-B", "B3"),
    ]
    
    # Generate the DrawIO XML
    drawio_xml = create_drawio_diagram(servers, switches, powermax_name, connections)
    
    # Save to file
    with open("san_powermax_connections.drawio", "w") as f:
        f.write(drawio_xml)
    
    print("DrawIO file created: san_powermax_connections.drawio")
```

## Key Improvements:

1. **Three-Tier Architecture**: Shows servers → SAN switches → PowerMax array
2. **Detailed Components**:
   - Servers with HBA ports
   - SAN switches with ports
   - PowerMax array with director ports
3. **Realistic Connections**: 
   - Server HBA connects to switch port 1
   - Switch port 2 connects to PowerMax director port
4. **Better Layout**:
   - Servers on the left
   - SAN switches in the middle
   - PowerMax on the right
5. **Color Coding**:
   - Servers: Light gray
   - SAN switches: Light yellow
   - PowerMax: Light blue
   - Ports: Color-coded by device type

## How to Use:

1. Modify the example data in the `if __name__ == "__main__":` section:
   - `servers`: List of your server hostnames
   - `switches`: List of your SAN switch hostnames
   - `powermax_name`: Name of your PowerMax array
   - `connections`: List of tuples showing (server, switch, powermax_port) paths

2. Run the script to generate a `.drawio` file

3. Open the generated file in DrawIO/diagrams.net

You can easily customize colors, shapes, and layout by modifying the style attributes in the code. The script provides a clear visualization of your SAN infrastructure including all intermediate switches.
