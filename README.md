# RProvider.SDKProject.Issue

Demonstrates an issue with using the RProvider from SDK-style projects

The contents of this repository (other than this README.md) were created as follows:

1. Create a new SDK-style project
2. Convert it to use net472 instead of the default netcoreapp2.0
3. Make it reference the RProvider
4. Build it

This can be scripted as follows for the sake of reproducibility, though there's nothing here that would be different to doing it manually:

```powershell
# Create a new console project
dotnet new console -lang f#

# Convert the project from netcoreapp2.0 to net472
$projectName = $(Get-Item .).Name
$projectFile = "$projectName.fsproj"
$contents = cat $projectFile
$contents -replace "netcoreapp2.1", "net472" | Set-Content -Encoding utf8 $projectFile

# Set up paket, and use paket to reference the RProvider package
mkdir .paket

[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-WebRequest https://github.com/fsprojects/Paket/releases/download/5.176.6/paket.bootstrapper.exe -OutFile .paket/paket.exe

.paket/paket init
Add-Content -Encoding utf8 paket.dependencies @"
storage: none
framework: net472
"@

.paket/paket convert-from-nuget --force
.paket/paket add RProvider -p $projectFile

# Build
msbuild /t:Restore,Build
```

## Expected

The project builds successfully.

## Actual

The following error is reported:

```powershell
Error FS3053 : The type provider 'RProvider.RProvider' reported an error : The type provider constructor has thrown an exception: Could not load file or assembly 'RDotNet, Version=1.6.5.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The system cannot find the file specified. [D:\git\RProvider. SDKProject.Issue\RProvider.SDKProject.Issue.fsproj]
```

The `rprovider.log` contains the following:

```
[23/08/2018 12:22:08] [Pid:23104, Tid:1, Apid:1] initAndGenerate: starting
[23/08/2018 12:22:08] [Pid:23104, Tid:1, Apid:1] Starting server 'C:\Users\Bill Hally\.nuget\packages\rprovider\1.1.20\lib\net40\RProvider.Server.exe' with arguments 'RInteropServer_23104_700453093_1957470444 "C:\Users\Bill Hally\AppData\Local\Temp\tmp10E5.tmp"' (exists=true)
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] Starting 'RProvider.Server' with arguments '[|"RInteropServer_23104_700453093_1957470444";
  "C:\Users\Bill Hally\AppData\Local\Temp\tmp10E5.tmp"|]'
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] Registering RInteropServer at channel 'RInteropServer_23104_700453093_1957470444'
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] Ready for connections..
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] Waiting for parent process pid=23104 (System.Diagnostics.Process (fsc))
[23/08/2018 12:22:08] [Pid:23104, Tid:1, Apid:1] Attempting to connect via IPC
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] Server started, running event loop
[23/08/2018 12:22:08] [Pid:23104, Tid:1, Apid:1] Got some server
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] server event loop: starting
[23/08/2018 12:22:08] [Pid:26548, Tid:5, Apid:1] Attempting resolution for 'RDotNet, Version=1.6.5.0, Culture=neutral, PublicKeyToken=null'
[23/08/2018 12:22:08] [Pid:26548, Tid:5, Apid:1] Probing locations: C:\Users\Bill Hally\.nuget\packages\rprovider\1.1.20\lib\net40\..\..\..\1.1.20\lib\net40
[23/08/2018 12:22:08] [Pid:26548, Tid:5, Apid:1] Assembly not found!
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] server event loop: failed with System.TypeInitializationException: The type initializer for '<StartupCode$RProvider-Runtime>.$RProvider.Internal.RInit' threw an exception. ---> System.IO.FileNotFoundException: Could not load file or assembly 'RDotNet, Version=1.6.5.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The system cannot find the file specified.
   at <StartupCode$RProvider-Runtime>.$RProvider.Internal.RInit..cctor()
   --- End of inner exception stack trace ---
   at RProvider.Server.EventLoop.startEventLoop() in C:\Tomas\Public\bmc\FSharp.RProvider\src\RProvider\RInteropServer.fs:line 28
[23/08/2018 12:22:08] [Pid:26548, Tid:1, Apid:1] Event loop finished, shutting down
[23/08/2018 12:22:08] [Pid:23104, Tid:1, Apid:1] RProvider constructor failed: System.TypeInitializationException: The type initializer for '<StartupCode$RProvider-Runtime>.$RProvider.Internal.RInit' threw an exception. ---> System.IO.FileNotFoundException: Could not load file or assembly 'RDotNet, Version=1.6.5.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The system cannot find the file specified.
   at <StartupCode$RProvider-Runtime>.$RProvider.Internal.RInit..cctor()
   --- End of inner exception stack trace ---

Server stack trace:
   at RProvider.Server.EventLoop.startEventLoop() in C:\Tomas\Public\bmc\FSharp.RProvider\src\RProvider\RInteropServer.fs:line 28

Exception rethrown at [0]:
   at System.Runtime.Remoting.Proxies.RealProxy.HandleReturnMessage(IMessage reqMsg, IMessage retMsg)
   at System.Runtime.Remoting.Proxies.RealProxy.PrivateInvoke(MessageData& msgData, Int32 type)
   at RProvider.Internal.IRInteropServer.get_InitializationErrorMessage()
   at RProvider.RInteropClient.tryGetInitializationError() in C:\Tomas\Public\bmc\FSharp.RProvider\src\RProvider.DesignTime\RInteropClient.fs:line 118
   at RProvider.RTypeBuilder.initAndGenerate@104.GenerateNext(IEnumerable`1& next) in C:\Tomas\Public\bmc\FSharp.RProvider\src\RProvider.DesignTime\RTypeBuilder.fs:line 107
   at Microsoft.FSharp.Core.CompilerServices.GeneratedSequenceBase`1.MoveNextImpl()
   at Microsoft.FSharp.Core.CompilerServices.GeneratedSequenceBase`1.System-Collections-IEnumerator-MoveNext()
   at Microsoft.FSharp.Collections.SeqModule.ToList[T](IEnumerable`1 source)
   at RProvider.RProvider.buildTypes() in C:\Tomas\Public\bmc\FSharp.RProvider\src\RProvider.DesignTime\RProvider.fs:line 32
```

## Problem diagnosis

The value of `ProbingLocations` is wrong - it doesn't go high enough in the directory hierarchy.

### Solution

Change the value of `ProbingLocations` from:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="ProbingLocations" value="../../../*/lib/net40" />
  </appSettings>
</configuration>
```

to:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="ProbingLocations" value="../../../../*/*/lib/net40" />
  </appSettings>
</configuration>
```

### Improved solution

The above fix won't work with the latest code in the RProvider repository, since that uses a version of DynamicInterop whose
assenblies are in a `netstandard1.2` folder rather than `net40`. So, an improved fix would be:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="ProbingLocations" value="../../../../*/*/lib/net40;../../../../*/*/lib/netstandard1.2" />
  </appSettings>
</configuration>
```
