# Stimulsoft Reports.NET — Developer FAQ

Quick answers for developers integrating Stimulsoft Reports.NET into WinForms applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports.NET?](#1-how-do-i-install-stimulsoft-reportsnet)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to save/load a report?](#3-how-to-saveload-a-report)
4. [How to load a DataSet XSD schema?](#4-how-to-load-a-dataset-xsd-schema)
5. [How to render a report?](#5-how-to-render-a-report)
6. [How to access variables?](#6-how-to-access-variables)
7. [How to attach data to a report?](#7-how-to-attach-data-to-a-report)
8. [How to run the report designer?](#8-how-to-run-the-report-designer)
9. [How to save/load a rendered report?](#9-how-to-saveload-a-rendered-report)
10. [How to print a report without preview?](#10-how-to-print-a-report-without-preview)
11. [How to export a report to PDF, Excel, Word?](#11-how-to-export-a-report-to-pdf-excel-word)
12. [How to connect JSON data from code?](#12-how-to-connect-json-data-from-code)

---

## 1. How do I install Stimulsoft Reports.NET?

Install via NuGet Package Manager in Visual Studio:

**Package Manager Console:**
```powershell
Install-Package Stimulsoft.Reports.Win
```

**.NET CLI:**
```bash
dotnet add package Stimulsoft.Reports.Win
```

Or search for `Stimulsoft.Reports.Win` in the NuGet UI inside Visual Studio.

**Supported frameworks:** .NET Framework 4.5.2+, .NET 6.0, .NET 8.0

After installation add the using directives you need:
```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Components;
using Stimulsoft.Report.Viewer;
using Stimulsoft.Base;
```

---

## 2. How to activate the license?

Call license activation **before** any other Stimulsoft API call — typically in the constructor of your main form or in `Program.cs`.

```csharp
// Option 1: key string directly in code
Stimulsoft.Base.StiLicense.Key = "your activation code...";

// Option 2: load from a .key file
Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

// Option 3: load from a byte array
Stimulsoft.Base.StiLicense.LoadFromBytes(bytes);

// Option 4: load from a stream
Stimulsoft.Base.StiLicense.LoadFromStream(stream);

// Option 5: load from an embedded assembly resource
Stimulsoft.Base.StiLicense.LoadFromEntryAssembly(assembly, "stimulsoft-license.key");
```

You can get your license key or download the license file from your [user account](https://devs.stimulsoft.com/).

---

## 3. How to save/load a report?

**Save a report template:**
```csharp
StiReport report = new StiReport();
report.Save("report.mrt");
```

**Load a report template:**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
```

Report templates are stored as `.mrt` files (XML-based). These are design-time files — they contain the layout but no rendered data.

---

## 4. How to load a DataSet XSD schema?

Use the `ImportXMLSchema` method to import an XSD schema into the report dictionary:

```csharp
StiReport report = new StiReport();

DataSet dataSet = new DataSet("Test");
dataSet.ReadXmlSchema("dataset.xsd");

report.Dictionary.ImportXMLSchema(dataSet);
```

This registers the schema structure in the report so it is available in the designer's data source list.

---

## 5. How to render a report?

**Render and show in a preview window in one call:**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.Show();
```

`Show()` automatically calls `Render()` if the report has not been rendered yet.

**Render explicitly (when you need to do something with pages before display):**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.Render();

// e.g. show in an embedded viewer control
stiViewerControl1.Report = report;
```

**Render asynchronously (non-blocking UI):**
```csharp
await report.RenderAsync();
```

---

## 6. How to access variables?

Variables are defined in the report designer. Read and write them from code after calling `Compile()`:

```csharp
StiReport report = new StiReport();
report.Load("Variables.mrt");
report.Compile(); // required before accessing variables

// Set a variable
report["VariableName"] = "Value";

// Set typed values
report["StartDate"] = new DateTime(2024, 1, 1);
report["IsActive"]  = true;
report["Amount"]    = 1500.00m;

// Get a variable value
object value = report["VariableName"];
```

Variable names are case-sensitive and must match the names defined in the designer.

---

## 7. How to attach data to a report?

**From XML files:**
```csharp
DataSet dataSet = new DataSet();
dataSet.ReadXmlSchema("Demo.xsd");
dataSet.ReadXml("Demo.xml");

StiReport report = new StiReport();
report.RegData(dataSet);
```

**From JSON:**
```csharp
var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile("Demo.json");

StiReport report = new StiReport();
report.RegData(dataSet);
```

**With a custom alias (when the template expects a specific name):**
```csharp
report.Dictionary.Databases.Clear(); // remove connections from the template
report.RegData("Demo", dataSet);     // register under the alias the template uses
```

**From business objects (C# collections):**
```csharp
IEnumerable<Customer> customers = GetCustomers();
report.RegData("Customers", customers);
```

---

## 8. How to run the report designer?

**Open an empty designer:**
```csharp
StiReport report = new StiReport();
report.Design();
```

**Open an existing template in the designer:**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.Design();
```

**Handle the save event:**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.StiReportDesigner.EventSave += (sender, args) =>
{
    var saved = (StiReport)sender;
    saved.Save("report.mrt");
};
report.Design();
```

---

## 9. How to save/load a rendered report?

A rendered report (`.mdc`) stores the output pages — no data source is needed to display it again.

**Save a rendered report:**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.Render();
report.SaveDocument("document.mdc");
```

**Load and display a previously rendered report:**
```csharp
StiReport report = new StiReport();
report.LoadDocument("document.mdc");
report.Show();
```

---

## 10. How to print a report without preview?

**Print directly from a template (renders first, then prints):**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.Print();
```

**Print from a previously rendered document:**
```csharp
StiReport report = new StiReport();
report.LoadDocument("document.mdc");
report.Print();
```

---

## 11. How to export a report to PDF, Excel, Word?

Always call `Render()` before exporting.

**Export to a file:**
```csharp
StiReport report = new StiReport();
report.Load("report.mrt");
report.Render();

report.ExportDocument(StiExportFormat.Pdf,        "report.pdf");
report.ExportDocument(StiExportFormat.Excel2007,  "report.xlsx");
report.ExportDocument(StiExportFormat.Word2007,   "report.docx");
report.ExportDocument(StiExportFormat.ImagePng,   "report.png");
```

**Export to a stream:**
```csharp
var stream = new MemoryStream();
report.ExportDocument(StiExportFormat.Pdf, stream);
stream.Seek(0, SeekOrigin.Begin);
// use the stream (send via HTTP, save to DB, etc.)
```

**Export asynchronously:**
```csharp
await report.ExportDocumentAsync(StiExportFormat.Pdf, "report.pdf");
```

**Common export formats:**

| Format | Constant |
|--------|----------|
| PDF | `StiExportFormat.Pdf` |
| Excel 2007+ | `StiExportFormat.Excel2007` |
| Word 2007+ | `StiExportFormat.Word2007` |
| PNG image | `StiExportFormat.ImagePng` |
| HTML | `StiExportFormat.Html` |
| CSV | `StiExportFormat.Csv` |
| Text | `StiExportFormat.Text` |
| RTF | `StiExportFormat.Rtf` |

---

## 12. How to connect JSON data from code?

```csharp
using Stimulsoft.Base;

var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile("data.json");

StiReport report = new StiReport();
report.Load("report.mrt");
report.Dictionary.Databases.Clear();
report.RegData("Demo", dataSet);
report.Show();
```

`StiJsonToDataSetConverterV2` automatically converts a JSON file into a `DataSet` with typed columns, ready for use in any report.

---

*For more details see the [official documentation](https://www.stimulsoft.com/en/documentation/online/programming-manual/reports_net.htm) and [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.NET-for-WinForms).*
