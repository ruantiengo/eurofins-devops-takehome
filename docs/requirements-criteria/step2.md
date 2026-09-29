# Step 2 - IIS Deployment Checklist

## Deployment script

- [x] Create a PowerShell script to deploy the application from Step 1.
- [x] Check that IIS and the ASP.NET Core Hosting Bundle are installed.
- [x] Extract the application package to the deployment directory.

## User and permissions

- [x] Create a local group.
- [x] Add the specified user to the local group.
- [x] Grant the permissions needed to access the application files.
- [x] Handle the user's password without storing it in source control.

## IIS configuration

- [x] Create an IIS website.
- [x] Add an HTTPS binding with a certificate.
- [x] Create an application pool and configure it to run as the specified user.
- [x] Set a custom directory for the website logs.
- [x] Create an application under the website.
- [x] Assign the application to the created application pool.

## Quality and validation

- [x] Ensure the script can be run again without duplicating existing resources.
- [x] Verify that Hello World is reachable through a localhost HTTP URL.
- [x] Verify that the HTTPS binding works.
- [x] Verify that the application's health endpoint returns 200 OK.
- [x] Verify that website logs are written to the configured directory.
- [x] Document the prerequisites, script parameters, and deployment command.

## Windows validation

Validated on Windows 11 (build 26200), Windows PowerShell 5.1, IIS and ASP.NET Core Hosting Bundle 10.0.12, using a package published locally with .NET SDK 10.0.401.

- Release build: no warnings or errors; both integration tests passed.
- First IIS deployment: Hello World and health returned 200 over HTTP (8080) and HTTPS (8443), under `/api`.
- The original redeployment failed because IIS held `HelloWorldApi.dll` open.
- With the pool shutdown/file-release fix, two deployments completed against the running IIS application. All four endpoint checks passed after each deployment.
- Before/after snapshots were identical: one site, one pool, one application, two bindings, one development certificate and unchanged group membership.
- Request logs were found under `C:\inetpub\logs\HelloWorldApi`.
- HTTPS checks trusted the specific localhost certificate rather than disabling certificate verification.

The deployment grants inherited read/execute permissions to the local group; all 12 deployed filesystem entries were processed successfully by icacls. A separate post-deployment audit of the effective ACL and pool identity was not executed because its UAC elevation was canceled. Endpoint and redeployment results above come from the completed elevated deployment validation.

This validates the local Windows/IIS scenario, not every server configuration or the CI-downloaded ZIP. Redeployment intentionally incurs brief downtime and does not implement rollback or concurrent-deployment locking. No zero-downtime guarantee is made.