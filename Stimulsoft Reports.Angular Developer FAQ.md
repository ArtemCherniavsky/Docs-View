# Stimulsoft Reports.Angular — Developer FAQ

Quick answers for developers integrating the Stimulsoft Angular Viewer and Designer into ASP.NET + Angular applications.

---

## Table of Contents

1. [How do I install Stimulsoft Reports for Angular?](#1-how-do-i-install-stimulsoft-reports-for-angular)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Viewer?](#3-how-to-show-a-report-in-the-viewer)
4. [How to configure the server-side controller for the Viewer?](#4-how-to-configure-the-server-side-controller-for-the-viewer)
5. [How to pass parameters from Angular to the server?](#5-how-to-pass-parameters-from-angular-to-the-server)
6. [How to handle Viewer events?](#6-how-to-handle-viewer-events)
7. [How to use the Viewer API methods?](#7-how-to-use-the-viewer-api-methods)
8. [How to customize the Viewer options?](#8-how-to-customize-the-viewer-options)
9. [How to use the Designer?](#9-how-to-use-the-designer)
10. [How to localize the Viewer?](#10-how-to-localize-the-viewer)
11. [How to send a report by email?](#11-how-to-send-a-report-by-email)
12. [How to use a Standalone Angular Component?](#12-how-to-use-a-standalone-angular-component)

---

## 1. How do I install Stimulsoft Reports for Angular?

Installation requires two parts — server-side NuGet package and client-side npm package.

**Server side (ASP.NET) — NuGet:**
```powershell
Install-Package Stimulsoft.Reports.Angular.NetCore
```

**.NET CLI:**
```bash
dotnet add package Stimulsoft.Reports.Angular.NetCore
```

**Client side (Angular) — npm:**
```bash
npm install stimulsoft-viewer-angular
npm install stimulsoft-designer-angular
```

**Supported frameworks:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0 (server side); Angular 19+ (client side).

**Register the module in `app.module.ts`:**
```typescript
import { StimulsoftViewerModule } from 'stimulsoft-viewer-angular';
import { StimulsoftDesignerModule } from 'stimulsoft-designer-angular';

@NgModule({
    imports: [
        BrowserModule,
        StimulsoftViewerModule,
        StimulsoftDesignerModule,
        BrowserAnimationsModule,
        FormsModule
    ],
    providers: [provideHttpClient(withInterceptorsFromDi())]
})
export class AppModule { }
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

Add the `<stimulsoft-viewer-angular>` component in your Angular template:

```html
<stimulsoft-viewer-angular
  [requestUrl]="'http://localhost:60801/Viewer/{action}'"
  [action]="'InitViewer'"
  [height]="'100vh'"
></stimulsoft-viewer-angular>
```

The component communicates with the server-side controller via `requestUrl`. The `{action}` placeholder is replaced automatically with the action name being called.

**Angular component:**
```typescript
import { Component } from '@angular/core';

@Component({
    selector: 'app-root',
    templateUrl: './app.component.html',
    standalone: false
})
export class AppComponent {
    title = 'ClientApp';
}
```

---

## 4. How to configure the server-side controller for the Viewer?

Create an ASP.NET controller with three actions — `InitViewer`, `GetReport`, and `ViewerEvent`:

```csharp
using Microsoft.AspNetCore.Mvc;
using Stimulsoft.Report;
using Stimulsoft.Report.Angular;

[Controller]
public class ViewerController : Controller
{
    [HttpPost]
    public IActionResult InitViewer()
    {
        var requestParams = StiAngularViewer.GetRequestParams(this);

        var options = new StiAngularViewerOptions();
        options.Actions.GetReport = "GetReport";
        options.Actions.ViewerEvent = "ViewerEvent";
        options.Appearance.ScrollbarsMode = true;

        return StiAngularViewer.ViewerDataResult(requestParams, options);
    }

    [HttpPost]
    public IActionResult GetReport()
    {
        var report = StiReport.CreateNewReport();
        var path = StiAngularHelper.MapPath(this, "Reports/MasterDetail.mrt");
        report.Load(path);

        return StiAngularViewer.GetReportResult(this, report);
    }

    [HttpPost]
    public IActionResult ViewerEvent()
    {
        return StiAngularViewer.ViewerEventResult(this);
    }
}
```

`StiAngularHelper.MapPath()` resolves the file path relative to the application root. The `Actions` property maps action names used in `requestUrl`.

---

## 5. How to pass parameters from Angular to the server?

Use the `[properties]` input to pass custom data from Angular to the server controller:

**Angular template:**
```html
<select (change)="updateProps($event.target.value)">
  <option *ngFor="let item of reports" [value]="item">{{ item }}</option>
</select>

<stimulsoft-viewer-angular
  [requestUrl]="'http://localhost:60801/Viewer/{action}'"
  [action]="'InitViewer'"
  [height]="'100vh'"
  [properties]="properties"
></stimulsoft-viewer-angular>
```

**Angular component:**
```typescript
export class AppComponent {
    reports = ['MasterDetail.mrt', 'EditableReport.mrt'];
    properties = { reportName: this.reports[0] };

    updateProps(reportName: string): void {
        this.properties = { reportName };
    }
}
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
    var path = StiAngularHelper.MapPath(this, $"Reports/{reportName}");
    report.Load(path);

    return StiAngularViewer.GetReportResult(this, report);
}
```

Properties are sent as a Base64-encoded JSON string and decoded on the server.

---

## 6. How to handle Viewer events?

Bind Angular event handlers to the Viewer component outputs:

**Angular template:**
```html
<stimulsoft-viewer-angular
  [requestUrl]="'http://localhost:60801/Viewer/{action}'"
  [action]="'InitViewer'"
  [height]="'100vh'"
  (loaded)="loaded()"
  (export)="export($event)"
></stimulsoft-viewer-angular>
```

**Angular component:**
```typescript
export class AppComponent {
    loaded(): void {
        console.log('Report loaded');
    }

    export(event: any): void {
        console.log(`Export to: ${event.exportFormat}`);
    }
}
```

Available events include: `(loaded)`, `(export)`, `(design)`, and others.

---

## 7. How to use the Viewer API methods?

Access the Viewer API using `@ViewChild`:

**Angular template:**
```html
Zoom is {{ viewer.api.zoom }}<br />
Current page {{ viewer.api.currentPage + 1 }}<br />
<input type="button" (click)="viewer.api.zoom = 50" value="Zoom to 50%" />
<input type="button"
  (click)="viewer.api.export('Pdf', { ImageResolution: 200 })"
  value="Export to PDF" />

<stimulsoft-viewer-angular
  #viewer
  [requestUrl]="'http://localhost:60801/Viewer/{action}'"
  [action]="'InitViewer'"
  [height]="'100vh'"
></stimulsoft-viewer-angular>
```

**Angular component:**
```typescript
import { ViewChild } from '@angular/core';
import { StimulsoftViewerComponent } from 'stimulsoft-viewer-angular';

export class AppComponent {
    @ViewChild('viewer') viewer: StimulsoftViewerComponent;
}
```

The `viewer.api` object provides properties and methods: `zoom`, `currentPage`, `export()`, and others.

---

## 8. How to customize the Viewer options?

Set options in the server-side `InitViewer` action using `StiAngularViewerOptions`:

```csharp
[HttpPost]
public IActionResult InitViewer()
{
    var requestParams = StiAngularViewer.GetRequestParams(this);

    var options = new StiAngularViewerOptions();
    options.Actions.GetReport = "GetReport";
    options.Actions.ViewerEvent = "ViewerEvent";
    options.Appearance.ScrollbarsMode = true;
    options.Toolbar.ShowDesignButton = true;
    options.Toolbar.ShowSendEmailButton = true;

    return StiAngularViewer.ViewerDataResult(requestParams, options);
}
```

**Key options:**

| Property | Description |
|----------|-------------|
| `options.Appearance.ScrollbarsMode` | Enable scroll bars |
| `options.Toolbar.ShowDesignButton` | Show the Design button to switch to designer |
| `options.Toolbar.ShowSendEmailButton` | Show the Send Email button |

---

## 9. How to use the Designer?

The Designer requires both an Angular component and a server-side controller.

**Angular template — switch between Viewer and Designer:**
```html
<stimulsoft-viewer-angular
  *ngIf="showViewer"
  [requestUrl]="'http://localhost:60801/Viewer/{action}'"
  [action]="'InitViewer'"
  [height]="'100%'"
  (design)="showViewer = false"
></stimulsoft-viewer-angular>

<stimulsoft-designer-angular
  #designer
  *ngIf="!showViewer"
  [requestUrl]="'http://localhost:60801/api/designer'"
  [height]="'100%'"
  [width]="'100%'"
  (designerLoaded)="this.designer.designerEl.nativeElement.style.height='100%'"
>
  Loading designer...
</stimulsoft-designer-angular>
```

**Angular component:**
```typescript
import { Component, ElementRef, ViewChild } from '@angular/core';

export class AppComponent {
    showViewer = true;
    @ViewChild('designer') designer: ElementRef;
}
```

**Server-side designer controller:**
```csharp
using Stimulsoft.Report.Angular;
using Stimulsoft.Report.Web;

[Produces("application/json")]
[Route("api/designer")]
public class DesignerController : Controller
{
    [HttpGet]
    public IActionResult Get()
    {
        var requestParams = StiAngularDesigner.GetRequestParams(this);
        if (requestParams.Action == StiAction.Undefined)
        {
            var options = new StiAngularDesignerOptions();
            return StiAngularDesigner.DesignerDataResult(requestParams, options);
        }

        return StiAngularDesigner.ProcessRequestResult(this);
    }

    [HttpPost]
    public IActionResult Post()
    {
        var requestParams = StiAngularDesigner.GetRequestParams(this);
        if (requestParams.ComponentType == StiComponentType.Designer)
        {
            switch (requestParams.Action)
            {
                case StiAction.GetReport:
                    return GetReport();
                case StiAction.SaveReport:
                    return SaveReport();
            }
        }

        return StiAngularDesigner.ProcessRequestResult(this);
    }

    public IActionResult GetReport()
    {
        var report = StiReport.CreateNewReport();
        var path = StiAngularHelper.MapPath(this, "Reports/MasterDetail.mrt");
        report.Load(path);
        return StiAngularDesigner.GetReportResult(this, report);
    }

    public IActionResult SaveReport()
    {
        var report = StiAngularDesigner.GetReportObject(this);
        var path = StiAngularHelper.MapPath(this, "Reports/MasterDetail.mrt");
        report.Save(path);
        return StiAngularDesigner.SaveReportResult(this);
    }
}
```

The `(design)` event on the Viewer fires when the user clicks the Design button, allowing you to toggle to the Designer component.

---

## 10. How to localize the Viewer?

Set the localization path in the server-side `InitViewer` action:

```csharp
[HttpPost]
public IActionResult InitViewer()
{
    var requestParams = StiAngularViewer.GetRequestParams(this);

    var options = new StiAngularViewerOptions();
    options.Actions.GetReport = "GetReport";
    options.Actions.ViewerEvent = "ViewerEvent";
    options.Appearance.ScrollbarsMode = true;
    options.Localization = StiAngularHelper.MapPath(this, "Localization/de.xml");

    return StiAngularViewer.ViewerDataResult(requestParams, options);
}
```

Place localization XML files in a `Localization` folder in your ASP.NET project. Stimulsoft provides localization files for many languages.

---

## 11. How to send a report by email?

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
    StiEmailOptions options = StiAngularViewer.GetEmailOptions(this);

    // Fill server-side SMTP settings
    options.AddressFrom = "admin_address@test.com";
    options.Host = "smtp.test.com";
    options.Port = 465;
    options.UserName = "admin_address@test.com";
    options.Password = "admin_password";

    return StiAngularViewer.EmailReportResult(this, options);
}
```

The recipient address, subject, and body are passed from the Viewer dialog. SMTP settings must be configured on the server side.

---

## 12. How to use a Standalone Angular Component?

For Angular 19+ with standalone components, import `StimulsoftViewerModule` directly in the component instead of `NgModule`:

**Component:**
```typescript
import { Component } from '@angular/core';
import { StimulsoftViewerModule } from 'stimulsoft-viewer-angular';

@Component({
    selector: 'app-root',
    imports: [StimulsoftViewerModule],
    templateUrl: './app.component.html',
    styleUrl: './app.component.css'
})
export class AppComponent {
    title = 'viewer';
}
```

**Bootstrap with `appConfig`:**
```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
    providers: [
        provideZoneChangeDetection({ eventCoalescing: true }),
        provideRouter(routes),
        provideHttpClient(),
        provideAnimationsAsync()
    ]
};
```

**`main.ts`:**
```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig)
    .catch((err) => console.error(err));
```

This approach does not require `AppModule` and uses the modern standalone component pattern.

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.WEB-for-ASP.NET-Angular).*
