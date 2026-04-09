# Stimulsoft Reports.AVALONIA — FAQ

Quick answers for developers integrating Stimulsoft Reports into Avalonia UI desktop applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports for Avalonia?](#1-how-do-i-install-stimulsoft-reports-for-avalonia)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Viewer?](#3-how-to-show-a-report-in-the-viewer)
4. [How to connect data to a report?](#4-how-to-connect-data-to-a-report)
5. [How to create a report at runtime?](#5-how-to-create-a-report-at-runtime)
6. [How to set report variables in code?](#6-how-to-set-report-variables-in-code)
7. [How to export a report from code?](#7-how-to-export-a-report-from-code)
8. [How to customize the Viewer?](#8-how-to-customize-the-viewer)
9. [How to change the theme (Light/Dark)?](#9-how-to-change-the-theme-lightdark)
10. [How to globalize a report?](#10-how-to-globalize-a-report)
11. [How to use business objects in a report?](#11-how-to-use-business-objects-in-a-report)
12. [How to manage sub-reports?](#12-how-to-manage-sub-reports)

---

## 1. How do I install Stimulsoft Reports for Avalonia?

Install the NuGet package:

```
dotnet add package Stimulsoft.Reports.Avalonia
```

**Supported frameworks:** .NET 6.0, .NET 8.0.

The project requires Avalonia UI packages (`Avalonia`, `Avalonia.Desktop`, `Avalonia.Themes.Fluent`). A minimal `.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Avalonia" Version="11.1.3" />
    <PackageReference Include="Avalonia.Desktop" Version="11.1.3" />
    <PackageReference Include="Avalonia.Themes.Fluent" Version="11.1.3" />
    <PackageReference Include="Stimulsoft.Reports.Avalonia" Version="2026.1.7" />
  </ItemGroup>
</Project>
```

In the `MainWindow` constructor, initialize the Avalonia mode and apply the theme:

```csharp
StiOptions.Configuration.IsAvalonia = true;

Application.Current.RequestedThemeVariant = ThemeVariant.Light;
StiAvaloniaTheme.ApplyNewTheme(StiAvaloniaAppThemeAppearance.Light);
```

---

## 2. How to activate the license?

Use one of the methods in `Stimulsoft.Base.StiLicense` before creating reports:

```csharp
// Option 1: set the key as a string
Stimulsoft.Base.StiLicense.Key = "6vJhGtLLLz2GNviWmUTrhSqnO...";

// Option 2: load from a file
Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

// Option 3: load from a stream
Stimulsoft.Base.StiLicense.LoadFromStream(stream);
```

Set the license once at application startup, before any report operations.

---

## 3. How to show a report in the Viewer?

Create an `StiViewerControl`, load a report, and display it in a window:

```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Viewer.Avalonia.Viewer;

var report = new StiReport();
var stream = AssetLoader.Open(new Uri("avares://MyApp/Reports/SimpleList.mrt"));
report.Load(stream);
report.CalculationMode = StiCalculationMode.Interpretation;

var window = new Window
{
    WindowState = WindowState.Maximized,
    Content = new StiViewerControl
    {
        Report = report
    }
};

await window.ShowDialog<bool?>(desktop.MainWindow);
```

Reports are loaded from Avalonia assets using `AssetLoader.Open()` with a URI in the format `avares://AssemblyName/Path/Report.mrt`. You can also load from a file path using `report.Load(filePath)`.

To embed the Viewer directly in a window, place `StiViewerControl` in AXAML:

```xml
<viewer:StiViewerControl x:Name="viewerControl" />
```

And assign the report in code-behind:

```csharp
viewerControl.Report = report;
```

---

## 4. How to connect data to a report?

Register data using `report.RegData()` and synchronize the dictionary:

```csharp
var dataSet = new DataSet();
dataSet.ReadXmlSchema(xsdStream);
dataSet.ReadXml(xmlStream);
dataSet.DataSetName = "Demo";

var report = new StiReport();
report.RegData(dataSet);
report.Dictionary.Synchronize();
```

Data can be loaded from XML files, databases, or any `DataSet`/`DataTable` source. Call `Dictionary.Synchronize()` when creating reports at runtime so the report dictionary picks up the registered data sources.

---

## 5. How to create a report at runtime?

Build report components programmatically using `StiHeaderBand`, `StiDataBand`, `StiFooterBand`, and `StiText`:

```csharp
var report = new StiReport();
report.RegData(dataSet);
report.Dictionary.Synchronize();

var page = report.Pages[0];

// Header
var headerBand = new StiHeaderBand();
headerBand.Height = 0.5;
headerBand.Name = "HeaderBand";
page.Components.Add(headerBand);

var headerText = new StiText(new RectangleD(0, 0, 5, 0.5));
headerText.Text = "CompanyName";
headerText.HorAlignment = StiTextHorAlignment.Center;
headerText.Brush = new StiSolidBrush(Color.LightGreen);
headerBand.Components.Add(headerText);

// Data band
var dataBand = new StiDataBand();
dataBand.DataSourceName = "Customers";
dataBand.Height = 0.5;
page.Components.Add(dataBand);

var dataText = new StiText(new RectangleD(0, 0, 5, 0.5));
dataText.Text = "{Line}.{Customers.CompanyName}";
dataBand.Components.Add(dataText);

// Footer
var footerBand = new StiFooterBand();
footerBand.Height = 0.5;
page.Components.Add(footerBand);

var footerText = new StiText(new RectangleD(0, 0, 5, 0.5));
footerText.Text = "Count - {Count()}";
footerText.HorAlignment = StiTextHorAlignment.Right;
footerBand.Components.Add(footerText);

report.CalculationMode = StiCalculationMode.Interpretation;
```

You can also create CrossTab reports at runtime using `StiCrossTab`, `StiCrossRow`, `StiCrossColumn`, and `StiCrossSummary` components.

---

## 6. How to set report variables in code?

Set variable values using the indexer `report["VariableName"]` after loading the report template:

```csharp
var report = new StiReport();
report.Load(stream);
report.CalculationMode = StiCalculationMode.Interpretation;

report["Name"] = "Maria";
report["Surname"] = "Anders";
report["Email"] = "m.anders@stimulsoft.com";
report["Address"] = "Obere Str. 57, Berlin";
report["Sex"] = true;
report["BirthDay"] = new DateTime(1982, 3, 20);
```

Variable value types must match the types defined in the report template. Variables can be populated from UI controls (TextBox, DatePicker, RadioButton, etc.) before displaying the report.

---

## 7. How to export a report from code?

First render the report with `RenderWithWpf()`, then use the async export methods on `StiViewerControl`:

```csharp
var report = new StiReport();
report.Load(stream);
report.CalculationMode = StiCalculationMode.Interpretation;
report.RenderWithWpf(false);

// Export to PDF
await StiViewerControl.ExportPdfAsync(report);

// Export to HTML
await StiViewerControl.ExportHtmlAsync(report);

// Export to Excel
await StiViewerControl.ExportExcelAsync(report);

// Export to Text
await StiViewerControl.ExportTxtAsync(report);

// Export to RTF
await StiViewerControl.ExportRtfAsync(report);
```

The export methods are static async methods on `StiViewerControl`. Available formats: PDF, HTML, Excel, Text, RTF.

---

## 8. How to customize the Viewer?

Control the visibility of Viewer UI elements through properties on `StiViewerControl`:

```csharp
var viewer = new StiViewerControl();

// Report operations
viewer.ShowReportOpen = true;
viewer.ShowReportSave = true;
viewer.ShowReportSaveDocument = true;

// Page navigation
viewer.ShowPageFirst = true;
viewer.ShowPagePrevious = true;
viewer.ShowPageGoTo = true;
viewer.ShowPageNext = true;
viewer.ShowPageLast = true;
viewer.ShowPageNew = false;
viewer.ShowPageDelete = false;

// Panels
viewer.ShowBookmarks = true;
viewer.ShowParameters = true;

// Tools
viewer.ShowToolEditor = true;
viewer.ShowToolFind = true;
viewer.ShowSignature = true;

// UI elements
viewer.ShowClose = true;
viewer.ShowStatusBar = true;
viewer.ShowHorScrollBar = true;
viewer.ShowVertScrollBar = true;
```

All properties can be bound to AXAML controls (e.g., CheckBox) for dynamic toggling at runtime.

---

## 9. How to change the theme (Light/Dark)?

Use `StiAvaloniaTheme.ApplyNewTheme()` together with Avalonia's `RequestedThemeVariant`:

```csharp
// Light theme
Application.Current.RequestedThemeVariant = ThemeVariant.Light;
StiAvaloniaTheme.ApplyNewTheme(StiAvaloniaAppThemeAppearance.Light);

// Dark theme
Application.Current.RequestedThemeVariant = ThemeVariant.Dark;
StiAvaloniaTheme.ApplyNewTheme(StiAvaloniaAppThemeAppearance.Dark);
```

Both the Avalonia theme and the Stimulsoft theme must be set together. The theme can be switched at runtime — for example, via a ComboBox selection handler. The current theme can be checked with `StiAvaloniaTheme.Appearance`.

---

## 10. How to globalize a report?

Implement `IStiGlobalizationManager` and assign it to the report's `GlobalizationManager` property:

```csharp
using System.Globalization;

var report = new StiReport();
report.Load(stream);
report.RegData(dataSet);

report.GlobalizationManager = new GlobalizationManager(
    "MyApp.MyResources",
    new CultureInfo("fr-FR")
);

report.CalculationMode = StiCalculationMode.Interpretation;
```

The `GlobalizationManager` class uses `ResourceManager` to load culture-specific strings. Supported cultures can include `fr-FR`, `de-DE`, `it-IT`, `ru-RU`, `es-ES`, `en-GB`, `en-US`, and any other .NET culture. Localized strings are stored in resource files (`.resx`) and referenced by the report components.

---

## 11. How to use business objects in a report?

Register business objects with `report.RegData()` or `report.RegBusinessObject()`:

**Using IEnumerable:**

```csharp
var report = new StiReport();
report.RegData("EmployeeIEnumerable", CreateBusinessObjectsIEnumerable.GetEmployees());
report.Load(stream);
report.CalculationMode = StiCalculationMode.Interpretation;
```

**Using ITypedList:**

```csharp
var report = new StiReport();
report.RegData("EmployeeITypedList", CreateBusinessObjectsITypedList.GetEmployees());
report.Load(stream);
report.CalculationMode = StiCalculationMode.Interpretation;
```

**Using LINQ queries:**

```csharp
var query = from item in items
            where item.Price > 9.99
            orderby item.Price
            select item;

var report = new StiReport();
report.Load(stream);
report.RegBusinessObject("MyData", "MyData", query);
report.CalculationMode = StiCalculationMode.Interpretation;
```

Both `IEnumerable` and `ITypedList` approaches are supported. Use `ITypedList` when you need to provide custom property descriptors for the report engine.

---

## 12. How to manage sub-reports?

Combine multiple reports into one document using `report.SubReports.Add()`:

```csharp
var report1 = new StiReport();
report1.Load(stream1);
report1.CalculationMode = StiCalculationMode.Interpretation;

var report2 = new StiReport();
report2.Load(stream2);
report2.CalculationMode = StiCalculationMode.Interpretation;

var report3 = new StiReport();
report3.Load(stream3);
report3.CalculationMode = StiCalculationMode.Interpretation;

var mainReport = new StiReport();
mainReport.SubReports.Add(report1);
mainReport.SubReports.Add(report2, resetPageNumber: true, printOnPreviousPage: false);
mainReport.SubReports.Add(report3, resetPageNumber: true, printOnPreviousPage: false);
mainReport.CalculationMode = StiCalculationMode.Interpretation;
```

The `SubReports.Add()` method supports optional parameters: `resetPageNumber` resets page numbering for each sub-report, and `printOnPreviousPage` allows the sub-report to start on the same page where the previous one ended.

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.AVALONIA).*
