# Pandas and XlsxWriter Cheat Sheet

## Pandas Basics

### Creating DataFrames
```python
import pandas as pd

# From dictionary
data = {'col1': [1, 2], 'col2': [3, 4]}
df = pd.DataFrame(data)

# From list of lists
df = pd.DataFrame([[1, 3], [2, 4]], columns=['col1', 'col2'])

# From CSV
df = pd.read_csv('file.csv')

# From Excel
df = pd.read_excel('file.xlsx', sheet_name='Sheet1')
```

### Viewing Data
```python
df.head()       # First 5 rows
df.tail(3)      # Last 3 rows
df.sample(5)    # Random 5 rows
df.describe()   # Summary statistics
df.info()       # DataFrame info
df.shape        # (rows, columns)
df.columns      # Column names
```

### Selecting Data
```python
df['col1']          # Single column
df[['col1', 'col2']] # Multiple columns
df.loc[0]           # Row by label
df.iloc[0]          # Row by index
df.loc[0, 'col1']   # Specific value
df.iloc[0, 0]       # Specific value by index
df[df['col1'] > 1]  # Filter rows
```

### Modifying Data
```python
df['new_col'] = df['col1'] * 2          # Add column
df = df.rename(columns={'col1': 'new'})  # Rename column
df = df.drop('col1', axis=1)            # Drop column
df = df.drop(0, axis=0)                  # Drop row
df = df.sort_values('col1')             # Sort
df = df.reset_index(drop=True)           # Reset index
```

### Grouping and Aggregating
```python
df.groupby('col1').mean()                # Group and average
df.groupby('col1').agg({'col2': 'sum', 'col3': 'mean'})
df.pivot_table(values='col2', index='col1', columns='col3', aggfunc='sum')
```

## XlsxWriter Basics

### Creating Excel Files
```python
import xlsxwriter

# Create workbook and worksheet
workbook = xlsxwriter.Workbook('output.xlsx')
worksheet = workbook.add_worksheet('Sheet1')

# Write data
worksheet.write(0, 0, 'Hello')       # Row, column, value
worksheet.write('A2', 'World')       # Excel notation

# Write formulas
worksheet.write_formula('A3', '=SUM(A1:A2)')

# Close workbook (must be done)
workbook.close()
```

### Formatting
```python
# Add formats
bold = workbook.add_format({'bold': True})
red_bg = workbook.add_format({'bg_color': 'red'})
border = workbook.add_format({'border': 1})

# Apply formats
worksheet.write(0, 0, 'Bold text', bold)
worksheet.set_column('A:A', 20)      # Set column width
worksheet.set_row(0, 30)            # Set row height
```

### Pandas with XlsxWriter
```python
# Create Pandas Excel writer using XlsxWriter
writer = pd.ExcelWriter('pandas_output.xlsx', engine='xlsxwriter')

# Convert dataframe to XlsxWriter Excel object
df.to_excel(writer, sheet_name='Sheet1', index=False)

# Get xlsxwriter objects
workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Add formatting
format = workbook.add_format({'num_format': '$#,##0'})
worksheet.set_column('B:B', 12, format)

# Close writer
writer.close()
```

### Advanced XlsxWriter Features
```python
# Conditional formatting
worksheet.conditional_format('B2:B10', {'type': '3_color_scale'})

# Charts
chart = workbook.add_chart({'type': 'column'})
chart.add_series({'values': '=Sheet1!$B$2:$B$10'})
worksheet.insert_chart('D2', chart)

# Autofilter
worksheet.autofilter('A1:B10')

# Freeze panes
worksheet.freeze_panes(1, 0)  # Freeze first row

# Data validation
worksheet.data_validation('C1:C10', {'validate': 'list', 'source': ['Yes', 'No']})
```

## Common Operations

### Save DataFrame to Excel with Formatting
```python
# Create writer
writer = pd.ExcelWriter('formatted.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Sheet1', index=False)

# Get objects
workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Add header format
header_format = workbook.add_format({
    'bold': True,
    'text_wrap': True,
    'valign': 'top',
    'fg_color': '#D7E4BC',
    'border': 1
})

# Write header
for col_num, value in enumerate(df.columns.values):
    worksheet.write(0, col_num, value, header_format)

# Close
writer.close()
```

### Multiple Sheets
```python
with pd.ExcelWriter('output.xlsx', engine='xlsxwriter') as writer:
    df1.to_excel(writer, sheet_name='Sheet1')
    df2.to_excel(writer, sheet_name='Sheet2')
```




Here's a Python function that takes a Pandas DataFrame and automatically applies header styling and autofilter to all columns when exporting to Excel using XlsxWriter:

```python
import pandas as pd

def export_with_styling(df, file_path, sheet_name='Sheet1'):
    """
    Export DataFrame to Excel with styled headers and autofilter
    
    Parameters:
    - df: pandas DataFrame to export
    - file_path: Output file path (e.g., 'output.xlsx')
    - sheet_name: Name of the worksheet (default: 'Sheet1')
    """
    # Create a Pandas Excel writer using XlsxWriter
    with pd.ExcelWriter(file_path, engine='xlsxwriter') as writer:
        # Convert the dataframe to an XlsxWriter Excel object
        df.to_excel(writer, sheet_name=sheet_name, index=False)
        
        # Get the xlsxwriter objects
        workbook = writer.book
        worksheet = writer.sheets[sheet_name]
        
        # Define header format
        header_format = workbook.add_format({
            'bold': True,
            'text_wrap': True,
            'valign': 'top',
            'fg_color': '#4472C4',  # Blue background
            'font_color': 'white',  # White text
            'border': 1,
            'border_color': 'white'
        })
        
        # Write the column headers with the defined format
        for col_num, value in enumerate(df.columns.values):
            worksheet.write(0, col_num, value, header_format)
        
        # Apply autofilter to all columns
        if len(df) > 0:
            worksheet.autofilter(0, 0, 0, len(df.columns) - 1)
        
        # Auto-adjust column widths
        for i, col in enumerate(df.columns):
            # Get max width of column
            max_len = max((
                df[col].astype(str).str.len().max(),  # Max data length
                len(str(col))  # Header length
            )) + 2  # Add padding
            
            # Set column width (with max limit of 50)
            worksheet.set_column(i, i, min(max_len, 50))

# Example usage:
# df = pd.DataFrame({'Name': ['Alice', 'Bob'], 'Age': [25, 30], 'City': ['NY', 'LA']})
# export_with_styling(df, 'styled_output.xlsx')
```

