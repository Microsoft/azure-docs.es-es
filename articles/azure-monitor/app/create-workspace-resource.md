---
title: Creación de nuevos recursos basados en área de trabajo de Application Insights de Azure Monitor | Microsoft Docs
description: Obtenga información sobre los pasos necesarios para habilitar los nuevos recursos basados en área de trabajo de Application Insights de Azure Monitor.
author: mrbullwinkle
ms.author: mbullwin
ms.topic: conceptual
ms.date: 08/24/2020
ms.openlocfilehash: d6d6731ae087604e0a53a6721bb76dfba5fbf40c
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783848"
---
# <a name="workspace-based-application-insights-resources-preview"></a>Recursos de Application Insights basados en área de trabajo (versión preliminar)

Los recursos basados en área de trabajo permiten la integración completa entre Application Insights y Log Analytics. Ahora puede optar por enviar la telemetría de Application Insights a un área de trabajo común de Log Analytics, lo que permite el acceso completo a todas las características de Log Analytics a la vez que se mantienen los registros de aplicación, infraestructura y plataforma en una única ubicación consolidada.

Esto también permite el control de acceso basado en rol (RBAC) común en los recursos y elimina la necesidad de consultas entre aplicaciones y áreas de trabajo.

> [!NOTE]
> La ingesta de datos y la retención de recursos de Application Insights basados en área de trabajo se facturan por medio del área de trabajo de Log Analytics en la que se encuentran los datos. [Más información]( ./pricing.md#workspace-based-application-insights) sobre la facturación de recursos de Application Insights basados en área de trabajo.

Para probar la nueva experiencia, inicie sesión en [Azure Portal](https://portal.azure.com) y cree un recurso de Application Insights:

![Recurso de Application Insights basado en área de trabajo](./media/create-workspace-resource/create-workspace-based.png)

Si aún no tiene un área de trabajo de Log Analytics, [vea la documentación de creación de áreas de trabajo de Log Analytics](../learn/quick-create-workspace.md).

**Los recursos basados en el área de trabajo están disponibles actualmente en todas las regiones comerciales y en Azure Government**.

Una vez creado el recurso, se ve la información del área de trabajo correspondiente en el panel de **información general**:

![Nombre del área de trabajo](./media/create-workspace-resource/workspace-name.png)

Al hacer clic en el texto del vínculo azul, se le lleva al área de trabajo de Log Analytics asociada, donde puede aprovechar el nuevo entorno de consultas unificado del área de trabajo.

> [!NOTE]
> Todavía se proporciona compatibilidad total con versiones anteriores con las consultas de recursos clásicas, los libros y las alertas basadas en registro de Application Insights dentro de la experiencia de Application Insights. Para consultar o ver en la [nueva estructura o esquema de tabla basados en área de trabajo](apm-tables.md), primero debe ir al área de trabajo de Log Analytics. Durante la versión preliminar, al seleccionar **Registros** en los paneles de Application Insights, se proporciona acceso a la experiencia de consulta clásica de Application Insights.

## <a name="copy-the-connection-string"></a>Copiar la cadena de conexión

La [cadena de conexión](./sdk-connection-string.md?tabs=net) identifica el recurso con el que se quieren asociar los datos de telemetría. También permite modificar los puntos de conexión que va a usar el recurso como destino de la telemetría. Tiene que copiar la cadena de conexión y agregarla al código de la aplicación o a una variable de entorno.

## <a name="monitoring-configuration"></a>Configuración de supervisión

Una vez creado un recurso de Application Insights basado en área de trabajo, la configuración de la supervisión es relativamente sencilla.

### <a name="code-based-application-monitoring"></a>Supervisión de aplicaciones basada en código

En la supervisión de aplicaciones basada en código, solo se debe instalar el SDK adecuado de Application Insights y apuntarlo a la clave de instrumentación o la cadena de conexión del recurso recién creado.  

Para obtener documentación detallada sobre la configuración de un SDK de Application Insights para la supervisión basada en código, vea la documentación específica del lenguaje o el marco:

- [ASP.NET](./asp-net.md)
- [ASP.NET Core](./asp-net-core.md)
- [Tareas en segundo plano y aplicaciones de consola modernas (.NET/.NET Core)](./worker-service.md)
- [Aplicaciones de consola clásicas (.NET)](./console.md) 
- [Java](./java-get-started.md?tabs=maven)
- [JavaScript](./javascript.md)
- [Node.js](./nodejs.md)
- [Python](./opencensus-python.md)

### <a name="codeless-monitoring-and-visual-studio-resource-creation"></a>Supervisión sin código y creación de recursos de Visual Studio

En la supervisión sin código de servicios como Azure Functions y Azure App Services, también tiene que crear primero el recurso de Application Insights basado en área de trabajo y luego apuntar a ese recurso durante la fase de configuración de la supervisión.

Aunque estos servicios ofrecen la opción de crear un nuevo recurso de Application Insights dentro de su propio proceso de creación de recursos, los recursos creados mediante estas opciones de la interfaz de usuario están restringidos actualmente a la experiencia clásica de Application Insights.

Lo mismo se aplica a la experiencia de creación de recursos de Application Insights en Visual Studio para ASP.NET y ASP.NET Core. Debe seleccionar un recurso existente basado en área de trabajo desde la interfaz de usuario de habilitación de la supervisión de Visual Studio. Si opta por crear el nuevo recurso desde Visual Studio, se le limita a crear un recurso de Application Insights clásico.

## <a name="creating-a-resource-automatically"></a>Creación de un recurso de forma automática

### <a name="azure-cli"></a>Azure CLI

Para acceder a los comandos de la CLI de Azure de Application Insights en versión preliminar, primero debe ejecutar:

```azurecli
 az extension add -n application-insights
```

Si no ejecuta el comando `az extension add`, aparece un mensaje de error que indica: `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`

Ahora puede ejecutar lo siguiente para crear el recurso de Application Insights:

```azurecli
az monitor app-insights component create --app
                                         --location
                                         --resource-group
                                         [--application-type]
                                         [--ingestion-access {Disabled, Enabled}]
                                         [--kind]
                                         [--only-show-errors]
                                         [--query-access {Disabled, Enabled}]
                                         [--tags]
                                         [--workspace]
```

#### <a name="example"></a>Ejemplo

```azurecli
az monitor app-insights component create --app demoApp --location eastus --kind web -g my_resource_group --workspace "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/test1234/providers/microsoft.operationalinsights/workspaces/test1234555"
```

Para obtener la documentación completa de la CLI de Azure para este comando, vea la [documentación de la CLI de Azure](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest#ext-application-insights-az-monitor-app-insights-component-create).

### <a name="azure-powershell"></a>Azure PowerShell

El comando `New-AzApplicationInsights` de PowerShell no admite actualmente la creación de un recurso de Application Insights basado en área de trabajo. Para crear un recurso basado en área de trabajo con PowerShell, puede usar las plantillas de Azure Resource Manager siguientes e implementar con PowerShell.

### <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure

#### <a name="template-file"></a>Archivo de plantilla

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string"
        },
        "type": {
            "type": "string"
        },
        "regionId": {
            "type": "string"
        },
        "tagsArray": {
            "type": "object"
        },
        "requestSource": {
            "type": "string"
        },
        "workspaceResourceId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "microsoft.insights/components",
            "location": "[parameters('regionId')]",
            "tags": "[parameters('tagsArray')]",
            "apiVersion": "2020-02-02-preview",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Application_Type": "[parameters('type')]",
                "Flow_Type": "Redfield",
                "Request_Source": "[parameters('requestSource')]",
                "WorkspaceResourceId": "[parameters('workspaceResourceId')]"
            }
        }
    ]
}
```

#### <a name="parameters-file"></a>Archivo de parámetros

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "type": {
            "value": "web"
        },
        "name": {
            "value": "customresourcename"
        },
        "regionId": {
            "value": "eastus"
        },
        "tagsArray": {
            "value": {}
        },
        "requestSource": {
            "value": "Custom"
        },
        "workspaceResourceId": {
            "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my_resource_group/providers/microsoft.operationalinsights/workspaces/myworkspacename"
        }
    }
}

```

