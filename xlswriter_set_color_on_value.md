Great! Here's a version that works for **any DataFrame**, automatically detects the **range**, and can apply **color formatting to both numbers and strings** based on values.

---

### ✅ Features:

* Auto-detects the cell range for each column.
* Handles both **numeric** and **string** values.
* Applies conditional formatting using **XlsxWriter**.

---

### 💡 Example Code: Dynamic formatting for numbers and strings

```python
import pandas as pd

# Example DataFrame with mixed data
df = pd.DataFrame({
    'Name': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'Score': [85, 40, 70, 55],
    'Status': ['Pass', 'Fail', 'Pass', 'Fail']
})

with pd.ExcelWriter('conditional_formatting.xlsx', engine='xlsxwriter') as writer:
    df.to_excel(writer, sheet_name='Sheet1', index=False)

    workbook  = writer.book
    worksheet = writer.sheets['Sheet1']

    # Define formats
    green_format = workbook.add_format({'bg_color': '#C6EFCE', 'font_color': '#006100'})  # for good values
    red_format = workbook.add_format({'bg_color': '#FFC7CE', 'font_color': '#9C0006'})    # for bad values

    # Determine number of rows (for range)
    num_rows = len(df) + 1  # +1 for header

    for col_num, column in enumerate(df.columns):
        col_letter = chr(ord('A') + col_num)  # Convert 0 -> A, 1 -> B, etc.
        cell_range = f'{col_letter}2:{col_letter}{num_rows}'  # e.g., B2:B5

        if pd.api.types.is_numeric_dtype(df[column]):
            # Numeric formatting: Green if >= 70, Red if < 70
            worksheet.conditional_format(cell_range, {
                'type': 'cell',
                'criteria': '>=',
                'value': 70,
                'format': green_format
            })
            worksheet.conditional_format(cell_range, {
                'type': 'cell',
                'criteria': '<',
                'value': 70,
                'format': red_format
            })
        elif pd.api.types.is_string_dtype(df[column]):
            # String formatting: Green if 'Pass', Red if 'Fail'
            worksheet.conditional_format(cell_range, {
                'type': 'text',
                'criteria': 'containing',
                'value': 'Pass',
                'format': green_format
            })
            worksheet.conditional_format(cell_range, {
                'type': 'text',
                'criteria': 'containing',
                'value': 'Fail',
                'format': red_format
            })
```

---

### 📌 Output:

* `Score` cells will be green if ≥ 70, red if < 70.
* `Status` cells will be green if they contain `"Pass"`, red if `"Fail"`.

---

### 🛠️ Tips:

* You can customize string matching (e.g. exact match, begins with).
* For more than two conditions or more complex logic, you could use formulas.

Would you like to make it support even more complex conditions (like formulas or multi-condition logic)?
