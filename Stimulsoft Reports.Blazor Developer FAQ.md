# Stimulsoft Reports.Blazor — Developer FAQ

Quick answers for developers integrating Stimulsoft Reports into Blazor Server applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports.Blazor?](#1-how-do-i-install-stimulsoft-reportsblazer)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Viewer?](#3-how-to-show-a-report-in-the-viewer)
4. [How to connect data to a report from code?](#4-how-to-connect-data-to-a-report-from-code)
5. [How to access variables?](#5-how-to-access-variables)
6. [How to export a report from code?](#6-how-to-export-a-report-from-code)
7. [How to save and load a report template?](#7-how-to-save-and-load-a-report-template)
8. [How to customize the Viewer?](#8-how-to-customize-the-viewer)
9. [How to change the Viewer and Designer theme?](#9-how-to-change-the-viewer-and-designer-theme)
10. [How to use the Designer?](#10-how-to-use-the-designer)
11. [How to localize the Viewer?](#11-how-to-localize-the-viewer)
12. [How to use business objects in a report?](#12-how-to-use-business-objects-in-a-report)

---

## 1. How do I install Stimulsoft Reports.Blazor?

Install via NuGet Package Manager in Visual Studio:

**Package Manager Console:**
```powershell
Install-Package Stimulsoft.Reports.Blazor
```

**.NET CLI:**
```bash
dotnet add package Stimulsoft.Reports.Blazor
```

Or search for `Stimulsoft.Reports.Blazor` in the NuGet UI inside Visual Studio.

**Supported frameworks:** .NET 6.0, .NET 8.0

After installation, add the following using directives in your Razor components:
```razor
@using Stimulsoft.Base
@using Stimulsoft.Report
@using Stimulsoft.Report.Blazor
```

**Configure SignalR message size** in `Startup.cs` or `Program.cs` to handle large reports:
```csharp
services.AddServerSideBlazor();
services.AddSignalR(hubOptions =>
{
    hubOptions.MaximumReceiveMessageSize = 10 * 1024 * 1024; // 10MB
});
```

---

## 2. How to activate the license?

Set the license key in `OnInitialized()` of your Razor component, before any other Stimulsoft API call:

```razor
@code
{
    protected override void OnInitialized()
    {
        // Option 1: key string directly in code
        Stimulsoft.Base.StiLicense.Key = "your-license-key-here";

        // Option 2: load from a file
        Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

        // Option 3: load from a stream
        Stimulsoft.Base.StiLicense.LoadFromStream(stream);
    }
}
```

---

## 3. How to show a report in the Viewer?

Use the `<StiBlazorViewer>` component in a Razor page:

```razor
@page "/"
@using Stimulsoft.Base
@using Stimulsoft.Report
@using Stimulsoft.Report.Blazor

<StiBlazorViewer Report="@Report" />

@code
{
    private StiReport Report;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Report = new StiReport();
        this.Report.Load("Reports/TwoSimpleLists.mrt");
    }
}
```

The `StiBlazorViewer` automatically renders the report and displays it. No explicit `Render()` call is needed.

---

## 4. How to connect data to a report from code?

Clone the report, clear existing connections, register new data, and assign back to the Viewer:

```razor
@code
{
    private StiReport Report;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Report = new StiReport();
        this.Report.Load("Reports/Report.mrt");
    }

    private void LoadData()
    {
        var report = this.Report.Clone() as StiReport;

        report.Dictionary.Databases.Clear();

        var data = new System.Data.DataSet();
        data.ReadXml("Data/Demo.xml");
        report.RegData(data);
        report.Dictionary.Synchronize();

        report.Render();

        this.Report = report;
    }
}
```

In Blazor, use `report.Clone()` to create a new report instance before modifying data — this triggers the Viewer to re-render when you assign the new report back.

---

## 5. How to access variables?

Bind a Blazor input to a variable and set it on the report before rendering:

```razor
@page "/"
@using Stimulsoft.Base
@using Stimulsoft.Report
@using Stimulsoft.Report.Blazor

<div align="center">
    <input type="text" @bind="@NewValueInput" />
    <button @onclick="@SetVariable">Set variable</button>
</div>

<StiBlazorViewer Report="@Report" />

@code
{
    private StiReport Report;
    private string NewValueInput;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Report = new StiReport();
        this.Report.Load("Reports/ReportWithVariable.mrt");
    }

    private void SetVariable()
    {
        var report = this.Report.Clone() as StiReport;

        report["MySuperUselessVariable"] = NewValueInput;
        report.Render();

        this.Report = report;
    }
}
```

Variable names are case-sensitive and must match the names defined in the report template. In Blazor, unlike WinForms/WPF, calling `Compile()` before setting variables is not required.

---

## 6. How to export a report from code?

There are two approaches — save to a file on the server, or send as a file download to the browser.

**Save to a file on the server:**
```razor
@code
{
    private void ExportReportAsPdf()
    {
        var report = new StiReport();
        report.Load("Reports/TwoSimpleLists.mrt");
        report.Render(false);

        report.ExportDocument(StiExportFormat.Pdf, "Reports/output.pdf");
        report.ExportDocument(StiExportFormat.Html, "Reports/output.html");
    }
}
```

**Send as a file download to the browser:**
```razor
@using Stimulsoft.Report.Web
@inject IJSRuntime JSRuntime

@code
{
    protected override void OnInitialized()
    {
        base.OnInitialized();
        StiBlazorHelper.Initialize(JSRuntime);
    }

    private void ExportReportAsPdf()
    {
        var report = new StiReport();
        report.Load("Reports/TwoSimpleLists.mrt");
        report.Render(false);

        StiReportResponse.ResponseAsPdf(report);
    }

    private void ExportReportAsHtml()
    {
        var report = new StiReport();
        report.Load("Reports/TwoSimpleLists.mrt");
        report.Render(false);

        StiReportResponse.ResponseAsHtml(report);
    }
}
```

For browser downloads, initialize `StiBlazorHelper` with `IJSRuntime` in `OnInitialized()`.

---

## 7. How to save and load a report template?

```razor
@code
{
    private StiReport Report;

    protected override void OnInitialized()
    {
        base.OnInitialized();
        this.Report = new StiReport();
    }

    private void LoadReport()
    {
        this.Report.Load("Reports/TwoSimpleLists.mrt");
    }

    private void SaveReportToJson()
    {
        var jsonString = this.Report.SaveToJsonString();
    }

    private void RenderReport()
    {
        this.Report.Render(false);
    }

    private void SaveRenderedDocument()
    {
        var jsonDocument = this.Report.SaveDocumentJsonToString();
    }
}
```

Report templates are stored as `.mrt` files (XML-based). Rendered reports can be saved as JSON strings using `SaveDocumentJsonToString()`.

---

## 8. How to customize the Viewer?

Pass an `Options` object and attributes to the `<StiBlazorViewer>` component:

```razor
@using Stimulsoft.Report.Web

<StiBlazorViewer Report="@Report" Options="@Options"
                 Width="650" Height="650" />

@code
{
    private StiReport Report;
    private StiBlazorViewerOptions Options;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Options = new StiBlazorViewerOptions();
        this.Options.Appearance.FullScreenMode = true;
        this.Options.Appearance.ScrollbarsMode = true;
        this.Options.Toolbar.Visible = false;
        this.Options.Toolbar.ViewMode = StiWebViewMode.Continuous;
        this.Options.Toolbar.ShowOpenButton = false;
        this.Options.Toolbar.DisplayMode = StiToolbarDisplayMode.Separated;

        this.Report = new StiReport();
        this.Report.Load("Reports/TwoSimpleLists.mrt");
    }
}
```

**Key options:**

| Property | Description |
|----------|-------------|
| `Options.Toolbar.Visible` | Show/hide the entire toolbar |
| `Options.Toolbar.ViewMode` | Page view mode (`SinglePage`, `Continuous`) |
| `Options.Toolbar.ShowOpenButton` | Show/hide the Open button |
| `Options.Toolbar.DisplayMode` | Toolbar layout (`Simple`, `Separated`) |
| `Options.Appearance.FullScreenMode` | Enable full-screen mode |
| `Options.Appearance.ScrollbarsMode` | Enable scroll bars |

---

## 9. How to change the Viewer and Designer theme?

Set the `Theme` attribute on the component:

**Viewer:**
```razor
<StiBlazorViewer Report="@Report"
                 Theme="StiViewerTheme.Office2022LightGrayCarmine" />
```

**Designer:**
```razor
<StiBlazorDesigner Report="@Report"
                   Theme="StiDesignerTheme.Office2022BlackCarmine" />
```

Available themes include: `Office2022LightGrayCarmine`, `Office2022BlackCarmine`, `Office2022WhiteTeal`, etc.

---

## 10. How to use the Designer?

Use the `<StiBlazorDesigner>` component with the `OnSaveReport` event:

```razor
@page "/"
@using Stimulsoft.Base
@using Stimulsoft.Report
@using Stimulsoft.Report.Blazor

<StiBlazorDesigner Report="@Report" OnSaveReport="@OnSaveReport" />

@code
{
    private StiReport Report;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Report = new StiReport();
        this.Report.Load("Reports/TwoSimpleLists.mrt");
    }

    private void OnSaveReport(StiSaveReportEventArgs args)
    {
        args.Report.Save("Reports/TwoSimpleLists.mrt");
    }
}
```

**Customize designer options:**
```razor
<StiBlazorDesigner Report="@Report" Options="@Options" />

@code
{
    private StiBlazorDesignerOptions Options;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Options = new StiBlazorDesignerOptions();
        this.Options.Toolbar.ShowPageButton = false;
        this.Options.Toolbar.ShowInsertButton = false;
        this.Options.Toolbar.ShowLayoutButton = false;
        this.Options.Appearance.ShowTooltips = false;
        this.Options.Appearance.ShowDialogsHelp = false;
    }
}
```

---

## 11. How to localize the Viewer?

Set the `Localization` attribute on the `<StiBlazorViewer>` component:

```razor
<StiBlazorViewer Report="@Report" Localization="Localization/pl.xml" />
```

Place localization XML files in the `Localization` folder of your Blazor application (under `wwwroot` or at the project root). Stimulsoft provides localization files for many languages.

---

## 12. How to use business objects in a report?

Register business objects using `RegBusinessObject()` and synchronize the dictionary:

```razor
@code
{
    private StiReport Report;

    protected override void OnInitialized()
    {
        base.OnInitialized();

        this.Report = new StiReport();
        this.Report.Load("Reports/BusinessObjects.mrt");

        var list = GetBusinessObject();
        this.Report.RegBusinessObject("MyList", list);
        this.Report.Dictionary.Synchronize();
    }

    public class BusinessEntity
    {
        public string Name { get; set; }
        public string Alias { get; set; }

        public BusinessEntity(string name, string alias)
        {
            Name = name;
            Alias = alias;
        }
    }

    private System.Collections.ArrayList GetBusinessObject()
    {
        var list = new System.Collections.ArrayList();
        list.Add(new BusinessEntity("name1", "alias1"));
        list.Add(new BusinessEntity("name2", "alias2"));
        list.Add(new BusinessEntity("name3", "alias3"));
        return list;
    }
}
```

`RegBusinessObject()` works with any collection type — `ArrayList`, `List<T>`, or LINQ query results.

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.WEB-for-Blazor-Server).*
