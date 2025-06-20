import xml.etree.ElementTree as ET
from collections import defaultdict

def analyze_xml(file_path):
    try:
        # Parse the XML file
        tree = ET.parse(file_path)
        root = tree.getroot()
        
        print(f"XML File Analysis Report for: {file_path}")
        print("=" * 60)
        
        # Basic information
        print(f"\nRoot Element: {root.tag}")
        
        # Count elements and attributes
        element_count = defaultdict(int)
        attribute_count = defaultdict(int)
        element_attributes = defaultdict(set)
        
        for elem in root.iter():
            element_count[elem.tag] += 1
            for attr in elem.attrib:
                attribute_count[attr] += 1
                element_attributes[elem.tag].add(attr)
        
        # Element statistics
        print("\nElement Statistics:")
        print(f"Total Elements: {sum(element_count.values())}")
        print(f"Unique Element Types: {len(element_count)}")
        for tag, count in sorted(element_count.items()):
            print(f"- {tag}: {count} occurrences")
        
        # Attribute statistics
        print("\nAttribute Statistics:")
        print(f"Total Attributes: {sum(attribute_count.values())}")
        print(f"Unique Attribute Names: {len(attribute_count)}")
        for attr, count in sorted(attribute_count.items()):
            print(f"- {attr}: {count} occurrences")
        
        # Element-Attribute relationships
        print("\nElement-Attribute Relationships:")
        for tag, attrs in sorted(element_attributes.items()):
            print(f"- {tag} has attributes: {', '.join(sorted(attrs)) or 'None'}")
        
        # Sample content from first few elements
        print("\nSample Content (first 3 elements of each type):")
        sampled_elements = defaultdict(list)
        for elem in root.iter():
            if len(sampled_elements[elem.tag]) < 3:
                text = elem.text.strip() if elem.text else ""
                sampled_elements[elem.tag].append((elem.attrib, text))
        
        for tag, samples in sorted(sampled_elements.items()):
            print(f"\n{tag}:")
            for i, (attrs, text) in enumerate(samples, 1):
                print(f"  Sample {i}:")
                if attrs:
                    print(f"    Attributes: {attrs}")
                if text:
                    print(f"    Text: {text[:100]}{'...' if len(text) > 100 else ''}")
                else:
                    print("    No text content")
        
        print("\nAnalysis complete.")
        
    except ET.ParseError as e:
        print(f"Error parsing XML file: {e}")
    except FileNotFoundError:
        print(f"Error: File not found at {file_path}")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")

if __name__ == "__main__":
    xml_file_path = input("Enter the path to the XML file: ")
    analyze_xml(xml_file_path)
