---
title: Características y extensiones de las máquinas virtuales de Azure para Linux
description: Obtenga información acerca de qué extensiones están disponibles para máquinas virtuales de Azure, agrupadas por lo que proporcionan o mejoran.
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 52f5d0ec-8f75-49e7-9e15-88d46b420e63
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/30/2018
ms.author: akjosh
ms.openlocfilehash: bc29a62f469b0b9d091fcdef2488afba764a09fe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080359"
---
# <a name="virtual-machine-extensions-and-features-for-linux"></a>Características y extensiones de las máquinas virtuales para Linux

Las extensiones de máquina virtual de Azure son aplicaciones pequeñas que realizan tareas de automatización y configuración posterior a la implementación en máquinas virtuales de Azure. Por ejemplo, si una máquina virtual necesita que se instale software, protección antivirus o ejecutar un script en ella, se puede usar una extensión de máquina virtual. Las extensiones de máquina virtual de Azure se pueden ejecutar con la CLI de Azure, PowerShell, plantillas de Azure Resource Manager y Azure Portal. Las extensiones se pueden empaquetar con la implementación de una máquina virtual nueva o se pueden ejecutar en cualquier sistema existente.

Este artículo proporciona información general sobre las extensiones de máquina virtual, requisitos previos para usar las extensiones de máquina virtual de Azure e instrucciones sobre cómo detectar, administrar y quitar extensiones de máquina virtual. Este artículo proporciona información generalizada porque muchas extensiones de máquina virtual están disponibles, cada una con una configuración potencialmente única. En cada documento específico de la extensión individual pueden encontrarse detalles específicos de extensión.

## <a name="use-cases-and-samples"></a>Casos de uso y ejemplos

Varias extensiones de máquina virtual de Azure diferentes están disponibles, cada una con un caso de uso específico. Estos son algunos ejemplos:

