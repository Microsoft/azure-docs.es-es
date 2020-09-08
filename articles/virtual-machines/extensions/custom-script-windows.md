---
title: Extensión del script personalizado para Windows
description: Automatización de tareas de configuración de máquinas virtuales Windows mediante la extensión de script personalizado
services: virtual-machines-windows
manager: carmonm
author: bobbytreed
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/31/2020
ms.author: robreed
ms.openlocfilehash: 5b7793d70e49b6b760bfba5186b53fb8b31ed16b
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297933"
---
# <a name="custom-script-extension-for-windows"></a>Extensión de la secuencia de comandos personalizada para Windows

La extensión de script personalizado descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión. La extensión de script personalizado se integra con las plantillas de Azure Resource Manager y se puede ejecutar mediante la CLI de Azure, PowerShell, Azure Portal o la API de REST de máquina virtual de Azure.

En este documento se detalla cómo usar la extensión de script personalizado mediante el módulo de Azure PowerShell y plantillas de Azure Resource Manager, y se detallan también los pasos para solucionar problemas en los sistemas Windows.

## <a name="prerequisites"></a>Requisitos previos

> [!NOTE]  
> No use la extensión de script personalizado para ejecutar Update-AzVM con la misma VM como su parámetro, ya tendrá que hacerlo ella misma.  

### <a name="operating-system"></a>Sistema operativo

La extensión de script personalizado para Windows se ejecutará en los sistemas operativos admitidos de la extensión.
### <a name="windows"></a>Windows

* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2
* Windows 10
* Windows Server 2016
* Windows Server 2016 Core
* Windows Server 2019
* Windows Server 2019 Core

### <a name="script-location"></a>Ubicación del script

Puede configurar la extensión para usar las credenciales de Azure Blob Storage a fin de acceder a Azure Blob Storage. La ubicación del script puede ser cualquier lugar, siempre y cuando la máquina virtual pueda enrutarse a ese punto final, como GitHub o un servidor de archivos interno.

### <a name="internet-connectivity"></a>Conectividad de Internet

