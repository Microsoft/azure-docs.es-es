---
title: Ejecución de la versión 1 de Extensión de script personalizado (en desuso) en máquinas virtuales Linux en Azure
description: Automatización de tareas de configuración de máquinas virtuales Linux mediante la extensión de script personalizado v1
services: virtual-machines-linux
documentationcenter: ''
author: danielsollondon
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/14/2018
ms.author: danis
ms.openlocfilehash: 1fe915fd58f60e4ad5b1e28b51911678ef2f866c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085714"
---
# <a name="use-the-azure-custom-script-extension-version-1-with-linux-virtual-machines"></a>Uso de la extensión de script personalizado de Azure versión 1 con máquinas virtuales Linux

[!INCLUDE [virtual-machines-extensions-deprecation-statement](../../../includes/virtual-machines-extensions-deprecation-statement.md)]

La extensión de script personalizado versión 1 descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage u otra ubicación de Internet accesible, o se pueden proporcionar al tiempo de ejecución de la extensión.

La extensión de script personalizado se integra con las plantillas de Azure Resource Manager. También puede ejecutarla mediante la CLI de Azure, PowerShell, Azure Portal o la API de REST de Azure Virtual Machines.

En este artículo se detalla cómo utilizar la extensión de script personalizado desde la CLI de Azure, y cómo ejecutar la extensión mediante una plantilla de Azure Resource Manager. En este artículo se proporcionan también los pasos para la solución de problemas para los sistemas Linux.

Hay dos extensiones de script personalizado para Linux:

* Versión 1: Microsoft.OSTCExtensions.CustomScriptForLinux

* Versión 2: Microsoft.Azure.Extensions.CustomScript

Cambie las implementaciones nuevas y existentes para utilizar la nueva versión ([Microsoft.Azure.Extensions.CustomScript](custom-script-linux.md)) en su lugar. La nueva versión está diseñada para que sea un sustituto directo. Por lo tanto, la migración es tan sencilla como cambiar el nombre y la versión; no necesita cambiar la configuración de la extensión.

### <a name="operating-system"></a>Sistema operativo

Distribuciones de Linux compatibles:

* CentOS 6.5 y posterior
* Debian 8 y posterior
  * Debian 8.7 no se distribuye con Python2 en las imágenes más recientes, lo que interrumpe CustomScriptForLinux.
* FreeBSD
* OpenSUSE 13.1 y posterior
* Oracle Linux 6.4 y posterior
* SUSE Linux Enterprise Server 11 SP3 y posterior
* Ubuntu 12.04 y posterior

### <a name="script-location"></a>Ubicación del script

Puede emplear la extensión para usar las credenciales de Azure Blob Storage, a fin de tener acceso Azure Blob Storage. Como alternativa, la ubicación del script puede ser cualquier lugar, siempre y cuando la máquina virtual pueda enrutarse a ese punto de conexión, como GitHub, un servidor de archivos internos, etc.

### <a name="internet-connectivity"></a>Conectividad de Internet

