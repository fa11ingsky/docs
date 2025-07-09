
# Dot NET Reflection


We can dynamically load arbitrary .NET programs through Reflection. The program must export public class and public static functions like the following:

```csharp
using System;
namespace Agent
{
    public class Program
    {
        public static void Main(string[] args)
        {
            Console.WriteLine("[+] Code execution !");
        }
    }
}
```

Compile and generate `.exe` with `csc.exe`

```bash
csc.exe program.cs
```

In powershell, running this program in memory:
```powershell
$path = "/path/to/file.exe"
[System.Reflection.Assembly]::Load([System.IO.File]::ReadAllBytes($path))
[Agent.Program]::Main("")

> [+] Code execution !
```

