# Eurofins DevOps Take-Home

Simple Hello World API built with .NET 10.

## Run locally

Install the .NET 10 SDK, then run these commands from the repository root:

```bash
dotnet restore
dotnet run --project src/HelloWorldApi --launch-profile http
```

The API runs at `http://localhost:5184`. Press `Ctrl+C` to stop it.

## Build and test

```bash
dotnet build --configuration Release
dotnet test --configuration Release
```

Tests start the API in memory, so you don't need to run it separately.

## Endpoints

| Method | Path | Response |
| --- | --- | --- |
| GET | `/` | `200 OK` — `Hello World!` |
| GET | `/health` | `200 OK` — `Healthy` |

## Download the package

Open the repository's **Actions** tab, select a successful **CI** run, and download **HelloWorldApi** from **Artifacts**. You need to be signed in to GitHub.

The ZIP contains the published API and its dependencies. The target server needs the ASP.NET Core 10 runtime.

## Deploy to IIS (Step 2)

Use Windows with IIS, the WebAdministration PowerShell module, and the .NET 10 Hosting Bundle. Install IIS before installing the Hosting Bundle (repair the bundle if it was installed first). Run the deployment in **64-bit Windows PowerShell 5.1 as Administrator**.

Save the downloaded artifact as `artifacts/HelloWorldApi.zip`. The ZIP must contain `web.config` and `HelloWorldApi.dll` at its root. To build a package locally with the .NET 10 SDK instead:

```powershell
dotnet publish src/HelloWorldApi/HelloWorldApi.csproj --configuration Release --output artifacts/HelloWorldApi
Compress-Archive -Path .\artifacts\HelloWorldApi\* -DestinationPath .\artifacts\HelloWorldApi.zip -Force
```

The script expects an enabled local account named `HelloWorldUser`. Create it once if it does not exist, supplying a password interactively:

```powershell
$password = Read-Host 'Password for HelloWorldUser' -AsSecureString
New-LocalUser -Name HelloWorldUser -Password $password
$password.Dispose()
```

From the repository root, deploy with:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\scripts\deployment-script.ps1
```

Enter the account password when prompted. The deployment script has no command-line parameters; the deployment path, account and group are configured at its top. It calls `configure-iis.ps1` with `-DeploymentDir` and an in-memory `-Credential`. Never put the password in source control.

| Resource | Default |
| --- | --- |
| Application files | `C:\inetpub\HelloWorldApi` |
| Website / application | `HelloWorldApi` / `/api` |
| Website root | `C:\inetpub\HelloWorldApiSite` |
| Application pool | `HelloWorldApiPool`, No Managed Code, `HelloWorldUser` identity |
| File access group | `HelloWorldApiUsers`, inherited read/execute |
| HTTP / HTTPS | `http://localhost:8080/api/` / `https://localhost:8443/api/` |
| Request logs | `C:\inetpub\logs\HelloWorldApi` |

The HTTPS binding uses a self-signed localhost certificate for development. It is not automatically trusted by browsers. Validate HTTPS with that certificate explicitly trusted by the test client.

### Redeployment and validation

Run the same deployment command again after calling the API. The script stops an existing running application pool and waits up to 60 seconds for its files to be released before extracting the package. The pool is restarted in `finally` if it was running before the deployment. An already stopped pool remains stopped. This causes brief downtime; it is not a zero-downtime deployment and does not provide file rollback after a partial extraction failure.

Verify Hello World and `/api/health` over both HTTP and HTTPS, inspect the configured request-log directory, and confirm that running the script again does not duplicate the site, pool, application, bindings, certificate or group membership. See [Step 2 validation](docs/requirements-criteria/step2.md) for the Windows results.