### Features of this function:
1. **Header Styling**:
   - Bold text
   - White text on blue background
   - Borders
   - Text wrapping

2. **Autofilter**:
   - Applied to all columns
   - Only added if DataFrame has rows

3. **Auto Column Widths**:
   - Adjusts based on content width
   - Caps at 50 characters to prevent overly wide columns

4. **Clean Output**:
   - No index column
   - Properly formatted headers

5. **Context Manager**:
   - Uses `with` statement to properly handle file operations

To use this function, simply pass your DataFrame and the desired output file path. The function handles all the styling and formatting automatically.




To set a default date style for an Excel file when exporting from Pandas using XlsxWriter, you can create a custom format and apply it to date columns. Here's how to do it:

### **1. Function to Export DataFrame with Default Date Style**
```python
import pandas as pd
from datetime import datetime

def export_with_date_style(
    df, 
    file_path, 
    sheet_name="Sheet1", 
    date_format="yyyy-mm-dd",  # Excel date format
    date_columns=None,        # List of date columns (optional)
):
    """
    Export DataFrame to Excel with default date formatting.
    
    Args:
        df (pd.DataFrame): DataFrame to export.
        file_path (str): Output Excel file path.
        sheet_name (str): Worksheet name.
        date_format (str): Excel-style date format (e.g., "dd/mm/yyyy", "mm/dd/yyyy").
        date_columns (list): List of columns to format as dates (if None, auto-detect).
    """
    with pd.ExcelWriter(file_path, engine="xlsxwriter", datetime_format=date_format) as writer:
        # Write DataFrame to Excel
        df.to_excel(writer, sheet_name=sheet_name, index=False)
        
        workbook = writer.book
        worksheet = writer.sheets[sheet_name]
        
        # If date_columns is None, auto-detect datetime columns
        if date_columns is None:
            date_columns = df.select_dtypes(include=["datetime64", "datetime"]).columns.tolist()
        
        # Apply date format to detected columns
        date_style = workbook.add_format({"num_format": date_format})
        
        for col_idx, col_name in enumerate(df.columns):
            if col_name in date_columns:
                # Apply date format to entire column
                worksheet.set_column(col_idx, col_idx, None, date_style)
        
        # Optional: Apply header styling (from previous example)
        header_format = workbook.add_format({
            "bold": True,
            "bg_color": "#4472C4",
            "font_color": "white",
            "border": 1,
        })
        
        for col_idx, col_name in enumerate(df.columns):
            worksheet.write(0, col_idx, col_name, header_format)
        
        # Optional: Autofilter
        if len(df) > 0:
            worksheet.autofilter(0, 0, 0, len(df.columns) - 1)

# Example Usage:
# df = pd.DataFrame({
#     "Date": [datetime(2023, 1, 1), datetime(2023, 1, 2)],
#     "Name": ["Alice", "Bob"],
#     "Value": [100, 200]
# })
# export_with_date_style(df, "output.xlsx", date_format="dd-mmm-yyyy")
```

---

### **2. Common Excel Date Formats**
You can customize the `date_format` parameter with Excel-style format strings:

| Format String | Example Output |
|--------------|----------------|
| `"yyyy-mm-dd"` | 2023-12-31 |
| `"mm/dd/yyyy"` | 12/31/2023 |
| `"dd-mmm-yyyy"` | 31-Dec-2023 |
| `"dd/mm/yyyy hh:mm:ss"` | 31/12/2023 23:59:59 |
| `"mmm-yy"` | Dec-23 |

---

### **3. Key Features**
✅ **Auto-detects datetime columns** (unless `date_columns` is specified).  
✅ **Applies consistent date formatting** across all date columns.  
✅ **Optional header styling & autofilter** (can be removed if not needed).  
✅ **Uses `with` statement** for safe file handling.  

---

### **4. Alternative: Formatting After Export**
If you prefer to format dates **after** exporting (e.g., for existing Excel files), you can use:
```python
import pandas as pd
from openpyxl import load_workbook
from openpyxl.styles import numbers

# Load an existing Excel file
wb = load_workbook("output.xlsx")
ws = wb.active

# Apply date format to column A (e.g., "yyyy-mm-dd")
for row in ws.iter_rows(min_row=2, min_col=1, max_col=1):
    for cell in row:
        cell.number_format = numbers.FORMAT_DATE_YYYYMMDD2  # or "yyyy-mm-dd"

wb.save("formatted_output.xlsx")
```

---

### **Conclusion**
- Use `datetime_format` in `pd.ExcelWriter` for basic date formatting.  
- For more control, manually apply formats using `workbook.add_format()`.  
- Auto-detect date columns with `df.select_dtypes(include=["datetime64"])`.  

This ensures all dates in your Excel file follow a consistent style. 🚀
