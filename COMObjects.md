---
description: Enumeration techniques using Microsoft COM objects
icon: computer
---

# COM Objects

Using powershell, list the available COM objects:

```powershell
gwmi Win32_COMSetting | ? {$_.progid} |  ft ProgId,Caption,InprocServer32
```

Looking for ones that are not exposed by core Microsoft services, filtering out the DLL's that are located in the `C:\windows\system32` directory:

```powershell
gwmi Win32_COMSetting | ? {$_.progid} |  ft ProgId,Caption,InprocServer32 | findstr /iv system32
```

Interact with a COM object by: 
```powershell
$o = [activator]::CreateInstance([type]::GetTypeFromProgID(("Shell.Application.1")))
```

Viewing the methods it exposes by:
```powershell
$o | gm 

   TypeName: System.__ComObject#{286e6f1b-7113-4355-9562-96b7e9d64c54}

Name                 MemberType Definition
----                 ---------- ----------
AddToRecent          Method     void AddToRecent (Variant, string)
BrowseForFolder      Method     Folder BrowseForFolder (int, string, int, Variant)
CanStartStopService  Method     Variant CanStartStopService (string)
CascadeWindows       Method     void CascadeWindows ()
ControlPanelItem     Method     void ControlPanelItem (string)
EjectPC              Method     void EjectPC ()
Explore              Method     void Explore (Variant)
ExplorerPolicy       Method     Variant ExplorerPolicy (string)
FileRun              Method     void FileRun ()
FindComputer         Method     void FindComputer ()
....

```

And calling methods:
```powershell
$o.ShellExecute('netstat')
```

