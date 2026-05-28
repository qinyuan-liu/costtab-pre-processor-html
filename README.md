# Costtab Pre-Processor

Local browser tool for converting raw IFAD costtab workbooks into formatted costtab workbooks.

## Use

1. Open `costtab_pre_processor.html` in a browser.
2. Upload the raw costtab workbook.
3. Enter the Project ID.
4. Select the summary tab and component/subcomponent columns.
5. Review and edit the generated mapping rows.
6. Click `Check`.
7. Click `Generate formatted costtab`.

All Excel processing runs locally in the browser. The workbook is not uploaded anywhere.

## Files

- `costtab_pre_processor.html` - the tool.
- `vendor/xlsx.full.min.js` - local SheetJS dependency for reading Excel files.
- `vendor/exceljs.min.js` - local ExcelJS dependency for writing styled Excel files.

Keep the `vendor` folder next to the HTML file.

## Validation Logic

The `Check` button runs the same validation that is run before `Generate formatted costtab`.
Generation stays disabled while any blocking error exists.

Blocking errors:

- A raw costtab workbook must be selected.
- Project ID is normalized to digits only and must be present in the embedded project list.
- A component financing summary tab must be selected. Summary tab candidates are workbook sheets whose names include `COM`; `COMFIN` is preferred when present. If no summary tab exists, the tool shows: `No component financing summary found. Please flag this in the tracking file and skip this project.`
- Component and subcomponent summary columns must be non-empty Excel column letters only, for example `A`, `D`, or `AA`.
- In component mode, the top-level subcomponent column input is hidden and subcomponent text is not checked. The former subcomponent level in the DT sheet should be treated as the detail start level.
- At least one active component/subcomponent mapping is required. A mapping is active when the component has text, or when a subcomponent has text, an investment column, a non-USD currency, a custom currency, a unit, or a source tab. Editing only detail start/end does not make an otherwise empty row active.
- Component text is required for every active component and must contain at least 2 words. A leading marker such as `1.` or `A)` is ignored for the word count.
- Each active component must have at least one active subcomponent.
- Subcomponent text is required for every active subcomponent and must contain at least 2 words. A leading marker such as `1.` or `A)` is ignored for the word count.
- Detail start, detail end, and investment columns are required for every active subcomponent and must be Excel column letters only.
- Detail end must not be before detail start.
- If currency is `Other`, the custom currency field is required.
- Unit is required. Supported units in the UI are `million`, `thousand ('000)`, and `unit (1)`.
- Source tab is required.
- For each valid source tab mapping, source rows are scanned from row 6 onward. Rows are ignored when they are header rows, empty detail rows, subtotal/total rows, detailed cost labels, or parent rows without unit-like data. Any remaining source row is an error if the first detail column is empty.
- For each valid source tab mapping, any remaining source row is an error if the investment column is empty, non-numeric, or `0`.

Non-blocking warnings:

- If the number of detail columns from detail start through detail end is not exactly 4, the tool warns: `There are N detail columns. Is it correct? Please recheck the costtab and make sure.` Generation is still allowed if there are no blocking errors.
- If the investment measure is changed to `Base Cost`, the tool shows `Please flag this in the tracking file`.
- If more than 70% of a subcomponent's tokens are made up of `project management`, `knowledge management`, or `M&E`, the tool suggests removing it unless it includes costs beyond pure project management.

Source row scanning details:

- Source rows before row 6 are skipped.
- A summary tab is optional. If no `COM` tab is available, component/subcomponent text can be filled from source tabs by entering source column letters in the mapping table.
- Component and subcomponent mapping columns override text loaded from the summary tab when provided.
- Header rows are skipped when the selected detail columns are empty or include `detailed costs`, `quantities`, `unit cost`, or `financing`.
- Rows are skipped when all selected detail cells are empty.
- Rows are skipped when any selected detail cell is a skip label: blank, `subtotal`, text beginning with `subtotal `, text containing `total project costs`, text containing `investment costs`, text containing `recurrent costs`, or exactly `detailed costs`.
- When the first selected detail cell is blank but lower-level detail cells are present, the first detail value is filled down from the previous valid detail start before the row is checked.
- Language-sensitive labels are centralized in `LANGUAGE_LABELS` in `costtab_pre_processor.html`; French and Spanish dictionaries are scaffolded but empty.
- In component mode, DT source tabs are assigned by component (`DT_1` for component 1, etc.) and optional source row start/end values limit source scanning.
- Numeric zero in a detail cell is treated as blank.
- Investment amounts accept numbers or numeric text with commas. Other text is treated as missing.