- Aplique configuraciones de estado deseado de PowerShell a una máquina virtual con la extensión DSC para Linux. Para obtener más información, consulte la sección sobre la [extensión de configuración de estado deseado de Azure](https://github.com/Azure/azure-linux-extensions/tree/master/DSC).
- Configure la supervisión de una máquina virtual con la extensión de máquina virtual de Microsoft Monitoring Agent. Para más información, consulte el artículo sobre la [supervisión de máquinas virtuales Linux](../linux/tutorial-monitor.md).
- Configure la supervisión de su infraestructura de Azure con la extensión de Chef o Datadog. Para más información, consulte la [documentación de Chef](https://docs.chef.io/azure_portal.html) o el [blog de Datadog](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/).

Además de las extensiones específicas de proceso, una extensión de script personalizado está disponible tanto para máquinas virtuales Windows como para máquinas virtuales Linux. La extensión de script personalizado para Linux permite que se ejecute cualquier script de Bash en una máquina virtual. Los scripts personalizados resultan útiles para diseñar implementaciones de Azure que requieren una configuración más allá de lo que las herramientas de Azure nativas pueden proporcionar. Para obtener más información, consulte la sección sobre la [extensión de script personalizado de máquina virtual Linux](custom-script-linux.md).

## <a name="prerequisites"></a>Requisitos previos

Para controlar la extensión en la máquina virtual, debe tener instalado el Agente Linux de Azure. Algunas extensiones individuales tienen requisitos previos, como el acceso a los recursos o las dependencias.

### <a name="azure-vm-agent"></a>Agente de máquina virtual de Azure

El agente de máquina virtual de Azure administra las interacciones entre una máquina virtual de Azure y el controlador de tejido de Azure. El agente de máquina virtual es responsable de muchos aspectos funcionales de la implementación y administración de las máquinas virtuales de Azure, incluida la ejecución de extensiones de máquina virtual. El agente de máquina virtual de Azure está preinstalado en imágenes de Azure Marketplace y se puede instalar manualmente en sistemas operativos compatibles. El agente de máquina virtual de Azure para Linux se conoce como el agente Linux.

Para obtener información sobre las instrucciones de instalación y los sistemas operativos compatibles, consulte [Agente de máquina virtual de Azure](agent-linux.md).

#### <a name="supported-agent-versions"></a>Versiones de agente compatibles

Para brindar la mejor experiencia posible, se requieren versiones mínimas del agente. Para obtener más información, consulte [este artículo](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

#### <a name="supported-oses"></a>Sistemas operativos compatibles

El agente Linux se ejecuta en varios sistemas operativos, pero el marco de las extensiones tiene un límite para los sistemas operativos de esas extensiones. Para obtener más información, consulte [este artículo](https://support.microsoft.com/en-us/help/4078134/azure-extension-supported-operating-systems
).

Algunas extensiones no son compatibles con todos los sistemas operativos y puede aparecer el *código de error 51, "Sistema operativo no compatible"* . Revise la documentación de cada una de las extensiones para información sobre la compatibilidad.

#### <a name="network-access"></a>Acceso de red

Los paquetes de extensiones se descargan del repositorio de extensiones de Azure Storage y las cargas de estado de las extensiones se publican en Azure Storage. Si usa una versión [compatible](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) de los agentes, no necesita autorizar el acceso a Azure Storage en la región de la máquina virtual, porque puede usar el agente para redirigir la comunicación al controlador de tejido de Azure para las comunicaciones de los agentes. Si usa una versión no compatible del agente, deberá autorizar el acceso de salida a Azure Storage en esa región desde la máquina virtual.

> [!IMPORTANT]
> Si bloqueó el acceso a *168.63.129.16* con el firewall invitado, las extensiones generarán un error con independencia de lo mencionado anteriormente.

Los agentes solo se pueden usar para descargar los paquetes de extensiones e informar el estado. Por ejemplo, si la instalación de una extensión requiere descargar un script de GitHub (script personalizado) o necesita acceso a Azure Storage (Azure Backup), se deben abrir puertos adicionales del firewall o del grupo de seguridad de red. Distintas extensiones tienen distintos requisitos, porque son aplicaciones por sí mismas. En el caso de las extensiones que requieren acceso a Azure Storage, puede permitir el acceso mediante las etiquetas del servicio NSG de Azure para [Storage](../../virtual-network/security-overview.md#service-tags).

Para redirigir las solicitudes de tráfico del agente, el agente Linux es compatible con el servidor proxy. Sin embargo, esta compatibilidad con el servidor proxy no se aplica a las extensiones. Debe configurar cada extensión individual para que funcione con un servidor proxy.

## <a name="discover-vm-extensions"></a>Detección de extensiones de máquina virtual

Hay muchas extensiones de máquina virtual diferentes disponibles para su uso con máquinas virtuales de Azure. Para ver una lista completa, use [az vm extension image list](/cli/azure/vm/extension/image#az-vm-extension-image-list). En el ejemplo siguiente se muestran todas las extensiones disponibles en la ubicación *westus*:

```azurecli
az vm extension image list --location westus --output table
```

## <a name="run-vm-extensions"></a>Ejecución de extensiones de máquina virtual

Las extensiones de máquinas virtuales de Azure se ejecutan en máquinas virtuales existentes, lo que resulta útil si es preciso realizar cambios de configuración o recuperar la conectividad en una máquina virtual ya implementada. Las extensiones de máquina virtual también pueden agruparse con las implementaciones de plantilla de Azure Resource Manager. Al usar las extensiones con plantillas de Resource Manager, las máquinas virtuales de Azure se pueden implementar y configurar sin intervención alguna después de la implementación.

Los métodos siguientes pueden usarse para ejecutar una extensión en una máquina virtual existente.

### <a name="azure-cli"></a>Azure CLI

Las extensiones de máquina virtual de Azure se pueden ejecutar en una máquina virtual existente con el comando [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set). En el ejemplo siguiente se ejecuta la extensión de script personalizado en una máquina virtual denominada *myVM* en un grupo de recursos con el nombre *myResourceGroup*. Reemplace los nombres de ejemplo del grupo de recursos, la máquina virtual y el script que se va a ejecutar (https:\//raw.githubusercontent.com/me/project/hello.sh) por su propia información. 

```azurecli
az vm extension set `
  --resource-group myResourceGroup `
  --vm-name myVM `
  --name customScript `
  --publisher Microsoft.Azure.Extensions `
  --settings '{"fileUris": ["https://raw.githubusercontent.com/me/project/hello.sh"],"commandToExecute": "./hello.sh"}'
```

Cuando la extensión se ejecuta correctamente, la salida es similar al ejemplo siguiente:

```bash
info:    Executing command vm extension set
+ Looking up the VM "myVM"
+ Installing extension "CustomScript", VM: "mvVM"
info:    vm extension set command OK
```

### <a name="azure-portal"></a>Azure portal

Las extensiones de máquina virtual se pueden aplicar a una máquina virtual existente a través de Azure Portal. Seleccione la máquina virtual en el portal, elija **Extensiones** y, luego, **Agregar**. Elija la extensión que quiera en la lista de extensiones disponibles y siga las instrucciones del asistente.

En la siguiente imagen se muestra la instalación de la extensión de script personalizado de Linux desde Azure Portal:

![Instalar la extensión de script personalizado](./media/features-linux/installscriptextensionlinux.png)

### <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure

Las extensiones de máquina virtual se pueden agregar a una plantilla de Azure Resource Manager y ejecutar con la implementación de la plantilla. Al implementar una extensión con una plantilla, puede crear implementaciones de Azure completamente configuradas. Por ejemplo, el siguiente JSON procede de una plantilla de Resource Manager que implementa un conjunto de máquinas virtuales de carga equilibrada y Azure SQL Database y, a continuación, instala una aplicación .NET Core en cada VM. La extensión de máquina virtual se encarga de la instalación de software.

Para obtener más información, consulte la [plantilla de Resource Manager](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-linux) completa.

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
    }
}
```

Para más información sobre como crear plantillas de Resource Manager, consulte el artículo sobre la [creación de plantillas de Azure Resource Manager](../windows/template-description.md#extensions).

## <a name="secure-vm-extension-data"></a>Protección de datos de extensión de máquina virtual

Cuando ejecuta una extensión de máquina virtual, puede que sea necesario incluir información confidencial como credenciales, nombres de cuenta de almacenamiento y claves de acceso de cuenta de almacenamiento. Muchas extensiones de máquina virtual incluyen una configuración protegida que cifra datos y los descifra solo dentro de la máquina virtual de destino. Cada extensión tiene un esquema específico de configuración protegida que se detalla de forma individual en la documentación específica de extensión.

En el ejemplo siguiente se muestra una instancia de la extensión de script personalizado para Linux. El comando que se ejecutará incluye un conjunto de credenciales. En este ejemplo, el comando que se ejecutará no está cifrado:

```json
{
  "apiVersion": "2015-06-15",
  "type": "extensions",
  "name": "config-app",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
      ],
      "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
  }
}
```

Al moverse la propiedad **command to execute** a la configuración **protegida**, se protege la cadena de ejecución tal como se muestra en el ejemplo siguiente:

```json
{
  "apiVersion": "2015-06-15",
  "type": "extensions",
  "name": "config-app",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
      ]
    },
    "protectedSettings": {
      "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
  }
}
```

### <a name="how-do-agents-and-extensions-get-updated"></a>¿Cómo se actualizan los agentes y las extensiones?

Los agentes y las extensiones comparten el mismo mecanismo de actualización. Algunas actualizaciones no necesitan reglas adicionales de firewall.

Cuando hay disponible una actualización, solo se instala en la máquina virtual cuando se produce un cambio en las extensiones y otros cambios del modelo de máquina virtual, como:

- Discos de datos.
- Extensiones
- Contenedor de diagnósticos de arranque
- Secretos del sistema operativo invitado
- Tamaño de VM
- Perfil de red

Los publicadores hacen que las actualizaciones estén disponibles en distintos momentos en las diferentes regiones, por lo que es posible que tenga máquinas virtuales en distintas regiones con versiones diferentes.

#### <a name="agent-updates"></a>Actualizaciones del agente

El agente de máquina virtual Linux contiene *código de aprovisionamiento del agente* y *código de administración de extensiones* en un solo paquete que no se puede separar. Puede deshabilitar el *agente de aprovisionamiento* cuando quiera aprovisionar en Azure con cloud-init. Para hacerlo, consulte cómo [usar cloud-init](../linux/using-cloud-init.md).

Las versiones compatibles de los agentes pueden usar actualizaciones automáticas. El único código que se puede actualizar es el *código de administración de extensiones* y no el código de aprovisionamiento. El *código de aprovisionamiento del agente* solo se ejecuta una vez.

El *código de administración de extensiones* es responsable de comunicarse con el tejido de Azure y administrar las operaciones de extensiones de máquinas virtuales como las instalaciones, informar el estado, actualizar las extensiones individuales y quitarlas. Las actualizaciones contienen revisiones de seguridad, correcciones de errores y mejoras en el *código de administración de extensiones*.

Cuando se instala el agente, se crea un demonio primario. Este proceso primario luego genera un proceso secundario que se usa para administrar las extensiones. Si hay disponible una actualización para el agente, se descarga, el proceso primario detiene el secundario, lo actualiza y lo reinicia. Si se produce un problema con la actualización, el proceso primario revierte el proceso secundario a su versión anterior.

El proceso primario no se puede actualizar automáticamente. El proceso primario solo se puede actualizar mediante la actualización de un paquete de distribución.

Para comprobar qué versión ejecuta, compruebe `waagent` de la siguiente manera:

```bash
waagent --version
```

La salida es similar a la del ejemplo siguiente:

```bash
WALinuxAgent-2.2.17 running on ubuntu 16.04
Python: 3.5.2
Goal state agent: 2.2.18
```

En la salida del ejemplo anterior, el proceso primario o la "versión implementada del paquete" es *WALinuxAgent-2.2.17*.

"Goal state agent" es la versión de actualización automática.

Se recomienda que siempre tenga activada la actualización automática del agente, [AutoUpdate.Enabled=y](./update-linux-agent.md). Si no tiene habilitada esta opción, debe seguir actualizando de manera manual el agente y así no recibirá las correcciones de errores y de seguridad.

#### <a name="extension-updates"></a>Actualizaciones de extensiones

Cuando hay disponible una actualización de extensión, el agente Linux la descarga y actualiza. Las actualizaciones de extensiones automáticas son *secundarias* o *revisiones*. Puede decidir si desea recibir o no las actualizaciones *secundarias* de las extensiones cuando aprovisiona la extensión. En el ejemplo siguiente se muestra cómo actualizar automáticamente las versiones secundarias en una plantilla de Resource Manager con *autoUpgradeMinorVersion": true,'* :

```json
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
        ]
    },
```

Para obtener las correcciones de errores secundarias más recientes, se recomienda encarecidamente que siempre seleccione la actualización automática en las implementaciones de sus extensiones. No es posible optar por no recibir actualizaciones de revisiones que incluyen correcciones de errores clave o revisiones de seguridad.

### <a name="how-to-identify-extension-updates"></a>Identificación de las actualizaciones de extensiones

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>Identificación de si la extensión tiene establecida la opción autoUpgradeMinorVersion en una máquina virtual

En el modelo de la máquina virtual puede ver si la extensión se aprovisionó con "autoUpgradeMinorVersion". Para comprobarlo, use [az vm show](/cli/azure/vm#az-vm-show) y proporcione el nombre de la máquina virtual y el grupo de recursos como se indica a continuación:

```azurecli
az vm show --resource-group myResourceGroup --name myVM
```

La salida del ejemplo siguiente muestra que la opción *autoUpgradeMinorVersion* está establecida en *true*:

```json
  "resources": [
    {
      "autoUpgradeMinorVersion": true,
      "forceUpdateTag": null,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM/extensions/CustomScriptExtension",
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>Identificación de cuando se produce un evento de autoUpgradeMinorVersion

Para ver cuando se produce una actualización de la extensión, revise los registros del agente de la máquina virtual en */var/log/waagent.log*.

En el ejemplo anterior, la máquina virtual tenía instalada *Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025*. Hubo disponible una revisión para *Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027*:

```bash
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Expected handler state: enabled
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Decide which version to use
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Use version: 2.3.9027
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Current handler state is: NotInstalled
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Download extension package
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Unpack extension package
INFO Event: name=Microsoft.OSTCExtensions.LinuxDiagnostic, op=Download, message=Download succeeded
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Initialize extension directory
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Update settings file: 0.settings
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025] Disable extension.
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025] Launch command:diagnostic.py -disable
...
INFO Event: name=Microsoft.OSTCExtensions.LinuxDiagnostic, op=Disable, message=Launch command succeeded: diagnostic.py -disable
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Update extension.
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Launch command:diagnostic.py -update
2017/08/14 20:21:57 LinuxAzureDiagnostic started to handle.
```

## <a name="agent-permissions"></a>Permisos del agente

Para realizar sus tareas, el agente se debe ejecutar como *raíz*.

## <a name="troubleshoot-vm-extensions"></a>Solución de problemas de extensiones de máquina virtual

Cada extensión de máquina virtual puede tener unos pasos de solución de problemas específicos de la extensión. Por ejemplo, cuando usa la extensión de script personalizado, se pueden encontrar detalles de ejecución de script localmente en la máquina virtual donde se ejecutó la extensión. Todos los pasos de solución de problemas específicos de extensión aparecen detallados en la documentación específica de extensión.

Los pasos de solución de problemas siguientes se aplican a todas las extensiones de máquina virtual.

1. Para comprobar el registro del agente Linux, examine la actividad cuando la extensión se aprovisionó en */var/log/waagent.log*.

2. Revise los registros reales de la extensión para ver más detalles en */var/log/azure/\<extensionName>* .

3. Revise las secciones de solución de problemas de la documentación específica de la extensión para ver los códigos de errores, los problemas conocidos, etc.

3. Examine los registros del sistema. Compruebe si hay otras operaciones que puedan haber interferido con la extensión, como una instalación de ejecución prolongada de otra aplicación que requería acceso exclusivo al administrador de paquetes.

### <a name="common-reasons-for-extension-failures"></a>Motivos comunes para los errores de extensiones

1. Las extensiones tienen 20 minutos para ejecutarse (las excepciones son las extensiones CustomScript, Chef y DSC que tienen 90 minutos). Si la implementación supera este tiempo, se marca como tiempo de expiración. La causa de esto puede deberse a máquinas virtuales de bajos recursos, otras tareas de inicio y configuraciones de máquinas virtuales que consumen grandes cantidades de recursos mientras la extensión intenta realizar el aprovisionamiento.

2. No se cumplen los requisitos previos mínimos. Algunas extensiones tienen dependencias de las SKU de la máquina virtual, como las imágenes de HPC. Las extensiones pueden requerir ciertos requisitos de acceso de red, como comunicaciones con Azure Storage o servicios públicos. Otros ejemplos podrían ser el acceso a repositorios de paquetes, quedarse sin espacio en disco o restricciones de seguridad.

3. Acceso exclusivo al administrador de paquetes. En algunos casos, podría encontrar un conflicto entre la configuración de máquina virtual de ejecución prolongada y la instalación de la extensión, donde ambas necesitan acceso exclusivo al administrador de paquetes.

### <a name="view-extension-status"></a>Consulta del estado de la extensión

Después de que una extensión de máquina virtual se ejecuta en una máquina virtual, use [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view) para devolver el estado de la extensión como se indica a continuación:

```azurecli
az vm get-instance-view \
    --resource-group rgName \
    --name myVM \
    --query "instanceView.extensions"
```

La salida es similar a la salida del ejemplo siguiente:

```bash
  {
    "name": "customScript",
    "statuses": [
      {
        "code": "ProvisioningState/failed/0",
        "displayStatus": "Provisioning failed",
        "level": "Error",
        "message": "Enable failed: failed to execute command: command terminated with exit status=127\n[stdout]\n\n[stderr]\n/bin/sh: 1: ech: not found\n",
        "time": null
      }
    ],
    "substatuses": null,
    "type": "Microsoft.Azure.Extensions.customScript",
    "typeHandlerVersion": "2.0.6"
  }
```

El estado de ejecución de extensión también puede encontrarse en Azure Portal. Para ver el estado de una extensión, seleccione la máquina virtual, elija **Extensiones** y seleccione la extensión deseada.

### <a name="rerun-a-vm-extension"></a>Repetición de ejecución de una extensión de máquina virtual

Puede haber casos en los que sea necesario volver a ejecutar una extensión de máquina virtual. Puede volver ejecutar la extensión si la quita y la vuelve a ejecutar con un método de ejecución de su elección. Para quitar una extensión, use [az vm extension delete](/cli/azure/vm/extension#az-vm-extension-delete) como se indica a continuación:

```azurecli
az vm extension delete \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name customScript
```

También puede quitar una extensión en Azure Portal como se indica a continuación:

1. Seleccione una máquina virtual.
2. Elija **Extensiones**.
3. Seleccione la extensión deseada.
4. Elija **Desinstalar**.

## <a name="common-vm-extension-reference"></a>Referencia de extensión de máquina virtual común

| Nombre de la extensión | Descripción | Más información |
| --- | --- | --- |
| Extensión de script personalizado para Linux |Ejecución de scripts en una máquina virtual de Azure |[Extensión de script personalizado para Linux](custom-script-linux.md) |
| Extensión de acceso a máquina virtual |Repetición de obtención de acceso a una máquina virtual de Azure |[Extensión de acceso a máquina virtual](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) |
| Extensión de Diagnósticos de Azure |Administración de Diagnósticos de Azure |[Extensión de Diagnósticos de Azure](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Extensión de acceso a máquina virtual de Azure |Administración de usuarios y credenciales |[Extensión de acceso a máquina virtual para Linux](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las extensiones de máquina virtual, consulte [Características y extensiones de las máquinas virtuales de Azure](overview.md).