Si necesita descargar un script externamente, por ejemplo desde GitHub o Azure Storage, deben abrirse puertos adicionales de firewall y grupo de seguridad de red. Por ejemplo, si el script se encuentra en Azure Storage, puede permitir el acceso mediante las etiquetas del servicio NSG de Azure para [Storage](../../virtual-network/security-overview.md#service-tags).

Si el script se encuentra en un servidor local, puede que aún necesite abrir puertos adicionales de firewall y grupo de seguridad de red.

### <a name="tips-and-tricks"></a>Trucos y sugerencias

* La mayor tasa de errores de esta extensión se debe a errores de sintaxis en el script. Compruebe que el script se ejecuta sin errores y además establezca registros adicionales en él para facilitar la búsqueda de dónde se ha producido el error.
* Escriba scripts que sean idempotentes. Esto garantiza que si se ejecutan de nuevo accidentalmente, no van a provocar cambios en el sistema.
* Asegúrese de que los scripts no requieran intervención del usuario cuando se ejecutan.
* Los scripts tienen permitido un plazo de 90 minutos para ejecutarse; todo lo que dure más provocará un error de aprovisionamiento de la extensión.
* No incluya reinicios en el script, ya que esta acción provocará errores con otras extensiones que se instalen. Después del reinicio, la extensión no continuará después del reinicio.
* Si tiene un script que provoca un reinicio, instala aplicaciones y ejecuta scripts, puede programar el reinicio con una tarea programada de Windows o usar herramientas como las extensiones DSC, Chef o Puppet.
* La extensión solo ejecutará un script una vez. Si quiere ejecutar un script en cada inicio, debe usar la extensión para crear una tarea programada de Windows.
* Si quiere programar cuándo se ejecutará un script, debe usar la extensión para crear una tarea programada de Windows.
* Cuando el script se esté ejecutando, solo verá un estado de extensión "en transición" desde Azure Portal o la CLI. Si quiere recibir actualizaciones de estado más frecuentes de un script en ejecución, debe crear su propia solución.
* La extensión de script personalizada no admite de forma nativa servidores proxy, pero puede usar una herramienta de transferencia de archivos que admita servidores proxy en el script, como *Curl*.
* Tenga en cuenta las ubicaciones de directorio no predeterminadas en las que se puedan basar los scripts o comandos y aplique lógica para controlar esta situación.
* La extensión de script personalizado se ejecutará con la cuenta LocalSystem
* Si planea usar las propiedades *storageAccountName* y *storageAccountKey*, deben colocarse en *protectedSettings*.

## <a name="extension-schema"></a>Esquema de extensión

La configuración de la extensión de script personalizado especifica aspectos como la ubicación del script y el comando que se ejecutará. Esta configuración se puede almacenar en archivos de configuración, o se puede especificar en la línea de comandos o en una plantilla de Azure Resource Manager.

Los datos confidenciales se pueden almacenar en una configuración protegida, que se cifra y se descifra solo dentro de la máquina virtual. La configuración protegida es útil cuando el comando de ejecución incluye secretos tales como una contraseña.

Estos elementos se deben tratar como datos confidenciales y se deben especificar en la configuración protegida de las extensiones. Los datos de configuración protegida de la extensión de VM de Azure están cifrados y solo se descifran en la máquina virtual de destino.

```json
{
    "apiVersion": "2018-06-01",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "virtualMachineName/config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'),copyindex())]",
        "[variables('musicstoresqlName')]"
    ],
    "tags": {
        "displayName": "config-app"
    },
    "properties": {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.10",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "fileUris": [
                "script location"
            ],
            "timestamp":123456789
        },
        "protectedSettings": {
            "commandToExecute": "myExecutionCommand",
            "storageAccountName": "myStorageAccountName",
            "storageAccountKey": "myStorageAccountKey",
            "managedIdentity" : {}
        }
    }
}
```

> [!NOTE]
> La propiedad managedIdentity **no debe** usarse junto con las propiedades storageAccountName o storageAccountKey

> [!NOTE]
> Solo se puede instalar una versión de una extensión en una máquina virtual en un momento dado. Si se especifica el script personalizado dos veces en la misma plantilla de Resource Manager para la misma máquina virtual, se produce un error.

> [!NOTE]
> Este esquema se puede usar dentro del recurso VirtualMachine o como un recurso independiente. El nombre del recurso debe tener el formato "virtualMachineName/extensionName" si se usa esta extensión como recurso independiente en la plantilla de ARM.

### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.Compute | string |
| type | CustomScriptExtension | string |
| typeHandlerVersion | 1.10 | int |
| fileUris (p. ej.) | https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1 | array |
| timestamp (p. ej.) | 123456789 | Entero de 32 bits |
| commandToExecute (p. ej.) | powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 | string |
| storageAccountName (p. ej.) | examplestorageacct | string |
| storageAccountKey (p. ej.) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | string |
| managedIdentity (p. ej.) | { } o {"clientId": "31b403aa-C364-4240-a7ff-d85fb6cd7232"} o {"objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" } | json object |

>[!NOTE]
>Los nombres de propiedad distinguen entre mayúsculas y minúsculas. Para evitar problemas de implementación, use los nombres como se muestran aquí.

#### <a name="property-value-details"></a>Detalles del valor de propiedad

* `commandToExecute` (**necesario**, cadena): script de punto de entrada que se va a ejecutar. Use este campo si el comando contiene secretos, como contraseñas, o si los fileUris son confidenciales.
* `fileUris` (opcional, matriz de cadenas): direcciones URL de los archivos que se van a descargar.
* `timestamp` (opcional, entero de 32 bits) use este campo solo para desencadenar una nueva ejecución del script; para ello, cambie el valor de este campo.  Se acepta cualquier valor entero; solo debe ser distinto del valor anterior.
* `storageAccountName` (opcional, cadena): nombre de la cuenta de almacenamiento. Si especifica credenciales de almacenamiento, todos los valores de `fileUris` deben ser direcciones URL de blobs de Azure.
* `storageAccountKey` (opcional, cadena): clave de acceso de la cuenta de almacenamiento.
* `managedIdentity`: (opcional, objeto JSON) la [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) para descargar archivos.
  * `clientId`: (opcional, cadena) el id. de cliente de la identidad administrada.
  * `objectId`: (opcional, cadena) el id. de objeto de la identidad administrada.

Los valores siguientes se pueden establecer en la configuración pública o protegida. La extensión rechazará una configuración si los valores siguientes están establecidos en la configuración tanto pública como protegida.

* `commandToExecute`

El empleo de la configuración pública puede resultar útil para la depuración, pero se recomienda usar la configuración protegida.

La configuración pública se envía en texto no cifrado a la máquina virtual donde se ejecutará el script.  La configuración protegida se cifra con una clave que solo conocen Azure y la máquina virtual. La configuración se guarda en la máquina virtual tal cual se ha enviado, es decir, si la configuración estaba cifrada, se guarda cifrada en la máquina virtual. El certificado usado para descifrar los valores cifrados se almacena en la máquina virtual y se usa para descifrar la configuración (si es necesario) en tiempo de ejecución.

####  <a name="property-managedidentity"></a>Propiedad: managedIdentity
> [!NOTE]
> Esta propiedad **debe** especificarse solo en la configuración protegida.

CustomScript (versión 1.10 en adelante) admite la [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) para descargar archivos de las direcciones URL proporcionadas en el valor "fileUris". Permite a CustomScript acceder a blobs o contenedores privados de Azure Storage sin que el usuario tenga que enviar secretos como tokens de SAS o claves de cuenta de almacenamiento.

Para usar esta característica, el usuario debe agregar una identidad [asignada por el sistema](../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity) o[asignada por el usuario](../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-user-assigned-identity) a la máquina virtual o VMSS donde se espera que se ejecute CustomScript y [conceder a la identidad administrada acceso al contenedor de Azure Storage o al blob](../../active-directory/managed-identities-azure-resources/tutorial-vm-windows-access-storage.md#grant-access).

Para usar la identidad asignada por el sistema en la máquina virtual/VMSS de destino, establezca el campo "managedidentity" a un objeto JSON vacío. 

> Ejemplo:
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : {}
> }
> ```

Para usar la identidad asignada por el usuario en la máquina virtual/VMSS de destino, configure el campo "managedidentity" con el identificador de cliente o el identificador de objeto de la identidad administrada.

> Ejemplos:
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : { "clientId": "31b403aa-c364-4240-a7ff-d85fb6cd7232" }
> }
> ```
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.ps1"],
>   "commandToExecute": "powershell.exe script1.ps1",
>   "managedIdentity" : { "objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" }
> }
> ```

> [!NOTE]
> La propiedad managedIdentity **no debe** usarse junto con las propiedades storageAccountName o storageAccountKey

## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. El esquema JSON detallado en la sección anterior se puede usar en una plantilla de Azure Resource Manager para ejecutar la extensión de script personalizado durante la implementación. En los ejemplos siguientes se muestra cómo usar la extensión de script personalizado:

* [Tutorial: Implementación de extensiones de máquina virtual con plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-tutorial-deploy-vm-extensions.md)
* [Deploy Two Tier Application on Windows and Azure SQL DB](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows) (Implementación de aplicaciones de dos niveles en Windows y Azure SQL DB)

## <a name="powershell-deployment"></a>Implementación de PowerShell

El comando `Set-AzVMCustomScriptExtension` se puede usar para agregar la extensión de script personalizado a una máquina virtual existente. Para más información, consulte [Set-AzVMCustomScriptExtension](/powershell/module/az.compute/set-azvmcustomscriptextension).

```powershell
Set-AzVMCustomScriptExtension -ResourceGroupName <resourceGroupName> `
    -VMName <vmName> `
    -Location myLocation `
    -FileUri <fileUrl> `
    -Run 'myScript.ps1' `
    -Name DemoScriptExtension
```

