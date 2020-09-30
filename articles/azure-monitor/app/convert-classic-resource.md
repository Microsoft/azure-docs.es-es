---
title: Migración de un recurso clásico de Application Insights de Azure Monitor a un recurso basado en áreas de trabajo | Microsoft Docs
description: Obtenga información sobre los pasos necesarios para actualizar un recurso clásico de Application Insights de Azure Monitor al nuevo modelo basado en áreas de trabajo.
author: mrbullwinkle
ms.author: mbullwin
ms.topic: conceptual
ms.date: 09/09/2020
ms.openlocfilehash: caaf5469eace891f2996a565af183b411ad1d740
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932061"
---
# <a name="migrate-to-workspace-based-application-insights-resources"></a>Migración a recursos de Application Insights basados en áreas de trabajo

Esta guía le orienta a lo largo del proceso de migración de un recurso clásico de Application Insights a un recurso basado en áreas de trabajo. Los recursos basados en área de trabajo permiten la integración completa entre Application Insights y Log Analytics. Los recursos basados en áreas de trabajo envían telemetría de Application Insights a un área de trabajo común de Log Analytics, lo que permite el acceso a las [características más recientes de Azure Monitor](#new-capabilities), a la vez que se mantienen los registros de aplicación, infraestructura y plataforma en una única ubicación consolidada.

Los recursos basados en áreas de trabajo permiten el control de acceso basado en rol (RBAC) común en los recursos y eliminan la necesidad de consultas entre aplicaciones y áreas de trabajo.

**Los recursos basados en áreas de trabajo están disponibles actualmente en todas las regiones comerciales y en Azure US Government**

## <a name="new-capabilities"></a>Funcionalidades nuevas

Application Insights basado en áreas de trabajo permite aprovechar todas las capacidades más recientes de Azure Monitor y Log Analytics, incluidas:

* Las [claves administradas por el cliente (CMK)](../platform/customer-managed-keys.md) proporcionan cifrado en reposo de los datos mediante claves de cifrado a las que solo tiene acceso el usuario.
* [Azure Private Link](../platform/private-link-security.md) le permite vincular de forma segura los servicios PaaS de Azure a la red virtual mediante puntos de conexión privados.
* [Traiga su propio almacenamiento (BYOS) para Profiler y Snapshot Debugger](./profiler-bring-your-own-storage.md) le proporciona control total sobre la directiva de cifrado en reposo, la directiva de administración de la duración y el acceso a la red para todos los datos asociados a Application Insights Profiler y Snapshot Debugger. 
* Los [niveles de reserva de capacidad](../platform/manage-cost-storage.md#pricing-model) permiten ahorrar hasta un 25 % en comparación con el precio de Pago por uso. 
* Ingesta de datos más rápida gracias a la ingesta de streaming de Log Analytics.

## <a name="migration-process"></a>Proceso de migración

Al migrar a un recurso basado en áreas de trabajo no se transfieren datos del almacenamiento del recurso clásico al nuevo almacenamiento basado en áreas de trabajo. Por el contrario, si se decide migrar, la ubicación donde se escriben los nuevos datos cambia a un área de trabajo de Log Analytics, a la vez que se conserva el acceso a los datos del recurso clásico. 

Los datos del recurso clásico se conservan y están sujetos a la configuración de retención en el momento de su ingesta. Todos los nuevos datos ingeridos después de la migración están sujetos a la configuración de retención del área de trabajo asociada de Log Analytics. 

El proceso de migración es **permanente y no se puede deshacer**. Una vez que se migra un recurso a Application Insights basado en áreas de trabajo, siempre va a ser un recurso basado en áreas de trabajo. Pero, una vez migrado, se puede cambiar el área de trabajo de destino siempre que sea necesario. 

> [!NOTE]
> La ingesta de datos y la retención de recursos de Application Insights basados en área de trabajo se facturan por medio del área de trabajo de Log Analytics en la que se encuentran los datos. [Más información]( ./pricing.md#workspace-based-application-insights) sobre la facturación de recursos de Application Insights basados en área de trabajo. (A los datos del recurso clásico de Application Insights ingeridos antes de la migración se les siguen aplicando los precios y la retención de Application Insights mientras se conserven). 

Si no tiene que migrar un recurso existente y quiere crear un nuevo recurso de Application Insights basado en áreas de trabajo, use la [guía de creación de recursos basados en áreas de trabajo](create-workspace-resource.md).

## <a name="pre-requisites"></a>Requisitos previos 

- Un área de trabajo de Log Analytics con el modo de control de acceso establecido en el valor **`use resource or workspace permissions`** . 

    - Los recursos de Application Insights basados en áreas de trabajo no son compatibles con las áreas de trabajo establecidas en el valor **`workspace based permissions`** dedicado. Para obtener más información sobre el control de acceso al área de trabajo de Log Analytics, vea la [guía de configuración del modo de control de acceso de Log Analytics](../platform/manage-access.md#configure-access-control-mode)

    - Si aún no tiene un área de trabajo de Log Analytics, [vea la documentación de creación de áreas de trabajo de Log Analytics](../learn/quick-create-workspace.md).
    
- La exportación continua no es compatible con los recursos basados en áreas de trabajo y se debe deshabilitar.
Una vez completada la migración, puede usar la [configuración de diagnóstico](../platform/diagnostic-settings.md) para configurar el archivado de datos en una cuenta de almacenamiento o el streaming al centro de eventos de Azure.  

- Compruebe la configuración de retención actual en **Genera** > **Uso y costos estimados** > **Retención de datos** del área de trabajo de Log Analytics. Este valor afecta al periodo de tiempo que se almacenan los nuevos datos ingeridos una vez migrado el recurso de Application Insights. Si actualmente está almacenando datos de Application Insights durante más tiempo que el valor predeterminado de 90 días y quiere mantener este período de retención mayor, es posible que tenga que ajustar la configuración de retención del área de trabajo.

## <a name="migrate-your-resource"></a>Migración del recurso

Esta sección le orienta a lo largo del proceso de migración de un recurso clásico de Application Insights al nuevo tipo de recurso basado en áreas de trabajo.

1. En el recurso de Application Insights, seleccione **Propiedades** en el título **Configurar** de la barra de menús de la izquierda.

    ![Propiedades resaltado en cuadro rojo](./media/convert-classic-resource/properties.png)

2. Seleccione **`Migrate to Workspace-based`**.
    
     ![Botón Migrar recurso](./media/convert-classic-resource/migrate.png)

3. Seleccione el área de trabajo de Log Analytics donde quiere que se almacene toda la telemetría de Application Insights ingerida en el futuro.

     ![Interfaz de usuario del asistente para migración con la opción para seleccionar el área de trabajo de destino](./media/convert-classic-resource/migration.png)
    

Una vez migrado el recurso, se ve la información del área de trabajo correspondiente en el panel **Información general**:

![Nombre del área de trabajo](./media/create-workspace-resource/workspace-name.png)

Al hacer clic en el texto del vínculo azul, se le lleva al área de trabajo de Log Analytics asociada, donde puede aprovechar el nuevo entorno de consultas unificado del área de trabajo.

## <a name="understanding-log-queries"></a>Descripción de las consultas de registro

Todavía se proporciona compatibilidad total con versiones anteriores con las consultas de recursos clásicas, los libros y las alertas basadas en registro de Application Insights dentro de la experiencia de Application Insights. 

Para escribir consultas para la [nueva estructura o esquema de tabla basados en áreas de trabajo](apm-tables.md), primero debe ir al área de trabajo de Log Analytics. 

Al consultar directamente desde la interfaz de usuario de Log Analytics en el área de trabajo, solo se ven los datos ingeridos después de la migración. Para ver los datos clásicos de Application Insights más los datos nuevos ingeridos después de la migración en una experiencia de consulta unificada, use la vista de consulta Registros (Analytics) desde el recurso de Application Insights migrado.

## <a name="programmatic-resource-migration"></a>Migración de recursos mediante programación

### <a name="azure-cli"></a>Azure CLI

Para acceder a los comandos de la CLI de Azure de Application Insights en versión preliminar, primero debe ejecutar:

```azurecli
 az extension add -n application-insights
```

Si no ejecuta el comando `az extension add`, aparece un mensaje de error que indica: `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`

Ahora puede ejecutar lo siguiente para crear el recurso de Application Insights:

```azurecli
az monitor app-insights component update --app
                                         --resource-group
                                         [--ingestion-access {Disabled, Enabled}]
                                         [--kind]
                                         [--query-access {Disabled, Enabled}]
                                         [--retention-time]
                                         [--workspace]
```

#### <a name="example"></a>Ejemplo

```azurecli
az monitor app-insights component update --app your-app-insights-resource-name -g your_resource_group --workspace "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/test1234/providers/microsoft.operationalinsights/workspaces/test1234555"
```

Para obtener la documentación completa de la CLI de Azure para este comando, vea la [documentación de la CLI de Azure](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest#ext-application-insights-az-monitor-app-insights-component-update).

### <a name="azure-powershell"></a>Azure PowerShell

El comando `Update-AzApplicationInsights` de PowerShell no admite actualmente la migración de un recurso clásico de Application Insights a basado en áreas de trabajo. Para crear un recurso basado en área de trabajo con PowerShell, puede usar las plantillas de Azure Resource Manager siguientes e implementar con PowerShell.

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

## <a name="modifying-the-associated-workspace"></a>Modificación del área de trabajo asociada

Una vez creado un recurso de Application Insights basado en área de trabajo, puede modificar el área de trabajo de Log Analytics asociada.

En el panel del recurso de Application Insights, seleccione **Propiedades** > **Cambiar el área de trabajo** > **Áreas de trabajo de Log Analytics**.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="access-mode"></a>Modo de acceso

**Mensaje de error:** *El área de trabajo seleccionada está configurada con el modo de acceso basado en áreas de trabajo. Algunas características de APM podrían verse afectadas. Seleccione otra área de trabajo o permita el acceso basado en recursos en la configuración del área de trabajo. Puede invalidar este error mediante la CLI.* 

Para que el recurso de Application Insights basado en áreas de trabajo funcione correctamente, debe cambiar el modo de control de acceso del área de trabajo de Log Analytics de destino en el valor **permisos de recurso o área de trabajo**. Este valor se encuentra en la interfaz de usuario del área de trabajo de Log Analytics, en **Propiedades** > **Modo de control de acceso**. Para obtener instrucciones detalladas, vea la [guía de configuración del modo de control de acceso de Log Analytics](../platform/manage-access.md#configure-access-control-mode). Si el modo de control de acceso está establecido en el valor exclusivo **Requerir permisos de área de trabajo**, la migración por medio de la experiencia de migración del portal permanece bloqueada.

Si no puede cambiar el modo de control de acceso del área de trabajo de destino actual por motivos de seguridad, se recomienda crear una nueva área de trabajo de Log Analytics para la migración. 

### <a name="continuous-export"></a>Exportación continua

**Mensaje de error:** *Se debe deshabilitar la exportación continua para continuar. Después de la migración, use la configuración de diagnóstico para la exportación.* 

La funcionalidad de exportación continua heredada no es compatible con los recursos basados en área de trabajo. Antes de la migración, debe deshabilitar la exportación continua.

1. En la vista del recurso de Application Insights, en el título **Configurar**, seleccione **Exportación continua**.

    ![Elemento de menú Exportación continua](./media/convert-classic-resource/continuous-export.png)

2. Seleccione **Deshabilitar**.

    ![Botón Deshabilitar exportación continua](./media/convert-classic-resource/disable.png)

- Una vez que haya seleccionado Deshabilitar, puede volver a la interfaz de usuario de migración. Si la página de edición de la exportación continua le pide que no guarde la configuración, puede seleccionar Aceptar en este mensaje, ya que no se refiere a la deshabilitación o habilitación de la exportación continua.

- Una vez que haya migrado correctamente el recurso de Application Insights a basado en áreas de trabajo, puede usar la configuración de diagnóstico para reemplazar la funcionalidad que solía proporcionar la exportación continua. Seleccione **Configuración de diagnóstico** > **Agregar configuración de diagnóstico** en el recurso de Application Insights. Puede seleccionar todas las tablas, o un subconjunto de tablas, para archivar en una cuenta de almacenamiento o para transmitir a un centro de eventos de Azure. Para obtener orientación detallada sobre la configuración de diagnóstico, vea la [guía de configuración de diagnóstico de Azure Monitor](../platform/diagnostic-settings.md).

### <a name="retention-settings"></a>Configuración de retención

**Mensaje de advertencia:** *La configuración de retención personalizada de Application Insights no se va a aplicar a los datos enviados al área de trabajo. Tiene que volver a configurarla por separado.*

No es necesario realizar ningún cambio antes de la migración, pero este mensaje avisa de que la configuración actual de retención de Application Insights no se ha establecido en el período de retención predeterminado de 90 días. Este mensaje de advertencia significa que es posible que quiera modificar la configuración de retención del área de trabajo de Log Analytics antes de migrar y comenzar a ingerir datos nuevos. 

Puede comprobar la configuración de retención actual de Log Analytics en **General** > **Uso y costos estimados** > **Retención de datos** desde la interfaz de usuario de Log Analytics. Este valor afecta al periodo de tiempo que se almacenan los nuevos datos ingeridos una vez migrado el recurso de Application Insights.

## <a name="next-steps"></a>Pasos siguientes

* [Exploración de métricas](../platform/metrics-charts.md)
* [Escribir consultas de Analytics](../log-query/log-query-overview.md)
