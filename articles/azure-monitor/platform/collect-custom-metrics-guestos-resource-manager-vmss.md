---
title: Recopilación de métricas del conjunto de escalado de Windows en Azure Monitor con plantillas
description: Enviar métricas de SO invitado al almacén de métricas de Azure Monitor con una plantilla de Resource Manager para un conjunto de escalado de máquinas virtuales de Windows
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: 76f73df01b34cb20be48aefa3b5b3a6392a35b8b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87045200"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-by-using-an-azure-resource-manager-template-for-a-windows-virtual-machine-scale-set"></a>Enviar métricas de SO invitado al almacén de métricas de Azure Monitor con una plantilla de Azure Resource Manager para un conjunto de escalado de máquinas virtuales de Windows

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Mediante el uso de la [extensión Windows Azure Diagnostics (WAD)](diagnostics-extension-overview.md) de Azure Monitor, puede recopilar métricas y registros del sistema operativo invitado (SO invitado) que se ejecuta como parte de una máquina virtual, servicio en la nube, o clúster de Azure Service Fabric. La extensión puede enviar datos de telemetría a muchas ubicaciones diferentes que aparecen en el artículo vinculado anteriormente.  

En este artículo se describe el proceso de envío de métricas de rendimiento del SO invitado para un conjunto de escalado de máquinas virtuales de Windows al almacén de datos de Azure Monitor. A partir de Windows Azure Diagnostics versión 1.11, puede escribir las métricas directamente en el almacén de métricas de Azure Monitor, donde ya se recopilan métricas de la plataforma estándar. Al almacenarlas en esta ubicación, se permite tener acceso a las mismas acciones disponibles para las métricas de la plataforma. Las acciones incluyen la generación de alertas casi en tiempo real, la creación de gráficos, el enrutamiento, el acceso desde la API REST y mucho más. Anteriormente, la extensión Windows Azure Diagnostics se escribía en Azure Storage, pero no en el almacén de datos de Azure Monitor.  

Si no está familiarizado con las plantillas de Resource Manager, obtenga información sobre las [implementaciones de plantilla](../../azure-resource-manager/management/overview.md) y su estructura y sintaxis.  

## <a name="prerequisites"></a>Prerrequisitos

