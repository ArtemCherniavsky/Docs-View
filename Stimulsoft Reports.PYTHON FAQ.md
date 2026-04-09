# Stimulsoft Reports.PYTHON — FAQ

Quick answers for developers integrating Stimulsoft Reports into Python applications using Flask, Django, or Tornado.

---

## Table of Contents

1. [How do I install Stimulsoft Reports for Python?](#1-how-do-i-install-stimulsoft-reports-for-python)
2. [How to activate the license?](#2-how-to-activate-the-license)
3. [How to show a report in the Viewer?](#3-how-to-show-a-report-in-the-viewer)
4. [How to show the Viewer in an HTML template?](#4-how-to-show-the-viewer-in-an-html-template)
5. [How to edit a report in the Designer?](#5-how-to-edit-a-report-in-the-designer)
6. [How to register data from code?](#6-how-to-register-data-from-code)
7. [How to export a report from code?](#7-how-to-export-a-report-from-code)
8. [How to render and export on the server side using Node.js?](#8-how-to-render-and-export-on-the-server-side-using-nodejs)
9. [How to save a report template on the server side?](#9-how-to-save-a-report-template-on-the-server-side)
10. [How to set report variables on the server side?](#10-how-to-set-report-variables-on-the-server-side)
11. [How to change export settings on the server side?](#11-how-to-change-export-settings-on-the-server-side)
12. [How to change the Viewer theme?](#12-how-to-change-the-viewer-theme)
13. [How to localize the Designer?](#13-how-to-localize-the-designer)
14. [How to send a report by email?](#14-how-to-send-a-report-by-email)
15. [How to use a handler in a separate function?](#15-how-to-use-a-handler-in-a-separate-function)
16. [How to configure and install Node.js?](#16-how-to-configure-and-install-nodejs)

---

## 1. How do I install Stimulsoft Reports for Python?

Install the package from PyPI:

```bash
pip install stimulsoft-reports
```

**Supported frameworks:** Flask, Django, Tornado.

**Supported Python versions:** Python 3.x.

The package includes the Viewer, Designer, and Report engine. For server-side report rendering and exporting, Node.js is additionally required (see [question 16](#16-how-to-configure-and-install-nodejs)).

---

## 2. How to activate the license?

Use `StiLicense` to set your license key before creating any report or viewer objects:

```python
from stimulsoft_reports import StiLicense

# Option 1: set the key as a string
StiLicense.setKey('your-license-key-here...')

# Option 2: load from a file
StiLicense.setFile('path/to/license.key')
```

The license must be set once at application startup, before any report operations.

---

## 3. How to show a report in the Viewer?

Create an `StiViewer` object, load a report, and return the response. The example below uses Flask:

```python
from flask import Blueprint, request, url_for
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.viewer import StiViewer

app = Blueprint('viewer', __name__)

@app.route('/viewer', methods=['GET', 'POST'])
def index():
    viewer = StiViewer()

    if viewer.processRequest(request):
        return viewer.getFrameworkResponse()

    report = StiReport()
    reportUrl = url_for('static', filename='reports/SimpleList.mrt')
    report.loadFile(reportUrl)

    viewer.report = report
    return viewer.getFrameworkResponse()
```

**Django** — the same pattern, but the view is a plain function receiving `request`, and URLs use `static()`:

```python
from django.templatetags.static import static
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.viewer import StiViewer

def index(request):
    viewer = StiViewer()

    if viewer.processRequest(request):
        return viewer.getFrameworkResponse()

    report = StiReport()
    report.loadFile(static('reports/SimpleList.mrt'))

    viewer.report = report
    return viewer.getFrameworkResponse()
```

**Tornado** — uses `RequestHandler` with a separate `StiHandler` for POST requests:

```python
from tornado.web import RequestHandler
from stimulsoft_reports import StiHandler
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.viewer import StiViewer

class IndexHandler(RequestHandler):
    handler = StiHandler()

    def get(self):
        viewer = StiViewer()
        viewer.handler = self.handler

        if viewer.processRequest(self.request):
            return viewer.getFrameworkResponse(self)

        report = StiReport()
        report.loadFile(self.static_url('reports/SimpleList.mrt'))

        viewer.report = report
        return viewer.getFrameworkResponse(self)

    def post(self):
        if self.handler.processRequest(self.request):
            return self.handler.getFrameworkResponse(self)
```

Key differences: in Tornado, `getFrameworkResponse()` requires passing `self` as a parameter, and a separate `StiHandler` object handles POST requests.

---

## 4. How to show the Viewer in an HTML template?

Instead of returning a full HTML page from `getFrameworkResponse()`, you can embed the Viewer into your own HTML template:

```python
from flask import Blueprint, request, render_template, url_for
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.viewer import StiViewer

app = Blueprint('viewer_template', __name__)

@app.route('/viewer-template', methods=['GET', 'POST'])
def index():
    viewer = StiViewer()
    viewer.options.appearance.scrollbarsMode = True
    viewer.options.width = '1000px'
    viewer.options.height = '600px'

    if viewer.processRequest(request):
        return viewer.getFrameworkResponse()

    report = StiReport()
    reportUrl = url_for('static', filename='reports/SimpleList.mrt')
    report.loadFile(reportUrl)

    viewer.report = report

    js = viewer.javascript.getHtml()
    html = viewer.getHtml()

    return render_template('viewer.html', viewerJavaScript=js, viewerHtml=html)
```

In the HTML template, output the JavaScript and HTML parts:

```html
<!DOCTYPE html>
<html>
<head>
    {{ viewerJavaScript|safe }}
</head>
<body>
    {{ viewerHtml|safe }}
</body>
</html>
```

Use `viewer.javascript.getHtml()` for the `<head>` section and `viewer.getHtml()` for the `<body>` section.

---

## 5. How to edit a report in the Designer?

The Designer follows the same pattern as the Viewer — use `StiDesigner` instead of `StiViewer`:

```python
from flask import Blueprint, request, url_for
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.designer import StiDesigner

app = Blueprint('designer', __name__)

@app.route('/designer', methods=['GET', 'POST'])
def index():
    designer = StiDesigner()

    if designer.processRequest(request):
        return designer.getFrameworkResponse()

    report = StiReport()
    reportUrl = url_for('static', filename='reports/SimpleList.mrt')
    report.loadFile(reportUrl)

    designer.report = report
    return designer.getFrameworkResponse()
```

The Designer can also be embedded into an HTML template using `designer.javascript.getHtml()` and `designer.getHtml()`, similar to the Viewer.

---

## 6. How to register data from code?

Use the `onBeginProcessData` event on the Viewer or Designer. When assigning a **string**, the function is called on the JavaScript client side; when assigning a **Python function**, it is called on the server side:

**Client-side data registration (JavaScript callback):**

```python
viewer = StiViewer()
viewer.onBeginProcessData += 'beginProcessData'
```

In the HTML template, define the JavaScript function:

```javascript
function beginProcessData(args) {
    // Register data on the client side
}
```

**Server-side data processing (Python callback):**

```python
from stimulsoft_reports.events import StiDataEventArgs

def beginProcessData(args: StiDataEventArgs):
    if args.dataSource == 'customers' and len(args.parameters) > 0:
        args.parameters['Country'].value = 'Germany'

viewer = StiViewer()
viewer.onBeginProcessData += beginProcessData
```

In the server-side callback, you can check and change the connection, data source, and query parameters. Changes are not sent to the client side.

---

## 7. How to export a report from code?

Use `StiReport` with `render()` and `exportDocument()` methods:

```python
from flask import Blueprint, request, render_template, url_for
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.report.enums import StiExportFormat

app = Blueprint('export', __name__)

@app.route('/export', methods=['GET', 'POST'])
def index():
    report = StiReport()

    if report.processRequest(request):
        return report.getFrameworkResponse()

    reportUrl = url_for('static', filename='reports/SimpleList.mrt')
    report.loadFile(reportUrl)

    report.render()
    report.exportDocument(StiExportFormat.PDF)

    js = report.javascript.getHtml()
    html = report.getHtml()

    return render_template('export.html', reportJavaScript=js, reportHtml=html)
```

**Available export formats:** `StiExportFormat.PDF`, `StiExportFormat.EXCEL`, `StiExportFormat.HTML`, `StiExportFormat.DOCUMENT`, and others.

Note: by default, `render()` and `exportDocument()` generate JavaScript code that runs on the client side. For server-side rendering via Node.js, see [question 8](#8-how-to-render-and-export-on-the-server-side-using-nodejs).

---

## 8. How to render and export on the server side using Node.js?

Set the engine type to `SERVER_NODE_JS` for server-side rendering and exporting. This requires Node.js to be installed (see [question 16](#16-how-to-configure-and-install-nodejs)):

```python
from flask import Blueprint, request, render_template, current_app
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.report.enums import StiEngineType, StiExportFormat

app = Blueprint('server_export', __name__)

@app.route('/server-export', methods=['GET', 'POST'])
def index():
    report = StiReport()
    report.engine = StiEngineType.SERVER_NODE_JS

    if report.processRequest(request):
        return report.getFrameworkResponse()

    # When using server-side engine, load the report by file path (not URL)
    reportPath = current_app.static_folder + '/reports/SimpleList.mrt'
    report.loadFile(reportPath)

    # render() executes via Node.js and returns True/False
    result = report.render()

    if result:
        # exportDocument() returns byte data or saves to file
        buffer = report.exportDocument(StiExportFormat.PDF)
        message = f'The exported document takes {len(buffer)} bytes.'

        # Or save to file:
        # report.exportDocument(StiExportFormat.PDF, '/path/to/output.pdf')
    else:
        message = report.nodejs.error

    return render_template('message.html', message=message)
```

You can also render a report and save the finished document:

```python
report.engine = StiEngineType.SERVER_NODE_JS
report.loadFile(reportPath)

result = report.render()
if result:
    document = report.saveDocument()
    # Or save to file:
    # report.saveDocument('/path/to/output.mdc')
```

When using `StiEngineType.SERVER_NODE_JS`, the `loadFile()` method loads the report file and stores it as a compressed string. The report is then built and exported using the Node.js engine.

---

## 9. How to save a report template on the server side?

Use the `onSaveReport` event on the Designer with a Python callback function:

```python
import json
import os
from flask import Blueprint, request, url_for
from stimulsoft_reports import StiResult
from stimulsoft_reports.designer import StiDesigner
from stimulsoft_reports.events import StiReportEventArgs
from stimulsoft_reports.report import StiReport

app = Blueprint('save_report', __name__)

def saveReport(args: StiReportEventArgs):
    filePath = os.path.normpath(
        os.getcwd() + url_for('static', filename='reports/' + args.fileName)
    )
    try:
        with open(filePath, mode='w', encoding='utf-8') as file:
            jsonReport = json.dumps(args.report, indent=4)
            file.write(jsonReport)
    except Exception as e:
        return StiResult.getError(str(e))

    return f'The report was successfully saved to a {args.fileName} file.'


@app.route('/save-report', methods=['GET', 'POST'])
def index():
    designer = StiDesigner()
    designer.onSaveReport += saveReport

    if designer.processRequest(request):
        return designer.getFrameworkResponse()

    report = StiReport()
    reportUrl = url_for('static', filename='reports/SimpleList.mrt')
    report.loadFile(reportUrl)

    designer.report = report
    return designer.getFrameworkResponse()
```

The `StiReportEventArgs` provides `args.report` (the report as a dictionary) and `args.fileName` (the file name). Return a string message on success, or use `StiResult.getError()` to pass error messages back to the Designer.

---

## 10. How to set report variables on the server side?

Use the `onPrepareVariables` event on the Viewer:

```python
from datetime import datetime
from stimulsoft_reports.events import StiVariablesEventArgs
from stimulsoft_reports.viewer import StiViewer

def prepareVariables(args: StiVariablesEventArgs):
    if len(args.variables) > 0:
        args.variables['Name'].value = 'Maria'
        args.variables['Surname'].value = 'Anders'
        args.variables['Email'].value = 'm.anders@stimulsoft.com'
        args.variables['Address'].value = 'Obere Str. 57, Berlin'
        args.variables['Sex'].value = False
        args.variables['BirthDay'].value = datetime(1982, 3, 20, 0, 0, 0)

viewer = StiViewer()
viewer.onPrepareVariables += prepareVariables
```

The function is called before building the report when preparing variable values. Variable value types must match the original types defined in the report template.

---

## 11. How to change export settings on the server side?

Use the `onBeginExportReport` event on the Viewer:

```python
from stimulsoft_reports.events import StiExportEventArgs
from stimulsoft_reports.export import StiPdfExportSettings
from stimulsoft_reports.report.enums import StiExportFormat
from stimulsoft_reports.viewer import StiViewer

def beginExportReport(args: StiExportEventArgs):
    args.fileName = 'MyExportedFileName.' + args.fileExtension

    if args.format == StiExportFormat.PDF:
        settings: StiPdfExportSettings = args.settings
        settings.creatorString = 'My Company Name'
        settings.embeddedFonts = False

viewer = StiViewer()
viewer.onBeginExportReport += beginExportReport
```

The set of available settings depends on the export format. You can also save the exported file on the server side using the `onEndExportReport` event:

```python
import os
from stimulsoft_reports import StiResult
from stimulsoft_reports.events import StiExportEventArgs

def endExportReport(args: StiExportEventArgs):
    filePath = os.path.normpath('reports/' + args.fileName)
    try:
        with open(filePath, mode='wb') as file:
            file.write(args.data)
    except Exception as e:
        return StiResult.getError(str(e))

    return f'The exported report was saved to {args.fileName}.'

viewer.onEndExportReport += endExportReport
```

---

## 12. How to change the Viewer theme?

Set the theme and appearance options on the Viewer:

```python
from stimulsoft_reports.viewer import StiViewer
from stimulsoft_reports.viewer.enums import StiViewerTheme, StiToolbarDisplayMode

viewer = StiViewer()
viewer.options.appearance.theme = StiViewerTheme.OFFICE_2022_BLACK_GREEN
viewer.options.appearance.backgroundColor = 'black'
viewer.options.toolbar.displayMode = StiToolbarDisplayMode.SEPARATED
```

Available themes are defined in `StiViewerTheme`. Additional appearance options include `scrollbarsMode`, `fullScreenMode`, and toolbar settings like `showDesignButton` and `showPinToolbarButton`.

---

## 13. How to localize the Designer?

Set the localization file in the Designer options:

```python
from stimulsoft_reports.designer import StiDesigner

designer = StiDesigner()

# Setting the main localization
designer.options.localization = 'de.xml'

# Adding optional localizations to the designer menu
designer.options.localizations.append('es.xml')
designer.options.localizations.append('pt.xml')
```

Localization files can be obtained from the [Stimulsoft Localization repository on GitHub](https://github.com/stimulsoft/Stimulsoft.Reports.Localization). The main localization is applied by default, and optional localizations are displayed in the localization menu in the Designer panel.

---

## 14. How to send a report by email?

Enable the email button on the Viewer and handle the `onEmailReport` event with SMTP settings:

```python
from stimulsoft_reports.events import StiEmailEventArgs
from stimulsoft_reports.viewer import StiViewer

def emailReport(args: StiEmailEventArgs):
    args.settings.fromAddr = 'sender@example.com'
    args.settings.host = 'smtp.example.com'
    args.settings.port = 465
    args.settings.login = 'smtp_login'
    args.settings.password = 'smtp_password'
    return 'The Email has been sent successfully.'

viewer = StiViewer()
viewer.options.toolbar.showSendEmailButton = True
viewer.onEmailReport += emailReport
```

The recipient address, subject, and body are filled in by the user in the Viewer email dialog. SMTP settings are configured on the server side and are not sent to the client.

---

## 15. How to use a handler in a separate function?

Use `StiHandler` to process requests in a separate route, separate from the main component route:

```python
from flask import Blueprint, request, render_template, url_for
from stimulsoft_reports import StiHandler
from stimulsoft_reports.report import StiReport
from stimulsoft_reports.viewer import StiViewer

app = Blueprint('handler_example', __name__)

mainHandler = StiHandler('/handler-example/handler')

@app.route('/handler-example')
def index():
    viewer = StiViewer()
    viewer.handler = mainHandler
    viewer.options.appearance.fullScreenMode = True

    report = StiReport()
    reportUrl = url_for('static', filename='reports/SimpleList.mrt')
    report.loadFile(reportUrl)

    viewer.report = report

    js = viewer.javascript.getHtml()
    html = viewer.getHtml()

    return render_template('handler.html', viewerJavaScript=js, viewerHtml=html)


@app.route('/handler-example/handler', methods=['GET', 'POST'])
def handler():
    mainHandler.processRequest(request)
    return mainHandler.getFrameworkResponse()
```

This pattern is useful when you need to embed the Viewer or Designer into an HTML template and handle POST requests at a separate URL. In Tornado, `StiHandler` is always required because GET and POST are handled by different methods.

---

## 16. How to configure and install Node.js?

Node.js is required for server-side report rendering and exporting. Use `StiNodeJs` to install and configure it:

```python
from stimulsoft_reports import StiNodeJs

nodejs = StiNodeJs()

# Optional: set the path to an already installed Node.js
# nodejs.binDirectory = '/usr/bin/nodejs'

# Optional: set the working directory for Node.js packages
# nodejs.workingDirectory = '/path/to/working/dir'

# Install Node.js from the official website
result = nodejs.installNodeJS()

# Install or update Stimulsoft packages for Node.js
if result:
    result = nodejs.updatePackages()

if result:
    print('Installation successful.')
else:
    print(nodejs.error)
```

After installation, set `report.engine = StiEngineType.SERVER_NODE_JS` on the report object to enable server-side rendering (see [question 8](#8-how-to-render-and-export-on-the-server-side-using-nodejs)).

---

*For more details see the [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.Python).*