Si necesita descargar un script externamente, como GitHub o Azure Storage, deben abrirse puertos adicionales de firewall/grupo de seguridad de red. Por ejemplo, si el script se encuentra en Azure Storage, puede permitir el acceso mediante las etiquetas del servicio NSG de Azure para [Storage](../../virtual-network/security-overview.md#service-tags).

Si el script se encuentra en un servidor local, puede que aún necesite que haya abiertos puertos adicionales de firewall/grupo de seguridad de red.

### <a name="tips-and-tricks"></a>Trucos y sugerencias

* La mayor tasa de errores para esta extensión se debe a errores de sintaxis en el script. Compruebe que el script se ejecute sin errores y también establezca registros adicionales en el script para facilitar la búsqueda de dónde se produjo el error.
* Escriba scripts que sean idempotentes, para que, si se ejecutan más de una vez por accidente, no provoquen cambios en el sistema.
* Asegúrese de que los scripts no requieran intervención del usuario cuando se ejecutan.
* Los scripts tienen permitido un plazo de 90 minutos para ejecutarse; todo lo que dure más provocará un error de aprovisionamiento de la extensión.
* No coloque reinicios dentro del script, ya que esto provocará problemas con otras extensiones que se estén instalando y, tras reiniciar el equipo, la extensión no continuará. 
* Si tiene un script que provocará un reinicio, instale las aplicaciones y ejecute los scripts, etc. Debe programar el reinicio de un trabajo de Cron o usar herramientas como DSC, o extensiones de Chef o Puppet.
* La extensión solo ejecutará un script una vez, si desea ejecutar un script en cada inicio, puede usar [cloud-init image](../linux/using-cloud-init.md) y usar un módulo [Scripts Per Boot](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#scripts-per-boot). Como alternativa, puede usar el script para crear una unidad de servicio de Systemd.
* Si desea programar cuándo se ejecutará un script, debe utilizar la extensión para crear un trabajo de Cron.
* Cuando el script se esté ejecutando, solo verá un estado de extensión "en transición" desde Azure Portal o la CLI. Si quiere recibir actualizaciones de estado más frecuentes de un script en ejecución, debe crear su propia solución.
* La extensión de script personalizada no admite de forma nativa servidores proxy, pero puede usar una herramienta de transferencia de archivos que admita servidores proxy en el script, como *Curl*.
* Tenga en cuenta las ubicaciones de directorio no predeterminadas en las que se puedan basar los scripts o comandos y aplique lógica para controlarlas.

## <a name="extension-schema"></a>Esquema de extensión

La configuración de la extensión de script personalizado especifica aspectos como la ubicación del script y el comando que se ejecutará. Esta configuración se puede almacenar en archivos de configuración, o se puede especificar en la línea de comandos o en una plantilla de Azure Resource Manager. 

Los datos confidenciales se pueden almacenar en una configuración protegida, que se cifra y se descifra solo dentro de la máquina virtual. La configuración protegida es útil cuando el comando de ejecución incluye secretos tales como una contraseña.

Estos elementos se deben tratar como datos confidenciales y se deben especificar en la configuración protegida de las extensiones. Los datos de configuración protegida de la extensión de VM de Azure están cifrados y solo se descifran en la máquina virtual de destino.

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.OSTCExtensions",
    "type": "CustomScriptForLinux",
    "typeHandlerVersion": "1.5",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "<url>"
      ],
      "enableInternalDNSCheck": true
    },
    "protectedSettings": {
      "storageAccountName": "<storage-account-name>",
      "storageAccountKey": "<storage-account-key>",
      "commandToExecute": "<command>"
    }
  }
}
```

### <a name="property-values"></a>Valores de propiedad

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.OSTCExtensions | string |
| type | CustomScriptForLinux | string |
| typeHandlerVersion | 1.5 | int |
| fileUris (p. ej.) | `https://github.com/MyProject/Archive/MyPythonScript.py` | array |
| commandToExecute (p. ej.) | python MyPythonScript.py \<my-param1\> | string |
| enableInternalDNSCheck | true | boolean |
| storageAccountName (p. ej.) | examplestorageacct | string |
| storageAccountKey (p. ej.) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | string |

### <a name="property-value-details"></a>Detalles del valor de propiedad

* `fileUris`: (opcional, matriz de cadenas) la lista de URI de los scripts
* `enableInternalDNSCheck`: (opcional, bool) el valor predeterminado es True; se establece en False para deshabilitar la comprobación de DNS.
* `commandToExecute`: (opcional, cadena) script de punto de entrada que se va a ejecutar
* `storageAccountName`: (opcional, cadena) nombre de la cuenta de almacenamiento
* `storageAccountKey` (opcional, cadena): clave de acceso de la cuenta de almacenamiento.

Los valores siguientes se pueden establecer en la configuración pública o protegida; no establezca estos valores en la configuración tanto pública como protegida.

* `commandToExecute`

Usar la configuración pública puede resultar útil para la depuración, pero se recomienda encarecidamente usar la configuración protegida.

