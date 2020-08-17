---
title: Agregación de eventos con Windows Azure Diagnostics
description: Obtenga información sobre la agregación y la recopilación de eventos con WAD para la supervisión y el diagnóstico de clústeres de Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 04/03/2018
ms.author: srrengar
ms.openlocfilehash: bcb9ca9e73c0898dc778202eca036a5ae92bebf8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076140"
---
# <a name="event-aggregation-and-collection-using-windows-azure-diagnostics"></a>Recopilación y agregación de eventos con Azure Diagnostics de Windows
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-event-aggregation-wad.md)
> * [Linux](service-fabric-diagnostics-event-aggregation-lad.md)
>
>

Cuando se ejecuta un clúster de Azure Service Fabric, es conveniente recopilar los registros de todos los nodos en una ubicación central. La presencia de los registros en una ubicación central facilita el análisis y la solución de los problemas del clúster o de las aplicaciones y los servicios que se ejecutan en ese clúster.

Uno de los métodos para cargar y recopilar registros es usar la extensión Azure Diagnostics de Windows (WAD), que carga registros en Azure Storage, y también tiene la opción de enviar registros a Azure Application Insights o Event Hubs. Además, puede usar un proceso externo para leer los eventos desde el almacenamiento y colocarlos en un producto de plataforma de análisis, como los [registros de Azure Monitor](./service-fabric-diagnostics-oms-setup.md) u otra solución de análisis de registros.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos
En este artículo se emplean las herramientas siguientes:

* [Azure Resource Manager](../azure-resource-manager/management/overview.md)
* [Azure PowerShell](/powershell/azure/)
* [Plantilla de Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json)

## <a name="service-fabric-platform-events"></a>Eventos de plataforma de Service Fabric
Service Fabric configura algunos [canales de registro predefinidos](service-fabric-diagnostics-event-generation-infra.md). Los que se indican a continuación se configuran previamente con la extensión para que envíen datos de supervisión y diagnóstico a una tabla de almacenamiento u otro lugar:
  * [Eventos operativos](service-fabric-diagnostics-event-generation-operational.md): operaciones de nivel superior realizadas por la plataforma Service Fabric. Algunos ejemplos son la creación de aplicaciones y servicios, los cambios de estado de nodo y la información sobre la actualización. Se emiten como registros de Seguimiento de eventos para Windows (ETW).
  * [Eventos del modelo de programación de Reliable Actors](service-fabric-reliable-actors-diagnostics.md)
  * [Eventos del modelo de programación de Reliable Services](service-fabric-reliable-services-diagnostics.md)

## <a name="deploy-the-diagnostics-extension-through-the-portal"></a>Implementación de la extensión de Diagnostics mediante el portal
El primer paso para recopilar registros es implementar la extensión de Diagnostics en los nodos del conjunto de escalado de máquinas virtuales del clúster de Service Fabric. La extensión de Diagnósticos recopila registros en cada máquina virtual y los carga a la cuenta de almacenamiento que especifique. Los siguientes pasos describen cómo lograrlo para los clústeres nuevos y existentes mediante Azure Portal y plantillas de Azure Resource Manager.

### <a name="deploy-the-diagnostics-extension-as-part-of-cluster-creation-through-azure-portal"></a>Implementación de la extensión Diagnostics como parte de la creación del clúster a través de Azure Portal
Al crear el clúster, en el paso de configuración de clúster, expanda la configuración opcional y asegúrese de que Diagnósticos está establecido en **Encendido** (valor predeterminado).

![Configuración de Diagnósticos de Azure en el portal para crear un clúster](media/service-fabric-diagnostics-event-aggregation-wad/azure-enable-diagnostics-new.png)

Es muy recomendable descargar la plantilla **antes de hacer clic en Crear** en el paso final. Para más información, vea [Configuración de un clúster de Service Fabric con una plantilla de Azure Resource Manager](service-fabric-cluster-creation-via-arm.md). Necesita la plantilla para realizar cambios en los canales (mencionados anteriormente) de los que recopilar los datos.

![Plantilla del clúster](media/service-fabric-diagnostics-event-aggregation-wad/download-cluster-template.png)

Ahora que agrega eventos a Azure Storage, [configure los registros de Azure Monitor](service-fabric-diagnostics-oms-setup.md) para obtener información y consultarlos en el portal de los registros de Azure Monitor.

