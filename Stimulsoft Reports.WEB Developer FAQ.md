# Stimulsoft Reports.WEB for ASP.NET — Developer FAQ

Quick answers for developers integrating Stimulsoft Reports.WEB into ASP.NET Web Forms applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports.WEB?](#1-how-do-i-install-stimulsoft-reportsweb)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Web Viewer?](#3-how-to-show-a-report-in-the-web-viewer)
4. [How to connect data to a report from code?](#4-how-to-connect-data-to-a-report-from-code)
5. [How to create a report at runtime?](#5-how-to-create-a-report-at-runtime)
6. [How to access variables?](#6-how-to-access-variables)
7. [How to export a report from code?](#7-how-to-export-a-report-from-code)
8. [How to print a report from code?](#8-how-to-print-a-report-from-code)
9. [How to customize the Web Viewer?](#9-how-to-customize-the-web-viewer)
10. [How to change the Viewer and Designer theme?](#10-how-to-change-the-viewer-and-designer-theme)
11. [How to use the Web Designer?](#11-how-to-use-the-web-designer)
12. [How to localize the Viewer and Designer?](#12-how-to-localize-the-viewer-and-designer)
13. [How to configure report caching?](#13-how-to-configure-report-caching)

---

## 1. How do I install Stimulsoft Reports.WEB?

Install via NuGet Package Manager in Visual Studio:

**Package Manager Console:**
```powershell
Install-Package Stimulsoft.Reports.Web
Install-Package Stimulsoft.Reports.WebDesign
```

After installation, register the assemblies in your ASPX pages:

```aspx
<%@ Register Assembly="Stimulsoft.Report.Web" Namespace="Stimulsoft.Report.Web" TagPrefix="cc1" %>
```

For the designer:

```aspx
<%@ Register Assembly="Stimulsoft.Report.WebDesign" Namespace="Stimulsoft.Report.Web" TagPrefix="cc1" %>
```

**Supported framework:** ASP.NET Web Forms (.NET Framework).

---

## 2. How to activate the license?

Set the license key in the static constructor of your page class, so it executes once before any report operations:

```csharp
static Default()
{
    // Option 1: key string directly in code
    Stimulsoft.Base.StiLicense.Key = "your-license-key-here";

    // Option 2: load from a file
    Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

    // Option 3: load from a stream
    Stimulsoft.Base.StiLicense.LoadFromStream(stream);
}
```

---

## 3. How to show a report in the Web Viewer?

**ASPX markup:**
```aspx
<form id="form1" runat="server">
    <cc1:StiWebViewer ID="StiWebViewer1" runat="server"
        OnGetReport="StiWebViewer1_GetReport" />
</form>
```

**Code-behind — using the GetReport event:**
```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Web;

protected void StiWebViewer1_GetReport(object sender, StiReportDataEventArgs e)
{
    var report = new StiReport();
    report.Load(Server.MapPath("Reports/SimpleList.mrt"));
    e.Report = report;
}
```

**Or assign the report directly (without event):**
```csharp
protected void Page_Load(object sender, EventArgs e)
{
    var report = new StiReport();
    report.Load(Server.MapPath("Reports/SimpleList.mrt"));
    StiWebViewer1.Report = report;
}
```

---

## 4. How to connect data to a report from code?

**Register data from XML:**
```csharp
protected void StiWebViewer1_GetReport(object sender, StiReportDataEventArgs e)
{
    var report = new StiReport();
    report.Load(Server.MapPath("Reports/SimpleList.mrt"));

    // Remove existing connections from the template
    report.Dictionary.Databases.Clear();

    // Load and register data
    var data = new DataSet();
    data.ReadXml(Server.MapPath("Data/Demo.xml"));
    report.RegData(data);

    e.Report = report;
}
```

**Provide data in the designer's preview:**
```csharp
protected void StiWebDesigner1_PreviewReport(object sender, StiReportDataEventArgs e)
{
    var report = e.Report;
    report.Dictionary.Databases.Clear();

    var data = new DataSet();
    data.ReadXml(Server.MapPath("Data/Demo.xml"));
    report.RegData(data);
}
```

---

## 5. How to create a report at runtime?

```csharp
using Stimulsoft.Base.Drawing;
using Stimulsoft.Report;
using Stimulsoft.Report.Components;

protected void Page_Load(object sender, EventArgs e)
{
    var data = new DataSet();
    data.ReadXmlSchema(Server.MapPath(@"Data\Demo.xsd"));
    data.ReadXml(Server.MapPath(@"Data\Demo.xml"));

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

    StiWebViewer1.Report = report;
}
```

---

## 6. How to access variables?

Variables are defined in the report designer. Compile the report first, then set values from code:

```csharp
var report = new StiReport();
report.Load(Server.MapPath(@"Reports\Variables.mrt"));

report.Compile();

report["Name"] = Request.Form["name"] ?? string.Empty;
report["Surname"] = Request.Form["surname"] ?? string.Empty;
report["Email"] = Request.Form["email"] ?? string.Empty;
report["Address"] = Request.Form["address"] ?? string.Empty;
report["Sex"] = Request.Form["sex"] != null && Convert.ToBoolean(Request.Form["sex"]);

StiWebViewer1.Report = report;
```

Variable names are case-sensitive and must match the names defined in the report template.

---

## 7. How to export a report from code?

Use `StiReportResponse` to send the exported report directly to the browser as a file download:

```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Web;

var report = new StiReport();
report.Load(Server.MapPath("Reports/SimpleList.mrt"));

// Export as PDF
StiReportResponse.ResponseAsPdf(report);

// Other formats:
// StiReportResponse.ResponseAsHtml(report);
// StiReportResponse.ResponseAsXls(report);
// StiReportResponse.ResponseAsText(report);
// StiReportResponse.ResponseAsRtf(report);
```

`StiReportResponse` automatically renders the report, exports it, and sends it to the client browser. No manual `report.Render()` call is needed.

---

## 8. How to print a report from code?

```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Web;

var report = new StiReport();
report.Load(Server.MapPath("Reports/SimpleList.mrt"));

// Print as PDF (opens a PDF in the browser for printing)
StiReportResponse.PrintAsPdf(report);

// Print as HTML (triggers the browser's print dialog)
StiReportResponse.PrintAsHtml(report);
```

---

## 9. How to customize the Web Viewer?

Set properties on the `StiWebViewer` control in ASPX markup:

```aspx
<cc1:StiWebViewer ID="StiWebViewer1" runat="server"
    Width="1000px"
    Height="800px"
    Theme="Office2013LightGrayPurple"
    ScrollBarsMode="true"
    ZoomPercent="150"
    ToolbarDisplayMode="Separated"
    ShowPrintButton="false"
    ShowExportToCsv="false"
    ShowExportToXml="false"
    OnGetReport="StiWebViewer1_GetReport" />
```

**Key properties:**

| Property | Description | Example |
|----------|-------------|---------|
| `Width` / `Height` | Viewer dimensions | `"1000px"`, `"100%"` |
| `Theme` | Visual theme | `"Office2013LightGrayPurple"` |
| `ScrollBarsMode` | Enable scroll bars | `"true"` |
| `ZoomPercent` | Default zoom level | `"150"` |
| `ToolbarDisplayMode` | Toolbar layout | `"Separated"` |
| `FullScreenMode` | Full-screen mode | `"true"` |
| `ShowPrintButton` | Show/hide print button | `"false"` |
| `ShowExportTo*` | Show/hide specific export formats | `"false"` |

You can also set these from code-behind:

```csharp
StiWebViewer1.Theme = StiViewerTheme.Office2022BlackOrange;
StiWebViewer1.FullScreenMode = true;
```

---

## 10. How to change the Viewer and Designer theme?

Themes can be changed dynamically at runtime:

**Viewer:**
```csharp
StiWebViewer1.Theme = (StiViewerTheme)Enum.Parse(
    typeof(StiViewerTheme), selectedThemeName);
```

**Designer:**
```csharp
StiWebDesigner1.Theme = (StiDesignerTheme)Enum.Parse(
    typeof(StiDesignerTheme), selectedThemeName);
```

Available theme enums include: `Office2013LightGrayPurple`, `Office2022BlackOrange`, `Office2022WhiteTeal`, etc.

---

## 11. How to use the Web Designer?

**ASPX markup:**
```aspx
<%@ Register Assembly="Stimulsoft.Report.WebDesign"
    Namespace="Stimulsoft.Report.Web" TagPrefix="cc1" %>

<cc1:StiWebDesigner ID="StiWebDesigner1" runat="server"
    OnCreateReport="StiWebDesigner1_CreateReport"
    OnPreviewReport="StiWebDesigner1_PreviewReport"
    OnSaveReport="StiWebDesigner1_SaveReport" />
```

**Code-behind:**
```csharp
protected void Page_Load(object sender, EventArgs e)
{
    var report = new StiReport();
    report.Load(Server.MapPath(@"Reports\Invoice.mrt"));
    StiWebDesigner1.Report = report;
}

// Fired when creating a new report — set up data sources
protected void StiWebDesigner1_CreateReport(object sender, StiReportDataEventArgs e)
{
    var data = new DataSet();
    data.ReadXmlSchema(Server.MapPath(@"Data\Demo.xsd"));
    data.ReadXml(Server.MapPath(@"Data\Demo.xml"));

    e.Report.RegData(data);
    e.Report.Dictionary.Synchronize();
}

// Fired when previewing — optionally provide data
protected void StiWebDesigner1_PreviewReport(object sender, StiReportDataEventArgs e)
{
    // Provide data for preview if not embedded in the template
}

// Fired when saving — implement your save logic
protected void StiWebDesigner1_SaveReport(object sender, StiSaveReportEventArgs e)
{
    var report = e.Report;
    report.Save(Server.MapPath(@"Reports\" + report.ReportName + ".mrt"));
}
```

**Designer events summary:**

| Event | When it fires | Typical use |
|-------|--------------|-------------|
| `GetReport` | Designer loads initially | Load the report template |
| `CreateReport` | User creates a new report | Register data and synchronize dictionary |
| `PreviewReport` | User clicks Preview | Provide runtime data for preview |
| `SaveReport` | User clicks Save | Save the report to file/database |
| `Exit` | User exits the designer | Redirect back to the viewer |

---

## 12. How to localize the Viewer and Designer?

Set the `Localization` property to point to a localization XML file:

**Viewer:**
```csharp
StiWebViewer1.Localization = $"Localization/{selectedLanguage}.xml";
StiWebViewer1.Report = report;
```

**Designer:**
```csharp
StiWebDesigner1.Localization = $"Localization/{selectedLanguage}.xml";
StiWebDesigner1.Report = report;
```

Place localization XML files in a `Localization` folder in your web application. Stimulsoft provides localization files for many languages.

---

## 13. How to configure report caching?

Report caching stores rendered reports between requests to improve performance. Implement a custom `StiCacheHelper` and assign it to `StiWebViewer.CacheHelper`.

**File-based cache example:**
```csharp
using Stimulsoft.Report;
using Stimulsoft.Report.Web;

public class StiFileCacheHelper : StiCacheHelper
{
    public override StiReport GetReport(string guid)
    {
        var path = HttpContext.Server.MapPath(Path.Combine("CacheFiles", guid));
        if (File.Exists(path))
        {
            var report = new StiReport();
            var packedReport = File.ReadAllText(path);
            if (guid.EndsWith(GUID_ReportTemplate))
                report.LoadPackedReportFromString(packedReport);
            else
                report.LoadPackedDocumentFromString(packedReport);
            return report;
        }
        return null;
    }

    public override void SaveReport(StiReport report, string guid)
    {
        var packedReport = guid.EndsWith(GUID_ReportTemplate)
            ? report.SavePackedReportToString()
            : report.SavePackedDocumentToString();
        var path = HttpContext.Server.MapPath(Path.Combine("CacheFiles", guid));
        File.WriteAllText(path, packedReport);
    }

    public override void RemoveReport(string guid)
    {
        var path = HttpContext.Server.MapPath(Path.Combine("CacheFiles", guid));
        if (File.Exists(path)) File.Delete(path);
    }
}

// Assign in the page constructor:
public MyPage()
{
    StiWebViewer.CacheHelper = new StiFileCacheHelper();
}
```

Other cache backends (default in-memory, SQL Server) follow the same pattern — override `GetReport`, `SaveReport`, and `RemoveReport`.

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.WEB-for-ASP.NET).*
