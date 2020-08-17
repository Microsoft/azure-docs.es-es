---
title: Soluciones de supervisión en Azure Monitor | Microsoft Docs
description: Las soluciones de supervisión de Azure Monitor son una colección de reglas de lógica, visualización y adquisición de datos que proporcionan métricas que giran en torno a una determinada área de problemas.  En este artículo se proporciona información sobre la instalación y el uso de soluciones de supervisión.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/12/2020
ms.openlocfilehash: a2ece33974e5bb4d8cb6257d3ca68a258f9b8c81
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326008"
---
# <a name="monitoring-solutions-in-azure-monitor"></a>Soluciones de supervisión en Azure Monitor

Las soluciones de supervisión aprovechan los servicios de Azure para proporcionar análisis adicional del funcionamiento de una aplicación o servicio determinados. En este artículo se proporciona una breve descripción de las soluciones de supervisión en Azure y los detalles sobre su uso e instalación. Puede agregar soluciones de supervisión a su instancia de Azure Monitor para todas las aplicaciones y los servicios que use. Normalmente están disponibles sin costo, excepto la recopilación de datos que podría suponer cargos por uso.

## <a name="use-monitoring-solutions"></a>Uso de soluciones de supervisión

Abra la página **Introducción** en Azure Monitor para mostrar un icono para cada solución instalada en el área de trabajo.

