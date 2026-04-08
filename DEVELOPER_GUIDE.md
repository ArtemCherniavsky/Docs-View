# Stimulsoft Reports.NET — Developer Guide

> **Product:** Stimulsoft Reports.NET for WinForms  
> **Supported frameworks:** .NET Framework 4.5.2+, .NET 6.0, .NET 7.0, .NET 8.0  
> **NuGet:** [Stimulsoft.Reports.Net](https://www.nuget.org/packages/Stimulsoft.Reports.Net)  
> **Official docs:** https://www.stimulsoft.com/en/documentation/online/programming-manual/reports_net.htm

---

## Table of Contents

1. [Getting Started](#getting-started)
   - [Installation](#installation)
   - [License Activation](#license-activation)
   - [Your First Report](#your-first-report)
2. [Architecture & Concepts](#architecture--concepts)
   - [Core Components](#core-components)
   - [Report Lifecycle](#report-lifecycle)
   - [Dictionary & Data Model](#dictionary--data-model)
   - [Rendering Engines](#rendering-engines)
3. [API Reference](#api-reference)
   - [StiReport](#stireport)
   - [Report Components](#report-components)
   - [Data Registration](#data-registration)
   - [Export Formats](#export-formats)
   - [Theming](#theming)
   - [Localization](#localization)
4. [How-To Guides](#how-to-guides)
   - [Show a Report in a Viewer](#show-a-report-in-a-viewer)
   - [Embed a Viewer Control in a Form](#embed-a-viewer-control-in-a-form)
   - [Connect Data from Code](#connect-data-from-code)
   - [Use Business Objects as Data Source](#use-business-objects-as-data-source)
   - [Create a Report Entirely from Code](#create-a-report-entirely-from-code)
   - [Work with Report Variables](#work-with-report-variables)
   - [Export a Report](#export-a-report)
   - [Merge Multiple Reports into One PDF](#merge-multiple-reports-into-one-pdf)
   - [Render and Export Asynchronously](#render-and-export-asynchronously)
   - [Open the Designer from Code](#open-the-designer-from-code)
   - [Add a Custom Font via Resources](#add-a-custom-font-via-resources)
   - [Implement a Custom Data Adapter](#implement-a-custom-data-adapter)
   - [Apply Drill-Down Interactivity](#apply-drill-down-interactivity)
   - [Localize the UI](#localize-the-ui)
   - [Change Viewer and Designer Theme](#change-viewer-and-designer-theme)
   - [Render in a Background Thread](#render-in-a-background-thread)
   - [Export to ZUGFeRD PDF](#export-to-zugferd-pdf)

---

## Getting Started

### Installation

Add the NuGet package to your project:

```bash
dotnet add package Stimulsoft.Reports.Win
```

Or via the Package Manager Console in Visual Studio:

```powershell
Install-Package Stimulsoft.Reports.Win
```

**Framework compatibility:**

| Package | Target Frameworks |
|---------|-------------------|
| `Stimulsoft.Reports.Win` | .NET Framework 4.5.2+, .NET 6.0, .NET 8.0 |

After installation, add the required using directives:

```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Components;
using Stimulsoft.Report.Viewer;
using Stimulsoft.Base;
```

---

### License Activation

Activate the license **before any Stimulsoft API call** — typically in the constructor of your main form or in `Program.cs`.

Three activation methods are supported:

```csharp
// Method 1: Embed the key string directly (suitable for development)
Stimulsoft.Base.StiLicense.Key = "YOUR_LICENSE_KEY_HERE";

// Method 2: Load from a .key file (recommended for production)
Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

// Method 3: Load from a stream (e.g., embedded resource)
using var stream = Assembly.GetExecutingAssembly()
    .GetManifestResourceStream("MyApp.license.key");
Stimulsoft.Base.StiLicense.LoadFromStream(stream);
```

> **Best practice:** For production deployments, store the license file as an embedded resource or in a protected directory. Never hardcode license strings in source control.

---

### Your First Report

The minimal example — load a `.mrt` template and show it:

```csharp
var report = new StiReport();
report.Load(@"Reports\MyReport.mrt");
report.Show();
```

That's it. Stimulsoft takes care of rendering, UI, and window management.

---

## Architecture & Concepts

### Core Components

Stimulsoft Reports.NET consists of several layers:

```
┌─────────────────────────────────────────────┐
│              Your Application               │
├─────────────────────────────────────────────┤
│   StiReport  ←  the central object          │
│   ├── Dictionary  (data sources, variables) │
│   ├── Pages[]    (layout and components)    │
│   └── RenderedPages[] (output after render) │
├─────────────────────────────────────────────┤
│  Rendering Engine (EngineV1 / EngineV2)     │
├──────────────┬──────────────────────────────┤
│   Viewer     │       Designer               │
│ (read-only)  │  (interactive template edit) │
└──────────────┴──────────────────────────────┘
```

**Key classes at a glance:**

| Class | Package | Purpose |
|-------|---------|---------|
| `StiReport` | `Stimulsoft.Report` | Root object for every report |
| `StiPage` | `Stimulsoft.Report.Components` | A single report page |
| `StiText` | `Stimulsoft.Report.Components` | Text / expression component |
| `StiDataBand` | `Stimulsoft.Report.Components` | Repeating data rows |
| `StiHeaderBand` | `Stimulsoft.Report.Components` | Page / group header |
| `StiFooterBand` | `Stimulsoft.Report.Components` | Page / group footer |
| `StiViewerControl` | `Stimulsoft.Report.Viewer` | Embeddable WinForms viewer |
| `StiResource` | `Stimulsoft.Report.Dictionary` | Embedded resource (font, image) |
| `StiOptions` | `Stimulsoft.Base` | Global settings |

---

### Report Lifecycle

A report goes through the following stages:

```
Load / Build  →  Compile  →  Render  →  Show / Export / Print
```

| Stage | Method | Notes |
|-------|--------|-------|
| **Load** | `report.Load(path)` | Reads a `.mrt` template from disk |
| **Load Document** | `report.LoadDocument(path/stream)` | Loads a pre-rendered `.mdc` document |
| **Compile** | `report.Compile()` | Compiles the report script; required before setting variables at runtime |
| **Render** | `report.Render()` | Processes data and generates `RenderedPages` |
| **Show** | `report.Show()` | Renders (if not yet rendered) and opens in a viewer window |
| **Export** | `report.ExportDocument(format, target)` | Exports rendered output to a file or stream |
| **Print** | `report.Print()` | Sends to the printer |

> `report.Show()` combines Render + open viewer in a single call. Use `report.Render()` explicitly only when you need to manipulate `RenderedPages` before display.

---

### Dictionary & Data Model

The **Dictionary** (`report.Dictionary`) is the metadata store for a report. It holds:

- **Databases** — connection configurations (SQL, NoSQL, custom)
- **Data Sources** — tables / views derived from databases
- **Relations** — foreign key links between data sources
- **Variables** — scalar values set from code or via UI prompts
- **Resources** — embedded binary assets (fonts, images)
- **Functions** — custom functions available in report expressions

```csharp
// Access the dictionary
var dict = report.Dictionary;

// Clear all database connections (useful before registering data manually)
dict.Databases.Clear();

// Register an in-memory DataSet
report.RegData("DatasetAlias", myDataSet);

// Synchronize — updates DataSources from registered data
dict.Synchronize();

// Add a resource
dict.Resources.Add(new StiResource("FontName", "Alias", false, StiResourceType.FontTtf, bytes, false));
```

---

### Rendering Engines

Stimulsoft Reports.NET supports two rendering engines:

| Engine | Enum Value | Description |
|--------|-----------|-------------|
| **EngineV1** | `StiEngineVersion.EngineV1` | Legacy engine; lower memory use in some edge cases |
| **EngineV2** | `StiEngineVersion.EngineV2` | Current default; better performance and feature support |

```csharp
var report = new StiReport();
report.EngineVersion = Stimulsoft.Report.Engine.StiEngineVersion.EngineV2; // default
```

> **Recommendation:** Always use EngineV2 for new projects. EngineV1 exists for backward compatibility.

---

## API Reference

### StiReport

```csharp
var report = new StiReport();
```

**Loading**

| Method | Signature | Description |
|--------|-----------|-------------|
| `Load` | `Load(string path)` | Load a template file (.mrt) |
| `Load` | `Load(Stream stream)` | Load a template from a stream |
| `LoadDocument` | `LoadDocument(string path)` | Load a rendered document (.mdc) |
| `LoadDocument` | `LoadDocument(Stream stream)` | Load a rendered document from stream |

**Rendering**

| Method | Description |
|--------|-------------|
| `Compile()` | Compile the report script (needed when setting variables) |
| `CompileAsync()` | Async version of Compile |
| `Render()` | Synchronously render the report |
| `RenderAsync()` | Asynchronously render the report |

**Display**

| Method | Description |
|--------|-------------|
| `Show()` | Render + show in a standalone viewer dialog |
| `ShowWithRibbonGUI()` | Show with a Ribbon-style toolbar |
| `Design()` | Open the report designer |

**Export & Print**

| Method | Signature | Description |
|--------|-----------|-------------|
| `ExportDocument` | `(StiExportFormat, Stream)` | Export to a stream |
| `ExportDocument` | `(StiExportFormat, string path)` | Export to a file |
| `ExportDocumentAsync` | `(StiExportFormat, string path)` | Async export to a file |
| `Print` | `Print()` | Print the report |

**Data**

| Method / Property | Description |
|-------------------|-------------|
| `RegData(string name, DataSet)` | Register a DataSet |
| `RegData(DataSet)` | Register a DataSet using its name |
| `RegData(string name, IEnumerable)` | Register a business object collection |
| `Dictionary` | Access the report dictionary |
| `report["VarName"]` | Get/set a report variable by name |

**Key Properties**

| Property | Type | Description |
|----------|------|-------------|
| `Pages` | `StiPagesCollection` | Collection of report pages |
| `RenderedPages` | `StiRenderedPagesCollection` | Pages after rendering |
| `ReportName` | `string` | Report name (used as default file name on export) |
| `ReportUnit` | `StiReportUnitType` | Measurement unit (cm, inches, HundredthsOfInch) |
| `EngineVersion` | `StiEngineVersion` | Rendering engine version |
| `ReportCacheMode` | `StiReportCacheMode` | Cache mode for large reports |
| `ScriptLanguage` | `StiReportLanguageType` | C# or VB.NET scripting |

---

### Report Components

All components live on a `StiPage` and are accessed via `page.Components`.

**Layout bands (top to bottom on a page)**

| Class | Description |
|-------|-------------|
| `StiReportTitleBand` | Printed once at the start of the report |
| `StiHeaderBand` | Page or group header |
| `StiDataBand` | Repeated for each data row |
| `StiGroupHeaderBand` | Group header |
| `StiGroupFooterBand` | Group footer |
| `StiFooterBand` | Page or report footer |
| `StiReportSummaryBand` | Printed once at the end of the report |
| `StiPageHeaderBand` | Printed at the top of every page |
| `StiPageFooterBand` | Printed at the bottom of every page |

**Content components**

| Class | Description |
|-------|-------------|
| `StiText` | Static text or expression (e.g., `{Customers.Name}`) |
| `StiImage` | Image from file, URL, or database field |
| `StiBarCode` | Barcode (supports many symbologies) |
| `StiChart` | Chart / graph component |
| `StiGauge` | Gauge indicator |
| `StiTable` | Table with fixed columns |
| `StiCrossTab` | Cross-tabulation (pivot) |
| `StiShape` | Rectangle, ellipse, line, and other shapes |

**Component placement**

Positions and sizes use `RectangleD` in the report's measurement units:

```csharp
// RectangleD(x, y, width, height)
var text = new StiText(new RectangleD(0, 0, 10, 1));
text.Text = "{DataSource.FieldName}";
text.HorAlignment = StiTextHorAlignment.Center;
band.Components.Add(text);
```

---

### Data Registration

**DataSet (XML / JSON / Database)**

```csharp
// From XML
var ds = new DataSet();
ds.ReadXmlSchema(@"Data\Demo.xsd");
ds.ReadXml(@"Data\Demo.xml");
report.RegData("Demo", ds);

// From JSON
var ds = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\Demo.json");
report.RegData("Demo", ds);

// Important: clear template connections before registering your own
report.Dictionary.Databases.Clear();
```

**Business Objects (IEnumerable)**

```csharp
var employees = GetEmployeeList(); // returns IEnumerable<Employee>
report.RegData("Employees", employees);
```

**Variables**

Report variables are defined in the designer. Set them from code after `Compile()`:

```csharp
report.Compile();
report["StartDate"] = DateTime.Today.AddMonths(-1);
report["EndDate"]   = DateTime.Today;
report["UserName"]  = currentUser.Name;
```

Variable values can be any serializable type: `string`, `int`, `bool`, `DateTime`, etc.

---

### Export Formats

Pass a `StiExportFormat` value to `ExportDocument`:

| Constant | Format | Extension |
|----------|--------|-----------|
| `StiExportFormat.Pdf` | PDF | `.pdf` |
| `StiExportFormat.Word2007` | Microsoft Word | `.docx` |
| `StiExportFormat.Excel2007` | Microsoft Excel | `.xlsx` |
| `StiExportFormat.Text` | Plain text | `.txt` |
| `StiExportFormat.Csv` | CSV | `.csv` |
| `StiExportFormat.Html` | HTML | `.html` |
| `StiExportFormat.Html5` | HTML5 | `.html` |
| `StiExportFormat.Rtf` | RTF | `.rtf` |
| `StiExportFormat.Ods` | OpenDocument Spreadsheet | `.ods` |
| `StiExportFormat.Odt` | OpenDocument Text | `.odt` |
| `StiExportFormat.Xml` | XML | `.xml` |
| `StiExportFormat.Json` | JSON | `.json` |
| `StiExportFormat.ImagePng` | PNG image (one file per page) | `.png` |
| `StiExportFormat.ImageJpeg` | JPEG image | `.jpg` |
| `StiExportFormat.ImageTiff` | TIFF | `.tiff` |
| `StiExportFormat.ImageSvg` | SVG | `.svg` |

**Export with options:**

```csharp
var pdfSettings = new Stimulsoft.Report.Export.StiPdfExportSettings
{
    EmbeddedFonts = true,
    ImageQuality  = 0.9f
};
report.ExportDocument(StiExportFormat.Pdf, stream, pdfSettings);
```

---

### Theming

Available only for .NET Framework 4.7.2 projects (WinForms designer / viewer).

```csharp
using Stimulsoft.Base.Theme;

// Set appearance mode
StiUXTheme.ApplyNewTheme(StiThemeAppearance.Dark, StiThemeAccentColor.Blue);

// Available appearance values
// StiThemeAppearance.Auto    — follows system setting
// StiThemeAppearance.Light
// StiThemeAppearance.Dark

// Available accent colors
// StiThemeAccentColor.Auto, Blue, Violet, Carmine, Teal, Green, Orange

// Read current state
var currentMode  = StiUXTheme.Appearance;
var currentColor = StiUXTheme.AccentColor;
```

---

### Localization

```csharp
using Stimulsoft.Base.Localization;

// Load a localization file before showing a report
StiOptions.Localization.Load(@"Localization\de.xml");

// Official localization files ship with the product.
// Custom files follow the same XML schema.
```

---

## How-To Guides

### Show a Report in a Viewer

Open a standalone viewer dialog — the simplest way to display a report.

```csharp
var report = new StiReport();
report.Load(@"Reports\Invoice.mrt");
report.Show();

// Alternative: Ribbon-style toolbar
// report.ShowWithRibbonGUI();
```

`Show()` internally calls `Render()` if the report has not been rendered yet.

---

### Embed a Viewer Control in a Form

Place `StiViewerControl` on your form in the Designer, then bind a report to it at runtime.

```csharp
// In Form Designer: drag StiViewerControl onto the form (stiViewerControl1)

private void LoadReportButton_Click(object sender, EventArgs e)
{
    var report = new StiReport();
    report.Load(@"Reports\Sales.mrt");
    report.Render();                        // Must render before assigning
    stiViewerControl1.Report = report;      // Bind
}
```

The control provides a full toolbar (print, export, navigation, search) out of the box.

---

### Connect Data from Code

Override the data connections defined in a template with your own DataSet.

```csharp
// Load data
var dataSet = new DataSet();
dataSet.ReadXmlSchema(@"Data\Schema.xsd");
dataSet.ReadXml(@"Data\Data.xml");

// Or from JSON
// var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\data.json");

var report = new StiReport();
report.Load(@"Reports\Report.mrt");

// Remove embedded database connections from the template
report.Dictionary.Databases.Clear();

// Register your data under the alias the template expects
report.RegData("Demo", dataSet);

report.Show();
```

> The string `"Demo"` must match the data source alias in the `.mrt` template.

---

### Use Business Objects as Data Source

Register typed .NET collections directly — no database or DataSet required.

```csharp
public class Employee
{
    public int    Id   { get; set; }
    public string Name { get; set; }
    public string Dept { get; set; }
}

// IEnumerable<T>
IEnumerable<Employee> employees = GetEmployees();
report.RegData("Employees", employees);
report.Load(@"Reports\EmployeeList.mrt");
report.Show();

// ITypedList  (gives the designer full column metadata at design time)
ITypedList typedList = GetTypedEmployeeList();
report.RegData("Employees", typedList);
```

In the template, refer to fields as `{Employees.Name}`, `{Employees.Dept}`, etc.

---

### Create a Report Entirely from Code

Build the full report structure without a `.mrt` file.

```csharp
var report = new StiReport();

// Register data
var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\Customers.json");
report.RegData(dataSet);
report.Dictionary.Synchronize();    // Discover table/column names

var page = report.Pages[0];

// --- Header ---
var header = new StiHeaderBand { Height = 0.5, Name = "Header" };
page.Components.Add(header);

var headerText = new StiText(new RectangleD(0, 0, 10, 0.5))
{
    Name         = "HeaderText",
    Text         = "Customer List",
    HorAlignment = StiTextHorAlignment.Center,
    Brush        = new StiSolidBrush(Color.SteelBlue),
    TextBrush    = new StiSolidBrush(Color.White)
};
header.Components.Add(headerText);

// --- Data Band ---
var dataBand = new StiDataBand
{
    DataSourceName = "Customers",   // Table name from the DataSet
    Height         = 0.5,
    Name           = "DataBand"
};
page.Components.Add(dataBand);

var nameText = new StiText(new RectangleD(0, 0, 5, 0.5))
{
    Text = "{Line}. {Customers.CompanyName}",
    Name = "NameText"
};
dataBand.Components.Add(nameText);

// --- Footer ---
var footer = new StiFooterBand { Height = 0.5, Name = "Footer" };
page.Components.Add(footer);

var countText = new StiText(new RectangleD(0, 0, 10, 0.5))
{
    Text         = "Total: {Count()}",
    HorAlignment = StiTextHorAlignment.Right,
    Name         = "CountText"
};
footer.Components.Add(countText);

report.Show();
```

---

### Work with Report Variables

Variables are declared in the designer. Set their values from code before rendering.

```csharp
var report = new StiReport();
report.Load(@"Reports\FilteredReport.mrt");

// Compile is required before setting variables
report.Compile();

// Set variables by name (case-sensitive, matches designer name)
report["DateFrom"]   = new DateTime(2024, 1, 1);
report["DateTo"]     = DateTime.Today;
report["Department"] = "Sales";
report["IsActive"]   = true;

report.Show();
```

**Reading a variable value:**

```csharp
var currentValue = report["Department"];
```

---

### Export a Report

Export to any supported format via `ExportDocument`.

```csharp
var report = new StiReport();
report.Load(@"Reports\Invoice.mrt");
report.Render();    // Must render before exporting

// --- Export to stream ---
var stream = new MemoryStream();
report.ExportDocument(StiExportFormat.Pdf, stream);
stream.Seek(0, SeekOrigin.Begin);

// Send stream to HTTP response, save to DB, etc.

// --- Export directly to file ---
report.ExportDocument(StiExportFormat.Excel2007, @"C:\Exports\report.xlsx");

// --- Export with settings ---
var settings = new Stimulsoft.Report.Export.StiPdfExportSettings
{
    EmbeddedFonts    = true,
    UseUnicode       = true,
    ExportRtfTextAsImage = false
};
report.ExportDocument(StiExportFormat.Pdf, stream, settings);
```

---

### Merge Multiple Reports into One PDF

Combine pages from several reports into a single PDF file.

```csharp
var merged = new StiReport();
merged.ReportCacheMode = StiReportCacheMode.On;
merged.RenderedPages.CanUseCacheMode = true;
merged.RenderedPages.CacheMode = true;
merged.RenderedPages.Clear();
merged.ReportUnit = StiReportUnitType.HundredthsOfInch;

var temp = new StiReport();

foreach (var reportPath in reportPaths)   // e.g., a list of .mdc files
{
    temp.LoadDocument(reportPath);

    foreach (StiPage page in temp.RenderedPages)
    {
        page.Report = temp;
        page.Guid   = Guid.NewGuid().ToString("N");
        merged.RenderedPages.Add(page);
    }
}

merged.ExportDocument(StiExportFormat.Pdf, @"C:\Output\merged.pdf");
```

> Set `ReportCacheMode = StiReportCacheMode.On` to avoid out-of-memory errors when merging many large reports.

---

### Render and Export Asynchronously

Avoid blocking the UI thread for long-running renders.

```csharp
public partial class MainForm : Form
{
    private StiReport _report;

    public MainForm()
    {
        InitializeComponent();
        _report = new StiReport();
        _report.Load(@"Reports\LargeReport.mrt");
    }

    private async void RenderButton_Click(object sender, EventArgs e)
    {
        statusLabel.Text = "Rendering...";
        RenderButton.Enabled = false;

        await _report.CompileAsync();
        await _report.RenderAsync();

        statusLabel.Text = "Done.";
        RenderButton.Enabled = true;
    }

    private async void ExportButton_Click(object sender, EventArgs e)
    {
        if (saveDialog.ShowDialog() != DialogResult.OK) return;

        statusLabel.Text = "Exporting...";
        await _report.ExportDocumentAsync(StiExportFormat.Pdf, saveDialog.FileName);
        statusLabel.Text = "Exported.";
    }
}
```

> `RenderAsync` / `ExportDocumentAsync` free the UI thread and integrate naturally with async/await patterns.

---

### Open the Designer from Code

Let users edit report templates at runtime.

```csharp
// Open a blank designer
var report = new StiReport();
report.Design();

// Open an existing template in the designer
var report = new StiReport();
report.Load(@"Reports\MyReport.mrt");
report.Design();

// React to save events
report.StiReportDesigner.EventSave += (sender, args) =>
{
    // args.Report contains the saved report object
    // Persist it, update the path, etc.
    var savedReport = (StiReport)sender;
    savedReport.Save(@"Reports\MyReport.mrt");
};
report.Design();
```

---

### Add a Custom Font via Resources

Embed a TTF font into the report so it renders correctly on any machine.

```csharp
using Stimulsoft.Report.Dictionary;

// 1. Load font bytes
byte[] fontBytes = File.ReadAllBytes(@"Fonts\Roboto-Bold.ttf");

// 2. Create and register a resource in the report dictionary
var resource = new StiResource(
    "Roboto-Bold",            // Resource name (unique key)
    "Roboto-Bold",            // Alias (display name)
    false,
    StiResourceType.FontTtf,
    fontBytes,
    false
);
report.Dictionary.Resources.Add(resource);

// 3. Register the font in Stimulsoft's font collection
StiFontCollection.AddResourceFont(
    resource.Name,
    resource.Content,
    "ttf",
    resource.Alias
);

// 4. Use the font in a component
var text = new StiText(new RectangleD(0, 0, 5, 1));
text.Font = StiFontCollection.CreateFont("Roboto-Bold", 14, FontStyle.Regular);
page.Components.Add(text);
```

> Embedding fonts in the report dictionary ensures the font travels with the `.mrt` file.

---

### Implement a Custom Data Adapter

Register a custom database type so it appears in the designer's data source wizard.

**Step 1 — Define the database descriptor:**

```csharp
public class MyDatabase : StiDatabase
{
    public MyDatabase() : base() { }
    public MyDatabase(string name, string connectionString) 
        : base(name, connectionString) { }

    public override StiDatabase CreateNew() => new MyDatabase();
}
```

**Step 2 — Implement the adapter service:**

```csharp
public class MyAdapterService : StiAdapterService
{
    public override Type GetDatabaseType() => typeof(MyDatabase);

    public override DataSet GetDataSet(StiDatabase database,
        StiDictionary dictionary, bool allowConnecting)
    {
        var db = (MyDatabase)database;
        // ... open connection, execute queries, return DataSet
        return myDataSet;
    }
}
```

**Step 3 — Register at application startup:**

```csharp
// Optionally clear built-in adapters
StiOptions.Services.Databases.Clear();

// Add your adapter
StiOptions.Services.Databases.Add(new MyDatabase());
StiOptions.Services.DataAdapters.Add(new MyAdapterService());
```

**Step 4 — Add a connection to a report:**

```csharp
var report = new StiReport();
var db = new MyDatabase("ConnectionName", "server=...;database=...;");
report.Dictionary.Databases.Add(db);
report.Design();
```

---

### Apply Drill-Down Interactivity

Open a detail report when a user clicks a component in the main report.

```csharp
private DataSet _dataSet;

private void ShowMasterReport()
{
    _dataSet = LoadData();

    var report = new StiReport();
    report.RegData(_dataSet);
    report.Load(@"Reports\Master.mrt");
    report.Compile();

    // Subscribe to click events on the compiled report
    report.CompiledReport.Click += OnReportComponentClick;
    report.Show();
}

private void OnReportComponentClick(object sender, EventArgs e)
{
    var component = sender as StiComponent;
    var customerId = component?.BookmarkValue as string;
    if (customerId == null) return;

    // Build and filter the detail report
    var detail = new StiReport();
    detail.RegData(_dataSet);
    detail.Load(@"Reports\Detail.mrt");

    var dataBand = (StiDataBand)detail.Pages["Page1"].Components["OrdersBand"];
    dataBand.Filters.Add(new StiFilter($"{{Orders.CustomerID==\"{customerId}\"}}"));

    detail.Show();
}
```

> `BookmarkValue` is set in the designer on the clickable component. It carries the key value used for filtering.

---

### Localize the UI

Change the language of the viewer and designer toolbars and menus.

```csharp
using Stimulsoft.Base.Localization;

// Load a localization file (XML format)
StiOptions.Localization.Load(@"Localization\ru.xml");

// Now any report.Show() or report.Design() will use the loaded language
var report = new StiReport();
report.Load(@"Reports\MyReport.mrt");
report.Show();
```

**Populating a language selector from a folder of localization files:**

```csharp
foreach (var file in Directory.GetFiles(@"Localization", "*.xml"))
{
    languageCombo.Items.Add(file);
}
languageCombo.SelectedIndexChanged += (s, e) =>
{
    StiOptions.Localization.Load(languageCombo.SelectedItem.ToString());
};
```

---

### Change Viewer and Designer Theme

*(Available in .NET Framework 4.7.2 projects)*

```csharp
using Stimulsoft.Base.Theme;

// Apply a theme (call before showing any report UI)
StiUXTheme.ApplyNewTheme(StiThemeAppearance.Dark, StiThemeAccentColor.Violet);

// Read current settings
Console.WriteLine(StiUXTheme.Appearance);   // Auto | Light | Dark
Console.WriteLine(StiUXTheme.AccentColor);  // Auto | Blue | Violet | Carmine | Teal | Green | Orange
```

**HiDPI support** — add this to `Main()` for sharp rendering on high-resolution displays:

```csharp
[STAThread]
static void Main()
{
    Stimulsoft.Report.Win.StiDpiAwarenessHelper.SetPerMonitorDpiAware();
    Application.EnableVisualStyles();
    Application.SetCompatibleTextRenderingDefault(false);
    Application.Run(new MainForm());
}
```

---

### Render in a Background Thread

Show a progress indicator while rendering a complex report.

```csharp
private void RenderWithProgress(StiReport report)
{
    var worker = new BackgroundWorker { WorkerReportsProgress = true };

    worker.DoWork += (s, e) =>
    {
        report.Render();
    };

    worker.RunWorkerCompleted += (s, e) =>
    {
        progressBar.Visible = false;
        report.Show(); // Show on UI thread after render completes
    };

    progressBar.Visible = true;
    worker.RunWorkerAsync();
}
```

> Prefer the `async/await` approach (`RenderAsync`) in new .NET 6+ projects.

---

### Export to ZUGFeRD PDF

*(Available in .NET Framework 4.7.2 projects)*

ZUGFeRD is a German e-invoice standard that embeds structured XML inside a PDF/A-3 document.

```csharp
var report = new StiReport();
report.Load(@"Reports\Invoice.mrt");
report.Render();

var settings = new Stimulsoft.Report.Export.StiPdfExportSettings
{
    // ZUGFeRD v2.1 settings
    ZugferdCompliance = Stimulsoft.Report.Export.StiPdfZugferdCompliance.ZUGFeRD2p1,
    ZugferdXml        = File.ReadAllText(@"Data\invoice.xml")
};

report.ExportDocument(StiExportFormat.Pdf, @"C:\Output\invoice-zugferd.pdf", settings);
```

---

## Additional Resources

| Resource | URL |
|----------|-----|
| Official documentation | https://www.stimulsoft.com/en/documentation/online/programming-manual/reports_net.htm |
| Live demo | http://demo.stimulsoft.com/#Net |
| NuGet package | https://www.nuget.org/packages/Stimulsoft.Reports.Net |
| Free download | https://www.stimulsoft.com/en/downloads |
| Product page | https://www.stimulsoft.com/en/products/reports-net |
| GitHub samples | https://github.com/stimulsoft/Samples-Reports.NET-for-WinForms |

---

*Generated from source code analysis of the official WinForms samples repository.*
