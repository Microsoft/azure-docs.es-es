---
title: 'Inicio rápido: Creación de una consulta compartida con la CLI de Azure'
description: En este inicio rápido, dará los pasos necesarios para habilitar la extensión de Resource Graph en la CLI de Azure y crear una consulta compartida.
ms.date: 06/29/2020
ms.topic: quickstart
ms.openlocfilehash: c5f7e6fbe1b462c1f9b6e8ad46c598398e1aca02
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050971"
---
# <a name="quickstart-create-a-resource-graph-shared-query-using-azure-cli"></a>Inicio rápido: Creación de una consulta compartida de Azure Resource Graph mediante la CLI de Azure

El primer paso para usar Azure Resource Graph con la [CLI de Azure](/cli/azure/) es comprobar que se instalado la extensión. Esta guía de inicio rápido lo guiará a través del proceso de agregar la extensión a la instalación de la CLI de Azure. Puede usar la extensión con la CLI de Azure instalada localmente o mediante [Azure Cloud Shell](https://shell.azure.com).

Al final de este proceso, habrá agregado la extensión a la instalación de la CLI de Azure que haya elegido y habrá creado una consulta compartida de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-extension"></a>Incorporación de la extensión de Resource Graph

Para que la CLI de Azure pueda trabajar con Azure Resource Graph, se debe agregar la extensión. Esta extensión funciona siempre que se puede usar la CLI de Azure, incluidos [Bash en Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (independiente y dentro del portal), la [imagen de Docker de la CLI de Azure](https://hub.docker.com/r/microsoft/azure-cli/) o instalado localmente.

1. Compruebe que está instalada la versión más reciente de la CLI de Azure (al menos la versión **2.8.0**). Si todavía no está instalado, siga [estas instrucciones](/cli/azure/install-azure-cli-windows).

1. En el entorno de la CLI de Azure que prefiera, use [az extension add](/cli/azure/extension#az-extension-add) para importar la extensión de Resource Graph con el siguiente comando:

   ```azurecli-interactive
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. Compruebe con [az extension list](/cli/azure/extension#az-extension-list) que la extensión se ha instalado y que es la versión esperada (al menos la versión **1.1.0**):

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="create-a-resource-graph-shared-query"></a>Creación de una consulta compartida de Resource Graph

Una vez agregada la extensión de la CLI de Azure al entorno de su elección, es el momento de crear una consulta compartida de Resource Graph. Las consultas compartidas son objetos de Azure Resource Manager para los que se puede conceder permiso o que se pueden ejecutar en el Probador de Azure Resource Graph. La consulta resume el recuento de todos los recursos agrupados por _ubicación_.

1. Cree un grupo de recursos con [az group create](/cli/azure/group#az-group-create) para almacenar la consulta compartida de Azure Resource Graph. Este grupo de recursos se denomina `resource-graph-queries` y la ubicación es `westus2`.

   ```azurecli-interactive
   # Login first with az login if not using Cloud Shell

   # Create the resource group
   az group create --name 'resource-graph-queries' --location 'westus2'
   ```

1. Cree la consulta compartida de Azure Resource Graph con la extensión de `graph` y el comando [az graph shared-query create](/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-create):

   ```azurecli-interactive
   # Create the Azure Resource Graph shared query
   az graph shared-query create --name 'Summarize resources by location' \
      --description 'This shared query summarizes resources by location for a pinnable map graphic.' \
      --graph-query 'Resources | summarize count() by location' \
      --resource-group 'resource-graph-queries'
   ```

1. Consulte una lista de las consultas compartidas del nuevo grupo de recursos. El comando [az graph shared-query list](/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-list) devuelve una matriz de valores.

   ```azurecli-interactive
   # List all the Azure Resource Graph shared queries in a resource group
   az graph shared-query list --resource-group 'resource-graph-queries'
   ```

1. Para obtener un solo resultado de consulta compartida, use el comando [az graph shared-query show](/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-show).

   ```azurecli-interactive
   # Show a specific Azure Resource Graph shared query
   az graph shared-query show --resource-group 'resource-graph-queries' \
      --name 'Summarize resources by location'
   ```

1. Ejecute la consulta compartida en la CLI de Azure con la sintaxis `{{shared-query-uri}}` en un comando [az graph query](/cli/azure/ext/resource-graph/graph#ext-resource-graph-az-graph-query).
   En primer lugar, copie el campo `id` del resultado del comando `show` anterior. Reemplace el texto `shared-query-uri` del ejemplo por el valor del campo `id`, pero deje los caracteres `{{` y `}}` que lo rodean.

   ```azurecli-interactive
   # Run a Azure Resource Graph shared query
   az graph query --graph-query "{{shared-query-uri}}"
   ```

   > [!NOTE]
   > La sintaxis `{{shared-query-uri}}` es una característica en **versión preliminar**.

Otra manera de buscar consultas compartidas de Resource Graph es mediante Azure Portal. En el portal, use la barra de búsqueda para buscar "Consultas de Resource Graph". Seleccione la consulta compartida. En la página **Información general**, la pestaña **Consulta** muestra la consulta guardada. El botón **Editar** la abre en el [Probador de Resource Graph](./first-query-portal.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere quitar la consulta compartida, el grupo de recursos y la extensión de Resource Graph de su entorno de la CLI de Azure, puede hacerlo mediante los siguientes comandos:

- [az graph shared-query delete](/cli/azure/ext/resource-graph/graph/shared-query#ext-resource-graph-az-graph-shared-query-delete)
- [az group delete](/cli/azure/group#az-group-delete)
- [az extension remove](/cli/azure/extension#az-extension-remove)

```azurecli-interactive
# Delete the Azure Resource Graph shared query
az graph shared-query delete --resource-group 'resource-graph-queries' \
   --name 'Summarize resources by location'

# Remove the resource group
# WARNING: This command deletes ALL resources you've added to this resource group without prompting for confirmation
az group delete --resource-group 'resource-graph-queries' --yes

# Remove the Azure Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha agregado la extensión de Resource Graph a su entorno de la CLI de Azure y ha creado una consulta compartida. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)