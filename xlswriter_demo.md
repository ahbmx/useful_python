# Extensive XlsxWriter Script Showcasing Multiple Styles and Cell Manipulations

Below is a comprehensive Python script using the XlsxWriter library to demonstrate various formatting styles, cell manipulations, and Excel features:

```python
import xlsxwriter
from datetime import datetime

# Create a new Excel file
workbook = xlsxwriter.Workbook('xlsxwriter_demo.xlsx')

# Add a worksheet
worksheet = workbook.add_worksheet('Demo Sheet')

# Add a second worksheet for charts
chart_sheet = workbook.add_worksheet('Charts')

# Set default row height and column width
worksheet.set_default_row(20)
worksheet.set_column('A:Z', 15)

## 1. Basic Cell Formatting ##

# Create some format objects
bold = workbook.add_format({'bold': True})
italic = workbook.add_format({'italic': True})
underline = workbook.add_format({'underline': True})
border = workbook.add_format({'border': 1})
center = workbook.add_format({'align': 'center'})
bg_color = workbook.add_format({'bg_color': '#FFFF00'})
font_size = workbook.add_format({'font_size': 16})
num_format = workbook.add_format({'num_format': '$#,##0.00'})
percent_format = workbook.add_format({'num_format': '0.00%'})
date_format = workbook.add_format({'num_format': 'yyyy-mm-dd', 'align': 'center'})

# Write formatted data
worksheet.write('A1', 'Basic Formatting Examples', bold)
worksheet.write('A2', 'Bold text', bold)
worksheet.write('A3', 'Italic text', italic)
worksheet.write('A4', 'Underlined text', underline)
worksheet.write('A5', 'Cell with border', border)
worksheet.write('A6', 'Centered text', center)
worksheet.write('A7', 'Yellow background', bg_color)
worksheet.write('A8', 'Larger font', font_size)
worksheet.write('A9', 1234.56, num_format)
worksheet.write('A10', 0.753, percent_format)
worksheet.write('A11', datetime(2023, 5, 15), date_format)

## 2. Combined Formatting ##

combined_format = workbook.add_format({
    'bold': True,
    'italic': True,
    'font_color': 'red',
    'bg_color': 'yellow',
    'border': 1,
    'align': 'center',
    'valign': 'vcenter'
})

worksheet.merge_range('C1:E1', 'Combined Formatting', combined_format)

## 3. Data Validation ##

worksheet.write('C3', 'Data Validation Examples', bold)
worksheet.data_validation('C4', {'validate': 'integer',
                                'criteria': 'between',
                                'minimum': 1,
                                'maximum': 100})

worksheet.data_validation('C5', {'validate': 'list',
                                'source': ['Option 1', 'Option 2', 'Option 3']})

## 4. Conditional Formatting ##

# Prepare data for conditional formatting
worksheet.write('C7', 'Conditional Formatting', bold)
for row in range(8, 18):
    for col in range(3, 6):
        worksheet.write(row, col, row * col)

# Add conditional formats
worksheet.conditional_format('C8:E17', {'type': 'cell',
                                       'criteria': '>=',
                                       'value': 50,
                                       'format': bg_color})

worksheet.conditional_format('C8:E17', {'type': 'data_bar',
                                      'bar_color': '#63C384'})

## 5. Formulas and Functions ##

worksheet.write('G1', 'Formulas and Functions', bold)
worksheet.write('G2', 'Sum of A9:A10', bold)
worksheet.write('H2', '=SUM(A9:A10)', num_format)

worksheet.write('G3', 'Average of C8:E17', bold)
worksheet.write('H3', '=AVERAGE(C8:E17)')

worksheet.write('G4', 'Today\'s Date', bold)
worksheet.write('H4', '=TODAY()', date_format)

## 6. Images ##

# Uncomment if you have an image to insert
# worksheet.insert_image('G6', 'python_logo.png', {'x_scale': 0.5, 'y_scale': 0.5})

## 7. Merged Cells ##

merge_format = workbook.add_format({
    'bold': 1,
    'border': 1,
    'align': 'center',
    'valign': 'vcenter',
    'fg_color': 'green'
})

worksheet.merge_range('G8:I10', 'Merged Cells Example', merge_format)

## 8. Cell Comments ##

worksheet.write('G12', 'Cell with Comment', bold)
worksheet.write_comment('H12', 'This is an important comment!')

## 9. Protection and Hidden Cells ##

locked = workbook.add_format({'locked': True})
hidden = workbook.add_format({'hidden': True})

worksheet.write('G14', 'Locked Cell (default)', locked)
worksheet.write('G15', 'Hidden Formula', hidden)
worksheet.write_formula('H15', '=A9*2', hidden)

# Protect the worksheet (password optional)
worksheet.protect()

## 10. Charts ##

# Create a chart object
chart = workbook.add_chart({'type': 'column'})

# Add data series for the chart
chart.add_series({
    'name': '=Demo Sheet!$C$7',
    'categories': '=Demo Sheet!$C$8:$C$17',
    'values': '=Demo Sheet!$D$8:$D$17',
})

# Configure the chart
chart.set_title({'name': 'Sample Chart'})
chart.set_x_axis({'name': 'Row numbers'})
chart.set_y_axis({'name': 'Values'})

# Insert the chart into the worksheet
worksheet.insert_chart('G18', chart)

# Alternatively, place on the chart sheet
chart_sheet.insert_chart('A1', chart)

## 11. Tables ##

# Write headers
worksheet.write('K1', 'Product', bold)
worksheet.write('L1', 'Q1 Sales', bold)
worksheet.write('M1', 'Q2 Sales', bold)

# Write data
data = [
    ['Apples', 10000, 12000],
    ['Oranges', 8000, 9000],
    ['Bananas', 15000, 17000],
    ['Pears', 7000, 8000],
]

for row_num, row_data in enumerate(data, start=2):
    worksheet.write_row(row_num, 10, row_data)

# Add a table
worksheet.add_table('K1:M5', {
    'columns': [
        {'header': 'Product'},
        {'header': 'Q1 Sales'},
        {'header': 'Q2 Sales'},
    ],
    'style': 'Table Style Medium 9',
})

## 12. Advanced Number Formats ##

worksheet.write('K7', 'Advanced Number Formats', bold)

custom_num_format = workbook.add_format({'num_format': '#,##0.00 "USD";[Red](#,##0.00 "USD")'})
worksheet.write('K8', 1234.56, custom_num_format)
worksheet.write('K9', -1234.56, custom_num_format)

phone_format = workbook.add_format({'num_format': '[<=9999999]###-####;(###) ###-####'})
worksheet.write('K10', 5551234, phone_format)
worksheet.write('K11', 18005551234, phone_format)

## 13. Hyperlinks ##

worksheet.write('K13', 'Hyperlinks', bold)
worksheet.write_url('K14', 'https://www.python.org', string='Python Website')
worksheet.write_url('K15', 'mailto:example@example.com', string='Send Email')

## 14. Freeze Panes ##
worksheet.freeze_panes(1, 1)  # Freeze first row and column

## 15. Page Setup ##
worksheet.set_landscape()
worksheet.set_paper(9)  # Paper size A4
worksheet.set_margins(left=0.7, right=0.7, top=0.75, bottom=0.75)
worksheet.set_header('&C&D &T', {'font_size': 10})
worksheet.set_footer('&CPage &P of &N')

## 16. Named Ranges ##
workbook.define_name('SalesData', '=Demo Sheet!$K$2:$M$5')

## 17. Rich Strings ##
red = workbook.add_format({'font_color': 'red'})
blue = workbook.add_format({'font_color': 'blue'})

worksheet.write_rich_string('K17', 
                          'This is a ', red, 'red', ' word and this is ', blue, 'blue', ' word.')

## 18. Sparklines ##
# Note: Sparklines are only available in Excel 2010+ and may not work in all viewers

# Prepare data
sparkline_data = [
    [-2, 2, 3, -1, 0],
    [30, 20, 33, 20, 15],
    [1, -1, -1, 2, -1]
]

worksheet.write_row('O1', ['Sparkline Examples'], bold)
for row_num, row_data in enumerate(sparkline_data, start=2):
    worksheet.write_row(f'O{row_num}', row_data)
    worksheet.add_sparkline(f'P{row_num}', {
        'range': f'Demo Sheet!O{row_num}:S{row_num}',
        'type': 'column',
    })

## 19. Outlines and Grouping ##
worksheet.write('U1', 'Outline Example', bold)

# Add sample data with outlines
for row in range(2, 12):
    worksheet.write(row, 20, f'Level {row % 3}')

# Set outline levels
worksheet.set_row(2, None, None, {'level': 1})
worksheet.set_row(3, None, None, {'level': 2})
worksheet.set_row(4, None, None, {'level': 2})
worksheet.set_row(5, None, None, {'level': 1})
worksheet.set_row(6, None, None, {'level': 2})
worksheet.set_row(7, None, None, {'level': 2})
worksheet.set_row(8, None, None, {'level': 1})
worksheet.set_row(9, None, None, {'level': 2})
worksheet.set_row(10, None, None, {'level': 2})

worksheet.outline_settings(True, False, True, False)

## 20. Dynamic Array Formulas (Excel 365) ##
worksheet.write('U13', 'Dynamic Arrays', bold)
worksheet.write('U14', '=SORT(UNIQUE(O2:S2))')

# Close the workbook
workbook.close()

print("Excel file created successfully!")
```

## Key Features Demonstrated:

1. **Basic Formatting**: Bold, italic, underline, borders, alignment, colors
2. **Combined Formatting**: Multiple style properties in one format
3. **Data Validation**: Restricting cell input to specific values
4. **Conditional Formatting**: Color scales and data bars
5. **Formulas and Functions**: Common Excel functions
6. **Images**: Inserting images into worksheets
7. **Merged Cells**: Combining multiple cells
8. **Cell Comments**: Adding notes to cells
9. **Protection**: Locking and hiding cells
10. **Charts**: Creating column charts
11. **Tables**: Creating formatted tables
12. **Advanced Number Formats**: Custom number displays
13. **Hyperlinks**: Adding clickable links
14. **Freeze Panes**: Keeping headers visible
15. **Page Setup**: Controlling print layout
16. **Named Ranges**: Defining named ranges
17. **Rich Strings**: Multiple formats in one cell
18. **Sparklines**: Mini charts in cells
19. **Outlines**: Grouping and collapsing rows
20. **Dynamic Arrays**: Excel 365 features

To run this script, you'll need to install XlsxWriter:
```
pip install XlsxWriter
```

The script creates a comprehensive Excel file demonstrating most of XlsxWriter's capabilities, which you can use as a reference for your own projects.
