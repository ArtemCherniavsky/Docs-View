# Документация по примерам Stimulsoft Reports.NET для WinForms

> **Репозиторий:** [Samples-Reports.NET-for-WinForms](https://github.com/stimulsoft/Samples-Reports.NET-for-WinForms)  
> **Продукт:** Stimulsoft Reports.NET  
> **Дата анализа:** 2026-04-08

---

## Содержание

1. [Обзор репозитория](#обзор-репозитория)
2. [Структура проектов](#структура-проектов)
3. [Активация лицензии](#активация-лицензии)
4. [Категории примеров](#категории-примеров)
   - [Отображение отчётов](#1-отображение-отчётов)
   - [Подключение данных](#2-подключение-данных)
   - [Создание отчётов из кода](#3-создание-отчётов-из-кода)
   - [Экспорт и печать](#4-экспорт-и-печать)
   - [Переменные отчёта](#5-переменные-отчёта)
   - [Дизайнер отчётов](#6-дизайнер-отчётов)
   - [Локализация и глобализация](#7-локализация-и-глобализация)
   - [Ресурсы (шрифты)](#8-ресурсы-шрифты)
   - [Кастомные адаптеры данных](#9-кастомные-адаптеры-данных)
   - [Расширенные возможности (только .NET Framework 4.7.2)](#10-расширенные-возможности-только-net-framework-472)
5. [Матрица совместимости примеров](#матрица-совместимости-примеров)
6. [Ключевые API и классы](#ключевые-api-и-классы)
7. [Паттерны использования](#паттерны-использования)
8. [Запуск примеров](#запуск-примеров)

---

## Обзор репозитория

Репозиторий содержит примеры исходного кода, демонстрирующие использование **Stimulsoft Reports.NET** — инструмента для создания отчётов в приложениях Windows Forms на C#. Примеры организованы по трём версиям фреймворка:

| Версия | Папка | Количество примеров |
|--------|-------|-------------------|
| .NET Framework 4.7.2 | `NET Framework 4.7.2/` | 29 примеров |
| .NET 6.0 | `NET 6.0/` | 9 примеров |
| .NET 8.0 | `NET 8.0/` | 9 примеров |

> Примеры для .NET 6.0 и .NET 8.0 идентичны по коду. .NET Framework 4.7.2 содержит более широкий набор с дополнительными продвинутыми сценариями.

---

## Структура проектов

Каждый пример — это отдельный проект Visual Studio. Типичная структура:

```
Название примера/
├── Form1.cs               # Основная логика примера
├── Form1.Designer.cs      # Сгенерированный код UI
├── Program.cs             # Точка входа
├── Reports/               # Шаблоны отчётов (.mrt, .mdc)
├── Data/                  # Тестовые данные (.xml, .json, .xsd)
├── Fonts/                 # Шрифты (если нужны)
└── Localization/          # Файлы локализации (.xml)
```

Решения Visual Studio:
- `NET Framework 4.7.2/NET 4.7.2 WinForms Samples.sln`
- `NET 6.0/NET 6.0 WinForms Samples.sln`
- `NET 8.0/NET 8.0 WinForms Samples.sln`

---

## Активация лицензии

Во всех примерах активация лицензии закомментирована и показана в конструкторе формы. Поддерживаются три способа:

```csharp
// Способ 1: Лицензионный ключ напрямую в коде
Stimulsoft.Base.StiLicense.Key = "6vJhGtLLLz2GNviWmUTrhSqnO...";

// Способ 2: Загрузка из файла
Stimulsoft.Base.StiLicense.LoadFromFile("license.key");

// Способ 3: Загрузка из потока
Stimulsoft.Base.StiLicense.LoadFromStream(stream);
```

> **Важно:** Активацию необходимо выполнять **до** любых обращений к API Stimulsoft. Рекомендуется размещать в конструкторе главной формы или в `Program.cs`.

---

## Категории примеров

### 1. Отображение отчётов

**Пример:** `Showing a Report in the Viewer`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Демонстрирует два способа отображения готового отчёта: встроенный элемент управления на форме и модальный диалог.

```csharp
// Вариант 1: Встраивание в элемент управления StiViewerControl
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
report.Render();
stiViewerControl1.Report = report; // Встроенный вьюер

// Вариант 2: Показ в отдельном окне-диалоге
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
report.Show();

// Альтернатива: с Ribbon-интерфейсом
// report.ShowWithRibbonGUI();
```

**Ключевые классы:**
- `StiReport` — основной класс отчёта
- `StiViewerControl` — WinForms-элемент для встроенного отображения
- `report.Show()` — открывает отчёт в отдельном диалоговом окне

---

### 2. Подключение данных

**Пример:** `Connecting to Data from Code`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Показывает, как подключить данные из кода — из XML и JSON файлов — без настройки источников данных в дизайнере.

```csharp
// Подключение из XML
var dataSet = new DataSet();
dataSet.ReadXmlSchema(@"Data\Demo.xsd");
dataSet.ReadXml(@"Data\Demo.xml");

var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");
report.Dictionary.Databases.Clear(); // Очищаем связи из шаблона
report.RegData("Demo", dataSet);     // Регистрируем данные
report.Show();

// Подключение из JSON
var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\Demo.json");
// ... далее аналогично XML
```

**Ключевые классы:**
- `StiJsonToDataSetConverterV2` — конвертер JSON в DataSet
- `report.RegData(name, dataSet)` — регистрация данных в отчёте
- `report.Dictionary.Databases.Clear()` — очистка источников данных

---

**Пример:** `Using Business Objects in the Report` (только .NET Framework 4.7.2)

Демонстрирует использование коллекций объектов C# как источников данных, поддерживая два интерфейса: `IEnumerable` и `ITypedList`.

```csharp
// IEnumerable — простые коллекции объектов
StiReport report = new StiReport();
report.RegData("EmployeeIEnumerable", CreateBusinessObjectsIEnumerable.GetEmployees());
report.Load(@"BusinessObjects_IEnumerable.mrt");
report.Show();

// ITypedList — типизированные списки с метаданными свойств
StiReport report = new StiReport();
report.RegData("EmployeeITypedList", CreateBusinessObjectsITypedList.GetEmployees());
report.Load(@"BusinessObjects_ITypedList.mrt");
report.Show();
```

---

**Пример:** `Using Linq in Reports` (только .NET Framework 4.7.2)

Позволяет использовать LINQ-запросы как источники данных для отчётов.

---

**Пример:** `Using User Data in Reports` (только .NET Framework 4.7.2)

Использование пользовательских провайдеров данных с рефлексией.

---

### 3. Создание отчётов из кода

**Пример:** `Creating Report at Runtime`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Полное создание структуры отчёта программно без использования дизайнера.

```csharp
var report = new StiReport();

// Подключение данных
var dataSet = StiJsonToDataSetConverterV2.GetDataSetFromFile(@"Data\Demo.json");
report.RegData(dataSet);
report.Dictionary.Synchronize(); // Синхронизация словаря

var page = report.Pages[0];

// Создание HeaderBand
var headerBand = new StiHeaderBand();
headerBand.Height = 0.5;
headerBand.Name = "HeaderBand";
page.Components.Add(headerBand);

// Текст в заголовке
var headerText = new StiText(new RectangleD(0, 0, 5, 0.5));
headerText.Text = "CompanyName";
headerText.HorAlignment = StiTextHorAlignment.Center;
headerText.Brush = new StiSolidBrush(Color.LightGreen);
headerBand.Components.Add(headerText);

// DataBand — полоса данных
var dataBand = new StiDataBand();
dataBand.DataSourceName = "Customers";
dataBand.Height = 0.5;
page.Components.Add(dataBand);

// Текст в полосе данных (поддерживает выражения)
var dataText = new StiText(new RectangleD(0, 0, 5, 0.5));
dataText.Text = "{Line}.{Customers.CompanyName}";
dataBand.Components.Add(dataText);

// FooterBand
var footerBand = new StiFooterBand();
footerBand.Height = 0.5;
page.Components.Add(footerBand);

var footerText = new StiText(new RectangleD(0, 0, 5, 0.5));
footerText.Text = "Count - {Count()}";
footerText.HorAlignment = StiTextHorAlignment.Right;
footerBand.Components.Add(footerText);

report.Show();
```

---

**Пример:** `Creating Chart at Runtime` (только .NET Framework 4.7.2)

Программное создание диаграммы Ганта с использованием `StiGanttArea` и `StiGanttSeries`.

---

### 4. Экспорт и печать

**Пример:** `Exporting a Report from Code`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Экспорт отчёта в различные форматы через код.

```csharp
var report = new StiReport();
report.Load(@"Reports\TwoSimpleLists.mrt");
report.Render();

var stream = new MemoryStream();

// Выбор формата экспорта
switch (selectedFormat)
{
    case "PDF":
        report.ExportDocument(StiExportFormat.Pdf, stream);
        break;
    case "Word":
        report.ExportDocument(StiExportFormat.Word2007, stream);
        break;
    case "Excel":
        report.ExportDocument(StiExportFormat.Excel2007, stream);
        break;
    case "Text":
        report.ExportDocument(StiExportFormat.Text, stream);
        break;
    case "Image":
        report.ExportDocument(StiExportFormat.ImagePng, stream);
        break;
}

// Сохранение в файл
using (var fileStream = File.Create(filePath))
{
    stream.Seek(0, SeekOrigin.Begin);
    stream.CopyTo(fileStream);
}

// Печать
report.Print();
```

**Поддерживаемые форматы `StiExportFormat`:**

| Константа | Формат |
|-----------|--------|
| `StiExportFormat.Pdf` | PDF |
| `StiExportFormat.Word2007` | Microsoft Word (.docx) |
| `StiExportFormat.Excel2007` | Microsoft Excel (.xlsx) |
| `StiExportFormat.Text` | Текстовый файл |
| `StiExportFormat.ImagePng` | PNG изображение |

---

**Пример:** `Exporting Many Files to Single PDF` (только .NET Framework 4.7.2)

Объединение нескольких отрендеренных отчётов в один PDF-файл с использованием кэша.

```csharp
var report = new StiReport();
report.ReportCacheMode = StiReportCacheMode.On;
report.RenderedPages.CanUseCacheMode = true;
report.RenderedPages.CacheMode = true;
report.RenderedPages.Clear();

var tempReport = new StiReport();
for (int index = 0; index < 30; index++)
{
    using (var stream = Assembly.GetExecutingAssembly()
        .GetManifestResourceStream("...MasterDetailSubdetail.mdc"))
    {
        tempReport.LoadDocument(stream);
    }

    foreach (StiPage page in tempReport.RenderedPages)
    {
        page.Report = tempReport;
        page.Guid = Guid.NewGuid().ToString().Replace("-", "");
        report.RenderedPages.Add(page);
    }
}

report.ExportDocument(StiExportFormat.Pdf, "output.pdf");
```

---

**Пример:** `Exporting a Report to ZUGFeRD PDF` (только .NET Framework 4.7.2)

Экспорт отчёта в формат ZUGFeRD PDF v2.1 — стандарт для электронных счетов-фактур с встроенными XML-данными.

---

**Пример:** `Rendering and Exporting a Report Asynchronously` (только .NET Framework 4.7.2)

Асинхронный рендеринг и экспорт без блокировки UI-потока.

```csharp
// Асинхронный рендеринг
private async void buttonRender_Click(object sender, EventArgs e)
{
    await Report.CompileAsync(); // Компиляция (при необходимости)
    await Report.RenderAsync();  // Рендеринг
}

// Асинхронный экспорт
private async void buttonExport_Click(object sender, EventArgs e)
{
    await Report.ExportDocumentAsync(StiExportFormat.Pdf, filePath);
}
```

---

### 5. Переменные отчёта

**Пример:** `Using Report Variables in Code`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Передача значений из формы приложения в переменные отчёта перед отображением.

```csharp
var report = new StiReport();
report.Load(@"Reports\Variables.mrt");

// Обязательно для режима компиляции
report.Compile();

// Установка переменных через индексатор (имя из дизайнера)
report["Name"]    = textBoxName.Text;
report["Surname"] = textBoxSurname.Text;
report["Email"]   = textBoxEmail.Text;
report["Address"] = textBoxAddress.Text;
report["Sex"]     = radioButtonMale.Checked; // bool

report.Show();
```

> **Примечание:** `report.Compile()` необходим для подготовки отчёта в режиме компиляции перед передачей переменных.

---

### 6. Дизайнер отчётов

**Пример:** `Editing a Report Template in the Designer`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Открытие шаблона отчёта в интерактивном дизайнере с обработкой события сохранения.

```csharp
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");

// Обработка события сохранения
report.StiReportDesigner.EventSave += (sender, args) =>
{
    // Логика сохранения
};

report.Design();
```

---

**Пример:** `Saving and Loading a Report in the Designer` (только .NET Framework 4.7.2)

Обработка событий сохранения и загрузки файлов в дизайнере.

---

**Пример:** `Adding a Custom Component to the Designer` (только .NET Framework 4.7.2)

Регистрация пользовательского компонента в дизайнере через `StiConfig`.

---

**Пример:** `Customizing the Viewer` (только .NET Framework 4.7.2)

Расширенная настройка внешнего вида и функциональности встроенного вьюера отчётов.

---

**Пример:** `Changing the Viewer and Designer Theme` (только .NET Framework 4.7.2)

Управление темой оформления (светлая/тёмная/авто) и акцентным цветом.

```csharp
// Установка темы
StiUXTheme.ApplyNewTheme(StiThemeAppearance.Dark, StiThemeAccentColor.Blue);

// Проверка текущей темы
switch (StiUXTheme.Appearance)
{
    case StiThemeAppearance.Auto:  break;
    case StiThemeAppearance.Light: break;
    case StiThemeAppearance.Dark:  break;
}

// Доступные акцентные цвета
// StiThemeAccentColor: Auto, Blue, Violet, Carmine, Teal, Green, Orange
```

**Включение HiDPI-режима** (рекомендуется для .NET Framework):
```csharp
// В Main() перед Application.Run()
Stimulsoft.Report.Win.StiDpiAwarenessHelper.SetPerMonitorDpiAware();
Application.EnableVisualStyles();
Application.Run(new Form1());
```

---

### 7. Локализация и глобализация

**Пример:** `Localizing the User Interface`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Загрузка XML-файлов локализации для перевода интерфейса вьюера и дизайнера.

```csharp
// Загрузка файлов локализации из папки
foreach (var fileName in Directory.GetFiles(@"Localization"))
{
    comboBoxLocalizations.Items.Add(fileName);
    if (fileName.EndsWith("en.xml"))
        comboBoxLocalizations.SelectedItem = fileName;
}

// Применение выбранной локализации
StiOptions.Localization.Load(comboBoxLocalizations.Text);

// Далее — открытие отчёта с применённой локализацией
var report = new StiReport();
report.Load(@"Reports\SimpleList.mrt");
report.Show(); // Интерфейс будет на выбранном языке
```

---

**Пример:** `Globalizing Reports` (только .NET Framework 4.7.2)

Настройка форматирования данных отчёта в зависимости от культуры (языковых стандартов).

---

### 8. Ресурсы (шрифты)

**Пример:** `Adding a Font to the Resource`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Встраивание TTF-шрифта в ресурсы отчёта и его использование в компонентах.

```csharp
// Загрузка шрифта из файла и добавление в ресурсы отчёта
var fileContent = File.ReadAllBytes("Fonts/Roboto-Black.ttf");
var resource = new StiResource(
    "Roboto-Black",           // Имя ресурса
    "Roboto-Black",           // Псевдоним
    false,
    StiResourceType.FontTtf,  // Тип ресурса
    fileContent,
    false
);
report.Dictionary.Resources.Add(resource);

// Регистрация шрифта в коллекции шрифтов
StiFontCollection.AddResourceFont(
    resource.Name,
    resource.Content,
    "ttf",
    resource.Alias
);

// Создание компонента с использованием нестандартного шрифта
var dataText = new StiText();
dataText.ClientRectangle = new RectangleD(1, 1, 3, 2);
dataText.Text = "Sample Text";
dataText.Font = StiFontCollection.CreateFont("Roboto-Black", 12, FontStyle.Regular);
dataText.Border.Side = StiBorderSides.All;
report.Pages[0].Components.Add(dataText);
```

**Ключевые классы:**
- `StiResource` — ресурс отчёта (шрифт, изображение и др.)
- `StiResourceType.FontTtf` — тип ресурса «шрифт TTF»
- `StiFontCollection` — коллекция шрифтов Stimulsoft
- `StiFontCollection.CreateFont()` — создание объекта Font из коллекции

---

### 9. Кастомные адаптеры данных

**Пример:** `Using a Custom Data Adapter`  
**Доступность:** .NET Framework 4.7.2, .NET 6.0, .NET 8.0

Реализация собственного адаптера базы данных (на примере PostgreSQL) для использования в дизайнере и отчётах.

```csharp
// В конструкторе — регистрация адаптера
StiOptions.Services.Databases.Clear(); // Очистка стандартных (при необходимости)
StiOptions.Services.Databases.Add(new CustomPostgreSQLDatabase());
StiOptions.Services.DataAdapters.Add(new CustomPostgreSQLAdapterService());

// Добавление подключения к отчёту из кода
var report = new StiReport();
var database = new CustomPostgreSQLDatabase(
    "CustomData1",
    "Server=127.0.0.1; Port=5432; Database=myDataBase; User Id=myUsername; Password=myPassword;"
);
report.Dictionary.Databases.Add(database);
report.Design(); // Открытие дизайнера с доступом к БД
```

Для реализации кастомного адаптера нужно создать два класса:
- Наследник от базового класса `StiDatabase` — описание подключения
- Наследник от `StiAdapterService` — логика выполнения запросов

---

### 10. Расширенные возможности (только .NET Framework 4.7.2)

#### Drill-down (детализация) в реальном времени

**Пример:** `Drilling Down Report in Live`

Интерактивная детализация: клик по элементу отчёта открывает дочерний отчёт с фильтром.

```csharp
private void button2_Click(object sender, EventArgs e)
{
    var report = new StiReport();
    report.RegData(dataSet1);
    report.Load(@"LiveReports.mrt");
    report.Compile();

    // Подписка на событие клика по компоненту отчёта
    report.CompiledReport.Click += new EventHandler(click);
    report.Show();
}

private void click(object sender, EventArgs e)
{
    var comp = sender as StiComponent;
    var customerID = (string)comp.BookmarkValue;

    if (customerID != null)
    {
        var report = new StiReport();
        report.RegData(dataSet1);
        report.Load(@"Details.mrt");

        // Программная фильтрация данных
        var dataBand = (StiDataBand)report.Pages["Page1"].Components["DataBand1"];
        var filter = new StiFilter("{Orders.CustomerID==\"" + customerID + "\"}");
        dataBand.Filters.Add(filter);
        report.Show();
    }
}
```

---

#### Управление подотчётами

**Пример:** `Managing Reports with Sub-Reports`

Объединение нескольких отчётов с возможностью управления нумерацией страниц.

---

#### Рендеринг в фоновом потоке

**Пример:** `Rendering a Report in the Thread`

Рендеринг в `BackgroundWorker` для отображения прогресса без блокировки UI.

---

**Пример:** `Showing a Progress while Rendering a Report`

Показ прогресс-бара во время рендеринга длинного отчёта.

---

#### Автообновление в реальном времени

**Пример:** `Previewing a Report with AutoUpdate in Realtime`

Периодическое обновление данных и перерисовка отчёта по таймеру для отображения live-данных.

---

#### RTL режим

**Пример:** `Using the Right-To-Left Mode in the Viewer`

```csharp
// Включение режима Right-To-Left
viewer.RightToLeft = StiRightToLeftType.Yes;
```

---

#### Использование страницы отчёта как Canvas

**Пример:** `Using the Report Page Canvas for the Copyrights`

Рисование произвольного содержимого поверх страниц отчёта через событие отрисовки страницы.

---

#### Тестирование производительности движков

**Пример:** `Testing Memory Usage in EngineV1 and EngineV2`

Сравнение использования памяти между движком V1 (устаревший) и V2 (актуальный). Полезно при выборе движка для рендеринга.

```csharp
// Выбор версии движка
stiReport.EngineVersion = Stimulsoft.Report.Engine.StiEngineVersion.EngineV2;
```

---

#### Выбор колонок через форму

**Пример:** `Using the Form to Select Columns`

Отображение диалога для выбора пользователем нужных колонок из DataSet перед построением отчёта.

---

## Матрица совместимости примеров

| Пример | .NET Framework 4.7.2 | .NET 6.0 | .NET 8.0 |
|--------|:--------------------:|:--------:|:--------:|
| Showing a Report in the Viewer | ✅ | ✅ | ✅ |
| Connecting to Data from Code | ✅ | ✅ | ✅ |
| Creating Report at Runtime | ✅ | ✅ | ✅ |
| Editing a Report Template in the Designer | ✅ | ✅ | ✅ |
| Exporting a Report from Code | ✅ | ✅ | ✅ |
| Localizing the User Interface | ✅ | ✅ | ✅ |
| Using Report Variables in Code | ✅ | ✅ | ✅ |
| Using a Custom Data Adapter | ✅ | ✅ | ✅ |
| Adding a Font to the Resource | ✅ | ✅ | ✅ |
| Adding a Custom Component to the Designer | ✅ | ❌ | ❌ |
| Changing the Viewer and Designer Theme | ✅ | ❌ | ❌ |
| Creating Chart at Runtime | ✅ | ❌ | ❌ |
| Customizing the Viewer | ✅ | ❌ | ❌ |
| Drilling Down Report in Live | ✅ | ❌ | ❌ |
| Exporting Many Files to Single PDF | ✅ | ❌ | ❌ |
| Exporting a Report to ZUGFeRD PDF | ✅ | ❌ | ❌ |
| Globalizing Reports | ✅ | ❌ | ❌ |
| Managing Reports with Sub-Reports | ✅ | ❌ | ❌ |
| Previewing a Report with AutoUpdate | ✅ | ❌ | ❌ |
| Rendering a Report in the Thread | ✅ | ❌ | ❌ |
| Rendering and Exporting Asynchronously | ✅ | ❌ | ❌ |
| Saving and Loading in the Designer | ✅ | ❌ | ❌ |
| Showing a Progress while Rendering | ✅ | ❌ | ❌ |
| Testing Memory Usage (EngineV1/V2) | ✅ | ❌ | ❌ |
| Using Business Objects in the Report | ✅ | ❌ | ❌ |
| Using Linq in Reports | ✅ | ❌ | ❌ |
| Using User Data in Reports | ✅ | ❌ | ❌ |
| Using the Form to Select Columns | ✅ | ❌ | ❌ |
| Using the Report Page Canvas | ✅ | ❌ | ❌ |
| Using the Right-To-Left Mode | ✅ | ❌ | ❌ |
| Demo | ✅ | ❌ | ❌ |

---

## Ключевые API и классы

### Пространства имён

```csharp
using Stimulsoft.Report;                   // StiReport, StiExportFormat
using Stimulsoft.Report.Components;        // StiText, StiDataBand, StiHeaderBand, StiPage...
using Stimulsoft.Report.Viewer;            // StiViewerControl
using Stimulsoft.Report.Design;            // Дизайнер
using Stimulsoft.Report.Dictionary;        // StiResource, базы данных
using Stimulsoft.Base;                     // StiJsonToDataSetConverterV2, StiOptions
using Stimulsoft.Base.Drawing;             // RectangleD, StiSolidBrush
using Stimulsoft.Base.Localization;        // StiOptions.Localization
using Stimulsoft.Base.Theme;               // StiUXTheme, StiThemeAppearance
```

### Класс `StiReport` — основные методы

| Метод / Свойство | Описание |
|-----------------|----------|
| `new StiReport()` | Создание нового отчёта |
| `report.Load(path)` | Загрузка шаблона (.mrt) |
| `report.LoadDocument(stream)` | Загрузка готового документа (.mdc) |
| `report.Render()` | Синхронный рендеринг |
| `report.RenderAsync()` | Асинхронный рендеринг |
| `report.Compile()` | Компиляция (необходима при использовании переменных) |
| `report.CompileAsync()` | Асинхронная компиляция |
| `report.Show()` | Отображение в диалоговом окне |
| `report.ShowWithRibbonGUI()` | Отображение с Ribbon-интерфейсом |
| `report.Design()` | Открытие в дизайнере |
| `report.Print()` | Печать |
| `report.ExportDocument(format, stream/path)` | Экспорт |
| `report.ExportDocumentAsync(format, path)` | Асинхронный экспорт |
| `report.RegData(name, dataSet)` | Регистрация данных |
| `report["VarName"] = value` | Установка переменной |
| `report.Dictionary` | Доступ к словарю (данные, ресурсы) |
| `report.Pages` | Коллекция страниц |
| `report.RenderedPages` | Коллекция отрендеренных страниц |

### Компоненты страницы

| Класс | Описание |
|-------|----------|
| `StiHeaderBand` | Полоса заголовка |
| `StiDataBand` | Полоса данных |
| `StiFooterBand` | Полоса нижнего колонтитула |
| `StiText` | Текстовый компонент |
| `StiImage` | Компонент изображения |
| `StiPage` | Страница отчёта |

---

## Паттерны использования

### Паттерн 1: Загрузка и показ отчёта (базовый)

```csharp
var report = new StiReport();
report.Load("report.mrt");
report.Show();
```

### Паттерн 2: Загрузка, регистрация данных, показ

```csharp
var report = new StiReport();
report.Load("report.mrt");
report.Dictionary.Databases.Clear();
report.RegData("DataName", dataSet);
report.Show();
```

### Паттерн 3: Создание отчёта программно

```csharp
var report = new StiReport();
report.RegData(dataSet);
report.Dictionary.Synchronize();
// ... добавление компонентов ...
report.Show();
```

### Паттерн 4: Экспорт в поток

```csharp
var report = new StiReport();
report.Load("report.mrt");
report.Render();
var stream = new MemoryStream();
report.ExportDocument(StiExportFormat.Pdf, stream);
```

### Паттерн 5: Передача переменных

```csharp
var report = new StiReport();
report.Load("report.mrt");
report.Compile();
report["VarName"] = value;
report.Show();
```

### Паттерн 6: Встроенный вьюер

```csharp
var report = new StiReport();
report.Load("report.mrt");
report.Render();
stiViewerControl1.Report = report;
```

### Паттерн 7: Асинхронный рендеринг и экспорт

```csharp
await report.CompileAsync();
await report.RenderAsync();
await report.ExportDocumentAsync(StiExportFormat.Pdf, filePath);
```

---

## Запуск примеров

1. Откройте файл решения (`.sln`) в Visual Studio.
2. Установите нужный проект как **Startup Project** (ПКМ → "Set as Startup Project").
3. Запустите (`F5`). NuGet-пакеты Stimulsoft будут загружены автоматически.

**NuGet-пакет:** [Stimulsoft.Reports.Net](https://www.nuget.org/packages/Stimulsoft.Reports.Net)

### Полезные ссылки

- [Live Demo](http://demo.stimulsoft.com/#Net)
- [Страница продукта](https://www.stimulsoft.com/en/products/reports-net)
- [Документация](https://www.stimulsoft.com/en/documentation/online/programming-manual/reports_net.htm)
- [Скачать бесплатно](https://www.stimulsoft.com/en/downloads)
- [NuGet](https://www.nuget.org/packages/Stimulsoft.Reports.Net)

---

*Документация создана на основе анализа исходного кода примеров.*
