---
title: Recopilar y analizar registros de recursos
description: Obtenga información sobre cómo enviar registros de recursos y datos de eventos de grupos de contenedores en Azure Container Instances a los registros de Azure Monitor.
ms.topic: article
ms.date: 07/13/2020
ms.author: danlep
ms.openlocfilehash: efd71d181059ab395aeec4da364110a42d2cccd3
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86524019"
---
# <a name="container-group-and-instance-logging-with-azure-monitor-logs"></a>Registro de instancias y grupos de contenedores con registros de Azure Monitor

Las áreas de trabajo de Log Analytics proporcionan una ubicación centralizada para almacenar y consultar datos de registro no solo de los recursos de Azure, sino también de los recursos locales y de los recursos de otras nubes. Azure Container Instances incluye compatibilidad integrada para el envío de registros y datos de evento a los registros de Azure Monitor.

Para enviar un registro de grupo de contenedores y datos de eventos a los registros de Azure Monitor, especifique una clave y un identificador de un área de trabajo de Log Analytics existente al configurar un grupo de contenedores. 

En las secciones siguientes se describe cómo crear un grupo de contenedores con el registro habilitado y cómo consultar registros. También puede [actualizar un grupo de contenedores](container-instances-update.md) con un identificador de área de trabajo y una clave de área de trabajo para habilitar el registro.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

> [!NOTE]
> Actualmente, solo puede enviar datos de eventos desde instancias de contenedor de Linux a Log Analytics.

## <a name="prerequisites"></a>Prerrequisitos

Para habilitar el registro en las instancias de los contenedores, necesita lo siguiente:

* [Área de trabajo de Log Analytics](../azure-monitor/learn/quick-create-workspace.md)
* [CLI de Azure](/cli/azure/install-azure-cli) (o [Cloud Shell](../cloud-shell/overview.md))

## <a name="get-log-analytics-credentials"></a>Obtención de las credenciales de Log Analytics

Azure Container Instances necesita permiso para enviar datos a su área de trabajo de Log Analytics. Para conceder este permiso y habilitar el registro, es preciso proporcionar el identificador del área de trabajo de Log Analytics y una de sus claves (la principal o la secundaria) al crear el grupo de contenedores.

Para obtener el identificador y la clave principal del área de trabajo de Log Analytics:

1. Vaya al área de trabajo de Log Analytics en Azure Portal
1. En **Configuración**, seleccione **Agents management** (Administración de agentes).
1. Anote el valor de:
   * **Id. de área de trabajo**
   * **Clave principal**

## <a name="create-container-group"></a>Creación de un grupo de contenedores

Ahora que tiene el identificador y la clave principal del área de trabajo de Log Analytics, ya puede crear un grupo de contenedores con el registro habilitado.

En los ejemplos siguientes se muestran dos maneras de crear un grupo de contenedores formado por un solo contenedor [fluentd][fluentd]: CLI de Azure y la CLI de Azure con una plantilla de YAML. El contenedor fluentd genera varias líneas de salida en su configuración predeterminada. Dado que esta salida se envía a su área de trabajo de Log Analytics, sirve para mostrar la visualización y consulta de registros.

### <a name="deploy-with-azure-cli"></a>Implementación con la CLI de Azure

Para implementar con la CLI de Azure, especifique los parámetros `--log-analytics-workspace` y `--log-analytics-workspace-key` en el comando [az container create][az-container-create]. Antes de ejecutar el siguiente comando, reemplace los dos valores de área de trabajo por los valores obtenidos en el paso anterior (y actualice el nombre del grupo de recursos).

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainergroup001 \
    --image fluent/fluentd \
    --log-analytics-workspace <WORKSPACE_ID> \
    --log-analytics-workspace-key <WORKSPACE_KEY>
```

### <a name="deploy-with-yaml"></a>Implementación con YAML

Utilice este método si prefiere implementar grupos de contenedores con YAML. El siguiente fragmento de código YAML define un grupo de contenedores con un solo contenedor. Copie el código YAML en un nuevo archivo y sustituya `LOG_ANALYTICS_WORKSPACE_ID` y `LOG_ANALYTICS_WORKSPACE_KEY` por los valores que obtuvo en el paso anterior. Guarde el archivo con el nombre **deploy-aci.yaml**.

```yaml
apiVersion: 2019-12-01
location: eastus
name: mycontainergroup001
properties:
  containers:
  - name: mycontainer001
    properties:
      environmentVariables: []
      image: fluent/fluentd
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
  diagnostics:
    logAnalytics:
      workspaceId: LOG_ANALYTICS_WORKSPACE_ID
      workspaceKey: LOG_ANALYTICS_WORKSPACE_KEY
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

