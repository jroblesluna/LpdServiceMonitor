# LpdServiceMonitor

Servicio de Windows que monitorea el estado de **LPDSVC** (Servicio LPD de Windows).  
Si detecta que está detenido, intenta levantarlo de forma automática, aplicando reglas de ventana de reintentos, cooldown y un flag de mantenimiento.

Windows Service that monitors the state of **LPDSVC** (Windows LPD Service).  
If it detects that the service is stopped, it will try to restart it automatically, applying retry window rules, cooldown, and a maintenance flag.

---

## 📂 Estructura / Structure

LpdServiceMonitor/  
 ├─ src/                      # Código fuente / Source code (C# .NET 8 Worker Service)  
 │   ├─ Program.cs  
 │   ├─ MonitorService.csproj  
 │   └─ appsettings.json  
 ├─ dist/                     # Binarios publicados / Published binaries (`dotnet publish`)  
 ├─ installer/                # Archivos WiX / WiX files for MSI packaging  
 │   └─ Product.wxs  
 └─ README.md  

---

## ⚙️ Configuración / Configuration

Archivo `appsettings.json` / The `appsettings.json` file:

```json
{
  "Monitor": {
    "TargetServiceName": "LPDSVC",
    "CheckIntervalMs": 5000,
    "StartTimeoutSeconds": 30,
    "MaxRestartsInWindow": 5,
    "RestartWindowSeconds": 300,
    "CooldownAfterBurstSeconds": 600,
    "MaintenanceFlagPath": "C:\\windows\\temp\\LPDSVCMONITOR.MAINTENANCE.flag"
  }
}
```

- **TargetServiceName**: Nombre interno / Internal service name (LPDSVC).  
- **CheckIntervalMs**: Intervalo de chequeo / Check interval (ms).  
- **StartTimeoutSeconds**: Tiempo de espera máximo / Max wait time (s).  
- **MaxRestartsInWindow** y **RestartWindowSeconds**: Control de ráfagas / Burst restart control.  
- **CooldownAfterBurstSeconds**: Tiempo de enfriamiento / Cooldown after burst.  
- **MaintenanceFlagPath**: Si existe, no reinicia / If file exists, service won’t restart.  

---

## 🚀 Publicación del ejecutable / Publish the executable

```powershell
dotnet publish .\LpdServiceMonitor.csproj `
  -c Release -r win-x64 `
  -p:PublishSingleFile=true `
  -p:SelfContained=true `
  -o .\dist
```

---

## 📦 Empaquetado MSI con WiX / MSI packaging with WiX

1. Asegúrate que los binarios estén en `dist/`.  
   Make sure binaries are in `dist/`.  

2. Construye el MSI con WiX (requiere la extensión `Util`):  
   Build the MSI with WiX (requires `Util` extension):  

```powershell
wix build .\installer\Product.wxs `
  -ext WixToolset.Util.wixext `
  -o .\installer\LpdServiceMonitor.msi
```

---

## 🖥️ Instalación / Installation

```powershell
msiexec /i .\installer\LpdServiceMonitor.msi /L*v install.log
```

Esto copia los binarios en:  
This copies binaries to:  
`C:\Program Files\RoblesAI\LPD Service Monitor`

Servicio creado / Service created:  
```powershell
Get-Service LPDSVC-Monitor
```

---

## 🛑 Desinstalación / Uninstall

```powershell
msiexec /x .\installer\LpdServiceMonitor.msi
```

---

## 📝 Notas / Notes

- El servicio corre como **LocalSystem** / Service runs as **LocalSystem**.  
- Los eventos se registran en el **Visor de eventos → Application → Source: ServiceMonitor**.  
  Events are logged under **Event Viewer → Application → Source: ServiceMonitor**.  
- Crear `C:\windows\temp\LPDSVCMONITOR.MAINTENANCE.flag` pausa la acción de reinicio.  
  Creating that file will pause restart actions.  
- Puedes usar **variables de entorno** prefijo `SM_` para overrides.  
  You can override config using **environment variables** prefixed with `SM_`.  

---

## 📌 Requisitos / Requirements

- Windows 10/11 o Windows Server con soporte .NET 8.  
  Windows 10/11 or Windows Server with .NET 8 support.  
- [.NET 8 Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) (si no es self-contained).  
  [.NET 8 Runtime](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) (if not self-contained).  
- [WiX Toolset 6](https://wixtoolset.org/releases/) para generar el MSI.  
  [WiX Toolset 6](https://wixtoolset.org/releases/) to build the MSI.  

---