La configuración pública se envía en texto no cifrado a la máquina virtual donde se ejecutará el script.  La configuración protegida se cifra con una clave que solo conocen Azure y la máquina virtual. La configuración se guarda en la máquina virtual tal cual se envió; es decir, si la configuración estaba cifrada, se guarda cifrada en la máquina virtual. El certificado usado para descifrar los valores cifrados se almacena en la máquina virtual y se usa para descifrar la configuración (si es necesario) en tiempo de ejecución.

## <a name="template-deployment"></a>Implementación de plantilla

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. El esquema JSON detallado en la sección anterior se puede usar en una plantilla de Azure Resource Manager para ejecutar la extensión de script personalizado durante la implementación de una plantilla de Azure Resource Manager.

```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.OSTCExtensions",
    "type": "CustomScriptForLinux",
    "typeHandlerVersion": "1.5",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": ["http://MyAccount.blob.core.windows.net/vhds/MyShellScript.sh"]
    },
    "protectedSettings": {
      "storageAccountName": "MyAccount",
      "storageAccountKey": "<storage-account-key>",
      "commandToExecute": "sh MyShellScript.sh"
    }
  }
}
```

>[!NOTE]
>Los nombres de propiedad distinguen entre mayúsculas y minúsculas. Para evitar problemas de implementación, use los nombres como se muestran aquí.

## <a name="azure-cli"></a>Azure CLI

Cuando se usa la CLI de Azure para ejecutar la extensión de script personalizado, cree un archivo o archivos de configuración. Como mínimo, debe tener "commandToExecute".

```azurecli
az vm extension set -n VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.5 \
  --vm-name MyVm --resource-group MyResourceGroup \
  --protected-settings '{"commandToExecute": "echo hello"}'
```

Opcionalmente, la configuración puede especificarse en el comando como una cadena con formato JSON. Esto permite especificar la configuración durante la ejecución sin un archivo de configuración independiente.

```azurecli
az vm extension set \
  --resource-group exttest \
  --vm-name exttest \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings '{"fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],"commandToExecute": "./config-music.sh"}'
```

### <a name="azure-cli-examples"></a>Ejemplos de la CLI de Azure

#### <a name="public-configuration-with-no-script-file"></a>Configuración pública sin archivo de script

```json
{
  "commandToExecute": "apt-get -y update && apt-get install -y apache2"
}
```

Comando de la CLI de Azure:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings ./script-config.json
```

#### <a name="public-and-protected-configuration-files"></a>Archivos de configuración pública y protegida

Use un archivo de configuración pública para especificar el URI del archivo de script. Use un archivo de configuración protegida para especificar el comando que se ejecutará.

Archivo de configuración pública:

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"]
}
```

Archivo de configuración protegida:  

```json
{
  "commandToExecute": "./config-music.sh <param1>"
}
```

Comando de la CLI de Azure:

```azurecli
az vm extension set
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name CustomScriptForLinux \
  --publisher Microsoft.OSTCExtensions \
  --settings ./script-config.json \
  --protected-settings ./protected-config.json
```

## <a name="troubleshooting"></a>Solución de problemas

Cuando la extensión de script personalizado se ejecuta, el script se crea o se descarga en un directorio similar al del ejemplo siguiente. La salida del comando se guarda también en este directorio, en los archivos `stdout` y `stderr`.

```bash
/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-<version>/download/1
```

Para solucionar problemas, primero compruebe el registro del agente de Linux, asegúrese de que se ejecutó la extensión, consulte:

```bash
/var/log/waagent.log
```

Debe buscar la ejecución de la extensión, tendrá un aspecto parecido al siguiente:

