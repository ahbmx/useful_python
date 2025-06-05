# Python Script to Generate DrawIO Diagram for PowerMax Connected Servers

Here's a Python script that generates a DrawIO XML file showing servers connected to a PowerMax storage array:

```python
import xml.etree.ElementTree as ET
from xml.dom import minidom

def create_drawio_diagram(servers, powermax_name, connections):
    """
    Creates a DrawIO diagram showing servers connected to a PowerMax array
    
    Args:
        servers (list): List of server hostnames
        powermax_name (str): Name of the PowerMax array
        connections (list of tuples): List of (server, powermax_port) connections
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
    root.set("pageWidth", "850")
    root.set("pageHeight", "1100")
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
    title_cell.set("value", f"Server Connections to {powermax_name}")
    title_cell.set("style", "text;html=1;strokeColor=none;fillColor=none;align=center;verticalAlign=middle;whiteSpace=wrap;rounded=0;fontSize=24;fontStyle=1")
    title_cell.set("vertex", "1")
    title_cell.set("parent", "1")
    geometry = ET.SubElement(title_cell, "mxGeometry")
    geometry.set("x", "300")
    geometry.set("y", "20")
    geometry.set("width", "250")
    geometry.set("height", "40")
    geometry.set("as", "geometry")
    
    # Create PowerMax array shape
    powermax_cell = ET.SubElement(parent, "mxCell")
    powermax_cell.set("id", "powermax")
    powermax_cell.set("value", powermax_name)
    powermax_cell.set("style", "swimlane;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;rounded=1;fontSize=16;fontStyle=1")
    powermax_cell.set("vertex", "1")
    powermax_cell.set("parent", "1")
    geometry = ET.SubElement(powermax_cell, "mxGeometry")
    geometry.set("x", "350")
    geometry.set("y", "100")
    geometry.set("width", "150")
    geometry.set("height", "300")
    geometry.set("as", "geometry")
    
    # Add ports to the PowerMax (simplified)
    port_positions = {
        "A1": (350, 150),
        "A2": (350, 200),
        "A3": (350, 250),
        "A4": (350, 300),
        "B1": (500, 150),
        "B2": (500, 200),
        "B3": (500, 250),
        "B4": (500, 300),
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
    
    # Create server shapes in two columns
    server_cells = {}
    server_x_positions = [100, 700]  # Left and right columns
    server_y_start = 150
    server_y_step = 80
    
    for i, server in enumerate(servers):
        col = 0 if i < len(servers)/2 else 1
        y_pos = server_y_start + (i % (len(servers)/2)) * server_y_step
        
        server_cell = ET.SubElement(parent, "mxCell")
        server_cell.set("id", f"server_{server}")
        server_cell.set("value", server)
        server_cell.set("style", "shape=rectangle;whiteSpace=wrap;html=1;fillColor=#f5f5f5;strokeColor=#666666;rounded=1;fontSize=12")
        server_cell.set("vertex", "1")
        server_cell.set("parent", "1")
        geometry = ET.SubElement(server_cell, "mxGeometry")
        geometry.set("x", str(server_x_positions[col]))
        geometry.set("y", str(y_pos))
        geometry.set("width", "120")
        geometry.set("height", "60")
        geometry.set("as", "geometry")
        server_cells[server] = server_cell
    
    # Create connections
    for i, (server, port) in enumerate(connections):
        edge = ET.SubElement(parent, "mxCell")
        edge.set("id", f"conn_{i}")
        edge.set("style", "edgeStyle=none;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;endArrow=classic;endFill=1;strokeWidth=2")
        edge.set("edge", "1")
        edge.set("parent", "1")
        edge.set("source", f"server_{server}")
        edge.set("target", f"port_{port}")
        geometry = ET.SubElement(edge, "mxGeometry")
        geometry.set("relative", "1")
        geometry.set("as", "geometry")
    
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
    # Define your servers and connections
    servers = ["web01", "web02", "db01", "db02", "app01", "app02", "file01", "file02"]
    powermax_name = "PowerMax-1234"
    connections = [
        ("web01", "A1"),
        ("web02", "A2"),
        ("db01", "A3"),
        ("db02", "A4"),
        ("app01", "B1"),
        ("app02", "B2"),
        ("file01", "B3"),
        ("file02", "B4"),
    ]
    
    # Generate the DrawIO XML
    drawio_xml = create_drawio_diagram(servers, powermax_name, connections)
    
    # Save to file
    with open("powermax_connections.drawio", "w") as f:
        f.write(drawio_xml)
    
    print("DrawIO file created: powermax_connections.drawio")
```

## How to Use This Script:

1. Modify the example data in the `if __name__ == "__main__":` section:
   - `servers`: List of your server hostnames
   - `powermax_name`: Name of your PowerMax array
   - `connections`: List of tuples showing which server connects to which PowerMax port

2. Run the script to generate a `.drawio` file

3. Open the generated file in DrawIO (now diagrams.net)

## Customization Options:

- To change the visual style, modify the `style` attributes in the code
- To add more ports, extend the `port_positions` dictionary
- To change the layout, adjust the positioning values (x, y coordinates)

## Requirements:

- Python 3.x
- Standard library modules (xml.etree.ElementTree, xml.dom.minidom)

The script creates a professional-looking diagram with:
- A title
- The PowerMax array in the center
- Servers arranged in two columns on either side
- Connection lines showing which server connects to which PowerMax port
