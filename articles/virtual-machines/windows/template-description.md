---
title: Máquinas virtuales de una plantilla de Azure Resource Manager | Microsoft Azure
description: Obtenga más información sobre cómo se define el recurso de máquina virtual en una plantilla de Azure Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/03/2019
ms.author: cynthn
ms.openlocfilehash: e420fe81941352d1002ed2c25c04fc686115c7c2
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87827459"
---
# <a name="virtual-machines-in-an-azure-resource-manager-template"></a>Máquinas virtuales de una plantilla de Azure Resource Manager

En este artículo se explican los aspectos de una plantilla de Azure Resource Manager que se aplican a las máquinas virtuales. En este artículo no se describe una plantilla completa para crear una máquina virtual. Para ello, necesita definiciones de recursos de cuentas de almacenamiento, interfaces de red, direcciones IP públicas y redes virtuales. Para obtener más información sobre cómo se pueden definir estos recursos juntos, consulte el [tutorial de plantilla de Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

Hay muchas [plantillas en la galería](https://azure.microsoft.com/documentation/templates/?term=VM) que incluyen el recurso de máquina virtual. No todos los elementos que pueden incorporarse en una plantilla se describen aquí.

 

En este ejemplo se muestra una sección de recursos típica de una plantilla para crear un número especificado de máquinas virtuales:

```json
"resources": [
  { 
    "apiVersion": "2016-04-30-preview", 
    "type": "Microsoft.Compute/virtualMachines", 
    "name": "[concat('myVM', copyindex())]", 
    "location": "[resourceGroup().location]",
    "copy": {
      "name": "virtualMachineLoop", 
      "count": "[parameters('numberOfInstances')]"
    },
    "dependsOn": [
      "[concat('Microsoft.Network/networkInterfaces/myNIC', copyindex())]" 
    ], 
    "properties": { 
      "hardwareProfile": { 
        "vmSize": "Standard_DS1" 
      }, 
      "osProfile": { 
        "computername": "[concat('myVM', copyindex())]", 
        "adminUsername": "[parameters('adminUsername')]", 
        "adminPassword": "[parameters('adminPassword')]" 
      }, 
      "storageProfile": { 
        "imageReference": { 
          "publisher": "MicrosoftWindowsServer", 
          "offer": "WindowsServer", 
          "sku": "2012-R2-Datacenter", 
          "version": "latest" 
        }, 
        "osDisk": { 
          "name": "[concat('myOSDisk', copyindex())]",
          "caching": "ReadWrite", 
          "createOption": "FromImage" 
        },
        "dataDisks": [
          {
            "name": "[concat('myDataDisk', copyindex())]",
            "diskSizeGB": "100",
            "lun": 0,
            "createOption": "Empty"
          }
        ] 
      }, 
      "networkProfile": { 
        "networkInterfaces": [ 
          { 
            "id": "[resourceId('Microsoft.Network/networkInterfaces',
              concat('myNIC', copyindex()))]" 
          } 
        ] 
      },
      "diagnosticsProfile": {
        "bootDiagnostics": {
          "enabled": "true",
          "storageUri": "[concat('https://', variables('storageName'), '.blob.core.windows.net')]"
        }
      } 
    },
    "resources": [ 
      { 
        "name": "Microsoft.Insights.VMDiagnosticsSettings", 
        "type": "extensions", 
        "location": "[resourceGroup().location]", 
        "apiVersion": "2016-03-30", 
        "dependsOn": [ 
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
        ], 
        "properties": { 
          "publisher": "Microsoft.Azure.Diagnostics", 
          "type": "IaaSDiagnostics", 
          "typeHandlerVersion": "1.5", 
          "autoUpgradeMinorVersion": true, 
          "settings": { 
            "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
            variables('wadmetricsresourceid'), 
            concat('myVM', copyindex()),
            variables('wadcfgxend')))]", 
            "storageAccount": "[variables('storageName')]" 
          }, 
          "protectedSettings": { 
            "storageAccountName": "[variables('storageName')]", 
            "storageAccountKey": "[listkeys(variables('accountid'), 
              '2015-06-15').key1]", 
            "storageAccountEndPoint": "https://core.windows.net" 
          } 
        } 
      },
      {
        "name": "MyCustomScriptExtension",
        "type": "extensions",
        "apiVersion": "2016-03-30",
        "location": "[resourceGroup().location]",
        "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
        ],
        "properties": {
          "publisher": "Microsoft.Compute",
          "type": "CustomScriptExtension",
          "typeHandlerVersion": "1.7",
          "autoUpgradeMinorVersion": true,
          "settings": {
            "fileUris": [
              "[concat('https://', variables('storageName'),
                '.blob.core.windows.net/customscripts/start.ps1')]" 
            ],
            "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
          }
        }
      } 
    ]
  } 
]
``` 

> [!NOTE] 
>Se basa en una cuenta de almacenamiento que se ha creado previamente. Puede crear la cuenta de almacenamiento implementando la plantilla. El ejemplo también se basa en una interfaz de red y sus recursos dependientes que se definirían en la plantilla. Estos recursos no se muestran en el ejemplo.
>
>

## <a name="api-version"></a>Versión de API

Cuando se implementan recursos mediante una plantilla, tendrá que especificar una versión de la API que se utilizará. En el ejemplo, se muestra el recurso de máquina virtual con este elemento apiVersion:

```json
"apiVersion": "2016-04-30-preview",
```

La versión de la API que especifica en la plantilla afecta a qué propiedades puede definir en la plantilla. En general, seleccionaría la versión de la API más reciente al crear nuevas plantillas. Para las plantillas existentes, puede decidir si quiere continuar usando una versión anterior de la API o actualizar la plantilla para que la versión más reciente aproveche las nuevas características.

Aproveche estas oportunidades para obtener las últimas versiones de API:

- API de REST: [Mostrar todos los proveedores de recursos](/rest/api/resources/providers)
- PowerShell - [Get AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider)
- CLI de Azure: [az provider show](/cli/azure/provider)


## <a name="parameters-and-variables"></a>Parámetros y variables

Los [parámetros](../../azure-resource-manager/templates/template-syntax.md) facilitan la tarea de especificar valores para la plantilla cuando la ejecuta. Esta sección de parámetros se utiliza en el ejemplo:

```json
"parameters": {
  "adminUsername": { "type": "string" },
  "adminPassword": { "type": "securestring" },
  "numberOfInstances": { "type": "int" }
},
```

Al implementar la plantilla de ejemplo, escriba valores para el nombre y la contraseña de la cuenta de administrador de cada máquina virtual y el número de máquinas virtuales que creará. Tiene la opción de especificar valores de parámetro en un archivo independiente que se administra con la plantilla, o bien proporcionarlos se le solicite.

Las [variables](../../azure-resource-manager/templates/template-syntax.md) facilitan la tarea de establecer valores en la plantilla que se usan varias veces en ella o que cambian con el tiempo. Esta sección de variables se utiliza en el ejemplo:

```json
"variables": { 
  "storageName": "mystore1",
  "accountid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name,
  '/providers/','Microsoft.Storage/storageAccounts/', variables('storageName'))]", 
  "wadlogs": "<WadCfg> 
    <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> 
      <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> 
      <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > 
        <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> 
        <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" />
      </WindowsEventLog>", 
  "wadperfcounters": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\">
      <PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\">
        <annotation displayName=\"Threads\" locale=\"en-us\"/>
      </PerformanceCounterConfiguration>
    </PerformanceCounters>", 
  "wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters'), 
    '<Metrics resourceId=\"')]", 
  "wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, 
    '/resourceGroups/', resourceGroup().name , 
    '/providers/', 'Microsoft.Compute/virtualMachines/')]", 
  "wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/>
    <MetricAggregation scheduledTransferPeriod=\"PT1M\"/>
    </Metrics></DiagnosticMonitorConfiguration>
    </WadCfg>"
}, 
```

Al implementar la plantilla de ejemplo, los valores de las variables se utilizan para el nombre y el identificador de la cuenta de almacenamiento creada anteriormente. También se usan variables para proporcionar los valores de la extensión de diagnóstico. Lea [Procedimientos recomendados para crear plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-best-practices.md) para ayudarlo a decidir cómo desea estructurar los parámetros y variables de la plantilla.

## <a name="resource-loops"></a>bucles de recursos

Si necesita más de una máquina virtual para la aplicación, puede utilizar un elemento de copia en una plantilla. Este elemento opcional recorre en bucle la creación del número de máquinas virtuales que especificó como un parámetro:

```json
"copy": {
  "name": "virtualMachineLoop", 
  "count": "[parameters('numberOfInstances')]"
},
```

Además, tenga en cuenta en el ejemplo se utiliza el índice de bucle al especificar algunos de los valores para el recurso. Por ejemplo, si ha especificado un recuento de las tres instancias, los nombres de los discos del SO son "myOSDisk1", "myOSDisk2" y "myOSDisk3":

```json
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
}
```

> [!NOTE] 
>Este ejemplo utiliza discos administrados para las máquinas virtuales.
>
>

Tenga en cuenta que la creación de un bucle para un recurso de la plantilla puede requerir que se utilice dicho bucle al crear otros recursos o acceder a ellos. Por ejemplo, varias máquinas virtuales no pueden usar la misma interfaz de red; por tanto, si recorre en bucle la creación de tres máquinas virtuales, también debe hacerlo con la generación de las tres interfaces de red. Al asignar una interfaz de red a una máquina virtual, el índice de bucle se utiliza para identificarlo:

```json
"networkInterfaces": [ { 
  "id": "[resourceId('Microsoft.Network/networkInterfaces',
    concat('myNIC', copyindex()))]" 
} ]
```

## <a name="dependencies"></a>Dependencias

La mayoría de los recursos dependen de otros para que funcionen correctamente. Las máquinas virtuales deben estar asociadas con una red virtual y, para tal fin, necesita una interfaz de red. El elemento [dependsOn](../../azure-resource-manager/templates/define-resource-dependency.md) se utiliza para asegurarse de que la interfaz de red está lista para usarse antes de que se creen las máquinas virtuales:

```json
"dependsOn": [
  "[concat('Microsoft.Network/networkInterfaces/', 'myNIC', copyindex())]" 
],
```

Resource Manager implementa en paralelo todos los recursos que no dependan de otro que se vaya a implementar. Tenga cuidado al establecer las dependencias porque puede ralentizar accidentalmente el proceso si define algunas que sean innecesarias. Pueden encadenar las dependencias a través de varios recursos. Por ejemplo, la interfaz de red depende de la dirección IP pública y los recursos de red virtual.

¿Cómo puede saber si se requiere una dependencia? Examine los valores establecidos en la plantilla. Si un elemento de la definición de máquina virtual apunta a otro recurso que se implementa en la misma plantilla, necesita una dependencia. Por ejemplo, la máquina virtual de ejemplo define un perfil de red:

```json
"networkProfile": { 
  "networkInterfaces": [ { 
    "id": "[resourceId('Microsoft.Network/networkInterfaces',
      concat('myNIC', copyindex())]" 
  } ] 
},
```

Para establecer esta propiedad, debe existir la interfaz de red. Por lo tanto, necesita una dependencia. También debe establecer una dependencia cuando se define un recurso (un elemento secundario) dentro de otro recurso (un elemento primario). Por ejemplo, las extensiones de script personalizado y la configuración de diagnóstico se definen como recursos secundarios de la máquina virtual. No se creará hasta que la máquina virtual exista. Por lo tanto, los recursos se marcan como dependientes de la máquina virtual.

## <a name="profiles"></a>Perfiles

Al definir un recurso de máquina virtual, se utilizan varios elementos de perfil. Algunos son necesarios y otros, opcionales. Por ejemplo, los elementos hardwareProfile, osProfile, storageProfile y networkProfile son necesarios, pero diagnosticsProfile es opcional. Estos perfiles definen opciones como:
   
- [size](../sizes.md)
- [Nombre](/azure/architecture/best-practices/resource-naming) y credenciales
- [Configuración del sistema operativo](cli-ps-findimage.md) y disco
- [Interfaz de red](/previous-versions/azure/virtual-network/virtual-network-deploy-multinic-classic-ps) 
- Diagnósticos de arranque

## <a name="disks-and-images"></a>Discos e imágenes
   
En Azure, los archivos de VHD pueden representar [discos o imágenes](managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Cuando el sistema operativo de un archivo de VHD está especializado para ser una máquina virtual específica, se conoce como "disco". Cuando el sistema operativo de un archivo de VHD está generalizado para crear muchas máquinas virtuales, se conoce como "imagen".   
    
### <a name="create-new-virtual-machines-and-new-disks-from-a-platform-image"></a>Creación de máquinas virtuales y discos a partir de una imagen de plataforma

Cuando se crea una máquina virtual, debe decidir qué sistema operativo usar. El elemento imageReference se utiliza para definir el sistema operativo de una nueva máquina virtual. En el ejemplo se muestra una definición de un sistema operativo Windows Server:

```json
"imageReference": { 
  "publisher": "MicrosoftWindowsServer", 
  "offer": "WindowsServer", 
  "sku": "2012-R2-Datacenter", 
  "version": "latest" 
},
```

Si desea crear un sistema operativo Linux, puede usar esta definición:

```json
"imageReference": {
  "publisher": "Canonical",
  "offer": "UbuntuServer",
  "sku": "14.04.2-LTS",
  "version": "latest"
},
```

Los valores de configuración del disco del SO se asignan con el elemento osDisk. En el ejemplo se define un nuevo disco administrado con el modo de almacenamiento en caché establecido en **ReadWrite** y que se va a crear el disco de un [imagen de plataforma](cli-ps-findimage.md):

```json
"osDisk": { 
  "name": "[concat('myOSDisk', copyindex())]",
  "caching": "ReadWrite", 
  "createOption": "FromImage" 
},
```

### <a name="create-new-virtual-machines-from-existing-managed-disks"></a>Creación de máquinas virtuales a partir de discos administrados existentes

Si desea crear máquinas virtuales a partir de discos existentes, quite los elementos imageReference y osProfile, y defina esta configuración de disco:

```json
"osDisk": { 
  "osType": "Windows",
  "managedDisk": { 
    "id": "[resourceId('Microsoft.Compute/disks', [concat('myOSDisk', copyindex())])]" 
  }, 
  "caching": "ReadWrite",
  "createOption": "Attach" 
},
```

### <a name="create-new-virtual-machines-from-a-managed-image"></a>Creación de máquinas virtuales a partir de una imagen administrada

Si desea crear una máquina virtual a partir de una imagen personalizada, cambie el elemento imageReference y defina esta configuración de disco:

```json
"storageProfile": { 
  "imageReference": {
    "id": "[resourceId('Microsoft.Compute/images', 'myImage')]"
  },
  "osDisk": { 
    "name": "[concat('myOSDisk', copyindex())]",
    "osType": "Windows",
    "caching": "ReadWrite", 
    "createOption": "FromImage" 
  }
},
```

### <a name="attach-data-disks"></a>Conexión de discos de datos

Si lo desea, puede agregar discos de datos a las máquinas virtuales. El [número de discos](../sizes.md) depende del tamaño del disco de sistema operativo que utilice. Con el tamaño de las máquinas virtuales establecido en Standard_DS1_v2, el número máximo de discos de datos que podría agregarse a es 2. En el ejemplo, se agrega un disco administrado a cada máquina virtual:

```json
"dataDisks": [
  {
    "name": "[concat('myDataDisk', copyindex())]",
    "diskSizeGB": "100",
    "lun": 0, 
    "caching": "ReadWrite",
    "createOption": "Empty"
  }
],
```

## <a name="extensions"></a>Extensiones

Aunque las [extensiones](../extensions/features-windows.md) son un recurso independiente, están estrechamente relacionadas con las máquinas virtuales. Las extensiones pueden agregar como un recurso secundario de la máquina virtual o como uno independiente. El ejemplo muestra la [extensión de diagnóstico](../extensions/diagnostics-template.md) que se agrega a las máquinas virtuales:

```json
{ 
  "name": "Microsoft.Insights.VMDiagnosticsSettings", 
  "type": "extensions", 
  "location": "[resourceGroup().location]", 
  "apiVersion": "2016-03-30", 
  "dependsOn": [ 
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]" 
  ], 
  "properties": { 
    "publisher": "Microsoft.Azure.Diagnostics", 
    "type": "IaaSDiagnostics", 
    "typeHandlerVersion": "1.5", 
    "autoUpgradeMinorVersion": true, 
    "settings": { 
      "xmlCfg": "[base64(concat(variables('wadcfgxstart'), 
      variables('wadmetricsresourceid'), 
      concat('myVM', copyindex()),
      variables('wadcfgxend')))]", 
      "storageAccount": "[variables('storageName')]" 
    }, 
    "protectedSettings": { 
      "storageAccountName": "[variables('storageName')]", 
      "storageAccountKey": "[listkeys(variables('accountid'), 
        '2015-06-15').key1]", 
      "storageAccountEndPoint": "https://core.windows.net" 
    } 
  } 
},
```

Este recurso de extensión usa la variable storageName y las variables de diagnóstico para proporcionar valores. Si desea cambiar los datos recopilados mediante esta extensión, puede agregar más contadores de rendimiento a la variable wadperfcounters. También podría elegir poner los datos de diagnóstico en una cuenta de almacenamiento diferente a donde se almacenan los discos de máquina virtual.

Existen muchas extensiones que se pueden instalar en una máquina virtual, pero la más útil es probablemente la de [script personalizado](../extensions/custom-script-windows.md). En el ejemplo, un script de PowerShell denominado "start.ps1" se ejecuta en cada máquina virtual cuando se inicia por primera vez:

```json
{
  "name": "MyCustomScriptExtension",
  "type": "extensions",
  "apiVersion": "2016-03-30",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/myVM', copyindex())]"
  ],
  "properties": {
    "publisher": "Microsoft.Compute",
    "type": "CustomScriptExtension",
    "typeHandlerVersion": "1.7",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "[concat('https://', variables('storageName'),
          '.blob.core.windows.net/customscripts/start.ps1')]" 
      ],
      "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
    }
  }
}
```

El script start.ps1 puede realizar muchas tareas de configuración. Por ejemplo, no se inicializan los discos de datos que se agregan a las máquinas virtuales en el ejemplo; puede usar un script personalizado para inicializarlos. Si tiene varias tareas de inicio para realizar, puede utilizar el archivo start.ps1 para llamar a otros scripts de PowerShell en Azure Storage. En el ejemplo se usa PowerShell, pero puede usar cualquier método de scripting que esté disponible en el sistema operativo que esté utilizando.

Puede ver el estado de las extensiones instaladas en la configuración de extensiones del portal:

![Consulta del estado de la extensión](./media/template-description/virtual-machines-show-extensions.png)

También puede obtener información de la extensión mediante el comando de PowerShell **Get-AzVMExtension**, el comando de la CLI de Azure **vm extension get** o la API REST **Get extension information**.

## <a name="deployments"></a>Implementaciones

Cuando se implementa una plantilla, Azure realiza un seguimiento de los recursos implementados como un grupo y asigna automáticamente un nombre para este grupo implementado. El nombre de la implementación es el mismo que el de la plantilla.

Si le preocupa el estado de los recursos de la implementación, consulte el grupo de recursos de Azure Portal:

![Obtención de la información de implementación](./media/template-description/virtual-machines-deployment-info.png)
    
No pasa nada por usar la misma plantilla para crear o actualizar recursos existentes. Al usar comandos para implementar plantillas, tiene la oportunidad de indicar qué [modo](../../azure-resource-manager/templates/deploy-powershell.md) usar. El modo se puede establecer en **Completo** o **Incremental**. El valor predeterminado son actualizaciones incrementales. Tenga precaución al utilizar el modo **Completo** porque se pueden eliminar accidentalmente los recursos. Cuando se establece el modo en **Completo**, Resource Manager elimina los recursos del grupo de recursos que no están en la plantilla.

## <a name="next-steps"></a>Pasos siguientes

- Cree su propia plantilla con las [Creación de plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).
- Implemente la plantilla que creó mediante [Creación de una máquina virtual Windows con una plantilla de Resource Manager](ps-template.md).
- Aprenda a administrar la máquina virtual que ha creado repasando el tema [Creación y administración de máquinas virtuales Windows con el módulo de Azure PowerShell](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
- Para información sobre la sintaxis de JSON y las propiedades de los tipos de recursos de las plantillas, consulte [Azure Resource Manager template reference](/azure/templates/) (Referencia de las plantillas de Azure Resource Manager).