A continuación, ejecute el siguiente comando para implementar el grupo de contenedores. Reemplace `myResourceGroup` por un grupo de recursos de su suscripción (o bien cree antes un grupo de recursos denominado "myResourceGroup"):

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainergroup001 --file deploy-aci.yaml
```

Debería recibir una respuesta de Azure con detalles de implementación poco después de emitir el comando.

## <a name="view-logs"></a>Ver registros

Una vez que haya implementado el grupo de contenedores, las primeras entradas de registro pueden tardar varios minutos (hasta 10) en aparecer en Azure Portal. 

Para ver los registros del grupo de contenedores en la tabla `ContainerInstanceLog_CL`:

1. Vaya al área de trabajo de Log Analytics en Azure Portal
1. En **General**, seleccione **Registros**.  
1. Escriba la consulta siguiente: `ContainerInstanceLog_CL | limit 50`
1. Seleccione **Ejecutar**.

Debería ver que la consulta muestra varios resultados. Si no ve ninguno, espere unos minutos y seleccione el botón **Ejecutar** para volver a ejecutar la consulta. De forma predeterminada, las entradas del registro aparecen en formato de **tabla**. Luego puede expandir una fila para ver el contenido de una entrada de registro individual.

![Resultados de Búsqueda de registros en Azure Portal][log-search-01]

## <a name="view-events"></a>Ver eventos

También puede ver los eventos de las instancias de contenedor en Azure Portal. Los eventos incluyen la hora a la que se crea la instancia y el momento en que se inicia. Para ver los datos de evento en la tabla `ContainerEvent_CL`:

1. Vaya al área de trabajo de Log Analytics en Azure Portal
1. En **General**, seleccione **Registros**.  
1. Escriba la consulta siguiente: `ContainerEvent_CL | limit 50`
1. Seleccione **Ejecutar**.

Debería ver que la consulta muestra varios resultados. Si no ve ninguno, espere unos minutos y seleccione el botón **Ejecutar** para volver a ejecutar la consulta. De forma predeterminada, las entradas aparecen en formato de **tabla**. Luego puede expandir una fila para ver el contenido de una entrada individual.

![Resultados de la búsqueda de registros en Azure Portal][log-search-02]

## <a name="query-container-logs"></a>Consulta de registros de contenedores

Los registros de Azure Monitor incluyen un amplio [lenguaje de consulta][query_lang] para poder extraer información de miles de líneas de la salida del registro.

La estructura básica de una consulta consiste en la tabla de origen (en este artículo, `ContainerInstanceLog_CL` o `ContainerEvent_CL`) seguida de una serie de operadores separados por el carácter de barra vertical (`|`). Puede encadenar varios operadores para refinar los resultados y realizar funciones avanzadas.

Para ver los resultados de una consulta de ejemplo, pegue la siguiente consulta en el cuadro de texto de consulta y seleccione el botón **Ejecutar** para ejecutar la consulta. Esta consulta muestra todas las entradas de registro cuyo campo "Message" contiene la palabra "warn":

```query
ContainerInstanceLog_CL
| where Message contains "warn"
```

También se admiten consultas más complejas. Por ejemplo, esta consulta muestra solo las entradas de registro del grupo de contenedores "mycontainergroup001" generadas en la última hora:

```query
ContainerInstanceLog_CL
| where (ContainerGroup_s == "mycontainergroup001")
| where (TimeGenerated > ago(1h))
```

## <a name="next-steps"></a>Pasos siguientes

### <a name="azure-monitor-logs"></a>Registros de Azure Monitor

Para más información acerca de cómo realizar consultas en registros y configurar alertas en registros de Azure Monitor, consulte:

* [Descripción de las búsquedas de registros en los registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Alertas unificadas en Azure Monitor](../azure-monitor/platform/alerts-overview.md)


### <a name="monitor-container-cpu-and-memory"></a>Supervisión de la CPU y la memoria de los contenedores

Para obtener información acerca de la supervisión de los recursos de CPU y de memoria de un instancia de contenedor, consulte:

* [Supervisión de los recursos de los contenedores en Azure Container Instances](container-instances-monitor.md).

<!-- IMAGES -->
[log-search-01]: ./media/container-instances-log-analytics/portal-query-01.png
[log-search-02]: ./media/container-instances-log-analytics/portal-query-02.png

<!-- LINKS - External -->
[fluentd]: https://hub.docker.com/r/fluent/fluentd/
[query_lang]: https://aka.ms/LogAnalyticsLanguage

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
