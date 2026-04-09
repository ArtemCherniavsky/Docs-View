# Stimulsoft Reports.JS — FAQ

Quick answers for developers integrating Stimulsoft Reports.JS into web applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports.JS?](#1-how-do-i-install-stimulsoft-reportsjs)
2. [How do I show a report in the viewer?](#2-how-do-i-show-a-report-in-the-viewer)
3. [How do I load and render a report without the viewer?](#3-how-do-i-load-and-render-a-report-without-the-viewer)
4. [How do I design a new report?](#4-how-do-i-design-a-new-report)
5. [How do I open a specific report in the designer?](#5-how-do-i-open-a-specific-report-in-the-designer)
6. [How do I save a report changed in the designer?](#6-how-do-i-save-a-report-changed-in-the-designer)
7. [How do I load data from XML?](#7-how-do-i-load-data-from-xml)
8. [How do I load data from JSON?](#8-how-do-i-load-data-from-json)
9. [How do I set a variable value from code?](#9-how-do-i-set-a-variable-value-from-code)
10. [How do I export a report to PDF or HTML?](#10-how-do-i-export-a-report-to-pdf-or-html)
11. [How do I print a report?](#11-how-do-i-print-a-report)
12. [How do I save a rendered report to a file?](#12-how-do-i-save-a-rendered-report-to-a-file)
13. [How do I merge several reports into one?](#13-how-do-i-merge-several-reports-into-one)
14. [How do I customize the viewer?](#14-how-do-i-customize-the-viewer)
15. [How do I change the viewer or designer theme?](#15-how-do-i-change-the-viewer-or-designer-theme)
16. [How do I add a custom font to a report?](#16-how-do-i-add-a-custom-font-to-a-report)
17. [How do I add a custom function?](#17-how-do-i-add-a-custom-function)
18. [How do I use a custom data adapter?](#18-how-do-i-use-a-custom-data-adapter)
19. [How do I supply custom HTTP headers for a JSON database?](#19-how-do-i-supply-custom-http-headers-for-a-json-database)
20. [How do I load only the scripts I need (split bundles)?](#20-how-do-i-load-only-the-scripts-i-need-split-bundles)

---

## 1. How do I install Stimulsoft Reports.JS?

**Via npm:**
```bash
npm install stimulsoft-reports-js
```

**Via CDN** — add to the `<head>` of your HTML page:
```html
<!-- Core engine (always required) -->
<script src="https://cdn.stimulsoft.com/js/stimulsoft.reports.js"></script>

<!-- Add if you need the viewer -->
<script src="https://cdn.stimulsoft.com/js/stimulsoft.viewer.js"></script>

<!-- Add if you need the designer -->
<script src="https://cdn.stimulsoft.com/js/stimulsoft.designer.js"></script>
```

**From local files** (after downloading the package):
```html
<script src="/scripts/stimulsoft.reports.js" type="text/javascript"></script>
<script src="/scripts/stimulsoft.viewer.js" type="text/javascript"></script>
<script src="/scripts/stimulsoft.designer.js" type="text/javascript"></script>
```

> Only load the scripts you actually need. See [question 20](#20-how-do-i-load-only-the-scripts-i-need-split-bundles) for splitting the bundle to reduce page weight.

---

## 2. How do I show a report in the viewer?

```html
<!-- scripts -->
<script src="/scripts/stimulsoft.reports.js"></script>
<script src="/scripts/stimulsoft.viewer.js"></script>

<script type="text/javascript">
    // Create viewer and report
    var viewer = new Stimulsoft.Viewer.StiViewer(null, "StiViewer", false);
    var report = new Stimulsoft.Report.StiReport();

    // Load report template
    report.loadFile("reports/SimpleList.mrt");

    // Assign report — it will render automatically when the viewer renders
    viewer.report = report;

    // Render the viewer into a specific element
    viewer.renderHtml("viewerContent");
</script>

<!-- Viewer container -->
<div id="viewerContent"></div>
```

**Or render the viewer inline** (right where the script is placed):
```javascript
viewer.renderHtml(); // no argument — renders at the current script position
```

---

## 3. How do I load and render a report without the viewer?

```javascript
// Only stimulsoft.reports.js is needed — no viewer script required
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/SimpleList.mrt");

// Render using callback
report.renderAsync(function () {
    console.log("Rendered pages: " + report.renderedPages.count);
});

// Or using async/await
await report.renderAsync2();
console.log("Rendered pages: " + report.renderedPages.count);
```

---

## 4. How do I design a new report?

```html
<script src="/scripts/stimulsoft.reports.js"></script>
<script src="/scripts/stimulsoft.viewer.js"></script>
<script src="/scripts/stimulsoft.designer.js"></script>
<script src="/scripts/stimulsoft.blockly.editor.js"></script>

<script type="text/javascript">
    var designer = new Stimulsoft.Designer.StiDesigner(null, "StiDesigner", false);
    designer.renderHtml("designerContent");
</script>

<div id="designerContent"></div>
```

This opens the designer with an empty new report.

---

## 5. How do I open a specific report in the designer?

```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/SimpleList.mrt");

var options = new Stimulsoft.Designer.StiDesignerOptions();
options.appearance.fullScreenMode = true; // optional

var designer = new Stimulsoft.Designer.StiDesigner(options, "StiDesigner", false);
designer.report = report;
designer.renderHtml("designerContent");
```

---

## 6. How do I save a report changed in the designer?

Use the `onSaveReport` event to intercept the save action and get the report data:

```javascript
var designer = new Stimulsoft.Designer.StiDesigner(null, "StiDesigner", false);

// Handle save event
designer.onSaveReport = function (e) {
    // Save as JSON string — send to your server or store locally
    var jsonString = e.report.saveToJsonString();
    console.log("Report saved:", jsonString);

    // Example: POST to your backend
    fetch("/api/save-report", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: jsonString
    });
};

// Handle new report creation — register data so the designer can see it
designer.onCreateReport = function (e) {
    var ds = new Stimulsoft.System.Data.DataSet("Demo");
    ds.readJsonFile("reports/Demo.json");
    e.report.regData("Demo", "Demo", ds);
    e.report.dictionary.synchronize();
};
```

---

## 7. How do I load data from XML?

```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/TwoSimpleLists.mrt");

// Create DataSet and load XML schema + data
var dataSet = new Stimulsoft.System.Data.DataSet("Demo");
dataSet.readXmlSchemaFile("reports/Demo.xsd"); // optional but recommended
dataSet.readXmlFile("reports/Demo.xml");

// Remove connections from template and register your data
report.dictionary.databases.clear();
report.regData("Demo", "Demo", dataSet);

report.renderAsync(function () {
    console.log("Done, pages: " + report.renderedPages.count);
});
```

---

## 8. How do I load data from JSON?

**Option 1 — via DataSet (full control):**
```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/SimpleList.mrt");

var dataSet = new Stimulsoft.System.Data.DataSet("Demo");
dataSet.readJsonFile("reports/Demo.json");

report.dictionary.databases.clear();
report.regData("Demo", "Demo", dataSet);

report.renderAsync(function () {
    console.log("Done, pages: " + report.renderedPages.count);
});
```

**Option 2 — shorthand with dictionary sync:**
```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/SimpleListEmptyDictionary.mrt");

// Register JSON and synchronize dictionary in one call
report.regJsonFile("Demo", "reports/Demo.json", true); // true = synchronize

var viewer = new Stimulsoft.Viewer.StiViewer(null, "StiViewer", false);
viewer.report = report;
viewer.renderHtml("viewerContent");
```

---

## 9. How do I set a variable value from code?

```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/ReportWithVariable.mrt");

// Get variable by name and set its value
var variable = report.dictionary.variables.getByName("Variable1");
variable.value = "Hello, World!";

report.renderAsync(function () {
    // Export to HTML and display inline
    report.exportDocumentAsync(function (htmlString) {
        document.getElementById("reportContainer").innerHTML = htmlString;
    }, Stimulsoft.Report.StiExportFormat.Html);
});
```

Variable names are case-sensitive and must match the names defined in the report template.

---

## 10. How do I export a report to PDF or HTML?

Always call `renderAsync` before exporting.

**Export to PDF and download:**
```javascript
report.renderAsync(function () {
    report.exportDocumentAsync(function (pdfData) {
        var fileName = report.reportAlias;
        Stimulsoft.System.StiObject.saveAs(pdfData, fileName + ".pdf", "application/pdf");
    }, Stimulsoft.Report.StiExportFormat.Pdf);
});
```

**Export to PDF and open in a new tab:**
```javascript
report.renderAsync(function () {
    report.exportDocumentAsync(function (pdfData) {
        var blob = new Blob([new Uint8Array(pdfData)], { type: "application/pdf" });
        var url = URL.createObjectURL(blob);
        window.open(url);
    }, Stimulsoft.Report.StiExportFormat.Pdf);
});
```

**Export to HTML and display on the page:**
```javascript
report.renderAsync(function () {
    report.exportDocumentAsync(function (htmlString) {
        document.getElementById("reportContainer").innerHTML = htmlString;
    }, Stimulsoft.Report.StiExportFormat.Html);
});
```

**Export to HTML and download:**
```javascript
report.renderAsync(function () {
    report.exportDocumentAsync(function (htmlString) {
        Stimulsoft.System.StiObject.saveAs(htmlString, report.reportAlias + ".html", "text/html;charset=utf-8");
    }, Stimulsoft.Report.StiExportFormat.Html);
});
```

**Using async/await (modern syntax):**
```javascript
await report.renderAsync2();
var pdfData = await report.exportDocumentAsync2(Stimulsoft.Report.StiExportFormat.Pdf);
Stimulsoft.System.StiObject.saveAs(pdfData, "report.pdf", "application/pdf");
```

**Available export formats:**

| Constant | Format |
|----------|--------|
| `StiExportFormat.Pdf` | PDF |
| `StiExportFormat.Html` | HTML |
| `StiExportFormat.Excel2007` | Excel (.xlsx) |
| `StiExportFormat.Word2007` | Word (.docx) |
| `StiExportFormat.Csv` | CSV |
| `StiExportFormat.Text` | Plain text |
| `StiExportFormat.ImagePng` | PNG image |

---

## 11. How do I print a report?

```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadFile("reports/SimpleList.mrt");

report.renderAsync(function () {
    // Uses the browser's built-in print dialog
    report.print();
});
```

You can also trigger print from the viewer toolbar — it is available by default.

---

## 12. How do I save a rendered report to a file?

A rendered report is saved as a `.mdc` file (JSON format) so it can be loaded later without a data source.

**Save:**
```javascript
report.renderAsync(function () {
    var json = report.saveDocumentToJsonString();
    Stimulsoft.System.StiObject.saveAs(json, report.reportAlias + ".mdc", "application/json;charset=utf-8");
});
```

**Load a previously saved rendered report:**
```javascript
var report = new Stimulsoft.Report.StiReport();
report.loadDocumentFromJsonString(savedJsonString);

var viewer = new Stimulsoft.Viewer.StiViewer(null, "StiViewer", false);
viewer.report = report;
viewer.renderHtml("viewerContent");
```

---

## 13. How do I merge several reports into one?

**Using async/await (recommended):**
```javascript
var report1 = new Stimulsoft.Report.StiReport();
report1.loadFile("reports/SimpleList.mrt");

var report2 = new Stimulsoft.Report.StiReport();
report2.loadFile("reports/SimpleGroup.mrt");

// Create the target merged report
var merged = new Stimulsoft.Report.StiReport();
merged.reportUnit = report1.reportUnit;
report2.reportUnit = merged.reportUnit;

await merged.renderAsync2();
merged.renderedPages.clear();

await report1.renderAsync2();
report1.renderedPages.list.forEach(function (page) {
    merged.renderedPages.add(page);
    page.report = merged;
});

await report2.renderAsync2();
report2.renderedPages.list.forEach(function (page) {
    merged.renderedPages.add(page);
    page.report = merged;
});

// Show in viewer
viewer.report = merged;
```

---

## 14. How do I customize the viewer?

Use `StiViewerOptions` to configure the viewer before creating it:

```javascript
var options = new Stimulsoft.Viewer.StiViewerOptions();

// Appearance
options.appearance.scrollbarsMode = true;
options.appearance.pageBorderColor = Stimulsoft.System.Drawing.Color.navy;
options.appearance.theme = Stimulsoft.Viewer.StiViewerTheme.Office2022WhiteBlue;

// Toolbar
options.toolbar.showPrintButton = false;
options.toolbar.showDesignButton = true;
options.toolbar.showViewModeButton = false;
options.toolbar.viewMode = Stimulsoft.Viewer.StiWebViewMode.WholeReport;
options.toolbar.zoom = 75; // default zoom percentage
options.toolbar.borderColor = Stimulsoft.System.Drawing.Color.navy;

// Size
options.width = "100%";
options.height = "600px";

var viewer = new Stimulsoft.Viewer.StiViewer(options, "StiViewer", false);
```

**Viewer events:**
```javascript
// Intercept data loading to inject data from code
viewer.onBeginProcessData = function (args) {
    var ds = new Stimulsoft.System.Data.DataSet("Demo");
    ds.readJsonFile("reports/Demo.json");
    args.report.regData("Demo", "Demo", ds);
};

// Intercept export to override settings
viewer.onBeginExportReport = function (args) {
    if (args.format === "Html") {
        args.settings.zoom = 2; // 200% zoom for HTML export
    }
};

// Handle design button click
viewer.onDesignReport = function (args) {
    // Open your own designer or handle custom logic
};
```

---

## 15. How do I change the viewer or designer theme?

**Set theme at creation time (viewer):**
```javascript
var options = new Stimulsoft.Viewer.StiViewerOptions();
options.appearance.theme = Stimulsoft.Viewer.StiViewerTheme.Office2022WhiteBlue;
var viewer = new Stimulsoft.Viewer.StiViewer(options, "StiViewer", false);
```

**Switch theme at runtime:**
```javascript
viewer.setTheme(Stimulsoft.Viewer.StiViewerTheme.SimpleGray);
```

**Available viewer themes:**
- `Office2022WhiteBlue`
- `Office2013WhiteGreen`
- `SimpleGray`
- `WindowsXP`

**Localize the designer:**
```javascript
Stimulsoft.Base.Localization.StiLocalization.setLocalizationFile("localization/de.xml");
```

---

## 16. How do I add a custom font to a report?

```javascript
// Load font file as byte array
var fontContent = Stimulsoft.System.IO.File.getFile("reports/Roboto-Black.ttf", true);

// Create a font resource and add it to the report dictionary
var resource = new Stimulsoft.Report.Dictionary.StiResource(
    "Roboto-Black",   // name
    "Roboto-Black",   // alias
    false,
    Stimulsoft.Report.Dictionary.StiResourceType.FontTtf,
    fontContent
);
report.dictionary.resources.add(resource);

// Use the font in a text component
var text = new Stimulsoft.Report.Components.StiText();
text.clientRectangle = new Stimulsoft.System.Drawing.Rectangle(1, 1, 5, 1);
text.text = "Custom font text";
text.font = new Stimulsoft.System.Drawing.Font("Roboto-Black");
report.pages.getByIndex(0).components.add(text);
```

> Requires `StiOptions.WebServer.url` to be set if loading fonts via the server-side helper.

---

## 17. How do I add a custom function?

```javascript
// Define the function logic
var myFunction = function (value) {
    return value.toUpperCase();
};

// Register it in the report engine
Stimulsoft.Report.Dictionary.StiFunctions.addFunction(
    "MyCategory",                          // category name in the designer
    "MyFunction",                          // function name
    "MyFunction",                          // display name
    "Returns the string in uppercase",     // description
    "",                                    // help URL
    String,                                // return type
    "Uppercase string",                    // return description
    [String],                              // parameter types
    ["value"],                             // parameter names
    ["The text to convert"],               // parameter descriptions
    myFunction                             // implementation
);

// Use it in a report expression
// {MyFunction("hello, world")}  →  "HELLO, WORLD"
```

After registration the function is available in the designer's function list under "MyCategory".

---

## 18. How do I use a custom data adapter?

Register a custom database that handles connection testing, schema retrieval, and data retrieval:

```javascript
Stimulsoft.Report.Dictionary.StiCustomDatabase.registerCustomDatabase({
    serviceName: "MyDatabase",
    sampleConnectionString: "server=myserver;database=mydb",

    process: function (command, callback) {
        switch (command.command) {
            case "TestConnection":
                // Test the connection and report success/failure
                callback({ success: true });
                break;

            case "RetrieveSchema":
                // Return table/column metadata
                callback({
                    success: true,
                    data: {
                        Orders: [{ OrderId: 1, CustomerName: "Test" }]
                    },
                    types: {
                        Orders: { OrderId: "number", CustomerName: "string" }
                    }
                });
                break;

            case "RetrieveData":
                // Return data for the requested table (command.queryString)
                fetch("/api/data/" + command.queryString)
                    .then(r => r.json())
                    .then(data => callback({ success: true, data: data }));
                break;
        }
    }
});
```

> `data` and `types` are both optional — the engine infers types from whichever you provide.

---

## 19. How do I supply custom HTTP headers for a JSON database?

Use the `onBeginProcessData` event to inject headers before each data request:

```javascript
var report = new Stimulsoft.Report.StiReport();

report.onBeginProcessData = function (args) {
    if (
        args.database === "JSON" &&
        args.command === "GetData" &&
        args.pathData &&
        args.pathData.indexOf("/api/protected-data.json") >= 0
    ) {
        // Add any custom HTTP header
        args.headers.push({ key: "X-Auth-Token", value: "YOUR_TOKEN_HERE" });
        args.headers.push({ key: "Authorization", value: "Bearer " + getAccessToken() });
    }
};

report.loadFile("reports/ReportWithProtectedJson.mrt");
```

This works for both the report engine and viewer — `onBeginProcessData` fires before every data fetch.

---

## 20. How do I load only the scripts I need (split bundles)?

Instead of the single `stimulsoft.reports.js` bundle, load individual modules to reduce page weight:

```html
<!-- Core engine — always required -->
<script src="/scripts/stimulsoft.reports.engine.js"></script>

<!-- Include only what you use: -->

<!-- Charts and graphs -->
<script src="/scripts/stimulsoft.reports.chart.js"></script>

<!-- Export to PDF, Word, Excel, etc. (omit if only HTML export is needed) -->
<script src="/scripts/stimulsoft.reports.export.js"></script>

<!-- Import data from .xlsx files -->
<script src="/scripts/stimulsoft.reports.import.xlsx.js"></script>

<!-- Map components -->
<script src="/scripts/stimulsoft.reports.maps.js"></script>

<!-- Blockly visual script editor in the designer -->
<script src="/scripts/stimulsoft.blockly.editor.js"></script>
```

**Common combinations:**

| Scenario | Scripts needed |
|----------|---------------|
| Render + HTML export only | `engine.js` |
| Render + all exports | `engine.js` + `export.js` |
| Viewer (all features) | `engine.js` + `export.js` + `chart.js` + `viewer.js` |
| Designer (full) | `engine.js` + `export.js` + `chart.js` + `viewer.js` + `designer.js` + `blockly.editor.js` |

---

*For more details see the [official documentation](https://www.stimulsoft.com/en/documentation/online/programming-manual/reports_net.htm) and [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.JS-for-HTML).*