- La suscripción debe estar registrada en [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Debe tener instalado [Azure PowerShell](/powershell/azure), o puede usar [Azure CloudShell](../../cloud-shell/overview.md). 

- El recurso de máquina virtual debe estar en una [región que admita métricas personalizadas](metrics-custom-overview.md#supported-regions).

## <a name="set-up-azure-monitor-as-a-data-sink"></a>Configuración de Azure Monitor como receptor de datos 
La extensión Azure Diagnostics usa una característica denominada **receptores de datos** para enrutar las métricas y los registros a distintas ubicaciones. En los pasos siguientes se muestra cómo usar una plantilla de Resource Manager y PowerShell para implementar una VM con el nuevo receptor de datos Azure Monitor. 

## <a name="author-a-resource-manager-template"></a>Creación de una plantilla de Resource Manager 
En este ejemplo, puede usar una [plantilla de ejemplo](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-autoscale) disponible públicamente.  

- **Azuredeploy.json** es una plantilla de Resource Manager configurada previamente para la implementación de un conjunto de escalado de máquinas virtuales.

- **Azuredeploy.Parameters.json** es un archivo de parámetros que almacena información como el nombre de usuario y la contraseña que quiere establecer para la VM. Durante la implementación, la plantilla de Resource Manager usa los parámetros establecidos en este archivo. 

Descargue y guarde ambos archivos localmente. 

###  <a name="modify-azuredeployparametersjson"></a>Modificación de azuredeploy.parameters.json
Abra el archivo **azuredeploy.parameters.json**:  
 
- Proporcione el objeto **vmSKU** que quiere implementar. Se recomienda Standard_D2_v3. 
- Especifique un valor de **windowsOSVersion** que le gustaría para el conjunto de escalado de máquinas virtuales. Se recomienda 2016-Datacenter. 
- Asigne un nombre al recurso del conjunto de escalado de máquinas virtuales que se va a implementar con la propiedad **vmssName**. Por ejemplo, **VMSS-WAD-TEST**.    
- Especifique el número de VM que quiere que se ejecuten en el conjunto de escalado de máquinas virtuales con la propiedad **instanceCount**.
- Especifique los valores de **adminUsername** y **adminPassword** para el conjunto de escalado de máquinas virtuales. Estos parámetros se utilizan para el acceso remoto a las VM del conjunto de escalado. Para evitar el secuestro de la VM, **no** use los valores de esta plantilla. Los bots buscan por Internet los nombres de usuario y las contraseñas en repositorios públicos de GitHub. Es probable que sean VM de prueba con estos valores predeterminados. 


###  <a name="modify-azuredeployjson"></a>Modificación de azuredeploy.json
Abra el archivo **azuredeploy.json**. 

Agregue una variable para retener la información de la cuenta de almacenamiento en la plantilla de Resource Manager. Todos los registros p contadores de rendimiento especificados en el archivo de configuración de diagnóstico se escriben en el almacén de métricas de Azure Monitor y en la cuenta de almacenamiento especificada aquí: 

```json
"variables": { 
//add this line       
"storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]", 
```
 
Busque la definición de·conjunto de escalado de máquinas virtuales en la sección de recursos y agregue la sección **identidad** a la configuración. Esto adición garantiza que Azure le asigne una identidad del sistema. Este paso también garantiza que las VM del conjunto de escalado puedan emitir métricas de invitado sobre sí mismas a Azure Monitor.  

```json
    { 
      "type": "Microsoft.Compute/virtualMachineScaleSets", 
      "name": "[variables('namingInfix')]", 
      "location": "[resourceGroup().location]", 
      "apiVersion": "2017-03-30", 
      //add these lines below
      "identity": { 
           "type": "systemAssigned" 
       }, 
       //end of lines to add
```

En el recurso del conjunto de escalado de máquinas virtuales, busque la sección **virtualMachineProfile**. Agregue un nuevo perfil llamado **extensionsProfile** para administrar las extensiones.  


En **extensionProfile**, agregue una nueva extensión a la plantilla, como se muestra en la **sección VMSS-WAD-extension**.  Esta sección es la extensión de identidades administradas para los recursos de Azure que garantiza que Azure Monitor acepta las métricas que se emiten. El campo **name** puede contener cualquier nombre. 

El código siguiente de la extensión MSI también agrega la extensión de diagnóstico y la configuración como recurso de extensión al recurso del conjunto de escalado de máquinas virtuales. No dude en agregar o quitar contadores de rendimiento según sea necesario: 

```json
          "extensionProfile": { 
            "extensions": [ 
            // BEGINNING of added code  
            // Managed identities for Azure resources   
                { 
                 "name": "VMSS-WAD-extension", 
                 "properties": { 
                       "publisher": "Microsoft.ManagedIdentity", 
                       "type": "ManagedIdentityExtensionForWindows", 
                       "typeHandlerVersion": "1.0", 
                       "autoUpgradeMinorVersion": true, 
                       "settings": { 
                             "port": 50342 
                           }, 
                       "protectedSettings": {} 
                     } 
                                
            }, 
            // add diagnostic extension. (Remove this comment after pasting.)
            { 
              "name": "[concat('VMDiagnosticsVmExt','_vmNodeType0Name')]", 
              "properties": { 
                   "type": "IaaSDiagnostics", 
                   "autoUpgradeMinorVersion": true, 
                   "protectedSettings": { 
                        "storageAccountName": "[variables('storageAccountName')]", 
                        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')),'2015-05-01-preview').key1]", 
                        "storageAccountEndPoint": "https://core.windows.net/" 
                   }, 
                   "publisher": "Microsoft.Azure.Diagnostics", 
                   "settings": { 
                        "WadCfg": { 
                              "DiagnosticMonitorConfiguration": { 
                                   "overallQuotaInMB": "50000", 
                                   "PerformanceCounters": { 
                                       "scheduledTransferPeriod": "PT1M", 
                                       "sinks": "AzMonSink", 
                                       "PerformanceCounterConfiguration": [ 
                                          { 
              "counterSpecifier": "\\Memory\\% Committed Bytes In Use", 
                                              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Available Bytes", 
              "sampleRate": "PT15S" 
           }, 
           { 
              "counterSpecifier": "\\Memory\\Committed Bytes", 
              "sampleRate": "PT15S" 
           } 
                                       ] 
                                 }, 
                                 "EtwProviders": { 
                                       "EtwEventSourceProviderConfiguration": [ 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Actors", 
                                              "scheduledTransferKeywordFilter": "1", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                              "eventDestination": "ServiceFabricReliableActorEventTable" 
                                           } 
                                           }, 
                                           { 
                                              "provider": "Microsoft-ServiceFabric-Services", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricReliableServiceEventTable" 
                                              } 
                                           } 
                                     ], 
                                     "EtwManifestProviderConfiguration": [ 
                                           { 
                                              "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8", 
                                              "scheduledTransferLogLevelFilter": "Information", 
                                              "scheduledTransferKeywordFilter": "4611686018427387904", 
                                              "scheduledTransferPeriod": "PT5M", 
                                              "DefaultEvents": { 
                                                   "eventDestination": "ServiceFabricSystemEventTable" 
                                              } 
                                          } 
                                     ] 
                               } 
                               }, 
                               "SinksConfig": { 
                                     "Sink": [ 
                                          { 
                                              "name": "AzMonSink", 
                                              "AzureMonitor": {} 
                                          } 
                                      ] 
                               } 
                         }, 
                         "StorageAccount": "[variables('storageAccountName')]" 
                         }, 
                        "typeHandlerVersion": "1.11" 
                  } 
           } 
            ] 
          }
          }
      }
    },
    //end of added code plus a few brackets. Be sure that the number and type of brackets match properly when done. 
    {
      "type": "Microsoft.Insights/autoscaleSettings",
...
```


Agregue **dependsOn** para la cuenta de almacenamiento a fin de asegurarse de que se cree en el orden correcto. 

```json
"dependsOn": [ 
"[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]", 
"[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]" 
//add this line below
"[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]" 
```

Cree una cuenta de almacenamiento si aún no se ha creado ninguna en la plantilla. 

```json
"resources": [
// add this code    
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2015-05-01-preview",
    "location": "[resourceGroup().location]",
    "properties": {
    "accountType": "Standard_LRS"
    }
},
// end added code
{ 
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[variables('virtualNetworkName')]",
```

Guarde y cierre ambos archivos. 

## <a name="deploy-the-resource-manager-template"></a>Implementación de la plantilla de Resource Manager 

> [!NOTE]  
> Debe estar ejecutando la versión de la extensión 1.5 o superior de Azure Diagnostics **y**, A LA VEZ, tener la propiedad **autoUpgradeMinorVersion:** establecida en **true** en la plantilla de Resource Manager. A continuación, Azure carga la extensión adecuada cuando se inicia la VM. Si no tiene estas opciones en la plantilla, modifíquelas y vuelva a implementar la plantilla. 


Para implementar la plantilla de Resource Manager, se usa Azure PowerShell:  

1. Inicie PowerShell. 
1. Inicie sesión en Azure con `Login-AzAccount`.
1. Obtenga una lista de suscripciones con `Get-AzSubscription`.
1. Establezca la suscripción que va a crear o actualice la máquina virtual: 

   ```powershell
   Select-AzSubscription -SubscriptionName "<Name of the subscription>" 
   ```
1. Cree un nuevo grupo de recursos para la VM que se va a implementar. Ejecute el siguiente comando: 

   ```powershell
    New-AzResourceGroup -Name "VMSSWADtestGrp" -Location "<Azure Region>" 
   ```

   > [!NOTE]  
   > Recuerde usar una región de Azure que esté habilitada para las métricas personalizadas. Recuerde usar una [región de Azure que esté habilitada para las métricas personalizadas](./metrics-custom-overview.md#supported-regions).
 
1. Ejecute los siguientes comandos para implementar la máquina virtual:  

   > [!NOTE]  
   > Si quiere actualizar un conjunto de escalado existente, agregue **-Mode Incremental** al final del comando. 
 
   ```powershell
   New-AzResourceGroupDeployment -Name "VMSSWADTest" -ResourceGroupName "VMSSWADtestGrp" -TemplateFile "<File path of your azuredeploy.JSON file>" -TemplateParameterFile "<File path of your azuredeploy.parameters.JSON file>"  
   ```

1. Una vez realizada la implementación correctamente, debería ver el conjunto de escalado de máquinas virtuales en Azure Portal. Debería emitir métricas a Azure Monitor. 

   > [!NOTE]  
   > Puede que surjan errores en torno al parámetro **vmSkuSize** seleccionado. En ese caso, vuelva al archivo **azuredeploy.json** y actualice el valor predeterminado del parámetro **vmSkuSize**. Le recomendamos que pruebe **Standard_DS1_v2**. 


## <a name="chart-your-metrics"></a>Gráfico de las métricas 

1. Inicie sesión en Azure Portal. 

1. En el menú de la izquierda, seleccione **Monitor**. 

1. En la página **Monitor**, seleccione **Métricas**. 

   ![Página Métricas del Monitor](media/collect-custom-metrics-guestos-resource-manager-vmss/metrics.png) 

1. Cambie el período de agregación a **Últimos 30 minutos**.  

1. En el menú desplegable del recurso, seleccione el conjunto de escalado de máquinas virtuales que creó.  

1. En el menú desplegable de espacios de nombres, seleccione **azure.vm.windows.guest**. 

1. En el menú desplegable de métricas, seleccione **Memory\%Committed Bytes in Use** (Memoria%Bytes confirmados en uso).  

A continuación, también puede elegir usar las dimensiones en esta métrica para graficarla para una VM en particular o para trazar cada VM del conjunto de escalado. 



## <a name="next-steps"></a>Pasos siguientes
- Más información acerca de las [métricas personalizadas](metrics-custom-overview.md).
