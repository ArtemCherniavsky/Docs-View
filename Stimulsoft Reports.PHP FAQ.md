# Stimulsoft Reports.PHP — FAQ

Quick answers for developers integrating Stimulsoft Reports.PHP into PHP applications.

> **Architecture note:** Stimulsoft Reports.PHP uses a client-server model. PHP generates JavaScript code that runs the report engine in the browser. Server-side rendering (via Node.js) is also supported for headless use cases.

---

## Table of Contents

1. [How do I install Stimulsoft Reports.PHP?](#1-how-do-i-install-stimulsoft-reportsphp)
2. [How do I activate the license?](#2-how-do-i-activate-the-license)
3. [How do I show a report in the viewer?](#3-how-do-i-show-a-report-in-the-viewer)
4. [How do I render a report from code (client-side)?](#4-how-do-i-render-a-report-from-code-client-side)
5. [How do I print a report from code?](#5-how-do-i-print-a-report-from-code)
6. [How do I export a report from code?](#6-how-do-i-export-a-report-from-code)
7. [How do I open the report designer?](#7-how-do-i-open-the-report-designer)
8. [How do I save a report changed in the designer on the server?](#8-how-do-i-save-a-report-changed-in-the-designer-on-the-server)
9. [How do I set report variable values?](#9-how-do-i-set-report-variable-values)
10. [How do I register data from code?](#10-how-do-i-register-data-from-code)
11. [How do I use SQL data sources?](#11-how-do-i-use-sql-data-sources)
12. [How do I use parameters in SQL queries?](#12-how-do-i-use-parameters-in-sql-queries)
13. [How do I change report properties on the server-side?](#13-how-do-i-change-report-properties-on-the-server-side)
14. [How do I change export settings on the server-side?](#14-how-do-i-change-export-settings-on-the-server-side)
15. [How do I customize the viewer?](#15-how-do-i-customize-the-viewer)
16. [How do I change the viewer or designer theme?](#16-how-do-i-change-the-viewer-or-designer-theme)
17. [How do I send a report by email?](#17-how-do-i-send-a-report-by-email)
18. [How do I render and export a report on the server-side (Node.js)?](#18-how-do-i-render-and-export-a-report-on-the-server-side-nodejs)
19. [Which databases are supported?](#19-which-databases-are-supported)

---

## 1. How do I install Stimulsoft Reports.PHP?

**Via Composer (recommended):**
```bash
composer require stimulsoft/reports-php
```

To update to the latest version:
```bash
composer update stimulsoft/reports-php
```

**Manual installation:** Copy all product files to your PHP server directory via FTP or your hosting panel, then open in the browser:
```
http://your-domain/index.php
```

After installation, add the autoloader at the top of every PHP file that uses Stimulsoft:
```php
require_once 'vendor/autoload.php';
```

---

## 2. How do I activate the license?

Always activate **before** calling `$report->process()` or any other Stimulsoft method.

**Option A — global activation (all components):**
```php
use Stimulsoft\StiLicense;

// From a key string
StiLicense::setPrimaryKey('your license key...');

// From a .key file
StiLicense::setPrimaryFile('license.key');
```

**Option B — per-component activation:**
```php
$report->license->setKey('your license key...');
$report->license->setFile('license.key');
```

**Option C — from JavaScript (client-side):**
```html
<script>
    Stimulsoft.Base.StiLicense.loadFromString('your license key...');
    // or
    Stimulsoft.Base.StiLicense.loadFromFile('license.key');
</script>
```

Download your license key from your [personal account](https://devs.stimulsoft.com/). The 30-day trial has no functional restrictions except for a watermark on report pages.

---

## 3. How do I show a report in the viewer?

The fastest way — `printHtml()` outputs a complete ready-to-use HTML page:

```php
<?php
require_once 'vendor/autoload.php';

use Stimulsoft\Report\StiReport;
use Stimulsoft\Viewer\StiViewer;

$viewer = new StiViewer();
$viewer->javascript->relativePath = './'; // path to the scripts folder

// process() handles requests from the viewer (navigation, export, etc.)
$viewer->process();

$report = new StiReport();
$report->loadFile('reports/SimpleList.mrt');

$viewer->report = $report;
$viewer->printHtml(); // outputs a full HTML page
```

**Embedding the viewer into an existing HTML template:**
```php
<?php
require_once 'vendor/autoload.php';

use Stimulsoft\Report\StiReport;
use Stimulsoft\Viewer\StiViewer;

$viewer = new StiViewer();
$viewer->javascript->relativePath = './';
$viewer->process();

$report = new StiReport();
$report->loadFile('reports/SimpleList.mrt');
$viewer->report = $report;
?>
<!DOCTYPE html>
<html>
<head>
    <?php $viewer->javascript->renderHtml(); // outputs required <script> tags ?>
</head>
<body>
    <?php $viewer->renderHtml(); // outputs the viewer HTML element ?>
</body>
</html>
```

---

## 4. How do I render a report from code (client-side)?

Use `StiReport` alone — without a viewer — when you need to render and work with the output yourself (e.g. show inline, export, save as JSON).

```php
<?php
require_once 'vendor/autoload.php';

use Stimulsoft\Report\StiReport;

$report = new StiReport();
$report->javascript->relativePath = './';

// Subscribe to the after-render event
$report->onAfterRender = 'onAfterRender';

$report->process();
$report->loadFile('reports/SimpleList.mrt');
$report->render();
?>
<!DOCTYPE html>
<html>
<head>
    <?php $report->javascript->renderHtml(); ?>
    <script>
        function onAfterRender(args) {
            var pages = args.report.renderedPages.count;
            document.getElementById("msg").innerText = "Rendered pages: " + pages;
        }
    </script>
</head>
<body>
    <?php $report->renderHtml(); ?>
    <div id="msg"></div>
</body>
</html>
```

> `loadFile()` and `render()` do **not** execute on the PHP server — they generate JavaScript that runs in the browser.

---

## 5. How do I print a report from code?

```php
<?php
require_once 'vendor/autoload.php';

use Stimulsoft\Report\StiReport;

$report = new StiReport();
$report->javascript->relativePath = './';

$report->process();
$report->loadFile('reports/SimpleList.mrt');
$report->render();
$report->print(); // triggers browser print dialog after rendering

// printHtml() outputs a minimal complete page
$report->printHtml();
```

---

## 6. How do I export a report from code?

```php
<?php
require_once 'vendor/autoload.php';

use Stimulsoft\Export\Enums\StiExportFormat;
use Stimulsoft\Report\StiReport;

$report = new StiReport();
$report->javascript->relativePath = './';

$report->process();
$report->loadFile('reports/SimpleList.mrt');
$report->render();
?>
<!DOCTYPE html>
<html>
<head>
    <?php $report->javascript->renderHtml(); ?>
    <script>
        function exportToPdf() {
            <?php
            $report->exportDocument(StiExportFormat::Pdf);
            echo $report->getHtml(\Stimulsoft\Enums\StiHtmlMode::Scripts);
            ?>
        }
        function exportToExcel() {
            <?php
            $report->exportDocument(StiExportFormat::Excel);
            echo $report->getHtml(\Stimulsoft\Enums\StiHtmlMode::Scripts);
            ?>
        }
        function exportToHtml() {
            <?php
            $report->exportDocument(StiExportFormat::Html);
            echo $report->getHtml(\Stimulsoft\Enums\StiHtmlMode::Scripts);
            ?>
        }
    </script>
</head>
<body>
    <button onclick="exportToPdf()">Export to PDF</button>
    <button onclick="exportToExcel()">Export to Excel</button>
    <button onclick="exportToHtml()">Export to HTML</button>
</body>
</html>
```

**Common export format constants (`StiExportFormat`):**

| Constant | Format |
|----------|--------|
| `StiExportFormat::Pdf` | PDF |
| `StiExportFormat::Excel` | Excel 97-2003 |
| `StiExportFormat::Excel2007` | Excel 2007+ (.xlsx) |
| `StiExportFormat::Word2007` | Word 2007+ (.docx) |
| `StiExportFormat::Html` | HTML |
| `StiExportFormat::Csv` | CSV |
| `StiExportFormat::Text` | Plain text |
| `StiExportFormat::ImagePng` | PNG image |

---

## 7. How do I open the report designer?

```php
<?php
require_once 'vendor/autoload.php';

use Stimulsoft\Designer\StiDesigner;
use Stimulsoft\Report\StiReport;

$designer = new StiDesigner();
$designer->javascript->relativePath = './';

$designer->process();

$report = new StiReport();
$report->loadFile('reports/SimpleList.mrt'); // open existing template
// $report = new StiReport();               // or start with an empty report

$designer->report = $report;
$designer->printHtml(); // outputs a complete page with the designer
```

**Embedding the designer into an HTML template:**
```php
<?php $designer->javascript->renderHtml(); // <script> tags in <head> ?>
...
<?php $designer->renderHtml(); // designer element in <body> ?>
```

---

## 8. How do I save a report changed in the designer on the server?

Use the `onSaveReport` event — it fires when the user clicks the Save button:

```php
use Stimulsoft\Designer\StiDesigner;
use Stimulsoft\Events\StiReportEventArgs;
use Stimulsoft\StiResult;

$designer = new StiDesigner();
$designer->javascript->relativePath = './';

$designer->onSaveReport = function (StiReportEventArgs $args) {
    // Get the file name (fallback to 'Report.mrt' if empty)
    $fileName = strlen($args->fileName) > 0 ? $args->fileName : 'Report.mrt';
    if (!str_ends_with($fileName, '.mrt'))
        $fileName .= '.mrt';

    // Save to the reports folder on the server
    $path = "reports/$fileName";
    $result = file_put_contents($path, $args->getReportJson());

    if ($result === false)
        return StiResult::getError('Failed to save the report file.');

    return StiResult::getSuccess("Saved to '$path'.");
};

$designer->process();

$report = new StiReport();
$report->loadFile('reports/SimpleList.mrt');
$designer->report = $report;
$designer->printHtml();
```

---

## 9. How do I set report variable values?

Use the `onPrepareVariables` viewer event — it fires on the PHP server before the report is built:

```php
use Stimulsoft\Events\StiVariablesEventArgs;
use Stimulsoft\Viewer\StiViewer;

$viewer = new StiViewer();
$viewer->javascript->relativePath = './';

$viewer->onPrepareVariables = function (StiVariablesEventArgs $args) {
    if (count($args->variables) > 0) {
        // String variable
        $args->variables['Name']->value = 'Maria';

        // Date variable (ISO format)
        $args->variables['BirthDay']->value = '1982-03-20 00:00:00';

        // Boolean variable
        $args->variables['IsActive']->value = true;

        // Range variable
        $args->variables['DateRange']->value->from = '2024-01-01';
        $args->variables['DateRange']->value->to   = '2024-12-31';

        // List variable
        $args->variables['StatusList']->value = ['Active', 'Pending'];

        // Add a new variable not present in the template
        $args->variables['NewVar'] = ['value' => 'injected value'];
    }
};

$viewer->process();

$report = new StiReport();
$report->loadFile('reports/Variables.mrt');
$viewer->report = $report;
$viewer->printHtml();
```

---

## 10. How do I register data from code?

Data registration runs in the browser (via JavaScript). Use the `onBeforeRender` report event:

```php
$report = new StiReport();
$report->javascript->relativePath = './';

// Assign a JavaScript function name — it will be called in the browser
$report->onBeforeRender = 'onBeforeRender';

$report->process();
$report->loadFile('reports/SimpleList.mrt');
$report->render();
?>
<head>
    <?php $report->javascript->renderHtml(); ?>
    <script>
        function onBeforeRender(args) {
            var dataSet = new Stimulsoft.System.Data.DataSet("Demo");

            // Load from XML
            dataSet.readXmlSchemaFile("data/Demo.xsd");
            dataSet.readXmlFile("data/Demo.xml");

            // Or load from JSON
            // dataSet.readJsonFile("data/Demo.json");

            args.report.dictionary.databases.clear();
            args.report.regData("Demo", "Demo", dataSet);
        }
    </script>
</head>
```

---

## 11. How do I use SQL data sources?

Override connection strings and queries using the `onBeginProcessData` event:

```php
use Stimulsoft\Events\StiDataEventArgs;
use Stimulsoft\Viewer\StiViewer;

$viewer = new StiViewer();
$viewer->javascript->relativePath = './';

$viewer->onBeginProcessData = function (StiDataEventArgs $args) {

    // Override connection string for a specific connection
    if ($args->connection == 'MyConnectionName')
        $args->connectionString = 'Server=localhost; Database=northwind; UserId=root; Pwd=secret;';

    // Override the SQL query for a specific data source
    if ($args->dataSource == 'MyDataSource')
        $args->queryString = 'SELECT * FROM Orders WHERE Active = 1';
};

$viewer->process();

$report = new StiReport();
$report->loadFile('reports/SqlReport.mrt');
$viewer->report = $report;
$viewer->printHtml();
```

**Supported databases:** MySQL, MSSQL (SQL Server), PostgreSQL, Oracle, Firebird, ODBC, and more.

---

## 12. How do I use parameters in SQL queries?

Define parameters in the SQL query using `@ParameterName` syntax, then set their values in `onBeginProcessData`:

```php
$viewer->onBeginProcessData = function (StiDataEventArgs $args) {

    // SQL in the template: SELECT * FROM Orders WHERE CustomerId = @CustomerId AND Date > @FromDate
    if ($args->dataSource == 'OrdersDataSource') {
        $args->parameters['CustomerId']->value = 42;
        $args->parameters['FromDate']->value   = '2024-01-01';
    }

    // Dynamic table name: SELECT * FROM @TableName
    if ($args->dataSource == 'DynamicTable') {
        $args->parameters['TableName']->value = 'Products';
    }
};
```

---

## 13. How do I change report properties on the server-side?

Load the `.mrt` file as JSON, modify it with PHP, then pass the modified object to the report:

```php
// Load the .mrt file as a JSON string
$reportJson = json_decode(file_get_contents('reports/Variables.mrt'));

// Modify properties
$reportJson->ReportAlias    = 'My Custom Report Title';
$reportJson->CalculationMode = 'Interpretation';

// Change a variable value directly in the JSON
$reportJson->Dictionary->Variables->{'0'}->Value = 'Value from PHP';

// Load from the modified JSON object
$report->load($reportJson);
$report->render();
```

---

## 14. How do I change export settings on the server-side?

Use the `onBeginExportReport` viewer event to intercept exports and modify settings before they run:

```php
use Stimulsoft\Events\StiExportEventArgs;
use Stimulsoft\Export\Enums\StiExportFormat;
use Stimulsoft\Export\StiPdfExportSettings;

$viewer->onBeginExportReport = function (StiExportEventArgs $args) {

    // Override the exported file name
    $args->fileName = "MyReport.$args->fileExtension";

    // Change PDF-specific settings
    if ($args->format == StiExportFormat::Pdf) {
        /** @var StiPdfExportSettings $settings */
        $settings = $args->settings;
        $settings->creatorString  = 'My Company';
        $settings->keywordsString = 'report export php';
        $settings->embeddedFonts  = true;
    }
};
```

---

## 15. How do I customize the viewer?

Set options on `$viewer->options` before calling `process()`:

```php
use Stimulsoft\Viewer\Enums\StiToolbarDisplayMode;
use Stimulsoft\Viewer\StiViewer;

$viewer = new StiViewer();
$viewer->javascript->relativePath = './';

// Appearance
$viewer->options->appearance->fullScreenMode     = true;
$viewer->options->appearance->backgroundColor    = '#f5f5f5';

// Toolbar
$viewer->options->toolbar->displayMode           = StiToolbarDisplayMode::Separated;
$viewer->options->toolbar->showPrintButton       = true;
$viewer->options->toolbar->showFullScreenButton  = false;
$viewer->options->toolbar->showSendEmailButton   = true;

// Email defaults (shown in the Send Email dialog)
$viewer->options->email->defaultEmailAddress = 'recipient@example.com';
$viewer->options->email->defaultEmailSubject = 'Monthly Report';
```

**Custom toolbar button:**
```php
use Stimulsoft\Viewer\StiToolButton;

$button = new StiToolButton();
$button->caption    = 'My Action';
$button->icon       = 'path/to/icon.png';
$button->onClick    = 'myButtonClick'; // JavaScript function name

$viewer->options->toolbar->buttons[] = $button;
```

---

## 16. How do I change the viewer or designer theme?

```php
use Stimulsoft\Viewer\Enums\StiViewerTheme;

$viewer->options->appearance->theme = StiViewerTheme::Office2022BlackGreen;
```

**Available viewer themes:**
- `Office2022WhiteBlue` *(default)*
- `Office2022BlackGreen`
- `Office2013WhiteGreen`
- `SimpleGray`

**Localize the designer:**
```php
use Stimulsoft\Designer\StiDesigner;

$designer = new StiDesigner();
$designer->options->localization = 'localization/de.xml';
```

---

## 17. How do I send a report by email?

Enable the Send Email button and handle the `onEmailReport` event to configure SMTP on the server:

```php
use Stimulsoft\Events\StiEmailEventArgs;
use Stimulsoft\StiResult;

$viewer->options->toolbar->showSendEmailButton        = true;
$viewer->options->email->defaultEmailAddress          = 'recipient@example.com';
$viewer->options->email->defaultEmailSubject          = 'Your Report';
$viewer->options->email->defaultEmailMessage          = 'Please find the report attached.';

$viewer->onEmailReport = function (StiEmailEventArgs $args) {
    // SMTP credentials stay on the server — never sent to the client
    $args->settings->from     = 'noreply@mycompany.com';
    $args->settings->host     = 'smtp.mycompany.com';
    $args->settings->port     = 587;
    $args->settings->secure   = 'tls';
    $args->settings->login    = 'smtp_user';
    $args->settings->password = 'smtp_password';

    // Optional CC / BCC
    // $args->settings->cc[]  = 'manager@mycompany.com';

    return StiResult::getSuccess('The email was sent successfully.');
};
```

---

## 18. How do I render and export a report on the server-side (Node.js)?

Server-side mode uses Node.js to render reports entirely on the PHP server — no browser required. Useful for scheduled reports, batch exports, or APIs.

**Prerequisites:** Node.js must be installed on the server. See `Working with Report on the Server-Side/Configuring and Installing NodeJs.php` for setup instructions.

**Render on the server:**
```php
use Stimulsoft\Report\Enums\StiEngineType;
use Stimulsoft\Report\StiReport;

chdir('../'); // set working directory to project root

$report = new StiReport();
$report->engine = StiEngineType::ServerNodeJS; // switch to server-side engine

$report->process();
$report->loadFile('reports/SimpleList.mrt', true); // true = load as compressed string for Node.js

$result = $report->render();

if ($result) {
    $documentString = $report->saveDocument();          // get as string
    // $report->saveDocument('reports/output.mdc');     // or save to file
    echo "Rendered successfully. Size: " . strlen($documentString) . " bytes";
} else {
    echo "Error: " . $report->nodejs->error;
}
```

**Export on the server:**
```php
use Stimulsoft\Export\Enums\StiExportFormat;

$report->engine = StiEngineType::ServerNodeJS;
$report->process();
$report->loadFile('reports/SimpleList.mrt');

if ($report->render()) {
    $pdfData = $report->exportDocument(StiExportFormat::Pdf);
    // $report->exportDocument(StiExportFormat::Pdf, null, false, 'output/report.pdf'); // save to file

    if ($pdfData !== false) {
        header('Content-Type: application/pdf');
        header('Content-Disposition: attachment; filename="report.pdf"');
        echo $pdfData;
    }
}
```

**Register data before server-side rendering:**
```php
// Pass JavaScript code as a string — it runs inside Node.js
$report->onBeforeRender = '
    let dataSet = new Stimulsoft.System.Data.DataSet("Demo");
    dataSet.readXmlSchemaFile("data/Demo.xsd");
    dataSet.readXmlFile("data/Demo.xml");
    args.report.dictionary.databases.clear();
    args.report.regData("Demo", "Demo", dataSet);
';
```

---

## 19. Which databases are supported?

Stimulsoft Reports.PHP supports connections to the following database types:

| Database | Notes |
|----------|-------|
| MySQL / MariaDB | Native PHP driver |
| Microsoft SQL Server | Via PDO or SQLSRV extension |
| PostgreSQL | Native PHP driver |
| Oracle | Via OCI8 or PDO_OCI |
| Firebird / InterBase | Via PDO_Firebird |
| ODBC | Via PHP ODBC extension |
| SQLite | Via PDO_SQLite |
| XML | File or URL |
| JSON | File or URL |

Connection strings are managed via the `onBeginProcessData` event (see [question 11](#11-how-do-i-use-sql-data-sources)) — connection credentials never leave the server.

---

*For more details see the [official documentation](https://www.stimulsoft.com/en/documentation/online/programming-manual/reports_and_dashboards_for_php.htm) and [code samples on GitHub](https://github.com/stimulsoft/Samples-Reports.PHP).*
