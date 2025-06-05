# Setting Cell Colors Based on Values in Pandas with XlsxWriter

To individually set cell colors based on their values when exporting a pandas DataFrame to Excel using XlsxWriter, you can use conditional formatting or apply formatting cell by cell. Here are both approaches:

## Method 1: Using Conditional Formatting

```python
import pandas as pd

# Create a sample DataFrame
data = {'Value': [10, 20, 30, 40, 50, -5, -15]}
df = pd.DataFrame(data)

# Create a Pandas Excel writer using XlsxWriter
writer = pd.ExcelWriter('formatted_output.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Sheet1', index=False)

# Get the xlsxwriter workbook and worksheet objects
workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Add conditional formats
# Format 1: Positive values in green
positive_format = workbook.add_format({'bg_color': '#C6EFCE',
                                      'font_color': '#006100'})
worksheet.conditional_format('B2:B8', {'type': 'cell',
                                       'criteria': '>',
                                       'value': 0,
                                       'format': positive_format})

# Format 2: Negative values in red
negative_format = workbook.add_format({'bg_color': '#FFC7CE',
                                      'font_color': '#9C0006'})
worksheet.conditional_format('B2:B8', {'type': 'cell',
                                       'criteria': '<',
                                       'value': 0,
                                       'format': negative_format})

# Close the Pandas Excel writer and output the Excel file
writer.close()
```

## Method 2: Applying Format Cell by Cell

```python
import pandas as pd

# Create a sample DataFrame
data = {'Value': [10, 20, 30, 40, 50, -5, -15]}
df = pd.DataFrame(data)

# Create a Pandas Excel writer using XlsxWriter
writer = pd.ExcelWriter('cell_formatted_output.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Sheet1', index=False)

# Get the xlsxwriter workbook and worksheet objects
workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Define formats
positive_format = workbook.add_format({'bg_color': '#C6EFCE',
                                      'font_color': '#006100'})
negative_format = workbook.add_format({'bg_color': '#FFC7CE',
                                      'font_color': '#9C0006'})

# Apply formatting cell by cell
for row_num in range(1, len(df)+1):  # Skip header row (0)
    cell_value = df.iloc[row_num-1, 0]
    
    if cell_value > 0:
        worksheet.write(row_num, 1, cell_value, positive_format)
    elif cell_value < 0:
        worksheet.write(row_num, 1, cell_value, negative_format)
    # No format for zero values

writer.close()
```

## Method 3: Using a Color Gradient (Advanced)

```python
import pandas as pd

# Create a sample DataFrame
data = {'Values': range(-10, 11)}
df = pd.DataFrame(data)

writer = pd.ExcelWriter('gradient_format.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Sheet1', index=False)

workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Apply a 3-color scale format
worksheet.conditional_format('B2:B21', {'type': '3_color_scale'})

writer.close()
```

The conditional formatting approach (Method 1) is generally more efficient for large datasets, while the cell-by-cell approach (Method 2) gives you more precise control over each cell's formatting. Method 3 shows how to apply gradient color scales based on value ranges.



# Setting Cell Colors Based on String Values in Pandas with XlsxWriter

Here's how to color cells based on string values, including coloring the target cell and the next two adjacent cells:

```python
import pandas as pd

# Create a sample DataFrame with strings
data = {
    'Category': ['Apple', 'Banana', 'Cherry', 'Apple', 'Banana', 'Cherry'],
    'Price': [1.20, 0.50, 2.10, 1.10, 0.45, 2.30],
    'Stock': [50, 120, 30, 45, 110, 25],
    'Supplier': ['A', 'B', 'C', 'A', 'B', 'C']
}
df = pd.DataFrame(data)

# Create a Pandas Excel writer using XlsxWriter
writer = pd.ExcelWriter('string_colored_output.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Sheet1', index=False)

# Get the xlsxwriter workbook and worksheet objects
workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Define color formats for different string values
formats = {
    'Apple': workbook.add_format({'bg_color': '#FFCCCB', 'font_color': '#8B0000'}),  # Light red
    'Banana': workbook.add_format({'bg_color': '#FFFF99', 'font_color': '#8B8000'}),  # Light yellow
    'Cherry': workbook.add_format({'bg_color': '#D8BFD8', 'font_color': '#4B0082'})   # Light purple
}

# Apply formatting to each row based on the 'Category' value
for row_num in range(1, len(df)+1):  # Skip header row (0)
    category = df.iloc[row_num-1, 0]  # Get value from first column ('Category')
    
    if category in formats:
        # Apply format to the category cell and next 2 cells
        for col_num in range(0, 3):  # Current column + next 2 columns
            worksheet.write(row_num, col_num, df.iloc[row_num-1, col_num], formats[category])

# Close the Pandas Excel writer and output the Excel file
writer.close()
```

## Alternative Approach Using Conditional Formatting

If you prefer conditional formatting (which is more efficient for large datasets):

```python
import pandas as pd

# Create a sample DataFrame
data = {
    'Category': ['Apple', 'Banana', 'Cherry', 'Apple', 'Banana', 'Cherry'],
    'Price': [1.20, 0.50, 2.10, 1.10, 0.45, 2.30],
    'Stock': [50, 120, 30, 45, 110, 25]
}
df = pd.DataFrame(data)

writer = pd.ExcelWriter('conditional_string_format.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Sheet1', index=False)

workbook = writer.book
worksheet = writer.sheets['Sheet1']

# Define formats
apple_format = workbook.add_format({'bg_color': '#FFCCCB', 'font_color': '#8B0000'})
banana_format = workbook.add_format({'bg_color': '#FFFF99', 'font_color': '#8B8000'})
cherry_format = workbook.add_format({'bg_color': '#D8BFD8', 'font_color': '#4B0082'})

# Apply conditional formatting for each category
for col in range(0, 3):  # Apply to first 3 columns
    # Apple
    worksheet.conditional_format(1, col, len(df), col, {
        'type': 'text',
        'criteria': 'containing',
        'value': 'Apple',
        'format': apple_format
    })
    
    # Banana
    worksheet.conditional_format(1, col, len(df), col, {
        'type': 'text',
        'criteria': 'containing',
        'value': 'Banana',
        'format': banana_format
    })
    
    # Cherry
    worksheet.conditional_format(1, col, len(df), col, {
        'type': 'text',
        'criteria': 'containing',
        'value': 'Cherry',
        'format': cherry_format
    })

writer.close()
```

### Key Notes:
1. The first method colors cells based on exact string matches in the first column and applies the format to that cell and the next two cells.
2. The second method uses conditional formatting to color any cell in the first three columns that contains the specified text.
3. Adjust the column ranges (`range(0, 3)`) if you need to format more or fewer adjacent cells.
4. You can customize the colors by changing the hex color codes in the format definitions.
