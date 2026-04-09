# Stimulsoft Reports.JAVA — FAQ

## Table of Contents

1. [How do I install Stimulsoft Reports.JAVA?](#1-how-do-i-install-stimulsoft-reportsjava)
2. [How do I activate the license?](#2-how-do-i-activate-the-license)
3. [How do I load and render a report from a template?](#3-how-do-i-load-and-render-a-report-from-a-template)
4. [How do I connect an XML data source to the report?](#4-how-do-i-connect-an-xml-data-source-to-the-report)
5. [How do I connect a MySQL (JDBC) database to the report?](#5-how-do-i-connect-a-mysql-jdbc-database-to-the-report)
6. [How do I create a report at runtime in code?](#6-how-do-i-create-a-report-at-runtime-in-code)
7. [How do I create a report with master-detail relations at runtime?](#7-how-do-i-create-a-report-with-master-detail-relations-at-runtime)
8. [How do I export a report from code?](#8-how-do-i-export-a-report-from-code)
9. [How do I export a report using the export settings dialog?](#9-how-do-i-export-a-report-using-the-export-settings-dialog)
10. [How do I print a report from code?](#10-how-do-i-print-a-report-from-code)
11. [How do I use render events?](#11-how-do-i-use-render-events)
12. [How do I copy components between reports?](#12-how-do-i-copy-components-between-reports)
13. [How do I deploy the Web Viewer using JSP (Java EE)?](#13-how-do-i-deploy-the-web-viewer-using-jsp-java-ee)
14. [How do I deploy the Web Viewer using JSP (Jakarta EE)?](#14-how-do-i-deploy-the-web-viewer-using-jsp-jakarta-ee)
15. [How do I customize the Web Viewer options?](#15-how-do-i-customize-the-web-viewer-options)
16. [How do I deploy the Web Designer using JSP?](#16-how-do-i-deploy-the-web-designer-using-jsp)
17. [How do I integrate the Viewer and Designer with Spring Boot?](#17-how-do-i-integrate-the-viewer-and-designer-with-spring-boot)
18. [How do I integrate with JavaServer Faces (JSF) / Jakarta Faces?](#18-how-do-i-integrate-with-javaserver-faces-jsf--jakarta-faces)
19. [How do I use the Angular Viewer with a Java backend?](#19-how-do-i-use-the-angular-viewer-with-a-java-backend)
20. [How do I merge several reports into one?](#20-how-do-i-merge-several-reports-into-one)
21. [How do I register custom functions for the designer?](#21-how-do-i-register-custom-functions-for-the-designer)

---

## 1. How do I install Stimulsoft Reports.JAVA?

Stimulsoft Reports.JAVA is distributed as a Maven artifact. Add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.stimulsoft</groupId>
    <artifactId>stimulsoft-reports-libs</artifactId>
    <version>2026.1.7</version>
</dependency>
```

For Spring Boot projects you may also need to exclude conflicting dependencies:

```xml
<dependency>
    <groupId>com.stimulsoft</groupId>
    <artifactId>stimulsoft-reports-libs</artifactId>
    <version>2026.1.7</version>
    <exclusions>
        <exclusion>
            <groupId>xml-apis</groupId>
            <artifactId>xml-apis</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.apache.xmlgraphics</groupId>
            <artifactId>batik-all</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**System requirements:** Java SE 1.8 or higher. For web projects: any Servlet container (Tomcat, Jetty, etc.). Spring Boot 3.x requires Java 17+.

**Supported platforms:** Java SE (Swing desktop), Jakarta EE (Servlet/JSP), Spring Boot, JavaServer Faces (JSF), Jakarta Faces, Angular (with Java backend).

---

## 2. How do I activate the license?

Set the license key in your code before using any Stimulsoft components:

```java
com.stimulsoft.base.licenses.StiLicense.setKey("your-license-key-here");
```

Place this call at application startup — for example, in your `main()` method for desktop applications, or in the JSP/Servlet initialization for web applications.

---

## 3. How do I load and render a report from a template?

Use `StiSerializeManager` to load a report template (`.mrt` file) and then call `render()`:

```java
import com.stimulsoft.report.StiReport;
import com.stimulsoft.report.StiSerializeManager;
import java.io.File;

StiReport report = StiSerializeManager.deserializeReport(new File("Reports/SimpleList.mrt"));
report.render();
```

You can also load from an `InputStream`:

```java
import java.io.FileInputStream;

StiReport report = StiSerializeManager.deserializeReport(new FileInputStream("Reports/SimpleList.mrt"));
```

To load an already compiled/rendered report (`.mdc` file):

```java
StiReport report = StiSerializeManager.deserializeDocument(new File("Reports/TwoSimpleLists.mdc")).getReport();
```

You can control the rendering mode using `StiCalculationMode`:

```java
import com.stimulsoft.report.enums.StiCalculationMode;

report.setCalculationMode(StiCalculationMode.Interpretation);
report.Render(false);
```

---

## 4. How do I connect an XML data source to the report?

Create an `StiXmlDatabase` with the database name, path to XSD schema, and path to XML data file, then add it to the report dictionary:

```java
import com.stimulsoft.report.dictionary.databases.StiXmlDatabase;

StiXmlDatabase xmlDatabase = new StiXmlDatabase("Demo", "Data/Demo.xsd", "Data/Demo.xml");
report.getDictionary().getDatabases().add(xmlDatabase);
```

In a web application, use `ServletContext.getRealPath()` to resolve file paths:

```java
String xsdPath = request.getSession().getServletContext().getRealPath("/data/Demo.xsd");
String xmlPath = request.getSession().getServletContext().getRealPath("/data/Demo.xml");
StiXmlDatabase xmlDatabase = new StiXmlDatabase("Demo", xsdPath, xmlPath);
report.getDictionary().getDatabases().add(xmlDatabase);
```

---

## 5. How do I connect a MySQL (JDBC) database to the report?

Use `StiMySqlDatabase` with a JDBC connection string:

```java
import com.stimulsoft.report.dictionary.databases.StiMySqlDatabase;
import com.stimulsoft.report.dictionary.dataSources.StiMySqlSource;
import com.stimulsoft.report.dictionary.StiDataColumnsCollection;

StiMySqlDatabase db = new StiMySqlDatabase(
    "test", "test",
    "url=jdbc:mysql://localhost:3306/sakila;user=root;password=mypassword;database=sakila"
);
report.getDictionary().getDatabases().add(db);

StiMySqlSource source = new StiMySqlSource(
    "test.actors", "actors", "actors",
    "select * from actor"
);
source.setDictionary(report.getDictionary());
source.setColumns(new StiDataColumnsCollection());
report.getDictionary().getDataSources().add(source);
```

The connection string format is: `url=jdbc:mysql://host:port/database;user=username;password=password;database=dbname`.

To retrieve column metadata automatically, use `StiMySqlAdapter` and `StiDataColumnsUtil`:

```java
import com.stimulsoft.report.dictionary.adapters.StiMySqlAdapter;
import com.stimulsoft.report.utils.data.StiDataColumnsUtil;
import com.stimulsoft.report.utils.data.StiTableFieldsRequest;
import com.stimulsoft.report.utils.data.StiSqlField;
import com.stimulsoft.report.dictionary.StiDataColumn;

StiMySqlAdapter adapter = new StiMySqlAdapter(db.getConnectionString());
Class.forName(adapter.getDriverName());
Connection con = com.stimulsoft.webdesigner.helper.StiDictionaryHelper.getConnection(
    adapter.getJdbcParameters()
);
StiTableFieldsRequest fieldRequest = StiDataColumnsUtil.getFields(con, source.getQuery(), source);
for (StiSqlField field : fieldRequest.getColumns()) {
    source.getColumns().add(
        new StiDataColumn(field.getName(), field.getName(), field.getSystemType())
    );
}
```

For other JDBC databases, use `StiJDBCDatabase` / `StiSqlDatabase` with the appropriate JDBC driver and connection string.

---

## 6. How do I create a report at runtime in code?

You can build a report entirely in Java code by creating pages, bands, and components programmatically:

```java
import com.stimulsoft.report.StiReport;
import com.stimulsoft.report.StiNameCreation;
import com.stimulsoft.report.components.StiPage;
import com.stimulsoft.report.components.bands.StiHeaderBand;
import com.stimulsoft.report.components.bands.StiDataBand;
import com.stimulsoft.report.components.simplecomponents.StiText;
import com.stimulsoft.report.components.simplecomponents.StiImage;
import com.stimulsoft.report.dictionary.StiDictionary;
import com.stimulsoft.report.dictionary.databases.StiXmlDatabase;
import com.stimulsoft.base.system.geometry.StiRectangle;
import com.stimulsoft.base.system.StiFont;
import com.stimulsoft.base.system.StiFontStyle;
import com.stimulsoft.base.drawing.enums.StiTextHorAlignment;
import com.stimulsoft.base.drawing.enums.StiBorderSides;
import com.stimulsoft.base.drawing.StiSolidBrush;
import com.stimulsoft.base.drawing.StiColorEnum;

StiReport report = new StiReport();

// Create page
StiPage page = new StiPage(report);
report.getPages().add(page);
page.setName(StiNameCreation.createName(report, StiNameCreation.generateName(page)));

// Set up dictionary and data
report.setDictionary(new StiDictionary(report));
StiXmlDatabase xmlDatabase = new StiXmlDatabase("Demo", "Data/Demo.xsd", "Data/Demo.xml");
report.getDictionary().getDatabases().add(xmlDatabase);

// Create header band
StiHeaderBand headerBand = new StiHeaderBand();
headerBand.setHeight(0.5);
headerBand.setName("HeaderBand");
page.getComponents().add(headerBand);

// Create text on header
StiText headerText = new StiText(new StiRectangle(0, 0, 3, 0.5));
headerText.setTextInternal("Column Name");
headerText.setHorAlignment(StiTextHorAlignment.Center);
headerText.setName("HeaderText1");
headerText.setBrush(new StiSolidBrush(StiColorEnum.Orange.color()));
headerText.getBorder().setSide(StiBorderSides.All);
headerBand.getComponents().add(headerText);

// Create data band
StiDataBand dataBand = new StiDataBand();
dataBand.setDataSourceName("Categories");
dataBand.setHeight(0.5);
dataBand.setName("DataBand");
page.getComponents().add(dataBand);

// Create data text with expression
StiText dataText = new StiText(new StiRectangle(0, 0, 3, 0.5));
dataText.setText("{Categories.CategoryName}");
dataText.setName("DataText1");
dataText.getBorder().setSide(StiBorderSides.All);
dataBand.getComponents().add(dataText);

// For image columns, use StiImage:
// StiImage dataImage = new StiImage(new StiRectangle(pos, 0, width, 0.5));
// dataImage.setDataColumn("Categories.Picture");
// dataBand.getComponents().add(dataImage);

report.Render();
```

To display the report in a Swing desktop viewer:

```java
import com.stimulsoft.viewer.StiViewerFx;
import com.stimulsoft.viewer.events.StiViewCommonEvent;
import com.stimulsoft.report.saveLoad.StiDocument;

StiViewerFx viewerPanel = new StiViewerFx(frame);
panel.add(viewerPanel);
// ... after rendering:
viewerPanel.getStiViewModel().getEventDispatcher()
    .dispatchStiEvent(new StiViewCommonEvent(
        StiViewCommonEvent.DOCUMENT_FILE_LOADED,
        new StiDocument(report), null));
```

---

## 7. How do I create a report with master-detail relations at runtime?

You can create master-detail relationships by implementing custom `StiDatabase` classes and using `StiDataRelation`:

```java
import com.stimulsoft.report.dictionary.StiDataRelation;
import com.stimulsoft.report.dictionary.dataSources.StiDataTableSource;

// Add parent and child databases (custom StiDatabase subclasses)
report.getDictionary().getDatabases().add(new ParentDatabase());
report.getDictionary().getDatabases().add(new ChildDatabase());

// Create data sources
StiDataTableSource parentSource = new StiDataTableSource(
    "ParentDB.parent", "parent", "parent");
parentSource.setDictionary(report.getDictionary());
// Add columns...
report.getDictionary().getDataSources().add(parentSource);

StiDataTableSource childSource = new StiDataTableSource(
    "ChildDB.child", "child", "child");
childSource.setDictionary(report.getDictionary());
// Add columns...
report.getDictionary().getDataSources().add(childSource);

// Create relation
StiDataRelation relation = new StiDataRelation(
    "parentRelation",    // name
    parentSource,        // parent source
    childSource,         // child source
    new String[] {"id"}, // parent columns
    new String[] {"parentid"} // child columns
);
report.getDictionary().getRelations().add(relation);
```

A custom `StiDatabase` subclass must override the `connect()` method to populate data:

```java
import com.stimulsoft.report.dictionary.databases.StiDatabase;
import com.stimulsoft.base.system.StiDataTable;
import com.stimulsoft.base.system.StiDataRow;

public class ParentDatabase extends StiDatabase {

    public ParentDatabase() {
        super("ParentDB", "ParentDB");
    }

    @Override
    public void connect(StiDataStoreSource dataStoreSource, Boolean fillTable) {
        StiDataTable dataTable = new StiDataTable("parent");
        dataTable.getColumns().add(new DataColumn("id", Integer.class));
        dataTable.getColumns().add(new DataColumn("name", String.class));

        StiDataRow row = dataTable.newRow();
        row.setValueByIndex(0, 1);
        row.setValueByIndex(1, "Parent Record 1");
        dataTable.getRows().add(row);

        dataStoreSource.setDataTable(dataTable);
    }
}
```

---

## 8. How do I export a report from code?

After rendering the report, use `StiExportManager` to export to any supported format:

```java
import com.stimulsoft.report.StiExportManager;
import java.io.FileOutputStream;

StiReport report = StiSerializeManager.deserializeReport(new File("Reports/SimpleList.mrt"));
report.setCalculationMode(StiCalculationMode.Interpretation);
report.Render(false);

FileOutputStream outputStream = new FileOutputStream("output.pdf");
StiExportManager.exportPdf(report, outputStream);
outputStream.close();
```

**Supported export formats and methods:**

| Format | Method |
|--------|--------|
| PDF | `StiExportManager.exportPdf(report, outputStream)` |
| XPS | `StiExportManager.exportXps(report, outputStream)` |
| HTML | `StiExportManager.exportHtml(report, outputStream)` |
| Plain Text | `StiExportManager.exportText(report, outputStream)` |
| RTF | `StiExportManager.exportRtf(report, outputStream)` |
| Microsoft Word | `StiExportManager.exportWord(report, outputStream)` |
| Microsoft Excel (xlsx) | `StiExportManager.exportExcel(report, outputStream)` |
| Excel XML | `StiExportManager.exportExcelXml(report, outputStream)` |
| Excel 97-2003 (xls) | `StiExportManager.exportExcelBiff(report, outputStream)` |
| CSV | `StiExportManager.exportCsv(report, outputStream)` |
| XML | `StiExportManager.exportXml(report, outputStream)` |
| SYLK | `StiExportManager.exportSylk(report, outputStream)` |
| BMP | `StiExportManager.exportImageBmp(report, outputStream)` |
| JPEG | `StiExportManager.exportImageJpeg(report, outputStream)` |
| PCX | `StiExportManager.exportImagePcx(report, outputStream)` |
| PNG | `StiExportManager.exportImagePng(report, outputStream)` |
| SVG | `StiExportManager.exportImageSvg(report, outputStream)` |
| SVGZ | `StiExportManager.exportImageSvgz(report, outputStream)` |

---

## 9. How do I export a report using the export settings dialog?

For desktop (Swing) applications, you can show a format-specific export dialog that lets the user configure export settings before exporting:

```java
import com.stimulsoft.report.StiExportManager;
import com.stimulsoft.report.enums.StiExportFormat;
import com.stimulsoft.viewer.controls.dialogs.StiFileSaveDialog;
import com.stimulsoft.viewer.controls.dialogs.StiPdfExportDialog;
import javax.swing.JFileChooser;
import java.io.FileOutputStream;

StiReport report = getReport(); // your rendered report

// Option 1: Use file save dialog with automatic format detection
StiFileSaveDialog fileChooser = new StiFileSaveDialog(
    StiExportFormat.Pdf, report, report.getReportAlias());
int result = fileChooser.showSaveDialog(parentComponent);
if (result == JFileChooser.APPROVE_OPTION) {
    FileOutputStream outputStream = new FileOutputStream(fileChooser.getFile());
    StiExportManager.exportPdf(report, outputStream);
    outputStream.close();
}

// Option 2: Use export settings dialog for fine-tuning
Object settings = StiPdfExportDialog.showDialog(parentFrame, report);
if (settings != null) {
    StiFileSaveDialog saveDialog = new StiFileSaveDialog(
        StiExportFormat.Pdf, report, report.getReportAlias());
    if (saveDialog.showSaveDialog(parentComponent) == JFileChooser.APPROVE_OPTION) {
        FileOutputStream outputStream = new FileOutputStream(saveDialog.getFile());
        StiExportManager.exportPdf(report, (StiPdfExportSettings) settings, outputStream);
        outputStream.close();
    }
}
```

Similar dialogs exist for other formats: `StiExcelExportDialog`, `StiHtmlExportDialog`, `StiWordExportDialog`, `StiRtfExportDialog`, `StiTextExportDialog`, `StiCsvExportDialog`, `StiImageExportDialog`, etc.

---

## 10. How do I print a report from code?

Use `StiPrintHelper` to print a rendered report. You can print with or without the print dialog:

```java
import com.stimulsoft.report.print.StiPrintHelper;
import java.awt.print.PrinterJob;

StiReport report = getReport(); // your rendered report

// Print with print dialog
PrinterJob printerJob = StiPrintHelper.preparePrinterJob(report.getRenderedPages());
StiPrintHelper.printJob(printerJob, report, true); // true = show dialog

// Print without print dialog (to a specific printer)
import javax.print.PrintService;
import javax.print.PrintServiceLookup;
import javax.print.attribute.AttributeSet;
import javax.print.attribute.HashAttributeSet;
import javax.print.attribute.standard.PrinterName;

PrinterJob printerJob = StiPrintHelper.preparePrinterJob(report.getRenderedPages());
AttributeSet attrSet = new HashAttributeSet();
PrintService printService = selectedPrinter; // e.g. from PrintServiceLookup
attrSet.add(new PrinterName(printService.getName(), null));
PrintService[] services = PrintServiceLookup.lookupPrintServices(null, attrSet);
printerJob.setPrintService(services[0]);
StiPrintHelper.printJob(printerJob, report); // no dialog
```

To get a list of available printers:

```java
PrintService[] printers = PrintServiceLookup.lookupPrintServices(null, null);
```

---

## 11. How do I use render events?

You can subscribe to report rendering events to track the rendering process. Events are available at the report level and at the page level:

```java
import com.stimulsoft.base.system.StiEventHandlerListener;
import com.stimulsoft.base.system.StiEventObject;

// Report-level events
report.handlerBeginRender.add(new StiEventHandlerListener() {
    public void invoke(StiEventObject myEvent) {
        System.out.println("Report rendering started");
    }
});

report.handlerRendering.add(new StiEventHandlerListener() {
    public void invoke(StiEventObject myEvent) {
        System.out.println("Report rendering in progress...");
    }
});

report.handlerEndRender.add(new StiEventHandlerListener() {
    public void invoke(StiEventObject myEvent) {
        System.out.println("Report rendering finished");
    }
});

// Page-level events
report.getPages().get(0).handlerBeginRender.add(new StiEventHandlerListener() {
    public void invoke(StiEventObject myEvent) {
        System.out.println("Page rendering started");
    }
});

report.getPages().get(0).handlerEndRender.add(new StiEventHandlerListener() {
    public void invoke(StiEventObject myEvent) {
        System.out.println("Page rendering finished");
    }
});
```

You can add multiple listeners to the same event. For background rendering in Swing applications, use `StiSimpleWorker`:

```java
import com.stimulsoft.base.worker.StiSimpleWorker;

StiSimpleWorker renderWorker = new StiSimpleWorker() {
    @Override
    protected void doInBackground() throws Throwable {
        report.render();
    }
};
renderWorker.execute();
```

---

## 12. How do I copy components between reports?

You can copy components (bands, text fields, etc.) from one report to another using serialization:

```java
import com.stimulsoft.report.StiSerializeManager;
import com.stimulsoft.base.serializing.StiSerializerControler;
import com.stimulsoft.base.serializing.StiDeserializerControler;
import com.stimulsoft.report.StiNameCreation;
import com.stimulsoft.report.components.StiComponent;
import com.stimulsoft.report.components.bands.StiHeaderBand;

StiReport originalReport = StiSerializeManager.deserializeReport(originalFile);
StiReport customerReport = StiSerializeManager.deserializeReport(customerFile);

// Get the component to copy from the source report
StiHeaderBand customerHeader = (StiHeaderBand) customerReport.getComponents().get("HeaderBand1");

// Serialize component to string
String headerStr = StiSerializerControler.serializedObjectAsString(customerHeader);

// Deserialize into a new component
StiHeaderBand newHeader = new StiHeaderBand();
StiDeserializerControler.deserializeFromString(headerStr, newHeader);

// Remove old component and add new one
StiHeaderBand originalHeader = (StiHeaderBand) originalReport.getComponents().get("HeaderBand1");
int index = originalHeader.getPage().getComponents().indexOf(originalHeader);
originalHeader.getPage().getComponents().remove(index);

// Add the copied component
newHeader.setPage(originalReport.getPages().get(0));
originalReport.getPages().get(0).getComponents().add(index, newHeader);
newHeader.setName(StiNameCreation.createName(originalReport, "HeaderBand"));

// Update child components
for (StiComponent component : newHeader.getComponents()) {
    component.setPage(originalReport.getPages().get(0));
    component.setParent(newHeader);
    component.setName(StiNameCreation.createName(
        originalReport, component.getName().replaceAll("\\d*", "")));
}
```

---

## 13. How do I deploy the Web Viewer using JSP (Java EE)?

To deploy the web viewer in a Java EE Servlet container (e.g. Tomcat), you need to configure the required servlets and use the JSP tag library.

**Step 1.** Add the Maven dependency to your `pom.xml` (see [Question 1](#1-how-do-i-install-stimulsoft-reportsjava)).

**Step 2.** Configure servlets in `WEB-INF/web.xml`:

```xml
<web-app>
    <display-name>stimulsoft_webviewer</display-name>
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

    <servlet>
        <servlet-name>StimulsoftResource</servlet-name>
        <servlet-class>com.stimulsoft.web.servlet.StiWebResourceServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>StimulsoftResource</servlet-name>
        <url-pattern>/stimulsoft_web_resource/*</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>StimulsoftAction</servlet-name>
        <servlet-class>com.stimulsoft.webviewer.servlet.StiWebViewerActionServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>StimulsoftAction</servlet-name>
        <url-pattern>/stimulsoft_webviewer_action</url-pattern>
    </servlet-mapping>
</web-app>
```

**Step 3.** Create the `index.jsp` page:

```jsp
<%@page import="com.stimulsoft.webviewer.StiWebViewerOptions"%>
<%@page import="com.stimulsoft.webviewer.StiWebViewer"%>
<%@page import="java.io.File"%>
<%@page import="com.stimulsoft.report.StiSerializeManager"%>
<%@page import="com.stimulsoft.report.StiReport"%>
<%@ taglib uri="http://stimulsoft.com/webviewer" prefix="stiwebviewer"%>
<html>
<head>
    <title>Stimulsoft Reports for Java</title>
</head>
<body>
    <%
        String reportPath = request.getSession().getServletContext()
            .getRealPath("/reports/SimpleList.mrt");
        StiReport report = StiSerializeManager.deserializeReport(new File(reportPath));
        report.render();
        StiWebViewerOptions options = new StiWebViewerOptions();
        pageContext.setAttribute("report", report);
        pageContext.setAttribute("options", options);
    %>
    <stiwebviewer:webviewer report="${report}" options="${options}" />
</body>
</html>
```

Place your `.mrt` report templates into the `reports/` folder of your web application, and data files into the `data/` folder.

---

## 14. How do I deploy the Web Viewer using JSP (Jakarta EE)?

For Jakarta EE (Java 11+ / Tomcat 10+), the setup is similar to Java EE but uses the Jakarta namespace. The JSP tag uses `stiwebviewer` instead of `webviewer`:

```jsp
<%@page import="com.stimulsoft.webviewer.StiWebViewerOptions"%>
<%@page import="com.stimulsoft.webviewer.StiWebViewer"%>
<%@page import="java.io.File"%>
<%@page import="com.stimulsoft.report.StiSerializeManager"%>
<%@page import="com.stimulsoft.report.StiReport"%>
<%@ taglib uri="http://stimulsoft.com/webviewer" prefix="stiwebviewer"%>
<html>
<head>
    <title>Stimulsoft Reports for Java</title>
</head>
<body>
    <%
        String reportPath = request.getSession().getServletContext()
            .getRealPath("/reports/Dashboards.mrt");
        StiReport report = StiSerializeManager.deserializeReport(new File(reportPath));
        report.render();
        StiWebViewerOptions options = new StiWebViewerOptions();
        pageContext.setAttribute("report", report);
        pageContext.setAttribute("options", options);
    %>
    <stiwebviewer:stiwebviewer report="${report}" options="${options}" />
</body>
</html>
```

Note the tag difference: Java EE uses `<stiwebviewer:webviewer>`, while Jakarta EE uses `<stiwebviewer:stiwebviewer>`.

The servlet classes for Jakarta EE also have the `Jk` suffix: `StiWebViewerActionServletJk`, `StiWebResourceServletJk`, `StiWebDesignerActionServletJk`.

---

## 15. How do I customize the Web Viewer options?

The `StiWebViewerOptions` class provides many properties to customize the viewer appearance and behavior:

```java
StiWebViewerOptions options = new StiWebViewerOptions();

// Localization
options.setLocalization(servletContext.getRealPath("/localization/de.xml"));

// Theme
import com.stimulsoft.webviewer.enums.StiWebViewerTheme;
options.setTheme(StiWebViewerTheme.Office2013DarkGrayBlue);

// Size
options.setWidth("100%");
options.setHeight("600px");

// Background color
import com.stimulsoft.base.drawing.StiColor;
options.setBackColor(new StiColor(240, 240, 240));

// Right-to-left support
options.setRightToLeft(true);

// Scrollbars
options.setScrollbarsMode(true);

// Menu
import com.stimulsoft.webviewer.enums.StiShowMenuMode;
options.setMenuAnimation(true);
options.setMenuShowMode(StiShowMenuMode.Click); // or StiShowMenuMode.Hover

// Pages view mode
import com.stimulsoft.web.enums.StiWebViewMode;
options.setPagesViewMode(StiWebViewMode.WholeReport); // or StiWebViewMode.OnePage

// Page alignment
import com.stimulsoft.web.enums.StiContentAlignment;
options.setPageAlignment(StiContentAlignment.Center);

// Toolbar
options.setToolbarVisible(true);
options.setShowButtonPrint(true);
options.setShowButtonSave(true);
options.setShowButtonZoom(true);

// Export visibility
options.setShowExportToDocument(true);
options.setShowExportToPdf(true);

// Print destination
import com.stimulsoft.webviewer.enums.StiPrintDestination;
options.setMenuPrintDestination(StiPrintDestination.Direct);

// Refresh timeout (seconds)
options.setRefreshTimeout(3);
```

For Spring Boot, you can also use the toolbar sub-options:

```java
options.getToolbar().setZoom(-1); // auto-zoom
options.getToolbar().setViewMode(StiWebViewMode.Continuous);
```

---

## 16. How do I deploy the Web Designer using JSP?

The web designer requires implementing the `StiWebDesigerHandler` interface (or `StiWebDesigerHandlerJk` for Jakarta EE) to handle report loading, opening, creating, and saving.

**Step 1.** Register the designer servlet in `web.xml` (in addition to the resource servlet):

```xml
<servlet>
    <servlet-name>StimulsoftDesignerAction</servlet-name>
    <servlet-class>com.stimulsoft.webdesigner.servlet.StiWebDesignerActionServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>StimulsoftDesignerAction</servlet-name>
    <url-pattern>/stimulsoft_webdesigner_action/*</url-pattern>
</servlet-mapping>
```

**Step 2.** Create the JSP page with the designer handler:

```jsp
<%@page import="com.stimulsoft.webdesigner.StiWebDesignerOptions"%>
<%@page import="com.stimulsoft.webdesigner.StiWebDesigerHandler"%>
<%@page import="com.stimulsoft.webdesigner.enums.StiWebDesignerTheme"%>
<%@page import="com.stimulsoft.web.classes.StiRequestParams"%>
<%@page import="com.stimulsoft.report.StiSerializeManager"%>
<%@page import="com.stimulsoft.report.StiReport"%>
<%@ taglib uri="http://stimulsoft.com/webdesigner" prefix="stiwebdesigner"%>

<%
    final String reportPath = request.getSession().getServletContext()
        .getRealPath("/reports/Master-Detail.mrt");
    final String savePath = request.getSession().getServletContext()
        .getRealPath("/save/");

    StiWebDesignerOptions options = new StiWebDesignerOptions();
    options.setTheme(StiWebDesignerTheme.Office2013DarkGrayBlue);

    StiWebDesigerHandler handler = new StiWebDesigerHandler() {
        // Called when loading the designer — return the report to edit
        public StiReport getEditedReport(HttpServletRequest request) {
            try {
                return StiSerializeManager.deserializeReport(
                    new java.io.FileInputStream(reportPath));
            } catch (Exception e) {
                e.printStackTrace();
            }
            return null;
        }

        // Called when opening a report template
        public void onOpenReportTemplate(StiReport report,
                HttpServletRequest request) {
            // Add data sources to the opened report if needed
        }

        // Called when creating a new report template
        public void onNewReportTemplate(StiReport report,
                HttpServletRequest request) {
            // Set up data sources for the new report
        }

        // Called when saving — implement your save logic
        public void onSaveReportTemplate(StiReport report,
                StiRequestParams requestParams, HttpServletRequest request) {
            try {
                java.io.FileOutputStream fos = new java.io.FileOutputStream(
                    savePath + requestParams.designer.fileName);
                if (requestParams.designer.password != null) {
                    StiSerializeManager.serializeReport(report, fos,
                        requestParams.designer.password);
                } else {
                    StiSerializeManager.serializeReport(report, fos, true);
                }
                fos.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    };
    pageContext.setAttribute("handler", handler);
    pageContext.setAttribute("options", options);
%>

<stiwebdesigner:webdesigner handler="${handler}" options="${options}" />
```

For Jakarta EE, use `StiWebDesigerHandlerJk` and `<stiwebdesigner:stiwebdesigner>` tag.

---

## 17. How do I integrate the Viewer and Designer with Spring Boot?

**Step 1.** Add the Maven dependencies to `pom.xml`:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.3</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
    </dependency>
    <dependency>
        <groupId>com.stimulsoft</groupId>
        <artifactId>stimulsoft-reports-libs</artifactId>
        <version>2026.1.7</version>
        <exclusions>
            <exclusion>
                <groupId>xml-apis</groupId>
                <artifactId>xml-apis</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.apache.xmlgraphics</groupId>
                <artifactId>batik-all</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

**Step 2.** Register Stimulsoft servlets in a configuration class:

```java
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.stimulsoft.web.servlet.StiWebResourceServletJk;
import com.stimulsoft.webdesigner.servlet.StiWebDesignerActionServletJk;
import com.stimulsoft.webviewer.servlet.StiWebViewerActionServletJk;
import jakarta.servlet.http.HttpServlet;

@Configuration
public class StimulsoftConfig {

    @Bean
    public ServletRegistrationBean<HttpServlet> webviewerActionBean() {
        ServletRegistrationBean<HttpServlet> bean = new ServletRegistrationBean<>();
        bean.setServlet(new StiWebViewerActionServletJk());
        bean.addUrlMappings("/stimulsoft_webviewer_action");
        bean.setLoadOnStartup(1);
        return bean;
    }

    @Bean
    public ServletRegistrationBean<HttpServlet> webResourceBean() {
        ServletRegistrationBean<HttpServlet> bean = new ServletRegistrationBean<>();
        bean.setServlet(new StiWebResourceServletJk());
        bean.addUrlMappings("/stimulsoft_web_resource/*");
        bean.setLoadOnStartup(1);
        return bean;
    }

    @Bean
    public ServletRegistrationBean<HttpServlet> webdesignerActionBean() {
        ServletRegistrationBean<HttpServlet> bean = new ServletRegistrationBean<>();
        bean.setServlet(new StiWebDesignerActionServletJk());
        bean.addUrlMappings("/stimulsoft_webdesigner_action/*");
        bean.setLoadOnStartup(1);
        return bean;
    }
}
```

**Step 3.** Create a Spring controller:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import com.stimulsoft.report.StiReport;
import com.stimulsoft.report.StiSerializeManager;
import com.stimulsoft.web.classes.StiRequestParams;
import com.stimulsoft.webdesigner.StiWebDesigerHandlerJk;
import com.stimulsoft.webdesigner.StiWebDesignerOptions;
import com.stimulsoft.webviewer.StiWebViewerOptions;
import jakarta.servlet.ServletContext;
import jakarta.servlet.http.HttpServletRequest;

@Controller
public class StimulsoftController {

    @Autowired
    ServletContext context;

    @GetMapping("/viewer")
    public String viewerAction(Model model) throws Exception {
        String reportPath = context.getRealPath("/reports/SimpleList.mrt");
        StiReport report = StiSerializeManager.deserializeReport(
            new java.io.File(reportPath));
        report.render();

        StiWebViewerOptions options = new StiWebViewerOptions();
        model.addAttribute("report", report);
        model.addAttribute("options", options);
        return "viewer";
    }

    @GetMapping("/designer")
    public String designerAction(Model model) throws Exception {
        String reportPath = context.getRealPath("/reports/SimpleList.mrt");
        StiReport report = StiSerializeManager.deserializeReport(
            new java.io.File(reportPath));

        StiWebDesignerOptions options = new StiWebDesignerOptions();
        StiWebDesigerHandlerJk handler = new StiWebDesigerHandlerJk() {
            public StiReport getEditedReport(HttpServletRequest request) {
                return report;
            }
            public void onOpenReportTemplate(StiReport report,
                    HttpServletRequest request) { }
            public void onNewReportTemplate(StiReport report,
                    HttpServletRequest request) { }
            public void onSaveReportTemplate(StiReport report,
                    StiRequestParams requestParams,
                    HttpServletRequest request) {
                // Implement save logic
            }
        };

        model.addAttribute("handler", handler);
        model.addAttribute("options", options);
        return "designer";
    }
}
```

**Step 4.** Create JSP views in `src/main/webapp/WEB-INF/jsp/`:

`viewer.jsp`:
```jsp
<%@ taglib uri="http://stimulsoft.com/webviewer" prefix="stiwebviewer"%>
<html>
<body>
    <stiwebviewer:stiwebviewer report="${report}" options="${options}" />
</body>
</html>
```

`designer.jsp`:
```jsp
<%@ taglib uri="http://stimulsoft.com/webdesigner" prefix="stiwebdesigner"%>
<html>
<body>
    <stiwebdesigner:stiwebdesigner handler="${handler}" options="${options}" />
</body>
</html>
```

---

## 18. How do I integrate with JavaServer Faces (JSF) / Jakarta Faces?

Create a managed bean that provides the report, viewer options, and (for the designer) a handler.

**Viewer bean (JavaServer Faces / javax):**

```java
import javax.faces.context.FacesContext;
import javax.servlet.http.HttpSession;
import com.stimulsoft.report.StiReport;
import com.stimulsoft.report.StiSerializeManager;
import com.stimulsoft.report.dictionary.databases.StiXmlDatabase;
import com.stimulsoft.webviewer.StiWebViewerOptions;

public class StiWebViewerBean {
    StiReport report;
    StiWebViewerOptions options;

    public StiReport getReport() throws Exception {
        if (report == null) {
            FacesContext facesContext = FacesContext.getCurrentInstance();
            HttpSession session = (HttpSession) facesContext
                .getExternalContext().getSession(false);
            String reportPath = session.getServletContext()
                .getRealPath("/reports/Master-Detail.mrt");
            report = StiSerializeManager.deserializeReport(
                new java.io.File(reportPath));
            String xmlPath = session.getServletContext()
                .getRealPath("/data/Demo.xml");
            String xsdPath = session.getServletContext()
                .getRealPath("/data/Demo.xsd");
            report.getDictionary().getDatabases()
                .add(new StiXmlDatabase("Demo", xsdPath, xmlPath));
            report.render();
        }
        return report;
    }

    public StiWebViewerOptions getOptions() {
        return new StiWebViewerOptions();
    }
}
```

**Viewer bean (Jakarta Faces):**

```java
import jakarta.enterprise.context.SessionScoped;
import jakarta.faces.context.FacesContext;
import jakarta.inject.Named;
import jakarta.servlet.http.HttpSession;

@Named("WebViewerBean")
@SessionScoped
public class StiWebViewerBean implements Serializable {
    // Same logic, using jakarta.* imports
}
```

The designer bean implements `StiWebDesigerHandler` (javax) or `StiWebDesigerHandlerJk` (jakarta) and provides methods: `getEditedReport()`, `onOpenReportTemplate()`, `onNewReportTemplate()`, `onSaveReportTemplate()`.

---

## 19. How do I use the Angular Viewer with a Java backend?

For Angular frontend with a Java backend, create a servlet (or Spring Boot controller) that implements `StiAngularViewerHandler`:

**Java EE Servlet approach:**

```java
import com.stimulsoft.webviewer.StiAngularViewerHandler;
import com.stimulsoft.webviewer.StiWebViewerOptions;
import com.stimulsoft.webviewer.servlet.StiWebViewerActionServletHelper;
import com.stimulsoft.web.proxyee.StiHttpServletRequest;
import com.stimulsoft.web.proxyee.StiHttpServletResponse;
import com.stimulsoft.base.json.JSONObject;
import com.stimulsoft.base.mail.StiMailProperties;

public class StiAngularViewerActionServlet extends HttpServlet
        implements StiAngularViewerHandler {

    protected void doGet(HttpServletRequest request,
            HttpServletResponse response) {
        processing(request, response);
    }

    protected void doPost(HttpServletRequest request,
            HttpServletResponse response) {
        processing(request, response);
    }

    protected void processing(HttpServletRequest request,
            HttpServletResponse response) {
        StiWebViewerActionServletHelper.processing(
            new StiHttpServletRequest(request, this),
            new StiHttpServletResponse(response));
    }

    public StiWebViewerOptions getOptions(StiHttpServletRequest request,
            StiHttpServletResponse response) {
        StiWebViewerOptions options = new StiWebViewerOptions();
        options.getAppearance().setScrollbarsMode(true);
        return options;
    }

    public StiReport getReport(StiHttpServletRequest request,
            StiHttpServletResponse response, JSONObject viewerProperties) {
        try {
            String reportName = viewerProperties.getString("reportName");
            File reportFile = new File(request.getSession()
                .getServletContext().getRealPath("reports/" + reportName));
            StiReport report = StiSerializeManager
                .deserializeReport(reportFile);
            report.render();
            return report;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public StiMailProperties getMailProperties(StiHttpServletRequest request,
            StiHttpServletResponse response) {
        return new StiMailProperties();
    }
}
```

**Spring Boot Controller approach:**

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.core.io.ClassPathResource;

@RestController
public class StiViewerController implements StiAngularViewerHandler {

    @RequestMapping("/webviewer/**")
    public void StiWebViewer(HttpServletRequest request,
            HttpServletResponse response) {
        StiWebViewerActionServletHelper.processing(
            new StiHttpServletRequest(request, this),
            new StiHttpServletResponse(response));
    }

    public StiReport getReport(StiHttpServletRequest request,
            StiHttpServletResponse response, JSONObject viewerProperties) {
        try {
            File reportFile = new ClassPathResource(
                viewerProperties.getString("reportName")).getFile();
            StiReport report = StiSerializeManager
                .deserializeReport(reportFile);
            report.render();
            return report;
        } catch (Exception e) {
            return null;
        }
    }

    // ... getOptions() and getMailProperties()
}
```

The Angular frontend communicates with these endpoints to load and interact with reports.

---

## 20. How do I merge several reports into one?

Render each report separately, then combine their rendered pages into a new report:

```java
import com.stimulsoft.report.StiReport;
import com.stimulsoft.report.StiSerializeManager;
import com.stimulsoft.report.components.StiPage;
import com.stimulsoft.report.components.StiComponent;
import com.stimulsoft.report.components.simplecomponents.StiText;

// Render individual reports
StiReport report1 = StiSerializeManager.deserializeReport(
    new File("reports/SimpleList.mrt"));
report1.render();

StiReport report2 = StiSerializeManager.deserializeReport(
    new File("reports/SimpleGroup.mrt"));
report2.render();

// Create merged report
StiReport mergedReport = StiReport.newInstance();
mergedReport.getRenderedPages().clear();

// Add pages from both reports
for (StiPage renderedPage : report1.getRenderedPages()) {
    mergedReport.getRenderedPages().add(renderedPage);
}
for (StiPage renderedPage : report2.getRenderedPages()) {
    mergedReport.getRenderedPages().add(renderedPage);
}

mergedReport.setIsRendered(true);

// Optional: fix page numbering across merged reports
for (int i = 0; i < mergedReport.getRenderedPages().size(); i++) {
    StiPage page = mergedReport.getRenderedPages().get(i);
    StiComponent comp = page.GetComponentByName("PageNumberText");
    if (comp != null && comp instanceof StiText) {
        ((StiText) comp).setText(
            "Page " + (i + 1) + " of " +
            mergedReport.getRenderedPages().size());
    }
}
```

---

## 21. How do I register custom functions for the designer?

You can register custom functions that will be available in report expressions within the web designer. Add them to the report's `customFunctions` collection:

```java
import com.stimulsoft.report.StiCustomFunction;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

report.getCustomFunctions().add(new StiCustomFunction() {

    public Object invoke(List<Object> args) {
        // Custom function logic
        String input = (String) args.get(0);
        int start = ((Long) args.get(1)).intValue();
        int end = ((Long) args.get(2)).intValue();
        return input.substring(start, end);
    }

    @SuppressWarnings({"rawtypes", "unchecked"})
    public List<Class> getParametersList() {
        return new ArrayList<Class>(
            Arrays.asList(String.class, Long.class, Long.class));
    }

    public String getFunctionName() {
        return "subStr";
    }
});
```

This function can then be used in report expressions as `subStr("text", 0, 3)`. The function name, parameter types, and return value are defined by implementing the `StiCustomFunction` interface.

You can also populate data sources when new or opened reports are created in the designer using the handler callbacks `onNewReportTemplate()` and `onOpenReportTemplate()`:

```java
public void onNewReportTemplate(StiReport report, HttpServletRequest request) {
    String xmlPath = request.getSession().getServletContext()
        .getRealPath("/data/Demo.xml");
    String xsdPath = request.getSession().getServletContext()
        .getRealPath("/data/Demo.xsd");
    report.getDictionary().getDatabases()
        .add(new StiXmlDatabase("Demo", xsdPath, xmlPath));

    // Parse XSD and create data sources automatically
    StiXmlTableFieldsRequest tables = StiDataColumnsUtil
        .parceXSDSchema(new FileInputStream(xsdPath));
    for (StiXmlTable table : tables.getTables()) {
        StiDataTableSource tableSource = new StiDataTableSource(
            "Demo." + table.getName(), table.getName(), table.getName());
        tableSource.setColumns(new StiDataColumnsCollection());
        for (StiSqlField field : table.getColumns()) {
            StiDataColumn column = new StiDataColumn(
                field.getName(), field.getName(), field.getSystemType());
            tableSource.getColumns().add(column);
        }
        tableSource.setDictionary(report.getDictionary());
        report.getDictionary().getDataSources().add(tableSource);
    }
}
```
