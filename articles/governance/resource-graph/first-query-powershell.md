---
title: 'Inicio rápido: La primera consulta en PowerShell'
description: En este inicio rápido, dará los pasos necesarios para habilitar el módulo Resource Graph de Azure PowerShell y ejecutar la primera consulta.
ms.date: 08/10/2020
ms.topic: quickstart
ms.openlocfilehash: 4cc7eb524060f2a797077f38321ae9ce6c9cf267
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056421"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-powershell"></a>Inicio rápido: Ejecutar la primera consulta de Resource Graph con Azure PowerShell

El primer paso para usar Azure Resource Graph es comprobar que el módulo de Azure PowerShell está instalado. Esta guía de inicio rápido lo guiará a través del proceso de agregar el módulo a la instalación de Azure PowerShell.

Al final de este proceso, habrá agregado el módulo a la instalación de Azure PowerShell que haya elegido y habrá ejecutado la primera consulta de Resource Graph.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-module"></a>Agregar el módulo de Resource Graph

Para habilitar Azure PowerShell para consultar Azure Resource Graph, se debe agregar el módulo. Este módulo se puede usar con PowerShell y PowerShell Core instalados localmente, con [Azure Cloud Shell](https://shell.azure.com) o con la [imagen de Docker de PowerShell](https://hub.docker.com/_/microsoft-powershell).

### <a name="base-requirements"></a>Requisitos básicos

El módulo de Azure Resource Graph requiere el software siguiente:

- Azure PowerShell 1.0.0 o posterior. Si todavía no está instalado, siga [estas instrucciones](/powershell/azure/install-az-ps).

- PowerShellGet 2.0.1 o una versión posterior. Si no está instalado o actualizado, siga [estas instrucciones](/powershell/scripting/gallery/installing-psget).

### <a name="install-the-module"></a>Instalación del módulo

El módulo de Resource Graph para PowerShell es **Az.ResourceGraph**.

1. Ejecute el siguiente comando como **administrador** en un símbolo del sistema de PowerShell:

   ```azurepowershell-interactive
   # Install the Resource Graph module from PowerShell Gallery
   Install-Module -Name Az.ResourceGraph
   ```

1. Asegúrese de que el módulo se haya importado y que sea la versión más reciente (0.7.5):

   ```azurepowershell-interactive
   # Get a list of commands for the imported Az.ResourceGraph module
   Get-Command -Module 'Az.ResourceGraph' -CommandType 'Cmdlet'
   ```

## <a name="run-your-first-resource-graph-query"></a>Ejecutar la primera consulta de Resource Graph

Una vez agregado el módulo de Azure PowerShell al entorno de su elección, es el momento de probar una consulta simple de Resource Graph. La consulta devolverá los cinco primeros recursos de Azure con el **nombre** y el **tipo de recurso** de cada recurso.

1. Ejecute la primera consulta de Azure Resource Graph mediante el cmdlet `Search-AzGraph`:

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   # Run Azure Resource Graph query
   Search-AzGraph -Query 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Actualice la consulta para `order by` la propiedad **Nombre**:

   ```azurepowershell-interactive
   # Run Azure Resource Graph query with 'order by'
   Search-AzGraph -Query 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Actualice la consulta para que primero se aplique `order by` a la propiedad **Nombre** y, después, `limit` a los cinco primeros resultados:

   ```azurepowershell-interactive
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   Search-AzGraph -Query 'Resources | project name, type | order by name asc | limit 5'
   ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

> [!NOTE]
> Si la consulta no devuelve los resultados de una suscripción a la que ya tiene acceso, tenga en cuenta que `Search-AzGraph` cmdlet utiliza de forma predeterminada las suscripciones en el contexto predeterminado. Para ver la lista de identificadores de suscripción que forman parte del contexto predeterminado, ejecute este comando `(Get-AzContext).Account.ExtendedProperties.Subscriptions`. Si quiere buscar en todas las suscripciones a las que tiene acceso, puede configurar PSDefaultParameterValues para el cmdlet `Search-AzGraph` mediante la ejecución de `$PSDefaultParameterValues=@{"Search-AzGraph:Subscription"= $(Get-AzSubscription).ID}`.
   
## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere quitar el módulo de Resource Graph de su entorno de Azure PowerShell, puede hacerlo mediante el comando siguiente:

```azurepowershell-interactive
# Remove the Resource Graph module from the current session
Remove-Module -Name 'Az.ResourceGraph'

# Uninstall the Resource Graph module from the environment
Uninstall-Module -Name 'Az.ResourceGraph'
```

> [!NOTE]
> De esta forma no se elimina el archivo de módulo descargado anteriormente. Solo lo quita de la sesión de PowerShell en ejecución.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha agregado el módulo Resource Graph a su entorno de Azure PowerShell y ha ejecutado la primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)