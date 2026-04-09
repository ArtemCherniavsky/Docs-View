# Stimulsoft Reports.REACT — FAQ

Quick answers for developers integrating the Stimulsoft React Viewer into ASP.NET + React applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports for React?](#1-how-do-i-install-stimulsoft-reports-for-react)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Viewer?](#3-how-to-show-a-report-in-the-viewer)
4. [How to configure the server-side controller for the Viewer?](#4-how-to-configure-the-server-side-controller-for-the-viewer)
5. [How to pass parameters from React to the server?](#5-how-to-pass-parameters-from-react-to-the-server)
6. [How to handle Viewer events?](#6-how-to-handle-viewer-events)
7. [How to use the Viewer API methods?](#7-how-to-use-the-viewer-api-methods)
8. [How to customize the Viewer options?](#8-how-to-customize-the-viewer-options)
9. [How to localize the Viewer?](#9-how-to-localize-the-viewer)
10. [How to send a report by email?](#10-how-to-send-a-report-by-email)

---

## 1. How do I install Stimulsoft Reports for React?

Installation requires two parts — server-side NuGet package and client-side npm package.

**Server side (ASP.NET) — NuGet:**
```powershell
Install-Package Stimulsoft.Reports.React.NetCore
```

**.NET CLI:**
```bash
dotnet add package Stimulsoft.Reports.React.NetCore
```

**Client side (React) — npm:**
```bash
npm install stimulsoft-viewer-react
```

**Supported frameworks:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0 (server side); React 18+ (client side).

**Configure `Startup.cs`:**
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseCors(builder => builder.AllowAnyOrigin().AllowAnyHeader().AllowAnyMethod());

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller}/{action=Index}/{id?}");
    });
}
```

---

## 2. How to activate the license?

Set the license key in the **static constructor** of your server-side controller, so it executes once before any report operations:

```csharp
static ViewerController()
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

## 3. How to show a report in the Viewer?

Use the `<StimulsoftViewer>` component in your React application:

```tsx
import React from 'react';
import { StimulsoftViewer } from 'stimulsoft-viewer-react';

export const App: React.FC = () => {
    return (
        <StimulsoftViewer
            requestUrl="/Viewer/{action}"
            action="InitViewer"
            height="100vh"
        />
    );
};
```

The component communicates with the server-side controller via `requestUrl`. The `{action}` placeholder is replaced automatically with the action name being called.

---

## 4. How to configure the server-side controller for the Viewer?

Create an ASP.NET controller with three actions — `InitViewer`, `GetReport`, and `ViewerEvent`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Stimulsoft.Report;
using Stimulsoft.Report.React;

[Controller]
public class ViewerController : Controller
{
    [HttpPost]
    public IActionResult InitViewer()
    {
        var requestParams = StiReactViewer.GetRequestParams(this);

        var options = new StiReactViewerOptions();
        options.Actions.GetReport = "GetReport";
        options.Actions.ViewerEvent = "ViewerEvent";
        options.Appearance.ScrollbarsMode = true;

        return StiReactViewer.ViewerDataResult(requestParams, options);
    }

    [HttpPost]
    public IActionResult GetReport()
    {
        var report = StiReport.CreateNewReport();
        var path = StiReactHelper.MapPath(this, "Reports/MasterDetail.mrt");
        report.Load(path);

        return StiReactViewer.GetReportResult(this, report);
    }

    [HttpPost]
    public IActionResult ViewerEvent()
    {
        return StiReactViewer.ViewerEventResult(this);
    }
}
```

`StiReactHelper.MapPath()` resolves the file path relative to the application root. The `Actions` property maps action names used in `requestUrl`.

---

## 5. How to pass parameters from React to the server?

Use the `properties` prop to pass custom data from React to the server controller:

**React component:**
```tsx
import React, { useState, useMemo } from 'react';
import { StimulsoftViewer } from 'stimulsoft-viewer-react';

const reports = ['MasterDetail.mrt', 'EditableReport.mrt'];

export const App: React.FC = () => {
    const [reportName, setReportName] = useState(reports[0]);
    const properties = useMemo(() => ({ reportName }), [reportName]);

    return (
        <div>
            <select onChange={(e) => setReportName(e.target.value)} value={reportName}>
                {reports.map((item) => (
                    <option key={item} value={item}>{item}</option>
                ))}
            </select>

            <StimulsoftViewer
                requestUrl="/Viewer/{action}"
                action="InitViewer"
                height="100vh"
                properties={properties}
            />
        </div>
    );
};
```

**Server-side — read the properties in the controller:**
```csharp
using Stimulsoft.Base.Json;
using Stimulsoft.Base.Json.Linq;

[HttpPost]
public IActionResult GetReport()
{
    var reportName = "MasterDetail.mrt";
    var httpContext = new Stimulsoft.System.Web.HttpContext(this.HttpContext);
    var properties = httpContext.Request.Params["properties"]?.ToString();
    if (properties != null)
    {
        var data = Convert.FromBase64String(properties);
        var json = Encoding.UTF8.GetString(data);
        var jsonObject = JsonConvert.DeserializeObject(json) as JToken;
        reportName = jsonObject["reportName"]?.ToString() ?? reportName;
    }

    var report = StiReport.CreateNewReport();
    var path = StiReactHelper.MapPath(this, $"Reports/{reportName}");
    report.Load(path);

    return StiReactViewer.GetReportResult(this, report);
}
```

Properties are sent as a Base64-encoded JSON string and decoded on the server.

---

## 6. How to handle Viewer events?

Pass event handler functions as props to the `<StimulsoftViewer>` component:

```tsx
import React from 'react';
import { StimulsoftViewer } from 'stimulsoft-viewer-react';

export const App: React.FC = () => {
    const loaded = (): void => {
        console.log('Report loaded');
    };

    const onExport = (event: any): void => {
        console.log(`Export to: ${event.format}`);
    };

    return (
        <StimulsoftViewer
            requestUrl="/Viewer/{action}"
            action="InitViewer"
            height="100vh"
            onLoaded={loaded}
            onExport={onExport}
        />
    );
};
```

Available event props include: `onLoaded`, `onExport`, and others.

---

## 7. How to use the Viewer API methods?

Access the Viewer API using `useRef`:

```tsx
import React, { useRef, useState, useEffect } from 'react';
import { StimulsoftViewer, StimulsoftViewerHandle } from 'stimulsoft-viewer-react';

export const App: React.FC = () => {
    const viewerRef = useRef<StimulsoftViewerHandle>(null);
    const [zoom, setZoom] = useState<number>(100);
    const [currentPage, setCurrentPage] = useState<number>(0);

    useEffect(() => {
        const interval = setInterval(() => {
            if (viewerRef.current) {
                setZoom(viewerRef.current.zoom);
                setCurrentPage(viewerRef.current.currentPage);
            }
        }, 200);
        return () => clearInterval(interval);
    }, []);

    return (
        <div>
            Zoom is {zoom}<br />
            Current page {currentPage + 1}<br />
            <input
                type="button"
                onClick={() => { if (viewerRef.current) viewerRef.current.zoom = 50; }}
                value="Zoom to 50%"
            />
            <input
                type="button"
                onClick={() => { if (viewerRef.current) viewerRef.current.export('Pdf', { ImageResolution: 200 }); }}
                value="Export to PDF"
            />

            <StimulsoftViewer
                ref={viewerRef}
                requestUrl="/Viewer/{action}"
                action="InitViewer"
                height="100vh"
            />
        </div>
    );
};
```

The `StimulsoftViewerHandle` ref provides properties and methods: `zoom`, `currentPage`, `export()`, and others.

---

## 8. How to customize the Viewer options?

Set options in the server-side `InitViewer` action using `StiReactViewerOptions`:

```csharp
[HttpPost]
public IActionResult InitViewer()
{
    var requestParams = StiReactViewer.GetRequestParams(this);

    var options = new StiReactViewerOptions();
    options.Actions.GetReport = "GetReport";
    options.Actions.ViewerEvent = "ViewerEvent";
    options.Toolbar.ShowPinToolbarButton = false;
    options.Appearance.ScrollbarsMode = true;
    options.Toolbar.ShowSendEmailButton = true;

    return StiReactViewer.ViewerDataResult(requestParams, options);
}
```

**Key options:**

| Property | Description |
|----------|-------------|
| `options.Appearance.ScrollbarsMode` | Enable scroll bars |
| `options.Toolbar.ShowPinToolbarButton` | Show/hide the Pin toolbar button |
| `options.Toolbar.ShowSendEmailButton` | Show the Send Email button |

---

## 9. How to localize the Viewer?

Set the localization path in the server-side `InitViewer` action:

```csharp
[HttpPost]
public IActionResult InitViewer()
{
    var requestParams = StiReactViewer.GetRequestParams(this);

    var options = new StiReactViewerOptions();
    options.Actions.GetReport = "GetReport";
    options.Actions.ViewerEvent = "ViewerEvent";
    options.Appearance.ScrollbarsMode = true;
    options.Localization = StiReactHelper.MapPath(this, "Localization/de.xml");

    return StiReactViewer.ViewerDataResult(requestParams, options);
}
```

Place localization XML files in a `Localization` folder in your ASP.NET project. Stimulsoft provides localization files for many languages.

---

## 10. How to send a report by email?

Enable the email button and configure SMTP settings in the server controller:

**Enable the button in `InitViewer`:**
```csharp
options.Actions.EmailReport = "EmailReport";
options.Toolbar.ShowSendEmailButton = true;
```

**Add the `EmailReport` action:**
```csharp
[HttpPost]
public IActionResult EmailReport()
{
    StiEmailOptions options = StiReactViewer.GetEmailOptions(this);

    // Fill server-side SMTP settings
    options.AddressFrom = "admin_address@test.com";
    options.Host = "smtp.test.com";
    options.Port = 465;
    options.UserName = "admin_address@test.com";
    options.Password = "admin_password";

    return StiReactViewer.EmailReportResult(this, options);
}
```

The recipient address, subject, and body are passed from the Viewer dialog. SMTP settings must be configured on the server side.

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.WEB-for-ASP.NET-React).*