## <a name="additional-examples"></a>Ejemplos adicionales

### <a name="using-multiple-scripts"></a>Empleo de varios scripts

En este ejemplo hay tres scripts que se usan para compilar el servidor. **commandToExecute** llama al primer script y luego se dispone de opciones sobre cómo llamar a los demás. Por ejemplo, puede tener un script maestro que controla la ejecución con el control de errores, el registro y la administración de estado adecuados. Los scripts se descargan en el equipo local para su ejecución. Por ejemplo, en `1_Add_Tools.ps1` se llamaría a `2_Add_Features.ps1` al agregar `.\2_Add_Features.ps1` al script y se repetiría este proceso para los demás scripts definidos en `$settings`.

```powershell
$fileUri = @("https://xxxxxxx.blob.core.windows.net/buildServer1/1_Add_Tools.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/2_Add_Features.ps1",
"https://xxxxxxx.blob.core.windows.net/buildServer1/3_CompleteInstall.ps1")

$settings = @{"fileUris" = $fileUri};

$storageAcctName = "xxxxxxx"
$storageKey = "1234ABCD"
$protectedSettings = @{"storageAccountName" = $storageAcctName; "storageAccountKey" = $storageKey; "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File 1_Add_Tools.ps1"};

#run command
Set-AzVMExtension -ResourceGroupName <resourceGroupName> `
    -Location <locationName> `
    -VMName <vmName> `
    -Name "buildserver1" `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -Settings $settings    `
    -ProtectedSettings $protectedSettings `
```

### <a name="running-scripts-from-a-local-share"></a>Ejecutar scripts desde un recurso compartido local

En este ejemplo es posible que quiera usar un servidor local de SMB para la ubicación del script. Así, no es necesario proporcionar ningún otro valor de configuración, excepto **commandToExecute**.

```powershell
$protectedSettings = @{"commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File \\filesvr\build\serverUpdate1.ps1"};

Set-AzVMExtension -ResourceGroupName <resourceGroupName> `
    -Location <locationName> `
    -VMName <vmName> `
    -Name "serverUpdate"
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -ProtectedSettings $protectedSettings

