In Python using the `xlsxwriter` library, you can easily add **hyperlinks** either to:

1. **Different sheets** within the same Excel workbook (internal links), or
2. **External URLs** or files (external links).

---

### 🔗 Example: Adding Hyperlinks with `xlsxwriter`

#### 1. **Internal Link (to another sheet)**

```python
import xlsxwriter

# Create a workbook and add worksheets
workbook = xlsxwriter.Workbook('hyperlink_example.xlsx')
sheet1 = workbook.add_worksheet('Main')
sheet2 = workbook.add_worksheet('Data')

# Write something on the target sheet
sheet2.write('A1', 'Welcome to the Data Sheet!')

# Add a hyperlink in Sheet1 that links to Sheet2!A1
sheet1.write_url('A1', "internal:'Data'!A1", string='Go to Data Sheet')

workbook.close()
```

---

#### 2. **External Link (URL)**

```python
import xlsxwriter

workbook = xlsxwriter.Workbook('hyperlink_example.xlsx')
worksheet = workbook.add_worksheet()

# Add a hyperlink to a website
worksheet.write_url('A1', 'https://www.openai.com', string='Visit OpenAI')

workbook.close()
```

---

#### 3. **External File Link**

```python
import xlsxwriter

workbook = xlsxwriter.Workbook('hyperlink_example.xlsx')
worksheet = workbook.add_worksheet()

# Add a hyperlink to a file
worksheet.write_url('A1', 'external:C:/Users/YourUser/Documents/report.pdf', string='Open Report')

workbook.close()
```

---

### ✅ Tips

* Always enclose **sheet names in quotes** if they contain spaces: `internal:'My Sheet'!A1`.
* You can style hyperlinks using a format with `underline=True` and `font_color='blue'`.

Let me know if you want an example with custom formatting or dynamic link generation!


In Python, using the `xlsxwriter` library, you can create fairly rich Excel files including:

1. **Navigation via buttons (simulated via hyperlinks)**
2. **Styling specific cells or columns**
3. **Inserting images and logos**

Here's how to achieve all three:

---

## ✅ 1. Add "Buttons" to Navigate Between Sheets

While `xlsxwriter` doesn't support actual **form controls**, you can simulate navigation with **hyperlinks to other sheets**:

```python
import xlsxwriter

workbook = xlsxwriter.Workbook('navigation.xlsx')

# Create sheets
sheet1 = workbook.add_worksheet('Sheet1')
sheet2 = workbook.add_worksheet('Sheet2')

# Add hyperlink to Sheet2 from Sheet1
sheet1.write_url('A1', "internal:'Sheet2'!A1", string='Go to Sheet2')

# Add hyperlink back to Sheet1 from Sheet2
sheet2.write_url('A1', "internal:'Sheet1'!A1", string='Back to Sheet1')

workbook.close()
```

This adds clickable "button-like" text that jumps between sheets.

---

## ✅ 2. Style the First Column

You can format the entire column like so:

```python
# Define a format
bold_format = workbook.add_format({
    'bold': True,
    'font_color': 'blue',
    'bg_color': '#D7E4BC',
    'border': 1,
    'align': 'left'
})

# Apply format to the entire first column (A)
sheet1.set_column('A:A', 20, bold_format)
```

This sets the width and style for the first column.

---

## ✅ 3. Add Images and Logos

You can insert images (like logos) using `.insert_image()`:

```python
# Insert image (adjust path to your image)
sheet1.insert_image('B2', 'logo.png', {'x_scale': 0.5, 'y_scale': 0.5})
```

You can control image size with `x_scale` and `y_scale`.

---

### 🔄 Full Example Combining All Three

```python
import xlsxwriter

workbook = xlsxwriter.Workbook('styled_navigation.xlsx')
sheet1 = workbook.add_worksheet('Dashboard')
sheet2 = workbook.add_worksheet('Data')

# Navigation links
sheet1.write_url('A1', "internal:'Data'!A1", string='➡ Go to Data Sheet')
sheet2.write_url('A1', "internal:'Dashboard'!A1", string='⬅ Back to Dashboard')

# Style first column in Dashboard
header_format = workbook.add_format({
    'bold': True, 'bg_color': '#F9DA04', 'border': 1, 'align': 'center'
})
sheet1.set_column('A:A', 25, header_format)

# Insert logo in Dashboard
sheet1.insert_image('C2', 'logo.png', {'x_scale': 0.4, 'y_scale': 0.4})

workbook.close()
```

---

### 📌 Notes:

* `xlsxwriter` is purely for writing; you can't modify existing Excel files (for that, use `openpyxl` or `xlwings`).
* Images must be in supported formats like PNG or JPEG.
* If you want actual button elements or macros, you'd need to use Excel VBA or `xlwings`.

Would you like to extend this to include charts, dropdowns, or conditional formatting as well?



No, **`XlsxWriter` does not support setting image size in exact pixels directly.**
Instead, it uses **scaling factors** relative to the image’s original dimensions:

```python
worksheet.insert_image('B2', 'logo.png', {
    'x_scale': 0.5,
    'y_scale': 0.5
})
```

---

### 🔄 Workaround to Match Pixel Dimensions

If you **must set a logo to a specific width/height in pixels**, here's how you can approximate it:

1. **Get original image dimensions in pixels** (e.g., using PIL).
2. **Compute scaling ratio** to get desired size.
3. **Apply that ratio to `x_scale` and `y_scale`**.

---

### ✅ Example: Resize to Specific Pixels

```python
from PIL import Image
import xlsxwriter

# Target pixel size
target_width_px = 100
target_height_px = 50

# Load image and get original size
image_path = 'logo.png'
with Image.open(image_path) as img:
    original_width, original_height = img.size

# Compute scale ratios
x_scale = target_width_px / original_width
y_scale = target_height_px / original_height

# Create workbook and insert image with pixel-based scaling
workbook = xlsxwriter.Workbook('image_scaled.xlsx')
worksheet = workbook.add_worksheet()

worksheet.insert_image('B2', image_path, {
    'x_scale': x_scale,
    'y_scale': y_scale
})

workbook.close()
```

This gives you **precise control over image dimensions in pixels** using scaling factors.

---

Would you like to automatically position the logo in a header or top-left cell area too?

