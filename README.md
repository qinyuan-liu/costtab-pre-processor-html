# Costtab Pre-Processor

Local browser tool for converting raw IFAD costtab workbooks into formatted costtab workbooks.

## Use

1. Open `costtab_pre_processor.html` in a browser.
2. Upload the raw costtab workbook.
3. Enter the Project ID.
4. Select the summary tab and component/subcomponent columns.
5. Review or edit the generated mapping rows.
6. Click `Check`.
7. Click `Generate formatted costtab`.

All Excel processing runs locally in the browser. The workbook is not uploaded anywhere.

## Files

- `costtab_pre_processor.html` - the tool.
- `vendor/xlsx.full.min.js` - local SheetJS dependency for reading Excel files.
- `vendor/exceljs.min.js` - local ExcelJS dependency for writing styled Excel files.

Keep the `vendor` folder next to the HTML file.
