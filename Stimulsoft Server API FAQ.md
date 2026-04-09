# Stimulsoft Server API — FAQ

Quick answers for developers integrating with Stimulsoft BI Server using the .NET Connect API and REST API.

---

## Table of Contents

1. [How do I install the Server API library?](#1-how-do-i-install-the-server-api-library)
2. [How to connect and log in to the Server?](#2-how-to-connect-and-log-in-to-the-server)
3. [How to connect via REST API?](#3-how-to-connect-via-rest-api)
4. [How to get a list of all workspaces?](#4-how-to-get-a-list-of-all-workspaces)
5. [How to run a report and export to PDF?](#5-how-to-run-a-report-and-export-to-pdf)
6. [How to check the status of a long task?](#6-how-to-check-the-status-of-a-long-task)
7. [How to upload a report template to the Server?](#7-how-to-upload-a-report-template-to-the-server)
8. [How to copy a report template to other workspaces?](#8-how-to-copy-a-report-template-to-other-workspaces)
9. [How to share a report snapshot via public URL?](#9-how-to-share-a-report-snapshot-via-public-url)
10. [How to use the Server API from JavaScript?](#10-how-to-use-the-server-api-from-javascript)

---

## 1. How do I install the Server API library?

Install via NuGet Package Manager in Visual Studio:

**Package Manager Console:**
```powershell
Install-Package Stimulsoft.Connect.API
```

**.NET CLI:**
```bash
dotnet add package Stimulsoft.Connect.API
```

**Supported framework:** .NET Framework 4.7.2

After installation, add the following using directives:
```csharp
using Stimulsoft.Server.Connect;
using Stimulsoft.Server.Objects;
```

For REST API usage without the .NET library, you only need `Newtonsoft.Json`:
```powershell
Install-Package Newtonsoft.Json
```

---

## 2. How to connect and log in to the Server?

Use `StiServerConnection` from the .NET Connect API:

```csharp
using Stimulsoft.Server.Connect;

var serverAddress = "localhost:17764";
var connection = new StiServerConnection(serverAddress);

// Log in
connection.Accounts.Users.Login("user@name.com", "password");

// Check if user is Supervisor
if (connection.Accounts.Roles.Current.IsSupervisor)
{
    // Supervisor has access to all workspaces
}

// Do work...

// Log out
connection.Accounts.Users.Logout();
```

The Supervisor role is required to access workspaces, manage users and roles across the Server.

---

## 3. How to connect via REST API?

All REST API requests go to `http://{ServerAddress}/1/{endpoint}`. Authentication uses custom HTTP headers.

**Log in and get a session key:**
```csharp
var url = "http://localhost:17764/1/login";

var request = WebRequest.Create(url);
request.Method = "GET";
request.Headers.Add("x-sti-UserName", "user@name.com");
request.Headers.Add("x-sti-Password", "password");

var response = GetResponseJson(request);
var sessionKey = response.Value<string>("ResultSessionKey");
```

**Use the session key in subsequent requests:**
```csharp
request.Headers.Add("x-sti-SessionKey", sessionKey);
```

**Log out:**
```csharp
var logOutRequest = WebRequest.Create("http://localhost:17764/1/logout");
logOutRequest.Method = "DELETE";
logOutRequest.Headers.Add("x-sti-SessionKey", sessionKey);
logOutRequest.GetResponse();
```

**Key REST API headers:**

| Header | Description |
|--------|-------------|
| `x-sti-UserName` | Username for login |
| `x-sti-Password` | Password for login |
| `x-sti-SessionKey` | Session key for authenticated requests |
| `x-sti-DestinationItemKey` | Destination item key for export/run operations |
| `x-sti-VersionKey` | Version key for chunked file uploads |

---

## 4. How to get a list of all workspaces?

**Using .NET Connect API:**
```csharp
var connection = new StiServerConnection(serverAddress);
connection.Accounts.Users.Login(username, password);

var workspaceNames = connection.Accounts.Workspaces.FetchAll()
    .Select(workspace => workspace.Name);

connection.Accounts.Users.Logout();
```

**Using REST API:**
```csharp
var request = WebRequest.Create("http://localhost:17764/1/workspaces");
request.Method = "GET";
request.Headers.Add("x-sti-SessionKey", sessionKey);

var response = GetResponseJson(request);
var workspaceNames = response
    .Value<JArray>("ResultWorkspaces")
    .Cast<JObject>()
    .Select(w => w.Value<string>("Name"));
```

Only a user with the Supervisor role can access the list of all workspaces.

---

## 5. How to run a report and export to PDF?

**Step 1 — Create a destination file item:**
```csharp
var createRequest = WebRequest.Create("http://localhost:17764/1/files");
createRequest.Method = "POST";
createRequest.Headers.Add("x-sti-SessionKey", sessionKey);

var body = new JObject
{
    new JProperty("Ident", "FileItem"),
    new JProperty("Name", "ReportExportedToPdf"),
    new JProperty("Description", "Export to PDF"),
    new JProperty("FileType", "Pdf")
};
// Write body to request stream...

var response = GetResponseJson(createRequest);
var pdfFileItemKey = response
    .Value<JArray>("ResultCommands")
    .Where(c => c.Value<string>("Ident") == "ItemSave").Single()
    .Value<JArray>("ResultItems").Single()
    .Value<string>("Key");
```

**Step 2 — Run the report template into the destination:**
```csharp
var reportTemplateKey = "your-report-template-key";
var runUrl = $"http://localhost:17764/1/reporttemplates/{reportTemplateKey}/run";

var runRequest = WebRequest.Create(runUrl);
runRequest.Method = "PUT";
runRequest.Headers.Add("x-sti-SessionKey", sessionKey);
runRequest.Headers.Add("x-sti-DestinationItemKey", pdfFileItemKey);

var exportBody = new JObject
{
    new JProperty("FileItemName", "ExportToPDF"),
    new JProperty("ExportSet", new JObject
    {
        new JProperty("Ident", "Pdf"),
        new JProperty("PageRange", new JObject { }),
        new JProperty("EmbeddedFonts", "false"),
        new JProperty("DitheringType", "None"),
        new JProperty("PdfACompliance", "false")
    })
};
// Write body to request stream...
```

---

## 6. How to check the status of a long task?

After starting an export or other long operation, the response contains `ResultTaskKey`. Use it to check progress:

```csharp
var taskKey = exportResponse.Value<string>("ResultTaskKey");

var statusRequest = WebRequest.Create($"http://localhost:17764/1/task/{taskKey}");
statusRequest.Method = "GET";
statusRequest.Headers.Add("x-sti-SessionKey", sessionKey);

var statusResponse = GetResponseJson(statusRequest);
var taskStatus = statusResponse.Value<JObject>("ResultStatus");

// Available status properties:
var isWaiting   = taskStatus.Value<bool>("IsWaiting");
var isRunning   = taskStatus.Value<bool>("IsRunning");
var isProcessed = taskStatus.Value<bool>("IsProcessed");
var isFinished  = taskStatus.Value<bool>("IsFinished");
var isStopped   = taskStatus.Value<bool>("IsStopped");
var isError     = taskStatus.Value<bool>("IsError");
var created     = taskStatus.Value<DateTime>("Created");
var started     = taskStatus.Value<DateTime>("Started");
```

Poll `GET /1/task/{taskKey}` periodically until `IsFinished` or `IsError` is `true`.

---

## 7. How to upload a report template to the Server?

The REST API limits uploads to 90 KB per request. Split large files into chunks.

**First request — create the item with the first chunk:**
```csharp
var createRequest = WebRequest.Create("http://localhost:17764/1/files");
createRequest.Method = "POST";
createRequest.Headers.Add("x-sti-SessionKey", sessionKey);

var body = new JObject
{
    new JProperty("Ident", "ReportTemplateItem"),
    new JProperty("Name", "MyReport"),
    new JProperty("Description", "My report template"),
    new JProperty("Resource", Convert.ToBase64String(firstChunkBytes))
};
// Write body to request stream...

var response = GetResponseJson(createRequest);
var fileKey = response.Value<JArray>("ResultCommands")[0]
    .Value<JArray>("ResultItems")[0].Value<string>("Key");
var versionKey = response.Value<JArray>("ResultCommands")[1]
    .Value<string>("ResultVersionKey");
```

**Subsequent requests — append remaining chunks:**
```csharp
var appendRequest = WebRequest.Create($"http://localhost:17764/1/files/{fileKey}");
appendRequest.Method = "PUT";
appendRequest.Headers.Add("x-sti-SessionKey", sessionKey);
appendRequest.Headers.Add("x-sti-VersionKey", versionKey);

var appendBody = new JObject
{
    new JProperty("Resource", Convert.ToBase64String(nextChunkBytes))
};
// Write body to request stream...

var appendResponse = GetResponseJson(appendRequest);
versionKey = appendResponse.Value<JArray>("ResultCommands")[0]
    .Value<string>("ResultVersionKey");
```

Each chunk response returns a new `ResultVersionKey` that must be passed in the next request.

---

## 8. How to copy a report template to other workspaces?

Only a user within a workspace can save items there. The pattern is: create a temporary user in each target workspace, log in as that user, upload the template, then clean up.

**Using .NET Connect API:**
```csharp
var connection = new StiServerConnection(serverAddress);
connection.Accounts.Users.Login(username, password);

// Get source template
var sourceItem = (StiReportTemplateItem)connection.Items.GetByKey(templateKey);
var sourceBytes = sourceItem.DownloadToArray();

var sourceWorkspace = connection.Accounts.Workspaces.Current;

var targetWorkspaces = connection.Accounts.Workspaces.FetchAll()
    .Where(w => w.Key != sourceWorkspace.Key);

foreach (var targetWorkspace in targetWorkspaces)
{
    // Create role with ReportTemplate permissions
    var role = connection.Accounts.Roles.New("CopierRole", new StiCommandPermissions
    {
        ItemReportTemplates = StiPermissions.Create | StiPermissions.Modify
    });
    role.WorkspaceKey = targetWorkspace.Key;
    role.Save();

    // Create temporary user
    var tempUser = connection.Accounts.Users.NewUser(tempUserName, tempPassword);
    tempUser.WorkspaceKey = targetWorkspace.Key;
    tempUser.RoleKey = role.Key;
    tempUser.Save();

    // Log in as temporary user and upload
    var tempConnection = new StiServerConnection(serverAddress);
    tempConnection.Login(tempUserName, tempPassword);

    var copy = tempConnection.Items.Root.NewReportTemplate("Copy of Template",
        StiItemContentType.Report);
    copy.Save();
    copy.UploadFromArray(sourceBytes);

    // Clean up
    tempConnection.Logout();
    connection.Accounts.Users.DeleteByKey(tempUser.Key);
    connection.Accounts.Roles.DeleteByKey(role.Key);
}

connection.Accounts.Users.Logout();
```

The same operation can be done via REST API — see the full sample in the repository.

---

## 9. How to share a report snapshot via public URL?

From JavaScript, run a report to a snapshot, share it publicly, then get the URL:

```javascript
// 1. Create snapshot
var req = new XMLHttpRequest();
req.open('POST', 'http://localhost:40010/1/reportsnapshots', false);
req.setRequestHeader("x-sti-SessionKey", sessionKey);

var expiresDate = new Date(new Date().getTime() + 5 * 60000);
var body = {
    'Ident': 'ReportSnapshotItem',
    'Name': 'Snapshot001',
    'Description': '',
    'Expires': expiresDate
};
req.send(JSON.stringify(body));
var snapshotKey = JSON.parse(req.responseText).ResultItems[0].Key;

// 2. Run report template to snapshot
var runReq = new XMLHttpRequest();
runReq.open('PUT', 'http://localhost:40010/1/reporttemplates/' + reportKey + '/run', false);
runReq.setRequestHeader("x-sti-SessionKey", sessionKey);
runReq.setRequestHeader("x-sti-DestinationItemKey", snapshotKey);
runReq.send();

// 3. Share snapshot publicly
var shareReq = new XMLHttpRequest();
shareReq.open('PUT', 'http://localhost:40010/1/items/' + snapshotKey + '/share', false);
shareReq.setRequestHeader("x-sti-SessionKey", sessionKey);
shareReq.send(JSON.stringify({
    'ShareLevel': 'Public',
    'ShareExpires': expiresDate
}));

// 4. Get the share URL
var urlReq = new XMLHttpRequest();
urlReq.open('GET', 'http://localhost:40010/1/items/' + snapshotKey + '/share', false);
urlReq.setRequestHeader("x-sti-SessionKey", sessionKey);
urlReq.send();

var shareUrl = JSON.parse(urlReq.responseText).ResultUrl;
location.href = shareUrl;
```

---

## 10. How to use the Server API from JavaScript?

Use `XMLHttpRequest` to call the REST API directly from a browser:

**Log in:**
```javascript
function Connect() {
    var req = new XMLHttpRequest();
    req.open('GET', 'http://localhost:40010/1/login', false);
    req.setRequestHeader("x-sti-UserName", "user@name.com");
    req.setRequestHeader("x-sti-Password", "password");
    req.send();

    var data = JSON.parse(req.responseText);
    return data.ResultSessionKey;
}
```

**Create a report snapshot:**
```javascript
var sessionKey = Connect();

var req = new XMLHttpRequest();
req.open('POST', 'http://localhost:40010/1/reportsnapshots', false);
req.setRequestHeader("x-sti-SessionKey", sessionKey);

var body = {
    'Ident': 'ReportSnapshotItem',
    'Name': 'MySnapshot',
    'Description': ''
};
req.send(JSON.stringify(body));
var result = JSON.parse(req.responseText);
```

**Key REST API endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/1/login` | Log in (headers: UserName, Password) |
| `DELETE` | `/1/logout` | Log out |
| `GET` | `/1/workspaces` | Get all workspaces |
| `POST` | `/1/files` | Create a file/report item |
| `PUT` | `/1/files/{key}` | Append data to file (chunked upload) |
| `GET` | `/1/files/{key}` | Download a file item |
| `PUT` | `/1/reporttemplates/{key}/run` | Run a report template |
| `PUT` | `/1/reportsnapshots/{key}/export` | Export a snapshot |
| `GET` | `/1/task/{key}` | Check long task status |
| `POST` | `/1/reportsnapshots` | Create a report snapshot |
| `PUT` | `/1/items/{key}/share` | Share an item publicly |
| `GET` | `/1/items/{key}/share` | Get share URL |
| `POST` | `/1/roles` | Create a role |
| `DELETE` | `/1/roles/{key}` | Delete a role |
| `POST` | `/1/users` | Create a user |
| `DELETE` | `/1/users/{name}` | Delete a user |

---

*For more details see the [official Server API documentation](https://www.stimulsoft.com/en/documentation/online/server-api/) and [code samples on GitHub](https://github.com/stimulsoft/Samples-Server-API-for-JS-and-NET).*