```

### <a name="how-to-run-custom-script-more-than-once-with-cli"></a>Cómo ejecutar un script personalizado más de una vez con la CLI

Si quiere ejecutar la extensión de script personalizado más de una vez, solo puede hacer esta acción si se cumplen las condiciones siguientes:

* El parámetro **Name** de la extensión coincide con la implementación anterior de la extensión.
* Actualice la configuración, ya que de lo contrario el comando no vuelve a ejecutarse. Puede agregar una propiedad dinámica en el comando, como una marca de tiempo.

También puede establecer la propiedad [ForceUpdateTag](/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension.forceupdatetag) en **true**.

### <a name="using-invoke-webrequest"></a>Empleo de Invoke-WebRequest

Si usa [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest) en el script, debe especificar el parámetro `-UseBasicParsing`, ya que de lo contrario aparece el siguiente error al comprobar el estado detallado:

```error
The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
```
## <a name="virtual-machine-scale-sets"></a>Virtual Machine Scale Sets

Para implementar la extensión de script personalizado en un conjunto de escalado, consulte [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension?view=azps-3.3.0).

## <a name="classic-vms"></a>Máquinas virtuales clásicas

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

Para implementar la extensión de script personalizado en máquinas virtuales clásicas, puede usar Azure Portal o los cmdlets clásicos de Azure PowerShell.

### <a name="azure-portal"></a>Azure portal

Vaya al recurso de máquina virtual clásica. Seleccione **Extensiones** en **Configuración**.

Haga clic en **+ Agregar** y, en la lista de recursos, seleccione **Extensión de script personalizado**.

En la página **Instalar extensión**, seleccione el archivo de PowerShell local, rellene los argumentos y haga clic en **Aceptar**.

### <a name="powershell"></a>PowerShell

Use el cmdlet [Set-AzureVMCustomScriptExtension](/powershell/module/servicemanagement/azure.service/set-azurevmcustomscriptextension) para agregar la extensión de script personalizado a una máquina virtual existente.

```powershell
# define your file URI
$fileUri = 'https://xxxxxxx.blob.core.windows.net/scripts/Create-File.ps1'

# create vm object
$vm = Get-AzureVM -Name <vmName> -ServiceName <cloudServiceName>

# set extension
Set-AzureVMCustomScriptExtension -VM $vm -FileUri $fileUri -Run 'Create-File.ps1'

# update vm
$vm | Update-AzureVM
```

## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante el módulo Azure PowerShell. Para ver el estado de implementación de las extensiones de una máquina virtual determinada, ejecute el comando siguiente:

```powershell
Get-AzVMExtension -ResourceGroupName <resourceGroupName> -VMName <vmName> -Name myExtensionName
```

La salida de la extensión se registra en archivos que se encuentran en la siguiente carpeta de la máquina virtual de destino.

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension
```

Los archivos especificados se descargan en la siguiente carpeta de la máquina virtual de destino.

```cmd
C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads\<n>
```

donde `<n>` es un entero decimal que puede variar entre las ejecuciones de la extensión.  El valor `1.*` coincide con el valor `typeHandlerVersion` actual y real de la extensión.  Por ejemplo, el directorio real podría ser `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2`.  

Cuando se ejecute el comando `commandToExecute`, la extensión establece este directorio (por ejemplo, `...\Downloads\2`) como directorio de trabajo actual. Este proceso permite el uso de rutas de acceso relativas para buscar los archivos descargados mediante la propiedad `fileURIs`. En la tabla siguiente se muestran algunos ejemplos.

Dado que la ruta de acceso absoluta de descarga puede variar con el paso del tiempo, es mejor optar por rutas de acceso relativas de archivo o script en la cadena `commandToExecute`, siempre que sea posible. Por ejemplo:

```json
"commandToExecute": "powershell.exe . . . -File \"./scripts/myscript.ps1\""
```

Se conserva la información de la ruta de acceso tras el primer segmento de URI de los archivos descargados a través de la lista de propiedades `fileUris`.  Como se muestra en la tabla siguiente, los archivos descargados se asignan en los subdirectorios de descarga para reflejar la estructura de los valores `fileUris`.  

#### <a name="examples-of-downloaded-files"></a>Ejemplos de archivos descargados

| URI en fileUris | Ubicación descargada relativa | Ubicación descargada absoluta<sup>1</sup> |
| ---- | ------- |:--- |
| `https://someAcct.blob.core.windows.net/aContainer/scripts/myscript.ps1` | `./scripts/myscript.ps1` |`C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\scripts\myscript.ps1`  |
| `https://someAcct.blob.core.windows.net/aContainer/topLevel.ps1` | `./topLevel.ps1` | `C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.8\Downloads\2\topLevel.ps1` |

<sup>1</sup> Las rutas de acceso absolutas del directorio cambian a lo largo del ciclo de vida de la máquina virtual, pero no durante una ejecución única de la extensión de script personalizado.

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/forums/). También puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione Obtener soporte. Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).
