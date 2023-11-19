# Dotnet 7 PowerShell Service Tutorial
This tutorial will guide you through the process of running PowerShell commands from a Dotnet 7 windows service. The service is set up to respond to PowerShell commands that are activated by a GET request on an API. The example project will execute a PowerShell script based on the parameter provided and display the result, along with the current user and timestamp, in a web browser. This result is also written to a file in the `C:\\temp` directory. Additionally, Windows event logging has been enabled in the `appsettings.json` file.

## Prerequisites
- Dotnet 7
- Windows OS
- Rider IDE

## Initial Configuration
The service is highly configurable and relies on the `appsettings.json` file for such settings. The specific port the service relies on is also defined within this file. Please note that the `appsettings.Development.json` file may have different port configurations that apply only in the development environment.

## Ensuring Secure API Access
By default, the service's API is only accessible locally via HTTP. For broader accessibility, like for machines on your network, it's advised to configure use of an SSL certificate. This ameliorates the risk of eavesdropping.

Follow these steps to configure an SSL certificate:

1. Install the certificate in the Windows Certificate Store using `certmgr.exe` to the Personal store.

2. Validate the location and the path of the certificate with this PowerShell command:

   ```Powershell
    PS C:\> Get-ChildItem -Path cert:\ -Recurse | Where-Object { $_.Subject -imatch "desk.domain.com" } | Select Subject, HasPrivateKey, PsParentPath
   ```

   This command will return the subject, private key status, and parent path of the installed certificate.

3.  Add the Fully Qualified Domain Name (FQDN) from the certificate to the `AllowedHosts` in `appsettings.json`:

    ``"AllowedHosts": "desk.domain.com;localhost"``

4. Add an HTTPS entry under the "Kestrel" section in the `appsettings.json`:

    ```json
   {
      "Kestrel": {
        "Endpoints": {
          "Http": {
            "Url": "http://localhost:5400"
          },
          "Https": {
            "Url": "https://desk.domain.com:5401",
            "Certificate": {
              "Subject": "desk.domain.com",
              "Store": "My",
              "Location": "LocalMachine",
              "AllowInvalid": false
            }
          }
        }
      }
   }
    ```

## Publishing the Project
To consolidate the project into a distributable format, mark the project as self-contained and target a specific OS runtime. Use the generic `win-x64` runtime.

   ```Powershell
   dotnet publish -o C:\Services\MinimalApiPowershellService\ --sc --runtime win10-x64
   ```

Rider users can also use the publishing profile found in the IDE navigation panel:

![Rider Publish Configuration](riderPublish.png)

## Creating a Windows Service
Once the project is published, it's time to create the actual service. The command to accomplish this is:

   ```Powershell
   sc create "MinimalApiPowershellService" binpath="c:\Services\MinimalApiPowershellService\MinimalApiPowershellService.exe"
   ```

This creates MinimalApiPowershellService:

![Create the service](serviceCreate.png)

After successful creation, you can choose to start it right away or specify a user context it should run under. Make sure the chosen user's permissions are appropriate for running the service.

## Testing the Service
To verify the service works as expected, navigate to `http://localhost:5400` in your web browser or use the following PowerShell command:

   ```Powershell
   Invoke-WebRequest -Uri "http://localhost:5400"
   ```

A successful response should look like this:

![Results from the service running](serviceResults.png)

## Troubleshooting
If you encounter difficulties running the service, consult the Windows Event Log for possible errors. It collects detailed information about system events that could be integral in diagnosing the issue.