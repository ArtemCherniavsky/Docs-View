# Stimulsoft Reports.WPF — Developer FAQ

Quick answers for developers integrating Stimulsoft Reports into WPF applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports.WPF?](#1-how-do-i-install-stimulsoft-reportswpf)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Viewer?](#3-how-to-show-a-report-in-the-viewer)
4. [How to connect data to a report from code?](#4-how-to-connect-data-to-a-report-from-code)
5. [How to create a report at runtime?](#5-how-to-create-a-report-at-runtime)
6. [How to access variables?](#6-how-to-access-variables)
7. [How to export a report from code?](#7-how-to-export-a-report-from-code)
8. [How to print a report from code?](#8-how-to-print-a-report-from-code)
9. [How to run the report designer?](#9-how-to-run-the-report-designer)
10. [How to localize the user interface?](#10-how-to-localize-the-user-interface)
11. [How to use business objects in a report?](#11-how-to-use-business-objects-in-a-report)
12. [How to connect JSON data from code?](#12-how-to-connect-json-data-from-code)

---

## 1. How do I install Stimulsoft Reports.WPF?

Install via NuGet Package Manager in Visual Studio:

**Package Manager Console:**
```powershell
Install-Package Stimulsoft.Reports.Wpf
```

**.NET CLI:**
```bash
dotnet add package Stimulsoft.Reports.Wpf
```

Or search for `Stimulsoft.Reports.Wpf` in the NuGet UI inside Visual Studio.

**Supported frameworks:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

After installation add the using directives you need:
```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Components;
using Stimulsoft.Base;
```

---

## 2. How to activate the license?

Set the license key **before** any other Stimulsoft API call — typically in the constructor of your main window or in `App.xaml.cs`.

```csharp
// Option 1: key string directly in code
Stimulsoft.Base.StiLicense.Key = "your-license-key-here";

// Option 2: load from a file
Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

// Option 3: load from a stream
Stimulsoft.Base.StiLicense.LoadFromStream(stream);
```

---

## 3. How to show a report in the Viewer?

There are two approaches — a dialog window or an embedded viewer control.

**Dialog window — `ShowWithWpf()`:**
```csharp
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
report.ShowWithWpf();
```

`ShowWithWpf()` automatically renders the report and opens it in a separate viewer window.

**Embedded viewer control in XAML:**

Add the namespace to your window:
```xml
<Window ...
    xmlns:wpfViewer="schemas-stimulsoft-com:wpf-viewer">

    <wpfViewer:StiWpfViewerControl Name="StiWpfViewerControl1" />
</Window>
```

Assign the report from code-behind:
```csharp
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
report.Render();
StiWpfViewerControl1.Report = report;
```

When using the embedded control, call `report.Render()` explicitly before assigning.

**Show with Ribbon GUI:**
```csharp
report.ShowWithWpfRibbonGUI();
```

---

## 4. How to connect data to a report from code?

**From XML files:**
```csharp
var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");

report.Dictionary.Databases.Clear();

var data = new DataSet();
data.ReadXmlSchema(@"Data\Demo.xsd");
data.ReadXml(@"Data\Demo.xml");
report.RegData("Demo", data);

report.ShowWithWpf();
```

**From JSON files:**
```csharp
var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");

report.Dictionary.Databases.Clear();

var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\Demo.json");
report.RegData("Demo", dataSet);

report.ShowWithWpf();
```

Call `report.Dictionary.Databases.Clear()` to remove connections defined in the template before registering your data.

---

## 5. How to create a report at runtime?

```csharp
using Stimulsoft.Base.Drawing;
using Stimulsoft.Report;
using Stimulsoft.Report.Components;

var data = new DataSet();
data.ReadXmlSchema(@"Data\Demo.xsd");
data.ReadXml(@"Data\Demo.xml");

var report = new StiReport();
report.RegData(data);
report.Dictionary.Synchronize();

var page = report.Pages[0];

// Create HeaderBand
var headerBand = new StiHeaderBand();
headerBand.Height = 0.5;
headerBand.Name = "HeaderBand";
page.Components.Add(headerBand);

var headerText = new StiText(new RectangleD(0, 0, 5, 0.5));
headerText.Text = "CompanyName";
headerText.HorAlignment = StiTextHorAlignment.Center;
headerText.Name = "HeaderText";
headerText.Brush = new StiSolidBrush(System.Drawing.Color.LightGreen);
headerBand.Components.Add(headerText);

// Create DataBand
var dataBand = new StiDataBand();
dataBand.DataSourceName = "Customers";
dataBand.Height = 0.5;
dataBand.Name = "DataBand";
page.Components.Add(dataBand);

var dataText = new StiText(new RectangleD(0, 0, 5, 0.5));
dataText.Text = "{Line}.{Customers.CompanyName}";
dataText.Name = "DataText";
dataBand.Components.Add(dataText);

// Create FooterBand
var footerBand = new StiFooterBand();
footerBand.Height = 0.5;
footerBand.Name = "FooterBand";
page.Components.Add(footerBand);

var footerText = new StiText(new RectangleD(0, 0, 5, 0.5));
footerText.Text = "Count - {Count()}";
footerText.HorAlignment = StiTextHorAlignment.Right;
footerText.Name = "FooterText";
footerText.Brush = new StiSolidBrush(System.Drawing.Color.LightGreen);
footerBand.Components.Add(footerText);

report.ShowWithWpf();
```

---

## 6. How to access variables?

Variables are defined in the report designer. Compile the report first, then set values from code:

```csharp
var report = new StiReport();
report.Load(@"Reports\Variables.mrt");

report.Compile();

report["Name"] = TextBoxName.Text;
report["Surname"] = TextBoxSurname.Text;
report["Email"] = TextBoxEmail.Text;
report["Address"] = TextBoxAddress.Text;
report["Sex"] = RadioButtonMale.IsChecked.GetValueOrDefault();

report.ShowWithWpf();
```

Variable names are case-sensitive and must match the names defined in the report template.

---

## 7. How to export a report from code?

Always call `Render()` before exporting.

```csharp
var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");
report.Render();

// Export to a MemoryStream
var stream = new MemoryStream();
report.ExportDocument(StiExportFormat.Pdf, stream);

// Save to a file using SaveFileDialog
var dialog = new SaveFileDialog { Filter = "PDF (*.pdf)|*.pdf" };
if (dialog.ShowDialog() == true)
{
    File.WriteAllBytes(dialog.FileName, stream.ToArray());
}
```

**Common export formats:**

| Format | Constant |
|--------|----------|
| PDF | `StiExportFormat.Pdf` |
| Excel 2007+ | `StiExportFormat.Excel2007` |
| Word 2007+ | `StiExportFormat.Word2007` |
| PNG image | `StiExportFormat.ImagePng` |
| Text | `StiExportFormat.Text` |

---

## 8. How to print a report from code?

```csharp
var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");
report.PrintWithWpf();
```

`PrintWithWpf()` automatically renders the report and opens the WPF print dialog.

---

## 9. How to run the report designer?

There are two approaches — a dialog window or an embedded designer control.

**Dialog window — `DesignV2WithWpf()`:**
```csharp
// Handle the save event
StiWpfDesigner.SavingReport += (s, args) =>
{
    args.Report.Save(args.Report.ReportFile);
};

// New report
var report = new StiReport();
report.DesignV2WithWpf();

// Or edit an existing template
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
report.DesignV2WithWpf();
```

**Embedded designer control in XAML:**

Add the namespace to your window:
```xml
<Window ...
    xmlns:WpfDesign="clr-namespace:Stimulsoft.Client.Designer;assembly=Stimulsoft.Client.Designer">

    <WpfDesign:StiDesignerControl Name="StiDesignerControl1" />
</Window>
```

Assign the report from code-behind:
```csharp
// New report
StiDesignerControl1.Report = new StiReport();

// Or load an existing template
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
StiDesignerControl1.Report = report;
```

---

## 10. How to localize the user interface?

Load a localization XML file before opening the viewer or designer:

```csharp
StiOptions.Localization.Load(fileName);

// Then open the viewer or designer
report.ShowWithWpf();
// or
report.DesignV2WithWpf();
```

Stimulsoft provides localization files for many languages. Place localization XML files in your application directory and load the desired one at startup.

---

## 11. How to use business objects in a report?

**Register IEnumerable collections:**
```csharp
var report = new StiReport();
report.RegData("EmployeeIEnumerable", GetEmployees());
report.Load(@"Reports\BusinessObjects_IEnumerable.mrt");
report.ShowWithWpf();
```

**Register ITypedList collections:**
```csharp
var report = new StiReport();
report.RegData("EmployeeITypedList", GetEmployeesTypedList());
report.Load(@"Reports\BusinessObjects_ITypedList.mrt");
report.ShowWithWpf();
```

**Using LINQ queries:**
```csharp
var items = GetItems();
var query = from i in items
            where i.Price > 9.99
            orderby i.Price
            select i;

var report = new StiReport();
report.RegBusinessObject("MyData", "MyData", query);
report.Load(@"Reports\LinqReport.mrt");
report.ShowWithWpf();
```

`RegData()` works with any `IEnumerable` or `ITypedList`. For LINQ results use `RegBusinessObject()`.

---

## 12. How to connect JSON data from code?

```csharp
using Stimulsoft.Base;

var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\Demo.json");

var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");
report.Dictionary.Databases.Clear();
report.RegData("Demo", dataSet);
report.ShowWithWpf();
```

`StiJsonToDataSetConverterV2` automatically converts a JSON file into a `DataSet` with typed columns, ready for use in any report.

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.WPF).*
