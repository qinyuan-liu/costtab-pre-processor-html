# Costtab Pre-Processor

Local browser tool for converting raw costtab workbooks into formatted costtab workbooks.
**All Excel processing runs locally in the browser. The workbook is not uploaded anywhere.**

## Use

1. Open `costtab_pre_processor.html` in a browser.
2. Upload the raw costtab workbook.
3. Enter the Project ID.
4. Select the summary tab and component/subcomponent columns when a summary tab is available.
5. Review and edit the generated mapping rows. Source tabs, detail columns, investment columns, and units may be auto-filled when the workbook structure is recognized.
6. Click `Check`.
7. Click `Generate formatted costtab`.

Uploading a new workbook clears the previous project ID, summary column inputs, investment measure selection, mapping rows, and validation state.

## Files

- `costtab_pre_processor.html` - the tool.
- `vendor/xlsx.full.min.js` - local SheetJS dependency for reading Excel files.
- `vendor/exceljs.min.js` - local ExcelJS dependency for writing styled Excel files.

Keep the `vendor` folder next to the HTML file.

## Workbook Setup

- Summary tab candidates are workbook sheets whose names include `COM`; `COMFIN` is preferred when present.
- If no summary tab exists, the tool shows: `No component financing summary found. Please flag this in the tracking file and skip this project.`
- Component and subcomponent text can be loaded from the summary tab by entering the component and subcomponent column letters.
- If no summary tab is available, component/subcomponent text can be filled from source tabs by entering source column letters in the mapping table.
- Source tabs are auto-assigned from `DT_` sheets. In subcomponent mode, DT tabs are assigned sequentially to subcomponents. In component mode, `DT_1` is assigned to component 1, `DT_2` to component 2, and so on.

## Auto-Fill Defaults

- Detail start/end defaults are detected by scanning the top-left area of each source sheet for `Investment Costs`, then looking below it for the first detail row.
- The default detail range starts at the first non-empty detail cell and initially spans up to five columns. If a `Unit` column is detected within that range, the unit column and following columns are excluded.
- Investment column and unit defaults are detected from the selected investment measure and currency. For example, `Totals Including Contingencies` plus `USD` looks for the matching `(US$ ...)` group and its `Total` column.
- With `Base Cost` mode, `Base Cost` is only auto-filled when a matching `Base Cost` group is found; otherwise the auto-filled investment column and unit stay blank.
- Auto-filled defaults can be manually overridden. The tool only replaces a default when the field is empty or still equals the previous auto-filled value.

## Validation Logic

The `Check` button runs the same validation that is run before `Generate formatted costtab`.
Generation stays disabled while any blocking error exists.

Blocking errors:

- A raw costtab workbook must be selected.
- Project ID is normalized to digits only and must be present in the embedded project list.
- If a summary tab is selected, the component summary column must be a non-empty Excel column letter, for example `A`, `D`, or `AA`.
- If a summary tab is selected in subcomponent mode, the subcomponent summary column is optional, but must be Excel column letters when provided.
- In component mode, the top-level subcomponent column input is hidden and subcomponent text is not checked. The former subcomponent level in the DT sheet should be treated as the detail start level.
- At least one active component/subcomponent mapping is required. A mapping is active when the component has text, or when a subcomponent has text, an investment column, a custom currency, a unit, or a source tab. Editing only detail start/end does not make an otherwise empty row active.
- Component text is required for every active component and must contain at least 2 words. A leading marker such as `1.` or `A)` is ignored for the word count.
- Detail start, detail end, and investment columns are required for every active subcomponent and must be Excel column letters only.
- Detail end must not be before detail start.
- If currency is `Other`, the custom currency field is required.
- Unit is required. Supported units in the UI are `million`, `thousand ('000)`, and `unit (1)`.
- Source tab is required.
- For each valid source tab mapping, data is scanned from row 6 onward. Rows are ignored when all selected detail cells are empty, when a selected detail cell is a skip label, or when the investment column is empty or non-numeric. Any remaining row is an error if the first detail column is empty.

Non-blocking warnings:

- If the number of detail columns from detail start through detail end is not exactly 4, the tool warns: `There are N detail columns. Is it correct? Please recheck the costtab and make sure.` Generation is still allowed if there are no blocking errors.
- Empty subcomponent text warns: `The subcomponent text is empty. Please make sure there is no subcomponent available.`
- Investment values of `0` are skipped with a warning: `Please make sure it is correct.`
- If the investment measure is changed to `Base Cost`, the tool shows `Please flag this in the tracking file`.
- If more than 70% of a component or subcomponent's tokens are made up of `project management`, `knowledge management`, `KM`, `monitoring and evaluation`, or `M&E`, the tool suggests removing it unless it includes costs beyond pure project management.