```output
2018/04/26 15:29:44.835067 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Target handler state: enabled
2018/04/26 15:29:44.867625 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] [Enable] current handler state is: notinstalled
2018/04/26 15:29:44.959605 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Download, message=Download succeeded, duration=59
2018/04/26 15:29:44.993269 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Initialize extension directory
2018/04/26 15:29:45.022972 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Update settings file: 0.settings
2018/04/26 15:29:45.051763 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Install extension [customscript.py -install]
2018/04/26 15:29:45 CustomScriptForLinux started to handle.
2018/04/26 15:29:45 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] cwd is /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2
2018/04/26 15:29:45 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Change log file to /var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5.2.2/extension.log
2018/04/26 15:29:46.088212 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Install, message=Launch command succeeded: customscript.py -install, duration=1005
2018/04/26 15:29:46.133367 INFO [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Enable extension [customscript.py -enable]
2018/04/26 15:29:46 CustomScriptForLinux started to handle.
..
2018/04/26 15:29:47.178163 INFO Event: name=Microsoft.OSTCExtensions.CustomScriptForLinux, op=Enable, message=Launch command succeeded: customscript.py -enable, duration=1012
```

Algunos puntos a tener en cuenta:

1. Enable es cuando el comando empieza a ejecutarse.
1. Download se relaciona con la descarga del paquete de extensiones de CustomScript de Azure, no los archivos de script especificados en fileUris.
1. También puede ver el archivo de registro donde se escribe: `/var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5.2.2/extension.log`

El paso siguiente consiste en comprobar el archivo de registro. Este es el formato:

```bash
/var/log/azure/<extension-name>/<version>/extension.log file.
```

Debe buscar la ejecución individual, que tendrá un aspecto parecido al siguiente:

```output
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Enable,transitioning,0,Launching the script...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] sequence number is 0
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] setting file path is/var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/config/0.settings
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] JSON config: {"runtimeSettings": [{"handlerSettings": {"protectedSettings": "MIIB0AYJKoZIhvcNAQcDoIIBwTCCAb0CAQAxggF+hnEXRtFKTTuKiFC8gTfHKupUSs7qI0zFYRya", "publicSettings": {"fileUris": ["https://dannytesting.blob.core.windows.net/demo/myBash.sh"]}, "protectedSettingsCertThumbprint": "4385AB21617C2452FF6998C0A37F71A0A01C8368"}}]}
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Config decoded correctly.
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Will try to download files, number of retries = 10, wait SECONDS between retrievals = 20s
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Downloading,transitioning,0,Downloading files...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] No azure storage account and key specified in protected settings. Downloading scripts from external links...
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Converting /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/download/0/myBash.sh from DOS to Unix formats: Done
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Removing BOM of /var/lib/waagent/Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2/download/0/myBash.sh: Done
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Succeeded to download files, retry count = 0
2018/04/26 15:29:46 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Internal DNS is ready, retry count = 0
2018/04/26 15:29:47 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Command is finished.
2018/04/26 15:29:47 ---stdout---
2018/04/26 15:29:47
2018/04/26 15:29:47 ---errout---
2018/04/26 15:29:47
2018/04/26 15:29:47
2018/04/26 15:29:47 [Microsoft.OSTCExtensions.CustomScriptForLinux-1.5.2.2] Daemon,success,0,Command is finished.
2018/04/26 15:29:47 ---stdout---
2018/04/26 15:29:47
2018/04/26 15:29:47 ---errout---
2018/04/26 15:29:47
2018/04/26 15:29:47
```

Aquí puede ver:

* El comando Enable que se inicia es este registro.
* La configuración que se pasó a la extensión.
* El archivo de descarga de la extensión y el resultado.
* El comando que se ejecuta y el resultado.

El estado de ejecución de la extensión de script personalizado también se puede recuperar con la CLI de Azure:

```azurecli
az vm extension list -g myResourceGroup --vm-name myVM
```

La salida tendrá un aspecto similar al siguiente:

```output
Name                  ProvisioningState    Publisher                   Version  AutoUpgradeMinorVersion
--------------------  -------------------  ------------------------  ---------  -------------------------
CustomScriptForLinux  Succeeded            Microsoft.OSTCExtensions        1.5  True
```

## <a name="next-steps"></a>Pasos siguientes

Para ver el código, los problemas actuales y las versiones, consulte el [repositorio de extensiones de CustomScript](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript).
