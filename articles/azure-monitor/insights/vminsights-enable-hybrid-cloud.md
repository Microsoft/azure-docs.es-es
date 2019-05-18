---
title: Habilitar Azure Monitor (versión preliminar) para un entorno híbrido | Microsoft Docs
description: En este artículo se describe cómo habilitar a Azure Monitor para las máquinas virtuales para un entorno de nube híbrida que contiene una o más máquinas virtuales.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2019
ms.author: magoedte
ms.openlocfilehash: 6b8870f0a6f14536fdf3a1ff675f2fbe3ce8aeec
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524196"
---
# <a name="enable-azure-monitor-for-vms-preview-for-a-hybrid-environment"></a>Habilitar a Monitor de Azure para máquinas virtuales (versión preliminar) para un entorno híbrido

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

En este artículo se explica cómo habilitar a Azure Monitor para las máquinas virtuales (versión preliminar) para máquinas virtuales o equipos físicos alojados en su centro de datos o en otro entorno de nube. Al final de este proceso que habrá ha confirmado correctamente las máquinas virtuales en su entorno de supervisión y obtenga información sobre si experimentan los problemas de rendimiento o disponibilidad. 

Antes de obtener iniciado, no olvide revisar la [requisitos previos](vminsights-enable-overview.md) y comprobar la suscripción y los recursos cumplan los requisitos. Revise los requisitos y los métodos de implementación para el [Agente Linux y Windows de Log Analytics](../../log-analytics/log-analytics-agent-overview.md).

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)]

>[!NOTE]
>La extensión Dependency Agent para Service Map de Azure Monitor para VM no transmite ningún dato y no requiere ningún cambio en firewalls o puertos. Los datos de Mapa siempre son transmitidos por el agente de Log Analytics al servicio de Azure Monitor, ya sea directamente o mediante la [Puerta de enlace de OMS](../../azure-monitor/platform/gateway.md), si las directivas de seguridad de TI no permiten que los equipos de la red se conecten a Internet.

Las tareas para completar esta tarea se resumen como sigue:

1. Instalar el agente de Log Analytics para Windows o Linux.