1. Vaya a [Azure Portal](https://ms.portal.azure.com). Busque y seleccione **Monitor**.
1. En el menú **Insights**, seleccione **Más**.
1. Use los cuadros de lista desplegable en la parte superior de la pantalla para cambiar el área de trabajo o el intervalo de tiempo usado para los iconos.
1. Haga clic en el icono de una solución para abrir la vista que incluye un análisis más detallado de los datos recopilados.

![Información general](media/solutions/overview.png)

Las soluciones de supervisión pueden contener varios tipos de recursos de Azure; puede ver todos los recursos que incluyen las soluciones igual que cualquier otro recurso. Por ejemplo, las consultas de registro incluidas en la solución se enumeran en **Consultas de solución** de [Explorador de consultas](../log-query/get-started-portal.md#load-queries). Puede usar esas consultas al llevar a cabo análisis ad hoc con [consultas de registro](../log-query/log-query-overview.md).

## <a name="list-installed-monitoring-solutions"></a>Lista de soluciones de supervisión instaladas

### <a name="portal"></a>[Portal](#tab/portal)

Utilice el procedimiento siguiente para enumerar las soluciones de supervisión instaladas en su suscripción.

1. Vaya a [Azure Portal](https://ms.portal.azure.com). Busque y seleccione **Soluciones**.
1. Se mostrarán las soluciones instaladas en todas las áreas de trabajo. Al nombre de la solución le sigue el nombre del área de trabajo donde está instalada.
1. Use los cuadros de lista desplegable de la parte superior de la pantalla para filtrar por suscripción o grupo de recursos.

![Lista de todas las soluciones](media/solutions/list-solutions-all.png)

Haga clic en el nombre de una solución para abrir su página de resumen. Esta página muestra las vistas incluidas en la solución y proporciona diferentes opciones para la solución y su área de trabajo. Para ver la página de resumen de una solución, siga uno de los procedimientos descritos anteriormente para mostrar las soluciones y haga clic en el nombre concreto.

![Propiedades de la solución](media/solutions/solution-properties.png)

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az monitor log-analytics solution list](/cli/azure/ext/log-analytics-solution/monitor/log-analytics/solution#ext-log-analytics-solution-az-monitor-log-analytics-solution-list) para enumerar las soluciones de supervisión instaladas en la suscripción.   Antes de ejecutar el comando `list`, siga los requisitos previos que se indican en [Instalación de una solución de supervisión](#install-a-monitoring-solution).

```azurecli
# List all log-analytics solutions in the current subscription.
az monitor log-analytics solution list

# List all log-analytics solutions for a specific subscription
az monitor log-analytics solution list --subscription MySubscription

# List all log-analytics solutions in a resource group
az monitor log-analytics solution list --resource-group MyResourceGroup
```

* * *

## <a name="install-a-monitoring-solution"></a>Instalación de una solución de supervisión

### <a name="portal"></a>[Portal](#tab/portal)

Las soluciones de supervisión de Microsoft y asociados están disponibles en [Azure Marketplace](https://azuremarketplace.microsoft.com). Puede buscar las soluciones disponibles e instalarlas mediante el procedimiento siguiente. Al instalar una solución, debe seleccionar un [área de trabajo de Log Analytics](../platform/manage-access.md) donde se instalará la solución y se recopilarán los datos.

1. Desde la [lista de soluciones para la suscripción](#list-installed-monitoring-solutions), haga clic en **Agregar**.
1. Examine o busque una solución. También puede examinar las soluciones desde [este vínculo de búsqueda](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/management-tools?page=1&subcategories=management-solutions).
1. Localice la solución de supervisión que quiere y lea su descripción.
1. Haga clic en **Crear** para iniciar el proceso de instalación.
1. Cuando se inicia el proceso de instalación, se le pedirá especificar el área de trabajo de Log Analytics y proporcionar cualquier configuración necesaria para la solución.

![Instalar una solución](media/solutions/install-solution.png)

### <a name="install-a-solution-from-the-community"></a>Instalación de una solución desde la comunidad

Los miembros de la comunidad pueden enviar soluciones de administración a las plantillas de inicio rápido de Azure. Puede instalar estas soluciones directamente o descargar plantillas para hacerlo más adelante.

1. Siga el proceso descrito en [Área de trabajo de Log Analytics y cuenta de Automation](#log-analytics-workspace-and-automation-account) para vincular un área de trabajo y una cuenta.
2. Visite [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/).
3. Busque una solución que le interese.
4. Seleccione la solución en los resultados para ver su información.
5. Haga clic en botón **Deploy to Azure** (Implementar en Azure).
6. Se le pedirá que proporcione información como el grupo de recursos y la ubicación, además de los valores de los parámetros de la solución.
7. Haga clic en **Adquirir** para instalar la solución.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

### <a name="prepare-your-environment"></a>Preparación del entorno

1. Instalación de la CLI de Azure

   Debe [instalar la CLI de Azure](/cli/azure/install-azure-cli) antes de ejecutar los comandos de la referencia de la CLI.  Si lo prefiere, también puede usar Azure Cloud Shell para completar los pasos de este artículo.  Azure Cloud Shell es un entorno de shell interactivo que se utiliza mediante el explorador.  Inicie Cloud Shell con uno de estos métodos:

   - Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell.

   - Seleccione el botón **Cloud Shell** en la barra de menús de la esquina superior derecha de [Azure Portal](https://portal.azure.com).

1. Inicie sesión.

   Si está usando una instalación local de la CLI, inicie sesión con el comando [az login](/cli/azure/reference-index#az-login).  Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.

    ```azurecli
    az login
    ```

1. Instale la extensión `log-analytics`

   El comando `log-analytics` es una extensión experimental de la CLI de Azure principal. Obtenga más información sobre las referencias de extensión en [Uso de extensiones con la CLI de Azure](/cli/azure/azure-cli-extensions-overview?).

   ```azurecli
   az extension add --name log-analytics
   ```

   Se espera la advertencia siguiente.

   ```output
   The installed extension `log-analytics` is experimental and not covered by customer support.  Please use with discretion.
   ```

### <a name="install-a-solution-with-the-azure-cli"></a>Instalación de una solución con la CLI de Azure

Al instalar una solución, debe seleccionar un [área de trabajo de Log Analytics](../platform/manage-access.md) donde se instalará la solución y se recopilarán los datos.  Con la CLI de Azure, las áreas de trabajo se administran mediante los comandos de referencia [az monitor log-analytics workspace](/cli/azure/monitor/log-analytics/workspace).  Siga el proceso descrito en [Área de trabajo de Log Analytics y cuenta de Automation](#log-analytics-workspace-and-automation-account) para vincular un área de trabajo y una cuenta.

Use [az monitor log-analytics solution create](/cli/azure/ext/log-analytics-solution/monitor/log-analytics/solution) para instalar una solución de supervisión.  Los parámetros entre corchetes son opcionales.

```azurecli
az monitor log-analytics solution create --name
                                         --plan-product
                                         --plan-publisher
                                         --resource-group
                                         --workspace
                                         [--no-wait]
                                         [--tags]
```

Este es un ejemplo de código que crea una solución de Log Analytics para el producto del plan OMSGallery/Containers.

```azurecli
az monitor log-analytics solution create --resource-group MyResourceGroup \
                                         --name Containers({SolutionName}) \
                                         --tags key=value \
                                         --plan-publisher Microsoft  \
                                         --plan-product "OMSGallery/Containers" \
                                         --workspace "/subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/ \
                                           Microsoft.OperationalInsights/workspaces/{WorkspaceName}"
```

* * *

## <a name="log-analytics-workspace-and-automation-account"></a>Área de trabajo de Log Analytics y cuenta de Automation

Todas las soluciones de supervisión requieren un [área de trabajo de Log Analytics](../platform/manage-access.md) para almacenar los datos recopilados por la solución y para hospedar sus vistas y búsquedas de registros. Algunas soluciones también requieren una [cuenta de Automation](../../automation/automation-security-overview.md) que contenga runbooks y recursos relacionados. El área de trabajo y la cuenta deben cumplir los siguientes requisitos.

* Cada instalación de la solución solo puede utilizar un área de trabajo de Log Analytics y una cuenta de Automation. Puede instalar la solución por separado en varias áreas de trabajo.
* Si una solución requiere una cuenta de Automation, el área de trabajo de Log Analytics y la cuenta de Automation solución deben estar vinculadas. Un área de trabajo de Log Analytics solo puede estar vinculada a una cuenta de Automation y viceversa.
* Para vincularse, el área de trabajo Log Analytics y la cuenta de Automation deben estar en la misma suscripción, pero pueden estar en distintos grupos de recursos implementados en la misma región. La excepción es un área de trabajo que se encuentre en la región Este de EE. UU. y una cuenta de Automation del Este de EE. UU. 2.

Al instalar una solución mediante Azure Marketplace, se le pedirá un área de trabajo y una cuenta de Automation. Se creará un vínculo entre ellas, si no están ya vinculadas.

### <a name="verify-the-link-between-a-log-analytics-workspace-and-automation-account"></a>Comprobación de un vínculo entre un área de trabajo de Log Analytics y una cuenta de Automation

Puede comprobar el vínculo entre un área de trabajo de Log Analytics y una cuenta de Automation mediante el procedimiento siguiente.

1. Seleccione la cuenta de Automation en Azure Portal.
1. Vaya a la sección **Related Resources** (Recursos relacionados) del menú y seleccione **Linked workspace** (Área de trabajo vinculada).
1. Si el **área de trabajo** está vinculada a una cuenta de Automation, en esta página se muestra el área de trabajo a la que está vinculada. Si selecciona el nombre del área de trabajo que aparece, se le redirigirá a la página de información general de dicha área de trabajo.

## <a name="remove-a-monitoring-solution"></a>Eliminación de una solución de supervisión

### <a name="portal"></a>[Portal](#tab/portal)

Para quitar una solución instalada mediante el portal, busque en la [lista de soluciones instaladas](#list-installed-monitoring-solutions). Haga clic en el nombre de la solución para abrir su página de resumen y haga clic en **Eliminar**.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para quitar una solución instalada mediante la CLI de Azure, use el comando [az monitor log-analytics solution delete](/cli/azure/ext/log-analytics-solution/monitor/log-analytics/solution#ext-log-analytics-solution-az-monitor-log-analytics-solution-delete).

```azurecli
az monitor log-analytics solution delete --name
                                         --resource-group
                                         [--no-wait]
                                         [--yes]
```

* * *

## <a name="next-steps"></a>Pasos siguientes

* Obtenga un [lista de soluciones de supervisión de Microsoft](/azure/azure-monitor/monitor-reference).
* Aprenda a [crear consultas](../log-query/log-query-overview.md) para analizar los datos recopilados por las soluciones de supervisión.
* Consulte todos los [comandos de la CLI de Azure para Azure Monitor](/cli/azure/azure-cli-reference-for-monitor).