>[!NOTE]
>Actualmente no existe ninguna manera de filtrar o limpiar los eventos que se envían a las tablas. Si no se implementa un proceso para quitar eventos de la tabla, esta seguirá aumentando (límite predeterminado: 50 GB). [Más adelante en este artículo](service-fabric-diagnostics-event-aggregation-wad.md#update-storage-quota) encontrará instrucciones sobre cómo cambiar esto. Además, hay un ejemplo de un servicio de limpieza de datos en ejecución en el [ejemplo de guardián](https://github.com/Azure-Samples/service-fabric-watchdog-service). Se recomienda que escriba uno para usted, a menos que tenga una buena razón para almacenar los registros durante más de 30 o 90 días.



## <a name="deploy-the-diagnostics-extension-through-azure-resource-manager"></a>Implementación de la extensión de Diagnostics mediante Azure Resource Manager

### <a name="create-a-cluster-with-the-diagnostics-extension"></a>Creación de un clúster con la extensión de Diagnostics
Para crear un clúster mediante el Resource Manager, tiene que agregar el JSON de la configuración de Diagnostics a la plantilla de Resource Manager completa. Dentro de los ejemplos de plantillas del Administrador de recursos, proporcionamos una plantilla de ejemplo del Administrador de recursos de clúster de cinco máquinas virtuales con la configuración de Diagnósticos añadida. Puede verlo en esta ubicación en la galería de ejemplos de Azure: [Clúster de cinco nodos con el ejemplo de Diagnósticos en la plantilla de Resource Manager](https://azure.microsoft.com/resources/templates/service-fabric-secure-cluster-5-node-1-nodetype/).

Para ver la configuración de Diagnósticos en la plantilla de Resource Manager, abra el archivo azuredeploy.json y busque **IaaSDiagnostics**. Para crear un clúster con esta plantilla, presione el botón **Deploy to Azure** (Implementar en Azure) disponible en el vínculo anterior.

También puede descargar el ejemplo de Resource Manager, modificarlo y crear un clúster con la plantilla modificada mediante el comando `New-AzResourceGroupDeployment` en una ventana de Azure PowerShell. Vea el código siguiente para los parámetros que se pasan al comando. Para más información sobre cómo implementar un grupo de recursos con PowerShell, vea el artículo sobre la [implementación de un grupo de recursos con la plantilla de Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).

### <a name="add-the-diagnostics-extension-to-an-existing-cluster"></a>Incorporación de la extensión de Diagnostics a un clúster existente
Si tiene un clúster existente sin la extensión de Diagnostics implementada, puede agregarla o actualizarla con la plantilla del clúster. Modifique la plantilla de Resource Manager usada para crear el clúster existente o descargue la plantilla desde el portal, tal como se describió anteriormente. Modifique el archivo template.json mediante la realización de las siguientes tareas:

Agregue un nuevo recurso de almacenamiento a la plantilla mediante una adición en la sección de recursos.

```json
{
    "apiVersion": "2018-07-01",
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
    "location": "[parameters('computeLocation')]",
    "sku": {
    "name": "[parameters('applicationDiagnosticsStorageAccountType')]"
    "tier": "standard"
  },
    "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
```

 A continuación, agréguelo a la sección de parámetros justo después de las definiciones de la cuenta de almacenamiento, entre `supportLogStorageAccountName`. Reemplace el texto de marcador de posición *aquí va el nombre de la cuenta de almacenamiento* por el nombre de la cuenta de almacenamiento que desee.

```json
    "applicationDiagnosticsStorageAccountType": {
      "type": "string",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS"
      ],
      "defaultValue": "Standard_LRS",
      "metadata": {
        "description": "Replication option for the application diagnostics storage account"
      }
    },
    "applicationDiagnosticsStorageAccountName": {
      "type": "string",
      "defaultValue": "**STORAGE ACCOUNT NAME GOES HERE**",
      "metadata": {
        "description": "Name for the storage account that contains application diagnostics data from the cluster"
      }
    },
```
Luego, actualice la sección `VirtualMachineProfile` del archivo template.json. Para ello, agregue el siguiente código dentro de la matriz de extensiones. Asegúrese de agregar una coma al principio o al final, según donde se inserte.

```json
{
    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
    "properties": {
        "type": "IaaSDiagnostics",
        "autoUpgradeMinorVersion": true,
        "protectedSettings": {
        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
        "storageAccountEndPoint": "https://core.windows.net/"
        },
        "publisher": "Microsoft.Azure.Diagnostics",
        "settings": {
        "WadCfg": {
            "DiagnosticMonitorConfiguration": {
            "overallQuotaInMB": "50000",
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
                },
                {
                    "provider": "02d06793-efeb-48c8-8f7f-09713309a810",
                    "scheduledTransferLogLevelFilter": "Information",
                    "scheduledTransferKeywordFilter": "4611686018427387904",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricSystemEventTable"
                    }
                }
                ]
            }
            }
        },
        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
        },
        "typeHandlerVersion": "1.5"
    }
}
```

Después de modificar el archivo template.json tal como se indicó, vuelva a publicar la plantilla de Resource Manager. En caso de que la plantilla se haya exportado, si ejecuta el archivo deploy.ps1 , se vuelve a publicar dicha plantilla. Después de realizar la implementación, asegúrese de que el estado **ProvisioningState** sea **Correcto**.

> [!TIP]
> Si va a implementar contenedores en el clúster, habilite WAD para recopilar estadísticas de Docker. Para ello, agregue lo siguiente a la sección **WadCfg > DiagnosticMonitorConfiguration**.
>
>```json
>"DockerSources": {
>    "Stats": {
>        "enabled": true,
>        "sampleRate": "PT1M"
>    }
>},
>```

### <a name="update-storage-quota"></a>Actualización de la cuota de almacenamiento

Como las tablas que rellena la extensión aumentan hasta que se alcanza la cuota, quizá quiera considerar reducir el tamaño de esta. El valor predeterminado es 50 GB y se puede configurar en la plantilla en el campo `overallQuotaInMB` de `DiagnosticMonitorConfiguration`

```json
"overallQuotaInMB": "50000",
```

## <a name="log-collection-configurations"></a>Configuraciones de recopilación de registros
Los registros de los canales adicionales también están disponibles para la recopilación, estas son algunas de las configuraciones más comunes que puede realizar en la plantilla para los clústeres que se ejecutan en Azure.

* Canal operativo, básico: Habilitadas de forma predeterminada, las operaciones de alto nivel que realizan Service Fabric y el clúster, incluidos eventos para un nodo próximo, una nueva aplicación que se implementa o la reversión de una actualización, etc. Para obtener una lista de eventos, consulte los [eventos de canal operativo](./service-fabric-diagnostics-event-generation-operational.md).
  
```json
      scheduledTransferKeywordFilter: "4611686018427387904"
  ```
* Canal operativo, detallado: incluye informes de estado y decisiones de equilibrio de carga, además de todos los elementos de canal operativo básico. Estos eventos son generados por el sistema o por el código mediante las API de generación de informes de estado de mantenimiento o carga, como [ReportPartitionHealth](/previous-versions/azure/reference/mt645153(v=azure.100)) o [ReportLoad](/previous-versions/azure/reference/mt161491(v=azure.100)). Para ver estos eventos en el Visor de eventos de diagnóstico de Visual Studio, agregue "Microsoft-ServiceFabric:4:0x4000000000000008" a la lista de proveedores de ETW.

```json
      scheduledTransferKeywordFilter: "4611686018427387912"
  ```

* Canal de datos y mensajería, básico: registros y eventos críticos generados en la ruta de acceso a mensajería (actualmente solo ReverseProxy) y a datos, además de los registros del canal operativo detallado. Estos eventos son errores de procesamiento de solicitud y otros problemas críticos en ReverseProxy y en las solicitudes procesadas. **Esta es nuestra recomendación para el registro completo**. Para ver estos eventos en el Visor de eventos de diagnóstico de Visual Studio, agregue "Microsoft-ServiceFabric:4:0x4000000000000010" a la lista de proveedores de ETW.

```json
      scheduledTransferKeywordFilter: "4611686018427387928"
  ```

* Canal de datos y mensajería, detallado: canal detallado que contiene todos los registros no críticos de datos y mensajería del clúster, y el canal operativo detallado. Para obtener información sobre solución de problemas de todos los eventos de proxy inverso, consulte la [Guía de diagnóstico de proxy inverso](service-fabric-reverse-proxy-diagnostics.md).  Para ver estos eventos en el Visor de eventos de diagnóstico de Visual Studio, agregue "Microsoft-ServiceFabric:4:0x4000000000000020" a la lista de proveedores de ETW.

```json
      scheduledTransferKeywordFilter: "4611686018427387944"
  ```

>[!NOTE]
>Este canal tiene un volumen muy grande de eventos; permitir la recopilación desde este canal detallado provoca la generación rápida de muchos seguimientos, lo que puede consumir capacidad de almacenamiento. Solo active esta opción si es absolutamente necesario.


Para habilitar el **Canal operativo básico**, nuestra recomendación para realizar el registro completo con el mínimo de inconvenientes es que `EtwManifestProviderConfiguration` en el elemento `WadCfg` de la plantilla tenga el mismo aspecto que le mostramos aquí:

```json
  "WadCfg": {
        "DiagnosticMonitorConfiguration": {
          "overallQuotaInMB": "50000",
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
              },
              {
                "provider": "02d06793-efeb-48c8-8f7f-09713309a810",
                "scheduledTransferLogLevelFilter": "Information",
                "scheduledTransferKeywordFilter": "4611686018427387904",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                "eventDestination": "ServiceFabricSystemEventTable"
                }
              }
            ]
          }
        }
      },
```

## <a name="collect-from-new-eventsource-channels"></a>Recopilar desde canales EventSource nuevos

Para actualizar Diagnostics de forma que recopile registros de canales EventSource nuevos que representan una nueva aplicación que vaya a implementar, siga los mismos pasos que se han descrito anteriormente para configurar Diagnostics para un clúster existente.

Actualice la sección `EtwEventSourceProviderConfiguration` del archivo template.json para agregar entradas para los canales EventSource nuevos antes de aplicar la actualización de la configuración mediante el comando `New-AzResourceGroupDeployment` de PowerShell. El nombre del origen del evento se define como parte del código en el archivo generado en Visual Studio ServiceEventSource.cs.

Por ejemplo, si el origen del evento se denomina My-Eventsource, agregue el código siguiente para colocar los eventos de My-Eventsource en una tabla denominada MyDestinationTableName.

```json
        {
            "provider": "My-Eventsource",
            "scheduledTransferPeriod": "PT5M",
            "DefaultEvents": {
            "eventDestination": "MyDestinationTableName"
            }
        }
```

Para recopilar registros de eventos o contadores de rendimiento, modifique la plantilla de Resource Manager con los ejemplos proporcionados en [Creación de una máquina virtual de Windows con supervisión y diagnóstico mediante una plantilla de Azure Resource Manager](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json). Luego, vuelva a publicar la plantilla de Resource Manager.

## <a name="collect-performance-counters"></a>Recopilar contadores de rendimiento

Para recopilar métricas de rendimiento del clúster, agregue los contadores de rendimiento a "WadCfg > DiagnosticMonitorConfiguration" en la plantilla de Resource Manager para el clúster. Vea [Supervisión del rendimiento con WAD](service-fabric-diagnostics-perf-wad.md) para conocer los pasos acerca de cómo modificar `WadCfg` para recopilar contadores de rendimiento específicos. Consulte en [Service Fabric Performance Counters](service-fabric-diagnostics-event-generation-perf.md) (Contadores de rendimiento de Service Fabric) una lista de contadores de rendimiento que se recomienda recopilar.
  
Si usa un receptor de Application Insights, como se describe en la siguiente sección, y quiere que estas métricas aparezcan en Application Insights, asegúrese de agregar el nombre del receptor en la sección "receptores", como se indicó anteriormente. Esto enviará automáticamente los contadores de rendimiento que se configuran individualmente para el recurso de Application Insights.


## <a name="send-logs-to-application-insights"></a>Enviar registros a Application Insights

### <a name="configuring-application-insights-with-wad"></a>Configuración de Application Insights con WAD

>[!NOTE]
>Solo se aplica a los clústeres de Windows por ahora.

Hay dos métodos principales para enviar datos de WAD a Azure Application Insights; se consigue mediante la adición de un receptor de Application Insights a la configuración de WAD, ya sea a través de Azure Portal o bien a través de una plantilla de Azure Resource Manager.

#### <a name="add-an-application-insights-instrumentation-key-when-creating-a-cluster-in-azure-portal"></a>Incorporación de una clave de instrumentación de Application Insights al crear un clúster en Azure Portal

![Adición de una clave de AI](media/service-fabric-diagnostics-event-analysis-appinsights/azure-enable-diagnostics.png)

Si se activa "Diagnostics" al crear un clúster, aparecerá un campo opcional para escribir una clave de instrumentación de Application Insights. Si pega aquí la clave de Application Insights, el receptor de Application Insights se configura automáticamente en la plantilla de Resource Manager que se usa para implementar el clúster.

#### <a name="add-the-application-insights-sink-to-the-resource-manager-template"></a>Incorporación del receptor de Application Insights a la plantilla de Resource Manager

En el archivo "WadCfg" de la plantilla de Resource Manager, agregue un "receptor" mediante la introducción de estos dos cambios:

1. Agregue la configuración del receptor directamente después de la confirmación de que `DiagnosticMonitorConfiguration` se ha completado:

    ```json
    "SinksConfig": {
        "Sink": [
            {
                "name": "applicationInsights",
                "ApplicationInsights": "***ADD INSTRUMENTATION KEY HERE***"
            }
        ]
    }

    ```

2. Incluya el receptor en `DiagnosticMonitorConfiguration`; para ello, agregue la siguiente línea en `DiagnosticMonitorConfiguration` de `WadCfg` (justo antes de que se confirmen `EtwProviders`):

    ```json
    "sinks": "applicationInsights"
    ```

En los dos fragmentos de código anteriores, el nombre "applicationInsights" se usó para describir el receptor. No se trata de un requisito y, siempre que el nombre del receptor se incluya en "sinks", puede establecer el nombre en cualquier cadena.

Actualmente, los registros del clúster se muestran como **seguimientos** en el visor de registros de Application Insights. Como la mayoría de los seguimientos de la plataforma son de tipo "Información", también puede considerar la opción de cambiar la configuración del receptor para que solo envíe registros de tipo "Advertencia" o "Error". Esto puede realizarse mediante la adición de "canales" al receptor, como se demuestra en [este artículo](../azure-monitor/platform/diagnostics-extension-to-application-insights.md).

>[!NOTE]
>Si usa una clave de Application Insights incorrecta en el portal o en la plantilla de Resource Manager, deberá cambiarla manualmente y actualizar el clúster o volver a implementarlo.

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya configurado correctamente Azure Diagnostics, verá los datos en las tablas de almacenamiento de los registros de ETW y EventSource. Si decide usar los registros de Azure Monitor, Kibana u otra plataforma de análisis y visualización de datos que no se configure directamente en la plantilla de Resource Manager, asegúrese de configurar la plataforma de su elección para leer los datos de estas tablas de almacenamiento. Es relativamente fácil hacerlo en el caso de los registros de Azure Monitor, como se explica en [Análisis de eventos y registro](service-fabric-diagnostics-event-analysis-oms.md). Application Insights es un caso especial en este sentido, ya que puede configurarse como parte de la configuración de la extensión de Diagnostics, por lo que debe leer el [artículo correspondiente](service-fabric-diagnostics-event-analysis-appinsights.md) si opta por usar AI.

>[!NOTE]
>Actualmente no existe ninguna manera de filtrar o limpiar los eventos que se envían a la tabla. Si no se implementa un proceso para quitar eventos de la tabla, la tabla seguirá aumentando. Actualmente, hay un ejemplo de un servicio de limpieza de datos en ejecución en el [ejemplo de guardián](https://github.com/Azure-Samples/service-fabric-watchdog-service). Se recomienda que escriba uno para sí mismo, a menos que tenga una buena razón para almacenar los registros durante más de 30 o 90 días.

* [Información sobre cómo recopilar registros o contadores de rendimiento mediante la extensión de Diagnósticos](../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json)
* [Análisis y visualización de eventos con Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Análisis y visualización de eventos con los registros de Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
* [Análisis y visualización de eventos con Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Análisis y visualización de eventos con los registros de Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