2. Descargar e instalar Dependency Agent para Service Map de Azure Monitor para VM en [Windows](https://aka.ms/dependencyagentwindows) o [Linux](https://aka.ms/dependencyagentlinux).

3. Habilitar la recopilación de contadores de rendimiento.

4. Implementar Azure Monitor para VM.

## <a name="install-the-dependency-agent-on-windows"></a>Instalar Dependency Agent en Windows
Para instalar Dependency Agent manualmente en equipos Windows, puede ejecutar `InstallDependencyAgent-Windows.exe`. Si ejecuta este archivo ejecutable sin opciones, se inicia un asistente para instalación que puede seguir para realizar la instalación del agente de forma interactiva.

>[!NOTE]
>Se requieren privilegios de *administrador* para instalar o desinstalar al agente.

En la tabla siguiente se destacan los parámetros que admite el programa de instalación para el agente desde la línea de comandos.

| Parámetro | DESCRIPCIÓN |
|:--|:--|
| /? | Devuelve una lista de las opciones de la línea de comandos. |
| /S | Realiza una instalación silenciosa sin interacción del usuario. |

Por ejemplo, para ejecutar el programa de instalación con el parámetro `/?`, escriba **InstallDependencyAgent-Windows.exe /?**.

Los archivos de Dependency Agent para Windows se instalan en *C:\Archivos de programa\Microsoft Dependency Agent* de manera predeterminada. Si Dependency Agent no se inicia una vez completada la instalación, compruebe los registros para obtener información detallada del error. El directorio de registro es *%Programfiles%\Microsoft Dependency Agent\logs*.

## <a name="install-the-dependency-agent-on-linux"></a>Instalación de Dependency Agent en Linux
Dependency Agent se instala en los servidores Linux desde *InstallDependencyAgent-Linux64.bin*, un script de shell con un archivo binario autoextraíble. Puede ejecutar el archivo mediante `sh` o agregar permisos de ejecución al propio archivo.

>[!NOTE]
> Es necesario el acceso raíz para instalar o configurar el agente.
>

| Parámetro | DESCRIPCIÓN |
|:--|:--|
| -help | Obtenga una lista de las opciones de la línea de comandos. |
| -s | Realice una instalación silenciosa sin preguntas. |
| --check | Compruebe los permisos y el sistema operativo, pero no instale el agente. |

Por ejemplo, para ejecutar el programa de instalación con el parámetro `-help`, escriba **InstallDependencyAgent-Linux64.bin -help**.

Para instalar Dependency en Linux como raíz, ejecute el comando `sh InstallDependencyAgent-Linux64.bin`.

Si Dependency Agent no se inicia, compruebe los registros para obtener información detallada del error. En los agentes de Linux, el directorio de registro es */var/opt/microsoft/dependency-agent/log*.

Los archivos de Dependency Agent se colocan en los directorios siguientes:

| Archivos | Ubicación |
|:--|:--|
| Archivos principales | /opt/microsoft/dependency-agent |
| Archivos de registro | /var/opt/microsoft/dependency-agent/log |
| Archivos de configuración | /etc/opt/microsoft/dependency-agent/config |
| Archivos ejecutables de servicio | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| Archivos binarios de almacenamiento | /var/opt/microsoft/dependency-agent/storage |

## <a name="enable-performance-counters"></a>Activar los contadores de rendimiento
Si el área de trabajo de Log Analytics a la que la solución hace referencia no está todavía configurada para recopilar los contadores de rendimiento que la solución requiere, tendrá que habilitarlos. Puede hacerlo de dos maneras:
* Manualmente, según se describe en [Orígenes de datos de rendimiento de Windows y Linux en Log Analytics](../../azure-monitor/platform/data-sources-performance-counters.md)
* Descargando y ejecutando un script de PowerShell que está disponible en [Galería de Azure PowerShell](https://www.powershellgallery.com/packages/Enable-VMInsightsPerfCounters/1.1)

## <a name="deploy-azure-monitor-for-vms"></a>Implementar Azure Monitor para VM
Este método incluye una plantilla JSON que especifica la configuración para habilitar los componentes de la solución en el área de trabajo de Log Analytics.

Si no sabe cómo implementar los recursos con una plantilla, consulte:
* [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/resource-group-template-deploy.md)
* [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/resource-group-template-deploy-cli.md)

Si decide usar la CLI de Azure, primero debe instalar y usar la CLI localmente. Debe ejecuta la versión 2.0.27 de la CLI de Azure, o cualquier versión posterior. Para identificar la versión, ejecute `az --version`. Si necesita instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

### <a name="create-and-execute-a-template"></a>Creación y ejecución de una plantilla

1. Copie y pegue la siguiente sintaxis JSON en el archivo:

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "WorkspaceName": {
                "type": "string"
            },
            "WorkspaceLocation": {
                "type": "string"
            }
        },
        "resources": [
            {
                "apiVersion": "2017-03-15-preview",
                "type": "Microsoft.OperationalInsights/workspaces",
                "name": "[parameters('WorkspaceName')]",
                "location": "[parameters('WorkspaceLocation')]",
                "resources": [
                    {
                        "apiVersion": "2015-11-01-preview",
                        "location": "[parameters('WorkspaceLocation')]",
                        "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "dependsOn": [
                            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        ],
                        "properties": {
                            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        },

                        "plan": {
                            "name": "[concat('ServiceMap', '(', parameters('WorkspaceName'),')')]",
                            "publisher": "Microsoft",
                            "product": "[Concat('OMSGallery/', 'ServiceMap')]",
                            "promotionCode": ""
                        }
                    },
                    {
                        "apiVersion": "2015-11-01-preview",
                        "location": "[parameters('WorkspaceLocation')]",
                        "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                        "type": "Microsoft.OperationsManagement/solutions",
                        "dependsOn": [
                            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        ],
                        "properties": {
                            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('WorkspaceName'))]"
                        },
                        "plan": {
                            "name": "[concat('InfrastructureInsights', '(', parameters('WorkspaceName'),')')]",
                            "publisher": "Microsoft",
                            "product": "[Concat('OMSGallery/', 'InfrastructureInsights')]",
                            "promotionCode": ""
                        }
                    }
                ]
            }
        ]
    }
    ```

1. Guarde este archivo como *installsolutionsforvminsights.json* en una carpeta local.

1. Capture los valores de *WorkspaceName*, *ResourceGroupName* y *WorkspaceLocation*. El valor de *WorkspaceName* es el nombre del área de trabajo de Log Analytics. El valor de *WorkspaceLocation* es la región en la que el área de trabajo está definida.

1. Ya puede implementar una plantilla mediante el siguiente comando de PowerShell:

    ```powershell
    New-AzResourceGroupDeployment -Name DeploySolutions -TemplateFile InstallSolutionsForVMInsights.json -ResourceGroupName ResourceGroupName> -WorkspaceName <WorkspaceName> -WorkspaceLocation <WorkspaceLocation - example: eastus>
    ```

    El cambio de configuración puede tardar unos minutos en completarse. Cuando se completa, se muestra un mensaje que incluye el resultado y que es similar al siguiente:

    ```powershell
    provisioningState       : Succeeded
    ```
   Después de habilitar la supervisión, pueden pasar unos 10 minutos hasta que pueda ver la métrica y el estado del equipo híbrido.

## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para las máquinas virtuales, esta información está disponible para su análisis con Azure Monitor para las máquinas virtuales. Para obtener información sobre cómo usar la característica de mantenimiento, consulte [Descripción del estado de las máquinas virtuales de Azure con Azure Monitor para VM (versión preliminar)](vminsights-health.md). Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md). Para identificar cuellos de botella y el uso general con el rendimiento de sus VM, consulte el artículo sobre cómo [ver el rendimiento de la VM de Azure](vminsights-performance.md), o bien, para ver las dependencias de la aplicación detectadas, consulte cómo [ver el Azure Monitor para la asignación de VM](vminsights-maps.md).