## <a name="new-capabilities"></a>Funcionalidades nuevas

Application Insights basado en áreas de trabajo le permite aprovechar las funcionalidades más recientes de Azure Monitor, como las siguientes:

* Las [claves administradas por el cliente (CMK)](../platform/customer-managed-keys.md) proporcionan cifrado en reposo de los datos mediante claves de cifrado a las que solo tiene acceso el usuario.
* [Azure Private Link](../platform/private-link-security.md) le permite vincular de forma segura los servicios PaaS de Azure a la red virtual mediante puntos de conexión privados.
* [Traiga su propio almacenamiento (BYOS) para Profiler y Snapshot Debugger](./profiler-bring-your-own-storage.md) le proporciona control total sobre la directiva de cifrado en reposo, la directiva de administración de la duración y el acceso a la red para todos los datos asociados a Application Insights Profiler y Snapshot Debugger. 

## <a name="modifying-the-associated-workspace"></a>Modificación del área de trabajo asociada

Una vez creado un recurso de Application Insights basado en área de trabajo, puede modificar el área de trabajo de Log Analytics asociada.

En el panel del recurso de Application Insights, seleccione **Propiedades** > **Cambiar el área de trabajo** > **Áreas de trabajo de Log Analytics**.

## <a name="export-telemetry"></a>Exportación de telemetría

La funcionalidad de exportación continua heredada no es compatible con los recursos basados en área de trabajo. En su lugar, seleccione **Configuración de diagnóstico** > **Agregar configuración de diagnóstico** en el recurso de Application Insights. Puede seleccionar todas las tablas, o un subconjunto de tablas, para archivar en una cuenta de almacenamiento o para transmitir a un centro de eventos de Azure.

## <a name="next-steps"></a>Pasos siguientes

* [Exploración de métricas](../platform/metrics-charts.md)
* [Escribir consultas de Analytics](../log-query/log-query-overview.md)

[api]: ./api-custom-events-metrics.md
[diagnostic]: ./diagnostic-search.md
[metrics]: ../platform/metrics-charts.md
[start]: ./app-insights-overview.md